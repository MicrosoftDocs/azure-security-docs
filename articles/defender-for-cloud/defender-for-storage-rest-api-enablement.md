---
title: Enable Microsoft Defender for Storage using REST API
description: Learn how to enable the Defender for Storage on your Azure subscription for Microsoft Defender for Cloud using REST API.
ms.topic: install-set-up-deploy
ms.date: 06/30/2025
---

# Enable and configure with REST API

We recommend that you enable Defender for Storage on the subscription level. Doing so ensures all storage accounts currently in the subscription are protected. Storage accounts that are created after enabling Defender for Storage on the subscription level will be protected up to 24 hours after creation.

> [!TIP]
> You can always [configure specific storage accounts](/azure/storage/common/azure-defender-storage-configure?toc=%2Fazure%2Fdefender-for-cloud%2Ftoc.json&tabs=enable-subscription#override-defender-for-storage-subscription-level-settings) with custom configurations that differ from the settings configured at the subscription level (override subscription-level settings).

## [Enable on a subscription](#tab/enable-subscription/)

To enable and configure Microsoft Defender for Storage at the subscription level using REST API, create a PUT request with this endpoint (replace the `subscriptionId` in the endpoint URL with your own Azure subscription ID):

```rest
PUT
https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Security/pricings/StorageAccounts?api-version=2023-01-01

```

And add the following request body:

```json
{
    "properties": {
        "extensions": [
            {
                "name": "OnUploadMalwareScanning",
                "isEnabled": "True",
                "additionalExtensionProperties": {
                    "CapGBPerMonthPerStorageAccount": "10000",
					
                }
            },
            {
                "name": "SensitiveDataDiscovery",
                "isEnabled": "True"
            }
        ],
        "subPlan": "DefenderForStorageV2",
        "pricingTier": "Standard"
    }
}
```

To modify the monthly threshold for malware scanning in your storage accounts, adjust the `CapGBPerMonthPerStorageAccount` parameter to your preferred value. This parameter sets a cap on the maximum data that can be scanned for malware each month, per storage account. If you want to permit unlimited scanning, assign the value -1. The default limit is set at 10,000 GB.

If you want to turn off the on-upload malware scanning or Sensitive data threat detection features, you can change the isEnabled value to **False** under Sensitive data discovery.

If you want to turn off the use of blob index tags, you can change the `BlobScanResultsOptions` value to __None__.  If it isn't set, all scan results are published to blob index tags.

To disable the entire Defender plan, set the `pricingTier` property value to **Free** and remove the `subPlan` and extensions properties.

Learn more about [updating Defender plans with the REST API](/rest/api/defenderforcloud-composite/pricings/update?view=rest-defenderforcloud-composite-latest&tabs=HTTP&preserve-view=true) in HTTP, Java, Go, and JavaScript.

## [Enable on a storage account](#tab/enable-storage-account/)

To enable and configure Microsoft Defender for Storage at the storage account level using REST API, create a PUT request with this endpoint. Replace the `subscriptionId`, `resourceGroupName`, and `accountName` in the endpoint URL with your own Azure subscription ID, resource group, and storage account names accordingly.

```rest
PUT
https://management.azure.com/{resourceId}/providers/Microsoft.Security/defenderForStorageSettings/current?api-version=2025-02-01-preview

```

And add the following request body:

```json
{
   "properties": {
        "isEnabled": true,
        "malwareScanning": {
            "onUpload": {
                "isEnabled": true,
                "capGBPerMonth": 10000,
                "filters": {
                    "excludeBlobsWithPrefix": [
                        "<excluded-container>/",
                        "<container>/<excluded-blob-prefix>"
                        "<excluded-containers-prefix>"
                    ],
                    "excludeBlobsWithSuffix": [
                        ".parquet",
                        ".tmp",
                        "<excluded-blob-name-suffix>.json"
                    ],
                    "excludeBlobsLargerThan": 2147483648
				}
            },
            "scanResultsEventGridTopicResourceId": "/subscriptions/<Subscription>/resourceGroups/<resourceGroup>/providers/Microsoft.EventGrid/topics/<topicName>"
        },
        "sensitiveDataDiscovery": {
            "isEnabled": true
        },
        "overrideSubscriptionLevelSettings": true
    },
    "scope": "[resourceId('Microsoft.Storage/storageAccounts', parameters('StorageAccountName'))]"
}
```

To modify the monthly threshold for malware scanning in your storage accounts, adjust the `capGBPerMonth` parameter to your preferred value. This parameter sets a cap on the maximum data that can be scanned for malware each month, per storage account. If you want to permit unlimited scanning, assign the value -1. The default limit is set at 10,000 GB.

To have more granular control over on-upload malware scanning and exclude specific containers, blob types, or sizes, use the `filters` property.
Learn more about the [on-upload malware scanning filters](on-upload-malware-scanning.md).

If you want to turn off the on-upload malware scanning or sensitive data threat detection features, you can change the `isEnabled` value to **False** under the `malwareScanning` or `sensitiveDataDiscovery` properties sections.

If you want to turn off the use of blob index tags, you can change the `BlobScanResultsOptions` value to __None__.  If it isn't set, all scan results are published to blob index tags.

To disable the entire Defender plan for the storage account, set the `isEnabled` property value to **False** and remove the `malwareScanning` and `sensitiveDataDiscovery` sections from the properties.

Learn more about the [Microsoft.Security/DefenderForStorageSettings API](/rest/api/defenderforcloud-composite/defender-for-storage/create?view=rest-defenderforcloud-composite-latest&tabs=HTTP&preserve-view=true) documentation.

---

> [!TIP]
> Malware scanning can be configured to send scanning results to: <br>  **Event Grid custom topic** - for near-real time automatic response based on every scanning result. Learn more how to [configure malware scanning to send scanning events to an Event Grid custom topic](/azure/storage/common/azure-defender-storage-configure?toc=%2Fazure%2Fdefender-for-cloud%2Ftoc.json&tabs=enable-storage-account#setting-up-event-grid-for-malware-scanning). <br> **Log Analytics workspace** - for storing every scan result in a centralized log repository for compliance and audit. Learn more how to [configure malware scanning to send scanning results to a Log Analytics workspace](/azure/storage/common/azure-defender-storage-configure?toc=%2Fazure%2Fdefender-for-cloud%2Ftoc.json&tabs=enable-storage-account#setting-up-logging-for-malware-scanning).

Learn more on how to [set up response for malware scanning](defender-for-storage-configure-malware-scan.md) results.

## Next steps

- Learn how to [enable and Configure the Defender for Storage plan at scale with an Azure built-in policy](defender-for-storage-policy-enablement.md).
