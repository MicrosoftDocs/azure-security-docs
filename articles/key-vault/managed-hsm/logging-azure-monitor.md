---
title: Azure Managed HSM logging
description: Use this tutorial to help you get started with Managed HSM logging.
services: key-vault
author: msmbaldwin
ms.author: mbaldwin
ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: tutorial
ms.date: 01/30/2025

#Customer intent: As a security administrator, I want to enable and configure logging for Azure Managed HSM using Azure Monitor so I can monitor and audit the security and usage of my managed hardware security modules.
---

# Monitor Azure Managed HSM

This article describes:

- The types of monitoring data you can collect for this service.
- Ways to analyze that data.

When you have critical applications and business processes that rely on Azure resources, you need to monitor and get alerts for your system. The Azure Monitor service collects and aggregates metrics and logs from every component of your system. Azure Monitor provides you with a view of availability, performance, and resilience, and notifies you of issues. You can use the Azure portal, PowerShell, Azure CLI, REST API, or client libraries to set up and view monitoring data.

For more information on Azure Monitor, see the [Azure Monitor overview](/azure/azure-monitor/overview). For more information on how to monitor Azure resources in general, see [Monitor Azure resources with Azure Monitor](/azure/azure-monitor/overview).

## Monitoring page in Azure portal

For each managed HSM, you can navigate to the **Metrics** tab in the left-hand sidebar, under **Monitoring**, in the Azure portal to view the following metrics:

- Overall service API latency
- Overall service availability
- Total service API hits

## Resource types

Azure uses the concept of resource types and IDs to identify everything in a subscription. Azure Monitor similarly organizes core monitoring data into metrics and logs based on resource types, also called namespaces. Different metrics and logs are available for different resource types. Your service might be associated with more than one resource type.

Resource types are also part of the resource IDs for every resource running in Azure. For example, one resource type for a virtual machine is `Microsoft.Compute/virtualMachines`. For a list of services and their associated resource types, see [Resource providers](/azure/azure-resource-manager/management/resource-providers-and-types).

For more information about the resource types for managed HSMs, see [Azure Key Vault monitoring data reference](../general/monitor-key-vault-reference.md).

## Data storage

For Azure Monitor:

- Metrics data is stored in the Azure Monitor metrics database.
- Log data is stored in the Azure Monitor logs store. Log Analytics is a tool in the Azure portal that can query this store.
- The Azure activity log is a separate store with its own interface in the Azure portal.

You can optionally route metric and activity log data to the Azure Monitor logs store. You can then use Log Analytics to query the data and correlate it with other log data.

Many services can use diagnostic settings to send metric and log data to other storage locations outside Azure Monitor. Examples include Azure Storage, hosted partner systems, and non-Azure partner systems, by using Event Hubs.

For detailed information on how Azure Monitor stores data, see [Azure Monitor data platform](/azure/azure-monitor/platform/data-platform).

## Collection and routing

Platform metrics and the Activity log are collected and stored automatically, but can be routed to other locations by using a diagnostic setting.

Resource Logs aren't collected and stored until you create a diagnostic setting and route them to one or more locations.

See [Create diagnostic setting to collect platform logs and metrics in Azure](/azure/azure-monitor/platform/diagnostic-settings) for the detailed process for creating a diagnostic setting using the Azure portal, CLI, or PowerShell. When you create a diagnostic setting, you specify which categories of logs to collect. The categories for Managed HSM are listed in [Key Vault monitoring data reference](../general/monitor-key-vault-reference.md).

To create a diagnostic setting for your key vault, see [Set up logging for managed HSM](logging.md). The metrics and logs you can collect are discussed in the following sections.

## Azure Monitor platform metrics

Azure Monitor provides platform metrics for most services. These metrics are:

- Individually defined for each namespace.
- Stored in the Azure Monitor time-series metrics database.
- Lightweight and capable of supporting near real-time alerting.
- Used to track the performance of a resource over time.

**Collection:** Azure Monitor collects platform metrics automatically. No configuration is required.

**Routing:** You can also usually route platform metrics to Azure Monitor Logs / Log Analytics so you can query them with other log data. For more information, see the [Metrics diagnostic setting](/azure/azure-monitor/platform/metrics-collection). For how to configure diagnostic settings for a service, see [Create diagnostic settings in Azure Monitor](/azure/azure-monitor/platform/diagnostic-settings).

For a list of all metrics it's possible to gather for all resources in Azure Monitor, see [Supported metrics in Azure Monitor](/azure/azure-monitor/platform/metrics-supported).

You can analyze metrics for Key Vault with metrics from other Azure services using metrics explorer by opening **Metrics** from the Azure Monitor menu. See [Analyze metrics with Azure Monitor metrics explorer](/azure/azure-monitor/visualize/metrics-explorer) for details on using this tool.

For a list of available metrics for managed HSM, see [Azure Key Vault monitoring data reference](../general/monitor-key-vault-reference.md).

## Azure Monitor resource logs

Resource logs provide insight into operations that were done by an Azure resource. Logs are generated automatically, but you must route them to Azure Monitor logs to save or query them. Logs are organized in categories. A given namespace might have multiple resource log categories.

**Collection:** Resource logs aren't collected and stored until you create a diagnostic setting and route the logs to one or more locations. When you create a diagnostic setting, you specify which categories of logs to collect. There are multiple ways to create and maintain diagnostic settings, including the Azure portal, programmatically, and though Azure Policy.

**Routing:** The suggested default is to route resource logs to Azure Monitor Logs so you can query them with other log data. Other locations such as Azure Storage, Azure Event Hubs, and certain Microsoft monitoring partners are also available. For more information, see [Azure resource logs](/azure/azure-monitor/platform/resource-logs) and [Resource log destinations](/azure/azure-monitor/platform/resource-logs-destinations).

For detailed information about collecting, storing, and routing resource logs, see [Diagnostic settings in Azure Monitor](/azure/azure-monitor/platform/diagnostic-settings).

For a list of all available resource log categories in Azure Monitor, see [Supported resource logs in Azure Monitor](/azure/azure-monitor/platform/resource-logs-categories).

All resource logs in Azure Monitor have the same header fields, followed by service-specific fields. The common schema is outlined in [Azure Monitor resource log schema](/azure/azure-monitor/platform/resource-logs-schema).

For the available resource log categories, their associated Log Analytics tables, and the log schemas for managed HSM, see [Azure Key Vault monitoring data reference](../general/monitor-key-vault-reference.md).

## Azure activity log

The activity log contains subscription-level events that track operations for each Azure resource as seen from outside that resource; for example, creating a new resource or starting a virtual machine.

**Collection:** Activity log events are automatically generated and collected in a separate store for viewing in the Azure portal.

**Routing:** You can send activity log data to Azure Monitor Logs so you can analyze it alongside other log data. Other locations such as Azure Storage, Azure Event Hubs, and certain Microsoft monitoring partners are also available. For more information on how to route the activity log, see [Overview of the Azure activity log](/azure/azure-monitor/platform/activity-log).

## Analyzing logs

Data in Azure Monitor Logs is stored in tables where each table has its own set of unique properties.

All resource logs in Azure Monitor have the same fields followed by service-specific fields. The common schema is outlined in [Azure Monitor resource log schema](/azure/azure-monitor/platform/resource-logs-schema).

The Activity log is a type of platform log for Azure that provides insight into subscription-level events. You can view it independently or route it to Azure Monitor Logs, where you can do much more complex queries using Log Analytics.

For a list of the types of resource logs collected for Managed HSM, see [Monitoring Key Vault data reference](../general/monitor-key-vault-reference.md).

For a list of the tables used by Azure Monitor Logs and queryable by Log Analytics, see [Monitoring Key Vault data reference](../general/monitor-key-vault-reference.md).

## Analyze monitoring data

There are many tools for analyzing monitoring data.

### Azure Monitor tools

Azure Monitor supports the following basic tools:

- **Metrics explorer**, a tool in the Azure portal that allows you to view and analyze metrics for Azure resources. For more information, see [Analyze metrics with Azure Monitor metrics explorer](/azure/azure-monitor/visualize/metrics-explorer).
- **Log Analytics**, a tool in the Azure portal that allows you to query and analyze log data by using the Kusto query language (KQL). For more information, see [Get started with log queries in Azure Monitor](/azure/azure-monitor/logs/log-analytics-tutorial).
- **The activity log**, which has a user interface in the Azure portal for viewing and basic searches. To do more in-depth analysis, you have to route the data to Azure Monitor logs and run more complex queries in Log Analytics.

Tools that allow more complex visualization include:

- **Dashboards** that let you combine different kinds of data into a single pane in the Azure portal.
- **Workbooks**, customizable reports that you can create in the Azure portal. Workbooks can include text, metrics, and log queries.
- **Grafana**, an open platform tool that excels in operational dashboards. You can use Grafana to create dashboards that include data from multiple sources other than Azure Monitor.
- **Power BI**, a business analytics service that provides interactive visualizations across various data sources. You can configure Power BI to automatically import log data from Azure Monitor to take advantage of these visualizations.

### Azure Monitor export tools

You can get data out of Azure Monitor into other tools by using the following methods:

- **Metrics:** Use the REST API for metrics to extract metric data from the Azure Monitor metrics database. The API supports filter expressions to refine the data retrieved. For more information, see [Azure Monitor REST API reference](https://docs.microsoft.com/rest/api/monitor/).
- **Logs:** Use the REST API or the associated client libraries.
- Another option is the workspace data export.

To get started with the REST API for Azure Monitor, see [Azure monitoring REST API walkthrough](/azure/azure-monitor/platform/rest-api-walkthrough).

### Kusto queries

You can analyze monitoring data in the Azure Monitor Logs / Log Analytics store by using the Kusto query language (KQL).

> **Important**
> When you select **Logs** from the service's menu in the portal, Log Analytics opens with the query scope set to the current service. This scope means that log queries will only include data from that type of resource. If you want to run a query that includes data from other Azure services, select **Logs** from the Azure Monitor menu. See [Log query scope and time range in Azure Monitor Log Analytics](/azure/azure-monitor/logs/log-analytics-tutorial#log-query-scope-and-time-range) for details.

For a list of common queries for any service, see the [Log Analytics queries interface](/azure/azure-monitor/logs/log-analytics-tutorial).

Here are some queries that you can enter into the Log search bar to help you monitor your managed HSM resources. These queries work with the new language.

**Are there any slow requests?**

```kusto
// List of Managed HSM requests that took longer than 1sec. 
// To create an alert for this query, click '+ New alert rule'
let threshold=1000; // let operator defines a constant that can be further used in the query

AzureDiagnostics
| where ResourceProvider =="MICROSOFT.KEYVAULT" 
| where ResourceType == "MANAGEDHSMS"
| where DurationMs > threshold
| summarize count() by OperationName, _ResourceId
```

**Are there any failures?**

```kusto
// Count of failed Managed HSM requests by status code. 
// To create an alert for this query, click '+ New alert rule'

AzureDiagnostics
| where ResourceProvider =="MICROSOFT.KEYVAULT" 
| where ResourceType == "MANAGEDHSMS"
| where httpStatusCode_d >= 300 and not(OperationName == "Authentication" and httpStatusCode_d == 401)
| summarize count() by requestUri_s, ResultSignature, _ResourceId
// ResultSignature contains HTTP status, e.g. "OK" or "Forbidden"
// httpStatusCode_d contains HTTP status code returned
```

**How active has this Managed HSM been?**

```kusto
// Line chart showing trend of Managed HSM requests volume, per operation over time. 
// KeyVault diagnostic currently stores logs in AzureDiagnostics table which stores logs for multiple services. 
// Filter on ResourceProvider for logs specific to a service.

AzureDiagnostics
| where ResourceProvider =="MICROSOFT.KEYVAULT" 
| where ResourceType == "MANAGEDHSMS"
| summarize count() by bin(TimeGenerated, 1h), OperationName // Aggregate by hour
| render timechart
```

**Who is calling this Managed HSM?**

```kusto
// List of callers identified by their IP address with their request count.  
// Managed HSM diagnostic currently stores logs in AzureDiagnostics table which stores logs for multiple services. 
// Filter on ResourceProvider for logs specific to a service.

AzureDiagnostics
| where ResourceProvider =="MICROSOFT.KEYVAULT"
| where ResourceType == "MANAGEDHSMS"
| summarize count() by CallerIPAddress
```

**How fast is this Managed HSM serving requests?**

```kusto
// Line chart showing trend of request duration over time using different aggregations. 

AzureDiagnostics
| where ResourceProvider =="MICROSOFT.KEYVAULT"
| where ResourceType == "MANAGEDHSMS" 
| summarize avg(DurationMs) by requestUri_s, bin(TimeGenerated, 1h) // requestUri_s contains the URI of the request
| render timechart
```

## Alerts

Azure Monitor alerts proactively notify you when specific conditions are found in your monitoring data. Alerts allow you to identify and address issues in your system before your customers notice them. For more information, see [Azure Monitor alerts](/azure/azure-monitor/platform/alerts-overview).

There are many sources of common alerts for Azure resources. For examples of common alerts for Azure resources, see [Sample log alert queries](/azure/azure-monitor/platform/alerts-log). The Azure Monitor Baseline Alerts (AMBA) site provides a semi-automated method of implementing important platform metric alerts, dashboards, and guidelines. The site applies to a continually expanding subset of Azure services, including all services that are part of the Azure Landing Zone (ALZ).

The common alert schema standardizes the consumption of Azure Monitor alert notifications. For more information, see [Common alert schema](/azure/azure-monitor/platform/alerts-common-schema).

### Types of alerts

You can alert on any metric or log data source in the Azure Monitor data platform. There are many different types of alerts depending on the services you're monitoring and the monitoring data you're collecting. Different types of alerts have various benefits and drawbacks. For more information, see [Choose the right monitoring alert type](/azure/azure-monitor/platform/alerts-overview#types-of-alerts).

The following list describes the types of Azure Monitor alerts you can create:

- **Metric alerts** evaluate resource metrics at regular intervals. Metrics can be platform metrics, custom metrics, logs from Azure Monitor converted to metrics, or Application Insights metrics. Metric alerts can also apply multiple conditions and dynamic thresholds.
- **Log alerts** allow users to use a Log Analytics query to evaluate resource logs at a predefined frequency.
- **Activity log alerts** trigger when a new activity log event occurs that matches defined conditions. Resource Health alerts and Service Health alerts are activity log alerts that report on your service and resource health.

Some Azure services also support smart detection alerts, Prometheus alerts, or recommended alert rules.

For some services, you can monitor at scale by applying the same metric alert rule to multiple resources of the same type that exist in the same Azure region. Individual notifications are sent for each monitored resource. For supported Azure services and clouds, see [Monitor multiple resources with one alert rule](/azure/azure-monitor/platform/alerts-metric-multiple-resources).

> **Note**
> If you're creating or running an application that runs on your service, Azure Monitor application insights might offer more types of alerts.

### Managed HSM alert rules

The following list contains some suggested alert rules for managed HSM. These alerts are just examples. You can set alerts for any metric, log entry, or activity log entry listed in the [Azure Key Vault monitoring data reference](../general/monitor-key-vault-reference.md).

- Managed HSM Availability drops below 100% (Static Threshold)
- Managed HSM Latency is greater than 1000 ms (Static Threshold)
- Total Error Codes higher than average (Dynamic Threshold)

For more information, see [Alerting for Azure Managed HSM](configure-alerts.md).

## Advisor recommendations

For some services, if critical conditions or imminent changes occur during resource operations, an alert displays on the service Overview page in the portal. You can find more information and recommended fixes for the alert in **Advisor recommendations** under **Monitoring** in the left menu. During normal operations, no advisor recommendations display.

For more information on Azure Advisor, see [Azure Advisor overview](/azure/advisor/).

## Next Steps

- [Monitor Azure Managed HSM](logging-azure-monitor.md)
- [Configure Managed HSM alerts](configure-alerts.md)
- [Monitoring Key Vault](../general/monitor-key-vault.md)
- [Monitoring Key Vault data reference](../general/monitor-key-vault-reference.md)
- [Create a log query alert for an Azure resource](/azure/azure-monitor/platform/alerts-log)
