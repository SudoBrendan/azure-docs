---
title: Azure Table indexer
titleSuffix: Azure Cognitive Search
description: Set up a search indexer to index data stored in Azure Table Storage for full text search in Azure Cognitive Search.

manager: nitinme
author: mgottein 
ms.author: magottei

ms.service: cognitive-search
ms.topic: how-to
ms.date: 02/28/2022
---

# Index data from Azure Table Storage

In this article, learn how to configure an [**indexer**](search-indexer-overview.md) that imports content from Azure Table Storage and makes it searchable in Azure Cognitive Search. Inputs to the indexer are your entities, in a single table. Output is a search index with searchable content and metadata stored in individual fields.

This article supplements [**Create an indexer**](search-howto-create-indexers.md) with information that's specific to indexing from Azure Table Storage. It uses the REST APIs to demonstrate a three-part workflow common to all indexers: create a data source, create an index, create an indexer. Data extraction occurs when you submit the Create Indexer request.

## Prerequisites

+ [Azure Table Storage](../storage/tables/table-storage-overview.md)

+ Tables containing text. If you have binary data, you can include [AI enrichment](cognitive-search-concept-intro.md) for image analysis.

+ Read permissions to access Azure Storage. A "full access" connection string includes a key that gives access to the content, but if you're using Azure roles, make sure the [search service managed identity](search-howto-managed-identities-data-sources.md) has **Data and Reader** permissions.

+ A REST client, such as [Postman](search-get-started-rest.md) or [Visual Studio Code with the extension for Azure Cognitive Search](search-get-started-vs-code.md) to send REST calls that create the data source, index, and indexer.

## Define the data source

The data source definition specifies the data to index, credentials, and policies for identifying changes in the data. A data source is defined as an independent resource so that it can be used by multiple indexers.

1. [Create or update a data source](/rest/api/searchservice/create-data-source) to set its definition: 

    ```json
    {
        "name" : "hotel-tables",
        "type" : "azuretable",
        "credentials" : { "connectionString" : "DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=<account key>;" },
        "container" : { "name" : "tblHotels", "query" : "PartitionKey eq '123'" }
    }
    ```

1. Set "type" to `"azuretable"` (required).

1. Set "credentials" to an Azure Storage connection string. The next section describes the supported formats.

1. Set "container" to the name of the table.

1. Optionally, set "query" to a filter on PartitionKey. This is a best practice that improves performance. If "query" is specified any other way, the indexer will execute a full table scan, resulting in poor performance if the tables are large.

A data source definition can also include [soft deletion policies](search-howto-index-changed-deleted-blobs.md), if you want the indexer to delete a search document when the source document is flagged for deletion.

<a name="Credentials"></a>

### Supported credentials and connection strings

Indexers can connect to a table using the following connections.

| Full access storage account connection string |
|-----------------------------------------------|
|`{ "connectionString" : "DefaultEndpointsProtocol=https;AccountName=<your storage account>;AccountKey=<your account key>;" }` |
| You can get the connection string from the Storage account page in Azure portal by selecting **Access keys** in the left navigation pane. Make sure to select a full connection string and not just a key. |

| Managed identity connection string |
|------------------------------------|
|`{ "connectionString" : "ResourceId=/subscriptions/<your subscription ID>/resourceGroups/<your resource group name>/providers/Microsoft.Storage/storageAccounts/<your storage account name>/;" }`|
|This connection string does not require an account key, but you must have previously configured a search service to [connect using a managed identity](search-howto-managed-identities-data-sources.md).|

| Storage account shared access signature** (SAS) connection string |
|-------------------------------------------------------------------|
| `{ "connectionString" : "BlobEndpoint=https://<your account>.blob.core.windows.net/;SharedAccessSignature=?sv=2016-05-31&sig=<the signature>&spr=https&se=<the validity end time>&srt=co&ss=b&sp=rl;" }` |
| The SAS should have the list and read permissions on tables and entities. |

| Container shared access signature |
|-----------------------------------|
| `{ "connectionString" : "ContainerSharedAccessUri=https://<your storage account>.blob.core.windows.net/<container name>?sv=2016-05-31&sr=c&sig=<the signature>&se=<the validity end time>&sp=rl;" }` |
| The SAS should have the list and read permissions on the container. For more information, see [Using Shared Access Signatures](../storage/common/storage-sas-overview.md). |

> [!NOTE]
> If you use SAS credentials, you will need to update the data source credentials periodically with renewed signatures to prevent their expiration. If SAS credentials expire, the indexer will fail with an error message similar to "Credentials provided in the connection string are invalid or have expired".  

<a name="Performance"></a>

### Partition for improved performance

By default, Azure Cognitive Search uses the following internal query filter to keep track of which source entities have been updated since the last run: `Timestamp >= HighWaterMarkValue`. Because Azure tables don’t have a secondary index on the `Timestamp` field, this type of query requires a full table scan and is therefore slow for large tables.

To avoid a full scan, you can use table partitions to narrow the scope of each indexer job.

+ If your data can naturally be partitioned into several partition ranges, create a data source and a corresponding indexer for each partition range. Each indexer now has to process only a specific partition range, resulting in better query performance. If the data that needs to be indexed has a small number of fixed partitions, even better: each indexer only does a partition scan. 

   For example, to create a data source for processing a partition range with keys from `000` to `100`, use a query like this: `"container" : { "name" : "my-table", "query" : "PartitionKey ge '000' and PartitionKey lt '100' " }`

+ If your data is partitioned by time (for example, if you create a new partition every day or week), consider the following approach: 

  + In the data source definition, specify a query similar to the following example: `(PartitionKey ge <TimeStamp>) and (other filters)`. 

  + Monitor indexer progress by using [Get Indexer Status API](/rest/api/searchservice/get-indexer-status), and periodically update the `<TimeStamp>` condition of the query based on the latest successful high-water-mark value. 

  + With this approach, if you need to trigger a complete reindexing, you need to reset the data source query in addition to resetting the indexer. 

## Add search fields to an index

In a [search index](search-what-is-an-index.md), add fields to accept the content and metadata of your table entities.

1. [Create or update an index](/rest/api/searchservice/create-index) to define search fields that will store content from entities:

    ```http
    POST https://[service name].search.windows.net/indexes?api-version=2020-06-30 
    {
      "name" : "my-search-index",
      "fields": [
        { "name": "ID", "type": "Edm.String", "key": true, "searchable": false },
        { "name": "SomeColumnInMyTable", "type": "Edm.String", "searchable": true }
      ]
    }
    ```

1. Create a document key field ("key": true), but allow the indexer to populate it automatically. Do not define a field mapping to alternative unique string field in your table. 

   A table indexer populates the key field with concatenated partition and row keys from the table. For example, if a row’s PartitionKey is `PK1` and RowKey is `RK1`, then the key value is `PK1RK1`. If the partition key is null, just the row key is used.

1. Create additional fields that correspond to entity fields. For example, if an entity looks like the following example, your search index should have fields for HotelName, Description, and Category.

   :::image type="content" source="media/search-howto-indexing-tables/table.png" alt-text="Screenshot of table content in Storage browser." border="true":::

   Using the same names and compatible [data types](/rest/api/searchservice/supported-data-types) minimizes the need for [field mappings](search-indexer-field-mappings.md).

## Configure and run the table indexer

Once the index and data source have been created, you're ready to create the indexer. Indexer configuration specifies the inputs, parameters, and properties controlling run time behaviors.

1. [Create or update an indexer](/rest/api/searchservice/create-indexer) by giving it a name and referencing the data source and target index:

    ```http
    POST https://[service name].search.windows.net/indexers?api-version=2020-06-30
    {
        "name" : "table-indexer",
        "dataSourceName" : "my-table-datasource",
        "targetIndexName" : "my-search-index",
        "parameters": {
            "batchSize": null,
            "maxFailedItems": null,
            "maxFailedItemsPerBatch": null,
            "base64EncodeKeys": null,
            "configuration:" { }
        },
        "schedule" : { },
        "fieldMappings" : [ ]
    }
    ```

1. [Specify field mappings](search-indexer-field-mappings.md) if there are differences in field name or type, or if you need multiple versions of a source field in the search index.

1. See [Create an indexer](search-howto-create-indexers.md) for more information about other properties.

An indexer runs automatically when it's created. You can prevent this by setting "disabled" to true. To control indexer execution, [run an indexer on demand](search-howto-run-reset-indexers.md) or [put it on a schedule](search-howto-schedule-indexers.md).

## Check indexer status

To monitor the indexer status and execution history, send a [Get Indexer Status](/rest/api/searchservice/get-indexer-status) request:

```http
GET https://myservice.search.windows.net/indexers/myindexer/status?api-version=2020-06-30
  Content-Type: application/json  
  api-key: [admin key]
```

The response includes status and the number of items processed. It should look similar to the following example:

```json
    {
        "status":"running",
        "lastResult": {
            "status":"success",
            "errorMessage":null,
            "startTime":"2022-02-21T00:23:24.957Z",
            "endTime":"2022-02-21T00:36:47.752Z",
            "errors":[],
            "itemsProcessed":1599501,
            "itemsFailed":0,
            "initialTrackingState":null,
            "finalTrackingState":null
        },
        "executionHistory":
        [
            {
                "status":"success",
                "errorMessage":null,
                "startTime":"2022-02-21T00:23:24.957Z",
                "endTime":"2022-02-21T00:36:47.752Z",
                "errors":[],
                "itemsProcessed":1599501,
                "itemsFailed":0,
                "initialTrackingState":null,
                "finalTrackingState":null
            },
            ... earlier history items
        ]
    }
```

Execution history contains up to 50 of the most recently completed executions, which are sorted in the reverse chronological order so that the latest execution comes first.

## Next steps

You can now [run the indexer](search-howto-run-reset-indexers.md), [monitor status](search-howto-monitor-indexers.md), or [schedule indexer execution](search-howto-schedule-indexers.md). The following articles apply to indexers that pull content from Azure Storage:

+ [Index large data sets](search-howto-large-index.md)
+ [Indexer access to content protected by Azure network security features](search-indexer-securing-resources.md)
