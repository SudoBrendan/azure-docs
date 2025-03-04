---
title: 'Quickstart: Build a Python app using Azure Cosmos DB SQL API account'
description: Presents a Python code sample you can use to connect to and query the Azure Cosmos DB SQL API
author: Rodrigossz
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: python
ms.topic: quickstart
ms.date: 08/26/2021
ms.author: rosouz
ms.custom: seodec18, seo-javascript-september2019, seo-python-october2019, devx-track-python, mode-api
---

# Quickstart: Build a Python application using an Azure Cosmos DB SQL API account
[!INCLUDE[appliesto-sql-api](../includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [.NET V3](create-sql-api-dotnet.md)
> * [.NET V4](create-sql-api-dotnet-V4.md)
> * [Java SDK v4](create-sql-api-java.md)
> * [Spring Data v3](create-sql-api-spring-data.md)
> * [Spark v3 connector](create-sql-api-spark.md)
> * [Node.js](create-sql-api-nodejs.md)
> * [Python](create-sql-api-python.md)
> * [Go](create-sql-api-go.md)

In this quickstart, you create and manage an Azure Cosmos DB SQL API account from the Azure portal, and from Visual Studio Code with a Python app cloned from GitHub. Azure Cosmos DB is a multi-model database service that lets you quickly create and query document, table, key-value, and graph databases with global distribution and horizontal scale capabilities.

## Prerequisites

- A Cosmos DB Account. You options are:
    * Within an Azure active subscription:
        * [Create an Azure free Account](https://azure.microsoft.com/free) or use your existing subscription 
        * [Visual Studio Monthly Credits](https://azure.microsoft.com/pricing/member-offers/credit-for-visual-studio-subscribers)
        * [Azure Cosmos DB Free Tier](../optimize-dev-test.md#azure-cosmos-db-free-tier)
    * Without an Azure active subscription:
        * [Try Azure Cosmos DB for free](https://azure.microsoft.com/try/cosmosdb/), a tests environment that lasts for 30 days.
        * [Azure Cosmos DB Emulator](https://aka.ms/cosmosdb-emulator) 
- [Python 2.7 or 3.6+](https://www.python.org/downloads/), with the `python` executable in your `PATH`.
- [Visual Studio Code](https://code.visualstudio.com/).
- The [Python extension for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-python.python#overview).
- [Git](https://www.git-scm.com/downloads). 
- [Azure Cosmos DB SQL API SDK for Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cosmos/azure-cosmos)

## Important update on Python 2.x Support

New releases of this SDK won't support Python 2.x starting January 1st, 2022. Please check the [CHANGELOG](./sql-api-sdk-python.md) for more information.

## Create a database account

[!INCLUDE [cosmos-db-create-dbaccount](../includes/cosmos-db-create-dbaccount.md)]

## Add a container

[!INCLUDE [cosmos-db-create-collection](../includes/cosmos-db-create-collection.md)]

## Add sample data

[!INCLUDE [cosmos-db-create-sql-api-add-sample-data](../includes/cosmos-db-create-sql-api-add-sample-data.md)]

## Query your data

[!INCLUDE [cosmos-db-create-sql-api-query-data](../includes/cosmos-db-create-sql-api-query-data.md)]

## Clone the sample application

Now let's clone a SQL API app from GitHub, set the connection string, and run it. This quickstart uses version 4 of the [Python SDK](https://pypi.org/project/azure-cosmos/#history).

1. Open a command prompt, create a new folder named git-samples, then close the command prompt.

    ```cmd
    md git-samples
    ```

   If you are using a bash prompt, you should instead use the following command:

   ```bash
   mkdir "git-samples"
   ```

2. Open a git terminal window, such as git bash, and use the `cd` command to change to the new folder to install the sample app.

    ```bash
    cd "git-samples"
    ```

3. Run the following command to clone the sample repository. This command creates a copy of the sample app on your computer. 

    ```bash
    git clone https://github.com/Azure-Samples/azure-cosmos-db-python-getting-started.git
    ```  

## Update your connection string

Now go back to the Azure portal to get your connection string information and copy it into the app.

1. In your Azure Cosmos DB account in the [Azure portal](https://portal.azure.com/), select **Keys** from the left navigation. Use the copy buttons on the right side of the screen to copy the **URI** and **Primary Key** into the *cosmos_get_started.py* file in the next step.

    :::image type="content" source="./media/create-sql-api-dotnet/access-key-and-uri-in-keys-settings-in-the-azure-portal.png" alt-text="Get an access key and URI in the Keys settings in the Azure portal":::

2. In Visual Studio Code, open the *cosmos_get_started.py* file in *\git-samples\azure-cosmos-db-python-getting-started*.

3. Copy your **URI** value from the portal (using the copy button) and make it the value of the **endpoint** variable in *cosmos_get_started.py*. 

    `endpoint = 'https://FILLME.documents.azure.com',`

4. Then copy your **PRIMARY KEY** value from the portal and make it the value of the **key** in *cosmos_get_started.py*. You've now updated your app with all the info it needs to communicate with Azure Cosmos DB. 

    `key = 'FILLME'`

5. Save the *cosmos_get_started.py* file.

## Review the code

This step is optional. Learn about the database resources created in code, or skip ahead to [Update your connection string](#update-your-connection-string).

The following snippets are all taken from the *cosmos_get_started.py* file.

* The CosmosClient is initialized. Make sure to update the "endpoint" and "key" values as described in the [Update your connection string](#update-your-connection-string) section. 

    [!code-python[](~/azure-cosmos-db-python-getting-started/cosmos_get_started.py?name=create_cosmos_client)]

* A new database is created.

    [!code-python[](~/azure-cosmos-db-python-getting-started/cosmos_get_started.py?name=create_database_if_not_exists)]

* A new container is created, with 400 RU/s of [provisioned throughput](../request-units.md). We choose `lastName` as the [partition key](../partitioning-overview.md#choose-partitionkey), which allows us to do efficient queries that filter on this property. 

    [!code-python[](~/azure-cosmos-db-python-getting-started/cosmos_get_started.py?name=create_container_if_not_exists)]

* Some items are added to the container. Containers are a collection of items (JSON documents) that can have varied schema. The helper methods ```get_[name]_family_item``` return representations of a family that are stored in Azure Cosmos DB as JSON documents.

    [!code-python[](~/azure-cosmos-db-python-getting-started/cosmos_get_started.py?name=create_item)]

* Point reads (key value lookups) are performed using the `read_item` method. We print out the [RU charge](../request-units.md) of each operation.

    [!code-python[](~/azure-cosmos-db-python-getting-started/cosmos_get_started.py?name=read_item)]

* A query is performed using SQL query syntax. Because we're using partition key values of ```lastName``` in the WHERE clause, Azure Cosmos DB will efficiently route this query to the relevant partitions, improving performance.

    [!code-python[](~/azure-cosmos-db-python-getting-started/cosmos_get_started.py?name=query_items)]
   
## Run the app

1. In Visual Studio Code, select **View** > **Command Palette**. 

2. At the prompt, enter  **Python: Select Interpreter** and then select the version of Python to use.

    The Footer in Visual Studio Code is updated to indicate the interpreter selected. 

3. Select **View** > **Integrated Terminal** to open the Visual Studio Code integrated terminal.

4. In the integrated terminal window, ensure you are in the *azure-cosmos-db-python-getting-started* folder. If not, run the following command to switch to the sample folder. 

    ```cmd
    cd "\git-samples\azure-cosmos-db-python-getting-started"`
    ```

5. Run the following command to install the azure-cosmos package. 

    ```python
    pip install --pre azure-cosmos
    ```

    If you get an error about access being denied when attempting to install azure-cosmos, you'll need to [run VS Code as an administrator](https://stackoverflow.com/questions/37700536/visual-studio-code-terminal-how-to-run-a-command-with-administrator-rights).

6. Run the following command to run the sample and create and store new documents in Azure Cosmos DB.

    ```python
    python cosmos_get_started.py
    ```

7. To confirm the new items were created and saved, in the Azure portal, select **Data Explorer** > **AzureSampleFamilyDatabase** > **Items**. View the items that were created. For example, here is a sample JSON document for the Andersen family:
   
   ```json
   {
       "id": "Andersen-1569479288379",
       "lastName": "Andersen",
       "district": "WA5",
       "parents": [
           {
               "familyName": null,
               "firstName": "Thomas"
           },
           {
               "familyName": null,
               "firstName": "Mary Kay"
           }
       ],
       "children": null,
       "address": {
           "state": "WA",
           "county": "King",
           "city": "Seattle"
       },
       "registered": true,
       "_rid": "8K5qAIYtZXeBhB4AAAAAAA==",
       "_self": "dbs/8K5qAA==/colls/8K5qAIYtZXc=/docs/8K5qAIYtZXeBhB4AAAAAAA==/",
       "_etag": "\"a3004d78-0000-0800-0000-5d8c5a780000\"",
       "_attachments": "attachments/",
       "_ts": 1569479288
   }
   ```

## Review SLAs in the Azure portal

[!INCLUDE [cosmosdb-tutorial-review-slas](../includes/cosmos-db-tutorial-review-slas.md)]

## Clean up resources

[!INCLUDE [cosmosdb-delete-resource-group](../includes/cosmos-db-delete-resource-group.md)]

## Next steps

In this quickstart, you've learned how to create an Azure Cosmos DB account, create a container using the Data Explorer, and run a Python app in Visual Studio Code. You can now import additional data to your Azure Cosmos DB account. 

Trying to do capacity planning for a migration to Azure Cosmos DB? You can use information about your existing database cluster for capacity planning.
* If all you know is the number of vcores and servers in your existing database cluster, read about [estimating request units using vCores or vCPUs](../convert-vcore-to-request-unit.md) 
* If you know typical request rates for your current database workload, read about [estimating request units using Azure Cosmos DB capacity planner](estimate-ru-with-capacity-planner.md)

> [!div class="nextstepaction"]
> [Import data into Azure Cosmos DB for the SQL API](../import-data.md)
