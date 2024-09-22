---
title: include file
description: include file
author: memildin
ms.service: azure-data-factory
ms.topic: include
ms.date: 08/18/2024
ms.author: memildin
ms.custom: include file
---
### The data model of the schema

|Field|Description|
|----|----|
|**AlertName**|Alert display name|
|**AlertType**|unique alert identifier|
|**ConfidenceLevel**|(Optional) The confidence level of this alert (High/Low)|
|**ConfidenceScore**|(Optional) Numeric confidence indicator of the security alert|
|**Description**|Description text for the alert|
|**DisplayName**|The alert's display name|
|**EndTime**|The effect end time of the alert (the time of the last event contributing to the alert)|
|**Entities**|A list of entities related to the alert. This list can hold a mixture of entities of diverse types|
|**ExtendedLinks**|(Optional) A bag for all links related to the alert. This bag can hold a mixture of links for diverse types|
|**ExtendedProperties**|A bag of extra fields, which are relevant to the alert|
|**IsIncident**|Determines if the alert is an incident or a regular alert. An incident is a security alert that aggregates multiple alerts into one security incident|
|**ProcessingEndTime**|UTC timestamp in which the alert was created|
|**ProductComponentName**|(Optional) The name of a component inside the product, which generated the alert.|
|**ProductName**|constant ('Azure Security Center')|
|**ProviderName**|unused|
|**RemediationSteps**|Manual action items to take to remediate the security threat|
|**ResourceId**|Full identifier of the affected resource|
|**Severity**|The alert severity (High/Medium/Low/Informational)|
|**SourceComputerId**|a unique GUID for the affected server (if the alert is generated on the server)|
|**SourceSystem**|unused|
|**StartTime**|The effect start time of the alert (the time of the first event contributing to the alert)|
|**SystemAlertId**|Unique identifier of this security alert instance|
|**TenantId**|the identifier of the parent Microsoft Entra ID tenant of the subscription under which the scanned resource resides|
|**TimeGenerated**|UTC timestamp on which the assessment took place (Security Center's scan time) (identical to DiscoveredTimeUTC)|
|**Type**|constant ('SecurityAlert')|
|**VendorName**|The name of the vendor that provided the alert (for example, 'Microsoft')|
|**VendorOriginalId**|unused|
|**WorkspaceResourceGroup**|in case the alert is generated on a Virtual Machine (VM), Server, Virtual Machine Scale Set, or App Service instance that reports to a workspace, contains that workspace resource group name|
|**WorkspaceSubscriptionId**|in case the alert is generated on a VM, Server, Virtual Machine Scale Set, or App Service instance that reports to a workspace, contains that workspace subscriptionId|
|||
