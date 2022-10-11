---
title: Azure Monitor Ingestion client library for .NET
keywords: Azure, dotnet, SDK, API, Azure.Monitor.Ingestion, monitor
author: SameergMS
ms.author: sameerg
ms.date: 10/11/2022
ms.topic: reference
ms.devlang: dotnet
ms.service: monitor
---
# Azure Monitor Ingestion client library for .NET - version 1.0.0-beta.4 


The Azure Monitor Ingestion client library is used to send custom logs to [Azure Monitor][azure_monitor_overview].

This library allows you to send data from virtually any source to supported built-in tables or to custom tables that you create in Log Analytics workspace. You can even extend the schema of built-in tables with custom columns.

**Resources:**

- [Source code](https://github.com/Azure/azure-sdk-for-net/blob/Azure.Monitor.Ingestion_1.0.0-beta.4/sdk/monitor/Azure.Monitor.Ingestion/src)
- [NuGet package](https://www.nuget.org/packages/Azure.Monitor.Ingestion)
- [Service documentation][azure_monitor_overview]
- [Change log](https://github.com/Azure/azure-sdk-for-net/blob/Azure.Monitor.Ingestion_1.0.0-beta.4/sdk/monitor/Azure.Monitor.Ingestion/CHANGELOG.md)

## Getting started

### Prerequisites

- An [Azure subscription](https://azure.microsoft.com/free/dotnet/)
- A [TokenCredential](https://learn.microsoft.com/dotnet/api/azure.core.tokencredential?view=azure-dotnet) implementation, such as an [Azure Identity library credential type](https://learn.microsoft.com/dotnet/api/overview/azure/Identity-readme#credential-classes).
- A [Data Collection Endpoint](https://learn.microsoft.com/azure/azure-monitor/essentials/data-collection-endpoint-overview)
- A [Data Collection Rule](https://learn.microsoft.com/azure/azure-monitor/essentials/data-collection-rule-overview)
- A [Log Analytics workspace](https://learn.microsoft.com/azure/azure-monitor/logs/log-analytics-workspace-overview)

### Install the package

Install the Azure Monitor Ingestion client library for .NET with NuGet:

```dotnetcli
dotnet add package Azure.Monitor.Ingestion --prerelease
```

### Authenticate the client

An authenticated client is required to ingest data. To authenticate, create an instance of a `TokenCredential` class. Pass it to the constructor of the `LogsIngestionClient` class.

To authenticate, the following example uses `DefaultAzureCredential` from the `Azure.Identity` package:

```C# Snippet:CreateLogsIngestionClient
var endpoint = new Uri("<data_collection_endpoint_uri>");
var credential = new DefaultAzureCredential();
var client = new LogsIngestionClient(endpoint, credential);
```

### Upload the logs

For examples of logs ingestion, see the [Examples](#examples) section.

## Key concepts

### Data collection endpoint

Data collection endpoints (DCEs) allow you to uniquely configure ingestion settings for Azure Monitor. [This article][data_collection_endpoint] provides an overview of DCEs, including their contents, structure, and how you can create and work with them.

### Data collection rule

Data collection rules (DCRs) define data collected by Azure Monitor and specify how and where that data should be sent or stored. The REST API call must specify a DCR to use. A single DCE can support multiple DCRs, so you can specify a different DCR for different sources and target tables.

The DCR must understand the structure of the input data and the structure of the target table. If the two don't match, it can use a transformation to convert the source data to match the target table. You may also use the transform to filter source data and perform any other calculations or conversions.

For more information, see [Data collection rules in Azure Monitor][data_collection_rule].

### Log Analytics workspace tables

Custom logs can send data to any custom table that you create and to certain built-in tables in your Log Analytics workspace. The target table must exist before you can send data to it. The following built-in tables are currently supported:

- [CommonSecurityLog](https://learn.microsoft.com/azure/azure-monitor/reference/tables/commonsecuritylog)
- [SecurityEvent](https://learn.microsoft.com/azure/azure-monitor/reference/tables/securityevent)
- [Syslog](https://learn.microsoft.com/azure/azure-monitor/reference/tables/syslog)
- [WindowsEvent](https://learn.microsoft.com/azure/azure-monitor/reference/tables/windowsevent)

### Thread safety

We guarantee that all client instance methods are thread-safe and independent of each other ([guideline](https://azure.github.io/azure-sdk/dotnet_introduction.html#dotnet-service-methods-thread-safety)). This design ensures that the recommendation of reusing client instances is always safe, even across threads.

### Additional concepts

<!-- CLIENT COMMON BAR -->
[Client options](https://github.com/Azure/azure-sdk-for-net/blob/Azure.Monitor.Ingestion_1.0.0-beta.4/sdk/core/Azure.Core/README.md#configuring-service-clients-using-clientoptions) |
[Accessing the response](https://github.com/Azure/azure-sdk-for-net/blob/Azure.Monitor.Ingestion_1.0.0-beta.4/sdk/core/Azure.Core/README.md#accessing-http-response-details-using-responset) |
[Long-running operations](https://github.com/Azure/azure-sdk-for-net/blob/Azure.Monitor.Ingestion_1.0.0-beta.4/sdk/core/Azure.Core/README.md#consuming-long-running-operations-using-operationt) |
[Handling failures](https://github.com/Azure/azure-sdk-for-net/blob/Azure.Monitor.Ingestion_1.0.0-beta.4/sdk/core/Azure.Core/README.md#reporting-errors-requestfailedexception) |
[Diagnostics](https://github.com/Azure/azure-sdk-for-net/blob/Azure.Monitor.Ingestion_1.0.0-beta.4/sdk/core/Azure.Core/samples/Diagnostics.md) |
[Mocking](https://github.com/Azure/azure-sdk-for-net/blob/Azure.Monitor.Ingestion_1.0.0-beta.4/sdk/core/Azure.Core/README.md#mocking) |
[Client lifetime](https://devblogs.microsoft.com/azure-sdk/lifetime-management-and-thread-safety-guarantees-of-azure-sdk-net-clients/)
<!-- CLIENT COMMON BAR -->

## Examples

- [Upload custom logs](#upload-custom-logs)
- [Verify logs](#verify-logs)

You can familiarize yourself with different APIs using [samples](https://github.com/Azure/azure-sdk-for-net/tree/Azure.Monitor.Ingestion_1.0.0-beta.4/sdk/monitor/Azure.Monitor.Ingestion/samples).

### Upload custom logs

You can upload logs using either the `LogsIngestionClient.Upload` or the `LogsIngestionClient.UploadAsync` method. Note the data ingestion [limits](https://learn.microsoft.com/azure/azure-monitor/service-limits#custom-logs).

```C# Snippet:UploadCustomLogsAsync
var endpoint = new Uri("<data_collection_endpoint>");
var ruleId = "<data_collection_rule_id>";
var streamName = "<stream_name>";

var credential = new DefaultAzureCredential();
LogsIngestionClient client = new(endpoint, credential);
DateTimeOffset currentTime = DateTimeOffset.UtcNow;

// Use BinaryData to serialize instances of an anonymous type into JSON
BinaryData data = BinaryData.FromObjectAsJson(
    new[] {
        new
        {
            Time = currentTime,
            Computer = "Computer1",
            AdditionalContext = new
            {
                InstanceName = "user1",
                TimeZone = "Pacific Time",
                Level = 4,
                CounterName = "AppMetric1",
                CounterValue = 15.3
            }
        },
        new
        {
            Time = currentTime,
            Computer = "Computer2",
            AdditionalContext = new
            {
                InstanceName = "user2",
                TimeZone = "Central Time",
                Level = 3,
                CounterName = "AppMetric1",
                CounterValue = 23.5
            }
        },
    });

// Upload our logs
Response response = await client.UploadAsync(
    ruleId,
    streamName,
    RequestContent.Create(data)).ConfigureAwait(false);
```

### Verify logs

You can verify that your data has been uploaded correctly by using the [Azure Monitor Query](https://github.com/Azure/azure-sdk-for-net/blob/Azure.Monitor.Ingestion_1.0.0-beta.4/sdk/monitor/Azure.Monitor.Query/README.md#install-the-package) library. Run the [Upload custom logs](#upload-custom-logs) sample first before verifying the logs.

```C# Snippet:VerifyLogsAsync
var workspaceId = "<log_analytics_workspace_id>";
var tableName = "<table_name>";

var credential = new DefaultAzureCredential();
LogsQueryClient logsQueryClient = new(credential);

LogsBatchQuery batch = new();
string query = tableName + " | Count;";
string countQueryId = batch.AddWorkspaceQuery(
    workspaceId,
    query,
    new QueryTimeRange(TimeSpan.FromDays(1)));

Response<LogsBatchQueryResultCollection> queryResponse =
    await logsQueryClient.QueryBatchAsync(batch).ConfigureAwait(false);

Console.WriteLine("Table entry count: " +
    queryResponse.Value.GetResult<int>(countQueryId).Single());
```

## Troubleshooting

### Enabling logging

The Azure SDK for .NET offers a consistent logging story to aid in troubleshooting application errors and expedite their resolution. The logs produced will capture the application's flow before reaching the terminal state to help locate the root issue. For information on enabling logging, see [Logging with the Azure SDK for .NET][logging].

## Next steps

To learn more about Azure Monitor, see the [Azure Monitor service documentation][azure_monitor_overview].

## Contributing

This project welcomes contributions and suggestions. Most contributions require you to agree to a Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us the rights to use your contribution. For details, visit [https://cla.microsoft.com](https://cla.microsoft.com).

When you submit a pull request, a CLA-bot will automatically determine whether you need to provide a CLA and decorate the PR appropriately. For example, labels and comments. Follow the instructions provided by the bot. You'll only need to sign the CLA once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). For more information, see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any questions or comments.

<!-- LINKS -->
[azure_monitor_overview]: https://learn.microsoft.com/azure/azure-monitor/overview
[data_collection_endpoint]: https://learn.microsoft.com/azure/azure-monitor/essentials/data-collection-endpoint-overview
[data_collection_rule]: https://learn.microsoft.com/azure/azure-monitor/essentials/data-collection-rule-overview
[logging]: https://learn.microsoft.com/dotnet/azure/sdk/logging

![Impressions](https://azure-sdk-impressions.azurewebsites.net/api/impressions/azure-sdk-for-net/sdk/monitor/Azure.Monitor.Ingestion/README.png)
