---
title: Interoperability with Azure services, Azure clouds, and client operating systems
description: Learn about the Azure cloud environments where Defender for Cloud can be used, the Azure services that Defender for Cloud protects, and the client operating systems that Defender for Cloud supports.
ms.topic: limits-and-quotas
author: dcurwin
ms.author: dacurwin
ms.date: 05/01/2025
---

# Support matrices for Defender for Cloud

This article describes Azure services and client operating systems that are supported by Microsoft Defender for Cloud. For Azure cloud support, review [this article](support-matrix-cloud-environment.md)

## Security benefits for Azure services

Defender for Cloud provides recommendations, security alerts, and vulnerability assessment for these Azure services:

|Service|[Recommendations](security-policy-concept.md) free with [Foundational CSPM](concept-cloud-security-posture-management.md) |[Security alerts](alerts-overview.md) |Vulnerability assessment|
|:----|:----:|:----:|:----:|
|Azure App Service|✔|✔|-|
|Azure Automation account|✔|-|-|
|Azure Batch account|✔|-|-|
|Azure Blob Storage|✔|✔|-|
|Azure Cache for Redis|✔|-|-|
|Azure Cloud Services|✔|-|-|
|Azure AI Search|✔|-|-|
|Azure AI Service|✔|✔|-|
|Azure Container Registry|✔|✔|[Defender for Containers](defender-for-containers-introduction.md)|
|Azure Cosmos DB*|✔|✔|-|
|Azure Data Lake Analytics|✔|-|-|
|Azure Data Lake Storage|✔|✔|-|
|Azure Database for MySQL*|-|✔|-|
|Azure Database for PostgreSQL*|-|✔|-|
|Azure Event Hubs namespace|✔|-|-|
|Azure Files|✔|✔|-|
|Azure Functions app|✔|-|-|
|Azure Key Vault|✔|✔|-|
|Azure Kubernetes Service|✔|✔|-|
|Azure Load Balancer|✔|-|-|
|Azure Logic Apps|✔|-|-|
|Azure SQL Database|✔|✔|[Defender for Azure SQL](defender-for-sql-introduction.md)|
|Azure SQL Managed Instance|✔|✔|[Defender for Azure SQL](defender-for-sql-introduction.md)|
|Azure Service Bus namespace|✔|-|-|
|Azure Service Fabric account|✔|-|-|
|Azure Stream Analytics|✔|-|-|
|Azure Subscription|✔ **|✔|-|
|Azure Virtual Network</br> (incl. subnets, NICs, and network security groups)|✔|-|-|

\* These features are currently supported in preview.

\*\* Microsoft Entra recommendations are available only for subscriptions with [enhanced security features enabled](enable-enhanced-security.md).

## Supported operating systems

Defender for Cloud depends on the [Azure Monitor Agent](/azure/azure-monitor/agents/agents-overview) or the [Log Analytics agent](/azure/azure-monitor/agents/log-analytics-agent). Make sure that your machines are running one of the supported operating systems as described on the following pages:

- Azure Monitor Agent
  - [Azure Monitor Agent for Windows supported operating systems](/azure/azure-monitor/agents/agents-overview#windows)
  - [Azure Monitor Agent for Linux supported operating systems](/azure/azure-monitor/agents/agents-overview#linux)
- Log Analytics agent
  - [Log Analytics agent for Windows supported operating systems](/azure/azure-monitor/agents/agents-overview#windows)
  - [Log Analytics agent for Linux supported operating systems](/azure/azure-monitor/agents/agents-overview#linux)

Also ensure your Log Analytics agent is [properly configured to send data to Defender for Cloud](working-with-log-analytics-agent.md#manual-agent).

To learn more about the specific Defender for Cloud features available on Windows and Linux, review:

- [Defender for Servers support](support-matrix-defender-for-servers.md)
- [Defender for Containers support](support-matrix-defender-for-containers.md)

> [!NOTE]
> Even though Microsoft Defender for Servers is designed to protect servers, some features are available on certain desktop operating systems. One feature that isn't currently supported for Windows desktop systems is [Defender for Cloud's integrated EDR solution: Microsoft Defender for Endpoint](integration-defender-for-endpoint.md).

## Next steps

This article explained how Microsoft Defender for Cloud is supported in the Azure, Azure Government, and Microsoft Azure operated by 21Vianet clouds. Now that you're familiar with the Defender for Cloud capabilities supported in your cloud, learn how to:

- [Manage security recommendations in Defender for Cloud](review-security-recommendations.md)
- [Manage and respond to security alerts in Defender for Cloud](managing-and-responding-alerts.yml)
