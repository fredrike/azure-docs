---
title: Azure Cosmos DB best practices for .NET SDK v3
description: Learn the best practices for using the Azure Cosmos DB .NET SDK v3
author: StefArroyo
ms.service: cosmos-db
ms.subservice: nosql
ms.topic: how-to
ms.date: 08/30/2022
ms.author: esarroyo
ms.reviewer: mjbrown
ms.custom: cosmos-db-video
---

# Best practices for Azure Cosmos DB .NET SDK
[!INCLUDE[NoSQL](../includes/appliesto-nosql.md)]

This article walks through the best practices for using the Azure Cosmos DB .NET SDK. Following these practices, will help improve your latency, availability, and boost overall performance.

Watch the video below to learn more about using the .NET SDK from an Azure Cosmos DB engineer!

>
> [!VIDEO https://aka.ms/docs.dotnet-best-practices]

## Checklist
|Checked  | Subject  |Details/Links  |
|---------|---------|---------|
|<input type="checkbox"/> |    SDK Version    |   Always using the [latest version](sdk-dotnet-v3.md) of the Azure Cosmos DB SDK available for optimal performance.     |
| <input type="checkbox"/>   |    Singleton Client     |       Use a [single instance](/dotnet/api/microsoft.azure.cosmos.cosmosclient?view=azure-dotnet&preserve-view=true) of `CosmosClient` for the lifetime of your application for [better performance](performance-tips-dotnet-sdk-v3.md#sdk-usage).     |
| <input type="checkbox"/>  |     Regions     |   Make sure to run your application in the same [Azure region](../distribute-data-globally.md) as your Azure Cosmos DB account, whenever possible to reduce latency. Enable 2-4 regions and replicate your accounts in multiple regions for [best availability](../distribute-data-globally.md). For production workloads, enable [automatic failover](../how-to-manage-database-account.md#configure-multiple-write-regions). In the absence of this configuration, the account will experience loss of write availability for all the duration of the write region outage, as manual failover won't succeed due to lack of region connectivity. To learn how to add multiple regions using the .NET SDK visit [here](tutorial-global-distribution.md)   |
| <input type="checkbox"/>   |   Availability and Failovers     |  Set the [ApplicationPreferredRegions](/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.applicationpreferredregions?view=azure-dotnet&preserve-view=true) or [ApplicationRegion](/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.applicationregion?view=azure-dotnet&preserve-view=true) in the v3 SDK, and the [PreferredLocations](/dotnet/api/microsoft.azure.documents.client.connectionpolicy.preferredlocations?view=azure-dotnet&preserve-view=true) in the v2 SDK using the  [preferred regions list](./tutorial-global-distribution.md?tabs=dotnetv3%2capi-async#preferred-locations). During failovers, write operations are sent to the current write region and all reads are sent to the first region within your preferred regions list. For more information about regional failover mechanics, see the [availability troubleshooting guide](troubleshoot-sdk-availability.md). |
|  <input type="checkbox"/> |    CPU     |  You may run into connectivity/availability issues due to lack of resources on your client machine. Monitor your CPU utilization on nodes running the Azure Cosmos DB client, and scale up/out if usage is high.      |
| <input type="checkbox"/>   |    Hosting      |   Use [Windows 64-bit host](performance-tips.md#hosting) processing for best performance, whenever possible.       |
| <input type="checkbox"/> |    Connectivity Modes    |    Use [Direct mode](sdk-connection-modes.md) for the best performance.  For instructions on how to do this, see the [V3 SDK documentation](performance-tips-dotnet-sdk-v3.md#networking) or the [V2 SDK documentation](performance-tips.md#networking).|
|<input type="checkbox"/>  |    Networking   | If using a virtual machine to run your application, enable [Accelerated Networking](../../virtual-network/create-vm-accelerated-networking-powershell.md) on your VM to help with bottlenecks due to high traffic and reduce latency or CPU jitter. You might also want to consider using a higher end Virtual Machine where the max CPU usage is under 70%.    |
|<input type="checkbox"/> |  Ephemeral Port Exhaustion      | For sparse or sporadic connections, we set the [`IdleConnectionTimeout`](/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.idletcpconnectiontimeout?view=azure-dotnet&preserve-view=true) and [`PortReuseMode`](/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.portreusemode?view=azure-dotnet&preserve-view=true) to `PrivatePortPool`. The `IdleConnectionTimeout` property helps which control the time unused connections are closed. This will reduce the number of unused connections. By default, idle connections are kept open indefinitely. The value set must be greater than or equal to 10 minutes. We recommended values between 20 minutes and 24 hours.  The `PortReuseMode` property allows the SDK to use a small pool of ephemeral ports for various Azure Cosmos DB destination endpoints.    |
|<input type="checkbox"/> |  Use Async/Await     |  Avoid blocking calls: `Task.Result`, `Task.Wait`, and `Task.GetAwaiter().GetResult()`. The entire call stack is asynchronous in order to benefit from [async/await](/dotnet/csharp/programming-guide/concepts/async/) patterns. Many synchronous blocking calls lead to [Thread Pool starvation](/archive/blogs/vancem/diagnosing-net-core-threadpool-starvation-with-perfview-why-my-service-is-not-saturating-all-cores-or-seems-to-stall) and degraded response times. |
|<input type="checkbox"/>  |   End-to-End Timeouts | To get end-to-end timeouts, you'll need to use both `RequestTimeout` and `CancellationToken` parameters. For more details on timeouts with Azure Cosmos DB [visit](troubleshoot-dotnet-sdk-request-timeout.md) |
|<input type="checkbox"/>  |   Retry Logic      |   A transient error is an error that has an underlying cause that soon resolves itself. Applications that connect to your database should be built to expect these transient errors. To handle them, implement retry logic in your code instead of surfacing them to users as application errors. The SDK has built-in logic to handle these transient failures on retryable requests like read or query operations. The SDK won't retry on writes for transient failures as writes aren't idempotent. The SDK does allow users to configure retry logic for throttles. For details on which errors to retry on [visit](conceptual-resilient-sdk-applications.md#should-my-application-retry-on-errors) |
|<input type="checkbox"/>  |     Caching database/collection names    |    Retrieve the names of your databases and containers from configuration or cache them on start. Calls like `ReadDatabaseAsync` or `ReadDocumentCollectionAsync` and  `CreateDatabaseQuery` or `CreateDocumentCollectionQuery` will result in metadata calls to the service, which consume from the system-reserved RU limit. `CreateIfNotExist` should also only be used once for setting up the database. Overall, these operations should be performed infrequently.       |
|<input type="checkbox"/> |     Bulk Support      |     In scenarios where you may not need to optimize for latency, we recommend enabling [Bulk support](https://devblogs.microsoft.com/cosmosdb/introducing-bulk-support-in-the-net-sdk/) for dumping large volumes of data.    |
| <input type="checkbox"/>  |     Parallel Queries     |    The Azure Cosmos DB SDK supports [running queries in parallel](performance-tips-query-sdk.md?pivots=programming-language-csharp) for better latency and throughput on your queries.  We recommend setting the `MaxConcurrency` property within the `QueryRequestsOptions` to the number of partitions you have. If you aren't aware of the number of partitions, start by using `int.MaxValue`, which will give you the best latency. Then decrease the number until it fits the resource restrictions of the environment to avoid high CPU issues. Also, set the `MaxBufferedItemCount` to the expected number of results returned to limit the number of pre-fetched results. |
| <input type="checkbox"/> |     Performance Testing Backoffs      |    When performing testing on your application,  you should implement backoffs at [`RetryAfter`](performance-tips-dotnet-sdk-v3.md#sdk-usage) intervals. Respecting the backoff helps ensure that you'll spend a minimal amount of time waiting between retries.   |
|  <input type="checkbox"/>   |   Indexing     |   The Azure Cosmos DB indexing policy also allows you to specify which document paths to include or exclude from indexing by using indexing paths (IndexingPolicy.IncludedPaths and IndexingPolicy.ExcludedPaths).  Ensure that you exclude unused paths from indexing for faster writes.  For a sample on how to create indexes using the SDK [visit](performance-tips-dotnet-sdk-v3.md#indexing-policy)   |
|  <input type="checkbox"/>   |    Document Size  |    The request charge of a specified operation correlates directly to the size of the document. We recommend reducing the size of your documents as operations on large documents cost more than operations on smaller documents.      |
| <input type="checkbox"/>   |     Increase the number of threads/tasks    |    Because calls to Azure Cosmos DB are made over the network, you might need to vary the degree of concurrency of your requests so that the client application spends minimal time waiting between requests. For example, if you're using the [.NET Task Parallel Library](/dotnet/standard/parallel-programming/task-parallel-library-tpl), create on the order of hundreds of tasks that read from or write to Azure Cosmos DB.     |
|  <input type="checkbox"/> |    Enabling Query Metrics     |  For more logging of your backend query executions, you can enable SQL Query Metrics using our .NET SDK. For instructions on how to collect SQL Query Metrics [visit](query-metrics-performance.md)    |
|  <input type="checkbox"/>    | SDK Logging   | Log [SDK diagnostics](#capture-diagnostics) for outstanding scenarios, such as exceptions or when requests go beyond an expected latency.
|  <input type="checkbox"/>    | DefaultTraceListener   | The DefaultTraceListener poses performance issues on production environments causing high CPU and I/O bottlenecks. Make sure you're using the latest SDK versions or [remove the DefaultTraceListener from your application](performance-tips-dotnet-sdk-v3.md#logging-and-tracing)  |

## Capture diagnostics

[!INCLUDE[cosmos-db-dotnet-sdk-diagnostics](../includes/dotnet-sdk-diagnostics.md)]

## Best practices when using Gateway mode

Increase `System.Net MaxConnections` per host when you use Gateway mode. Azure Cosmos DB requests are made over HTTPS/REST when you use Gateway mode. They're subject to the default connection limit per hostname or IP address. You might need to set `MaxConnections` to a higher value (from 100 through 1,000) so that the client library can use multiple simultaneous connections to Azure Cosmos DB. In .NET SDK 1.8.0 and later, the default value for `ServicePointManager.DefaultConnectionLimit` is 50. To change the value, you can set `Documents.Client.ConnectionPolicy.MaxConnectionLimit` to a higher value.

## Best practices for write-heavy workloads

For workloads that have heavy create payloads, set the `EnableContentResponseOnWrite` request option to `false`. The service will no longer return the created or updated resource to the SDK. Normally, because the application has the object that's being created, it doesn't need the service to return it. The header values are still accessible, like a request charge. Disabling the content response can help improve performance, because the SDK no longer needs to allocate memory or serialize the body of the response. It also reduces the network bandwidth usage to further help performance.

## Next steps

For a sample application that's used to evaluate Azure Cosmos DB for high-performance scenarios on a few client machines, see [Performance and scale testing with Azure Cosmos DB](performance-testing.md).

To learn more about designing your application for scale and high performance, see [Partitioning and scaling in Azure Cosmos DB](../partitioning-overview.md).

Trying to do capacity planning for a migration to Azure Cosmos DB? You can use information about your existing database cluster for capacity planning.
* If all you know is the number of vCores and servers in your existing database cluster, read about [estimating request units using vCores or vCPUs](../convert-vcore-to-request-unit.md) 
* If you know typical request rates for your current database workload, read about [estimating request units using Azure Cosmos DB capacity planner](estimate-ru-with-capacity-planner.md)
