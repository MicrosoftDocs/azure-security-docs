---
title: Tutorial - Configure and Query Operation Event Logging for Azure Cloud HSM
description: Learn how to configure and query operation event logging for Azure Cloud HSM by using a Log Analytics workspace.
author: keithp
manager: keithp
ms.service: azure-cloud-hsm
ms.topic: tutorial
ms.date: 03/20/2025
ms.author: keithp

#Customer Intent: As an IT pro, I want to set up and use operation event logging for Azure Cloud HSM to help ensure security and compliance.

---

# Tutorial: Configure and query operation event logging for Azure Cloud HSM

Azure Cloud HSM Preview supports operation event logging through Log Analytics workspaces. This capability enables the centralized collection, analysis, and monitoring of logs across your Cloud HSM resources.

Operation event logging is crucial for the overall security of a hardware security module (HSM). It provides a transparent and immutable record of all access and operations, to help ensure accountability and traceability.

By capturing details such as user activities, key management actions, and system events, operation logs help you detect unauthorized access, investigate security incidents, and comply with regulatory requirements. They also play a vital role in identifying anomalies that could indicate potential breaches or misconfigurations. In this way, they strengthen an organization's ability to maintain the integrity and confidentiality of its cryptographic operations.

In this tutorial, you:

> [!div class="checklist"]
>
> - Set up and configure operation event logs, including the creation of a storage account and a Log Analytics workspace.
> - Query operation event logs to retrieve specific HSM operation events.
> - Get a comprehensive list of HSM operation events.

> [!IMPORTANT]
> To maintain security and privacy, logging excludes sensitive details such as key IDs, key names, and other identifiable information related to keys, users, or sessions. Logs capture the HSM operation performed, the time of the operation, and relevant HSM metadata.
>
> Azure Cloud HSM operation event logging can't determine whether an HSM operation succeeded or failed. It can only log the fact that the operation was executed. This limitation exists during the preview of the service because the HSM operation occurs within the inner TLS channel, which is not exposed outside that boundary.

## Prerequisites

- An Azure account with an active subscription. You can [create an account for free](https://azure.microsoft.com/free).
- An Azure Cloud HSM resource that you deployed, initialized, and configured. For information, refer to the [Azure Cloud HSM onboarding guide](onboarding-guide.md).

## Set up and configure operation event logs

Use the commands in the following sections to set up the resources that you want to monitor.

### Create a storage account to store HSM logs

To create a storage account for storing HSM logs, you first need to create a resource group. You also need to create the storage account within that resource group. Use the following commands to create all these items:

```bash
az group create --name <ResourceGroupName> --location <RegionName>
az storage account create --name <StorageAccountName> --resource-group <ResourceGroupName> --location <RegionName> --sku Standard_LRS --kind StorageV2
```

### Create a Log Analytics workspace

To create a Log Analytics workspace for storing and analyzing HSM logs, use the following command:

```bash
az monitor log-analytics workspace create --resource-group <ResourceGroupName> --workspace-name <WorkspaceName>
```

For more information about creating a Log Analytics workspace for Azure Monitor, see [Create a Log Analytics workspace](/azure/azure-monitor/logs/quick-create-workspace?tabs=azure-cli).

### Enable diagnostic settings by using the Azure CLI or Azure PowerShell

To set variables and run the command to enable diagnostic settings for Azure Cloud HSM operation event logging, use the following code. Replace the placeholders for `SubscriptionId`, `ResourceGroupName`, `HSMName`, `StorageAccountName`, and `WorkspaceName` with the appropriate values for your environment.

#### Azure CLI

```azurecli
$resourceId = "/subscriptions/<SubscriptionId>/resourceGroups/<ResourceGroupName>/providers/Microsoft.HardwareSecurityModules/cloudHsmClusters/<HSMName>"
 
$storageAccountId = "/subscriptions/<SubscriptionId>/resourceGroups/<ResourceGroupName>/providers/Microsoft.Storage/storageAccounts/<StorageAccountName>"
 
$workspaceId = "/subscriptions/<SubscriptionId>/resourcegroups/<ResourceGroupName>/providers/microsoft.operationalinsights/workspaces/<WorkspaceName>"

az monitor diagnostic-settings create --resource $resourceId -n "my-chsmAuditLogs" --storage-account $storageAccountId --logs "[{category:HsmServiceOperations,enabled:true}]"  --workspace $workspaceId
```

#### Azure PowerShell

```azurepowershell
$resourceId = "/subscriptions/<SubscriptionId>/resourceGroups/<ResourceGroupName>/providers/Microsoft.HardwareSecurityModules/cloudHsmClusters/<HSMName>"
 
$storageAccountId = "/subscriptions/<SubscriptionId>/resourceGroups/<ResourceGroupName>/providers/Microsoft.Storage/storageAccounts/<StorageAccountName>"
 
$workspaceId = "/subscriptions/<SubscriptionId>/resourcegroups/<ResourceGroupName>/providers/microsoft.operationalinsights/workspaces/<WorkspaceName>"

New-AzDiagnosticSetting -ResourceId $resourceId -Name "my-chsmAuditLogs" -StorageAccountId $storageAccountId -WorkspaceId $workspaceId -Enabled $true -Category "HsmServiceOperations"
```

### Verify the configuration of Cloud HSM logging

After you create the diagnostic setting, the logs start flowing in one to two minutes.

You can query Cloud HSM operation event logs from the Azure portal via the Log Analytics workspace.

:::image type="content" source="./media/operation-event-logs-portal.png" lightbox="./media/operation-event-logs-portal.png" alt-text="Screenshot of Azure Cloud HSM operation event logs in the Azure portal.":::

You can also query Cloud HSM operation event logs by using the Azure CLI and Azure PowerShell. In this example, you update `ResourceGroupName` and `WorkspaceName`:

```bash
$workspaceId = az monitor log-analytics workspace show --resource-group <ResourceGroupName> --workspace-name <WorkspaceName> --query customerId -o tsv
az monitor log-analytics query -w $workspaceId --analytics-query "CloudHsmServiceOperationAuditLogs | take 10"
```

:::image type="content" source="./media/operation-event-logs-cli.png" lightbox="./media/operation-event-logs-cli.png" alt-text="Screenshot of Azure Cloud HSM operation event logs in the command-line interface.":::

#### Registration error

If you get the error message "\<Subscription\> is not registered to use microsoft.insights," your Azure subscription is not registered to use the `Microsoft.Insights` resource provider. To resolve this problem, you need to register with the `Microsoft.Insights` provider in your subscription.

To register `Microsoft.Insights` by using the Azure CLI, run the following command:

```bash
az provider register --namespace Microsoft.Insights
az provider show --namespace Microsoft.Insights --query "registrationState" --output table
```

To register `Microsoft.Insights` by using Azure PowerShell, run the following command:

```bash
Register-AzResourceProvider -ProviderNamespace Microsoft.Insights
Get-AzResourceProvider -ProviderNamespace Microsoft.Insights | Select-Object ProviderNamespace, > RegistrationState
```

After you run the command, ensure that the resource provider is registered. If it's still registering, you might need to wait a few moments and check again.

## Query operation event logs

You can use the following commands to retrieve specific HSM operation events recorded in operation logs. To list all operation events, just run `CloudHsmServiceOperationAuditLogs`.

For details on other operations that can be queried, refer to the [comprehensive list of recorded operations](#glossary-of-operations) later in this article.

### Login and session events

```bash
// Find login and session events
CloudHsmServiceOperationAuditLogs
| where OperationName in ("CN_LOGIN", "CN_AUTHORIZE_SESSION")
| project OperationName, MemberId, CallerIpAddress, TimeGenerated
```

### Events for creating and deleting users

```bash
// Find user creation and deletion events 
CloudHsmServiceOperationAuditLogs
| where OperationName in ("CN_CREATE_USER", "CN_DELETE_USER")
| project OperationName, MemberId, CallerIpAddress, TimeGenerated
```

### Events for key creation

```bash
// Find key creation events
CloudHsmServiceOperationAuditLogs
| where OperationName in ("CN_GENERATE_KEY", "CN_GENERATE_KEY_PAIR")
| project OperationName, MemberId, CallerIpAddress, TimeGenerated
```

### Events for key deletion

```bash
// Find key deletion events
CloudHsmServiceOperationAuditLogs
| where OperationName == "CN_TOMBSTONE_OBJECT"
| project OperationName, MemberId, CallerIpAddress, TimeGenerated
```

## Glossary of operations

The following names are related to HSM operation events.

### azcloudhsm_util

| Operation name              | Command name       | Description                                                   |
|-----------------------------|--------------------|---------------------------------------------------------------|
| `CN_LOGIN`                    | `loginHSM`           | Logs in to the HSM.                                             |
| `CN_LOGOUT`                   | `logoutHSM`          | Logs out of the HSM.                                            |
| `CN_GENERATE_KEY`             | `genSymKey`          | Generates a symmetric key.                                     |
| `CN_GENERATE_KEY_PAIR`        | `genRSAKeyPair`      | Generates an RSA key pair.                                     |
| `CN_GENERATE_KEY_PAIR`        | `genECCKeyPair`      | Generates an ECC key pair.                                     |
| `CN_SHARE_OBJECT`             | `shareKey`           | Shares/unshares an existing key with other users.              |
| `CN_TOMBSTONE_OBJECT`         | `deleteKey`          | Deletes a key.                                                |
| `CN_FIND_OBJECTS_FROM_INDEX`  | `findSingleKey`      | Finds a single key.                                           |
| `CN_FIND_OBJECTS_USING_COUNT` | `findKey`            | Finds a key.                                                  |
| `CN_GET_OBJECT_INFO`          | `getKeyInfo`         | Gets key info about shared users/sessions.                    |
| `HASH_SINGLE_CALL`            | `sign`               | Generates a signature (`ME_PKCS_PKCS1v15_CRT_ENCRYPT`).         |
| `HASH_SINGLE_CALL`            | `verify`             | Verifies a signature (`ME_PKCS_PKCS1v15_DECRYPT`).              |
| `CN_LIST_TOKENS`              | `listTokens`         | Lists all tokens in the current partition.                    |
| `CN_GET_TOKEN`                | `getToken`           | Gets a token.                                                 |
| `CN_CREATE_USER`              | `createUser`         | Creates a user.                                               |
| `CN_DELETE_USER`              | `deleteUser`         | Deletes a user.                                               |
| `CN_LIST_USERS`               | `listUsers`          | Lists users.                                                  |
| `CN_CHANGE_PSWD`              | `changePswd`         | Changes a password.                                           |
| `CN_MODIFY_OBJECT`            | `setAttribute`       | Sets an attribute of an object.                               |
| `CN_GET_ATTRIBUTE_VALUE`<br>`CN_GET_ALL_ATTRIBUTE_VALUE`<br>`CN_GET_ATTRIBUTE_SIZE`<br>`CN_GET_ALL_ATTRIBUTE_SIZE` | `getAttribute`       | Gets an attribute of an object.                               |
| `CN_TOKEN_INFO`               | `getHSMInfo`         | Gets the HSM information.                                     |
| `CN_PARTITION_INFO`           | `getPartitionInfo`   | Gets the partition information.                               |
| ---                         | `getClusterInfo`     | Not recorded.                                                 |
| ---                         | `server`             | Not recorded.                                                 |

### azcloudhsm_mgmt

| Operation name              | Command name       | Description                                                   |
|-----------------------------|--------------------|---------------------------------------------------------------|
| `CN_LOGIN`                    | `loginHSM`           | Logs in to the HSM.                                             |
| `CN_LOGOUT`                   | `logoutHSM`          | Logs out of the HSM.                                            |
| `CN_GENERATE_KEY`             | `genSymKey`          | Generates a symmetric key.                                     |
| `CN_GENERATE_KEY_PAIR`        | `genRSAKeyPair`      | Generates an RSA key pair.                                     |
| `CN_GENERATE_KEY_PAIR`        | `genECCKeyPair`      | Generates an ECC key pair.                                     |
| `CN_SHARE_OBJECT`             | `shareKey`           | Shares/unshares an existing key with other users.              |
| `CN_TOMBSTONE_OBJECT`         | `deleteKey`          | Deletes a key.                                                |
| `CN_FIND_OBJECTS_FROM_INDEX`  | `findSingleKey`      | Finds a single key.                                           |
| `CN_FIND_OBJECTS_USING_COUNT` | `findKey`            | Finds a key.                                                  |
| `CN_GET_OBJECT_INFO`          | `getKeyInfo`         | Gets key info about shared users/sessions.                    |
| `HASH_SINGLE_CALL`            | `sign`               | Generates a signature (`ME_PKCS_PKCS1v15_CRT_ENCRYPT`).         |
| `HASH_SINGLE_CALL`            | `verify`             | Verifies a signature (`ME_PKCS_PKCS1v15_DECRYPT`).              |
| `CN_LIST_TOKENS`              | `listTokens`         | Lists all tokens in the current partition.                    |
| `CN_GET_TOKEN`                | `getToken`           | Gets a token.                                                 |
| `CN_CREATE_USER`              | `createUser`         | Creates a user.                                               |
| `CN_DELETE_USER`              | `deleteUser`         | Deletes a user.                                               |
| `CN_LIST_USERS`               | `listUsers`          | Lists users.                                                  |
| `CN_CHANGE_PSWD`              | `changePswd`         | Changes a password.                                           |
| `CN_MODIFY_OBJECT`            | `setAttribute`       | Sets an attribute of an object.                               |
| `CN_GET_ATTRIBUTE_VALUE`<br>`CN_GET_ALL_ATTRIBUTE_VALUE`<br>`CN_GET_ATTRIBUTE_SIZE`<br>`CN_GET_ALL_ATTRIBUTE_SIZE` | `getAttribute`       | Gets an attribute of an object.                               |
| `CN_TOKEN_INFO`               | `getHSMInfo`         | Gets the HSM information.                                     |
| `CN_PARTITION_INFO`           | `getPartitionInfo`   | Gets the partition information.                               |
| ---                         | `getClusterInfo`     | Not recorded.                                                 |
| ---                         | `server`             | Not recorded.                                                 |

### Backup and restore

| Operation name | Command name | Description |
|----------------|--------------|-------------|
|                | `/backup`      |             |
|                | `/restore`     |             |

## Related content

- [Azure Cloud HSM overview](overview.md)
- [Tutorial: Deploy Azure Cloud HSM](tutorial-deploy-cloud-hsm.md)
