---
author: craigshoemaker
ms.service: azure-functions
ms.topic: include
ms.date: 02/21/2020
ms.author: cshoe
---

::: zone pivot="programming-language-csharp"
## Install extension

The extension NuGet package you install depends on the C# mode you're using in your function app: 

# [In-process](#tab/in-process)

Functions execute in the same process as the Functions host. To learn more, see [Develop C# class library functions using Azure Functions](../articles/azure-functions/functions-dotnet-class-library.md).

# [Isolated process](#tab/isolated-process)

Functions execute in an isolated C# worker process. To learn more, see [Guide for running functions on .NET 5.0 in Azure](../articles/azure-functions/dotnet-isolated-process-guide.md).

# [C# script](#tab/csharp-script)

Functions run as C# script, which is supported primarily for C# portal editing. To update existing binding extensions for C# script apps running in the portal without having to republish your function app, see [Update your extensions].

---

The functionality of the extension varies depending on the extension version:

# [Extension v5.x+](#tab/extensionv5/in-process)

[!INCLUDE [functions-bindings-event-hubs-extension-5](functions-bindings-event-hubs-extension-5.md)] 

This version uses the new newer Event Hubs binding type [Azure.Messaging.EventHubs.EventData](/dotnet/api/azure.messaging.eventhubs.eventdata).

This extension version is available by installing the [NuGet package], version 5.x

# [Extension v3.x+](#tab/extensionv3/in-process)

Supports the original Event Hubs binding parameter type of [Microsoft.Azure.EventHubs.EventData](/dotnet/api/microsoft.azure.eventhubs.eventdata).

Add the extension to your project by installing the [NuGet package], version 3.x or 4.x.

# [Functions v1.x](#tab/functionsv1/in-process)

Version 1.x of the Functions runtime doesn't require an extension. 

# [Extension v5.x+](#tab/extensionv5/isolated-process)

[!INCLUDE [functions-bindings-event-hubs-extension-5](functions-bindings-event-hubs-extension-5.md)] 

Add the extension to your project by installing the [NuGet package](https://www.nuget.org/packages/Microsoft.Azure.Functions.Worker.Extensions.EventHubs), version 5.x.

# [Extension v3.x+](#tab/extensionv3/isolated-process)

Add the extension to your project by installing the [NuGet package](https://www.nuget.org/packages/Microsoft.Azure.Functions.Worker.Extensions.EventHubs), version 4.x.

# [Functions v1.x](#tab/functionsv1/isolated-process)

Version 1.x of the Functions runtime doesn't supported running in an isolated process. 

# [Extension v5.x+](#tab/extensionv5/csharp-script)

[!INCLUDE [functions-bindings-event-hubs-extension-5](functions-bindings-event-hubs-extension-5.md)]  

This version uses the new newer Event Hubs binding type [Azure.Messaging.EventHubs.EventData](/dotnet/api/azure.messaging.eventhubs.eventdata).

You can install this version of the extension in your function app by registering the [extension bundle], version 3.x.

# [Extension v3.x+](#tab/extensionv3/csharp-script)

Supports the original Event Hubs binding parameter type of [Microsoft.Azure.EventHubs.EventData](/dotnet/api/microsoft.azure.eventhubs.eventdata).

You can install this version of the extension in your function app by registering the [extension bundle], version 2.x. 

# [Functions v1.x](#tab/functionsv1/csharp-script)

Version 1.x of the Functions runtime doesn't require an extension. 

---

::: zone-end  

::: zone pivot="programming-language-javascript,programming-language-python,programming-language-java,programming-language-powershell"  

## Install bundle

The Event Hubs extension is part of an [extension bundle], which is specified in your host.json project file. You may need to modify this bundle to change the version of the Event Grid binding, or if bundles aren't already installed. To learn more, see [extension bundle].

# [Bundle v3.x](#tab/extensionv5)

[!INCLUDE [functions-bindings-event-hubs-extension-5](functions-bindings-event-hubs-extension-5.md)] 

You can add this version of the extension from the extension bundle v3 by adding or replacing the following code in your `host.json` file:

```json
{
  "version": "2.0",
  "extensionBundle": {
    "id": "Microsoft.Azure.Functions.ExtensionBundle",
    "version": "[3.3.0, 4.0.0)"
  }
}
```

To learn more, see [Update your extensions].

# [Bundle v2.x](#tab/extensionv3)

You can install this version of the extension in your function app by registering the [extension bundle], version 2.x.

# [Functions v1.x](#tab/functionsv1)

Version 1.x of the Functions runtime doesn't require extension bundles. 

---

::: zone-end

## host.json settings
<a name="host-json"></a>

The [host.json](../articles/azure-functions/functions-host-json.md#eventhub) file contains settings that control behavior for the Event Hubs trigger. The configuration is different depending on the extension version.

# [Extension v5.x+](#tab/extensionv5)

```json
{
    "version": "2.0",
    "extensions": {
        "eventHubs": {
            "maxEventBatchSize" : 10,
            "batchCheckpointFrequency" : 5,
            "prefetchCount" : 300,
            "transportType" : "amqpWebSockets",
            "webProxy" : "https://proxyserver:8080",
            "customEndpointAddress" : "amqps://company.gateway.local",
            "initialOffsetOptions" : {
                "type" : "fromStart",
                "enqueuedTimeUtc" : ""
            },
            "clientRetryOptions":{
                "mode" : "exponential",
                "tryTimeout" : "00:01:00",
                "delay" : "00:00:00.80",
                "maxDelay" : "00:01:00",
                "maxRetries" : 3
            }
        }
    }
}  
```

|Property  |Default | Description |
|---------|---------|---------|
| maxEventBatchSize| 10| The maximum number of events that will be included in a batch for a single invocation. Must be at least 1.|
| batchCheckpointFrequency| 1| The number of batches to process before creating  a checkpoint for the event hub.|
| prefetchCount| 300| The number of events that will be eagerly requested from Event Hubs and held in  a local cache to allow reads to avoid waiting on a network operation|
| transportType| amqpTcp | The protocol and transport that is used for communicating with Event Hubs. Available options: `amqpTcp`, `amqpWebSockets`|
| webProxy| | The proxy to use for communicating with Event Hubs over web sockets. A proxy cannot be used with the `amqpTcp` transport. |
| customEndpointAddress | | The address to use when establishing a connection to Event Hubs, allowing network requests to be routed through an application gateway or other path needed for the host environment. The fully qualified namespace for the event hub is still needed when a custom endpoint address is used and must be specified explicitly or via the connection string.|
| initialOffsetOptions/type | fromStart| The location in the event stream to start processing when a checkpoint does not exist in storage. Applies to all partitions. For more information, see the  [OffsetType documentation](/dotnet/api/microsoft.azure.webjobs.eventhubs.offsettype). Available options: `fromStart`, `fromEnd`, `fromEnqueuedTime`|
| initialOffsetOptions/enqueuedTimeUtc | | Specifies the enqueued time of the event in the stream from which to start processing. When `initialOffsetOptions/type` is configured as `fromEnqueuedTime`, this setting is mandatory. Supports time in any format supported by [DateTime.Parse()](/dotnet/standard/base-types/parsing-datetime), such as  `2020-10-26T20:31Z`. For clarity, you should also specify a timezone. When timezone isn't specified, Functions assumes the local timezone of the machine running the function app, which is UTC when running on Azure. |
| clientRetryOptions/mode | exponential | The approach to use for calculating retry delays. Exponential mode will retry attempts with a delay based on a back-off strategy where each attempt will increase the duration that it waits before retrying. The fixed mode will retry attempts at fixed intervals with each delay having a consistent duration. Available options: `exponential`, `fixed`|
| clientRetryOptions/tryTimeout | 00:01:00 | The maximum duration to wait for an Event Hubs operation to complete, per attempt.|
| clientRetryOptions/delay | 00:00:00.80 | The delay or back-off factor to apply between retry attempts.|
| clientRetryOptions/maxDelay | 00:00:01 | The maximum delay to allow between retry attempts. |
| clientRetryOptions/maxRetries | 3 | The maximum number of retry attempts before considering the associated operation to have failed.|

For a reference of host.json in Azure Functions 2.x and beyond, see [host.json reference for Azure Functions](../articles/azure-functions/functions-host-json.md).

# [Extension v3.x+](#tab/extensionv3)

```json
{
    "version": "2.0",
    "extensions": {
        "eventHubs": {
            "batchCheckpointFrequency": 5,
            "eventProcessorOptions": {
                "maxBatchSize": 256,
                "prefetchCount": 512
            },
            "initialOffsetOptions": {
                "type": "fromStart",
                "enqueuedTimeUtc": ""
            }
        }
    }
}  
```

|Property  |Default | Description |
|---------|---------|---------|
|batchCheckpointFrequency|1|The number of event batches to process before creating an EventHub cursor checkpoint.|
|eventProcessorOptions/maxBatchSize|10|The maximum event count received per receive loop.|
|eventProcessorOptions/prefetchCount|300|The default prefetch count used by the underlying `EventProcessorHost`. The minimum allowed value is 10.|
|initialOffsetOptions/type<sup>1</sup>|fromStart|The location in the event stream from which to start processing when a checkpoint doesn't exist in storage. Options are `fromStart` , `fromEnd` or `fromEnqueuedTime`. `fromEnd` processes new events that were enqueued after the function app started running. Applies to all partitions. For more information, see the [EventProcessorOptions documentation](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessoroptions.initialoffsetprovider).|
|initialOffsetOptions/enqueuedTimeUtc<sup>1</sup>| | Specifies the enqueued time of the event in the stream from which to start processing. When `initialOffsetOptions/type` is configured as `fromEnqueuedTime`, this setting is mandatory. Supports time in any format supported by [DateTime.Parse()](/dotnet/standard/base-types/parsing-datetime), such as  `2020-10-26T20:31Z`. For clarity, you should also specify a timezone. When timezone isn't specified, Functions assumes the local timezone of the machine running the function app, which is UTC when running on Azure. For more information, see the [EventProcessorOptions documentation](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessoroptions.initialoffsetprovider).|

<sup>1</sup> Support for `initialOffsetOptions` begins with [EventHubs v4.2.0](https://github.com/Azure/azure-functions-eventhubs-extension/releases/tag/v4.2.0).

For a reference of host.json in Azure Functions 2.x and beyond, see [host.json reference for Azure Functions](../articles/azure-functions/functions-host-json.md).

# [Functions v1.x](#tab/functionsv1)

```json
{
    "eventHub": {
      "maxBatchSize": 64,
      "prefetchCount": 256,
      "batchCheckpointFrequency": 1
    }
}
```

|Property  |Default | Description |
|---------|---------|---------| 
|maxBatchSize|64|The maximum event count received per receive loop.|
|prefetchCount| 300 |The default prefetch that will be used by the underlying `EventProcessorHost`.| 
|batchCheckpointFrequency|1|The number of event batches to process before creating an EventHub cursor checkpoint.| 

For a reference of host.json in Azure Functions 1.x, see [host.json reference for Azure Functions 1.x](../articles/azure-functions/functions-host-json-v1.md).

---


[NuGet package]: https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.EventHubs
[extension bundle]: ../articles/azure-functions/functions-bindings-register.md#extension-bundles
[Update your extensions]: ../articles/azure-functions/functions-bindings-register.md
