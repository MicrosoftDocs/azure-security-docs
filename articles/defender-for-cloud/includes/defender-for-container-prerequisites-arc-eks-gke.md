---
ms.service: defender-for-cloud
ms.topic: include
ms.date: 07/27/2022
ms.author: dacurwin
author: dcurwin
---

## Network requirements

Validate that the following endpoints for public cloud deployments are configured for outbound access. Configuring them for outbound access helps ensure that the Defender sensor can connect to Microsoft Defender for Cloud to send security data and events.

| Azure domain  | Azure Government domain  | Azure operated by 21Vianet domain | Port |
| -------------------------- | -------------------------- | -------------------------- |---- |
| *.ods.opinsights.azure.com | *.ods.opinsights.azure.us | *.ods.opinsights.azure.cn  | 443  |
| *.oms.opinsights.azure.com | *.oms.opinsights.azure.us | *.oms.opinsights.azure.cn | 443 |
| login.microsoftonline.com  | login.microsoftonline.us | login.chinacloudapi.cn  | 443  |
| *.cloud.defender.microsoft.com| N/A                          | N/A                                | 443  |

You also need to validate the [Azure Arc-enabled Kubernetes network requirements](/azure/azure-arc/kubernetes/network-requirements).
