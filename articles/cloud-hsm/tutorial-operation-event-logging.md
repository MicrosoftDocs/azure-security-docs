---
title: Tutorial - Operation Event Logging for Azure Cloud HSM
description: Learn how to configure and query operation event logging for Azure Cloud HSM using Azure Log Analytics.
author: keithp
manager: keithp
ms.service: azure-dedicated-hsm
ms.topic: tutorial
ms.date: 03/20/2025
ms.author: keithp

#Customer Intent: As an IT Pro, I want to set up and use operation event logging for Azure Cloud HSM to ensure security and compliance.

---

# Tutorial: Operation event logging

Microsoft Azure Cloud HSM supports operation event logging through Azure Log Analytics, which is vital for security as it enables centralized collection, analysis, and monitoring of logs across your Cloud HSM resources.

> [!IMPORTANT]
> To maintain security and privacy, logging excludes sensitive details such as key IDs, key names, and other identifiable information related to keys, users, or sessions. Logs capture the HSM operation performed, the time of the operation, and relevant HSM metadata. 
>
> Azure Cloud HSM operation event logging cannot determine whether an operation performed by the HSM succeeded or failed. It can only log the fact that the operation was executed. This limitation exists during public preview because the HSM operation occurs within the inner TLS channel, which is not exposed outside that boundary.

## Prerequisites

The following prerequisites are required to support operation event logging with Azure Cloud HSM. Refer to the Azure Cloud HSM onboarding guide for SDK installation and Azure Cloud HSM configuration if you haven't completed your HSM deployment.

### System requirements

* Azure Cloud HSM resource has been deployed, initialized, and configured.
* Creation of Azure Storage Account
* Creation of Azure Log Analytics Workspace and Azure Monitor(s)

## Operation event log setup and configuration

Operation event logging is crucial for HSM’s and overall security because it provides a transparent and immutable record of all access and operations, ensuring accountability and traceability. By capturing details such as user activities, key management actions, and system events, operation logs help detect unauthorized access, investigate security incidents, and comply with regulatory requirements. They also play a vital role in identifying anomalies that could indicate potential breaches or misconfigurations, strengthening an organization's ability to maintain the integrity and confidentiality of its cryptographic operations.

### Create a storage account to store HSM logs

To create a storage account for storing HSM logs, you first need to create a resource group, and then create the storage account within that resource group using the following commands:

```bash
az group create --name <ResourceGroupName> --location <RegionName>
az storage account create --name <StorageAccountName> --resource-group <ResourceGroupName> --location <RegionName> --sku Standard_LRS --kind StorageV2
```
### Create a log analytics workspace

To create a Log Analytics workspace for storing and analyzing HSM logs, use the following command:

```bash
az monitor log-analytics workspace create --resource-group <ResourceGroupName> --workspace-name <WorkspaceName>
```

For more information about creating a Log Analytics workspace, see [create log analytics workspaces for Azure Monitor](/azure/azure-monitor/logs/quick-create-workspace?tabs=azure-cli).

### Enable diagnostic-settings using Azure Monitor CLI or PowerShell

Before running the command to enable diagnostic settings for Azure Cloud HSM operation event logging, you need to set the following variables. Replace the placeholders for SubscriptionId, ResourceGroup, HSMName, StorageAccountName, and WorkspaceName with the appropriate values for your environment.

Azure CLI:

```azurecli
$resourceId = "/subscriptions/<SubscriptionId>/resourceGroups/<ResourceGroupName>/providers/Microsoft.HardwareSecurityModules/cloudHsmClusters/<HSMName>"
 
$storageAccountId = "/subscriptions/<SubscriptionId>/resourceGroups/<ResourceGroupName>/providers/Microsoft.Storage/storageAccounts/<StorageAccountName>"
 
$workspaceId = "/subscriptions/<SubscriptionId>/resourcegroups/<ResourceGroupName>/providers/microsoft.operationalinsights/workspaces/<WorkspaceName>"

az monitor diagnostic-settings create --resource $resourceId -n "my-chsmAuditLogs" --storage-account $storageAccountId --logs "[{category:HsmServiceOperations,enabled:true}]"  --workspace $workspaceId
```

Azure PowerShell:

```azurepowershell
$resourceId = "/subscriptions/<SubscriptionId>/resourceGroups/<ResourceGroupName>/providers/Microsoft.HardwareSecurityModules/cloudHsmClusters/<HSMName>"
 
$storageAccountId = "/subscriptions/<SubscriptionId>/resourceGroups/<ResourceGroupName>/providers/Microsoft.Storage/storageAccounts/<StorageAccountName>"
 
$workspaceId = "/subscriptions/<SubscriptionId>/resourcegroups/<ResourceGroupName>/providers/microsoft.operationalinsights/workspaces/<WorkspaceName>"

New-AzDiagnosticSetting -ResourceId $resourceId -Name "my-chsmAuditLogs" -StorageAccountId $storageAccountId -WorkspaceId $workspaceId -Enabled $true -Category "HsmServiceOperations"
```

### Verify Cloud HSM logging is configured

After the diagnostic setting is created, the logs will start flowing within 1-2 minutes. The logs can be seen on the Azure portal or using CLI and PowerShell.

**Azure portal**: You can query Azure Cloud HSM operation event logs from the Azure portal via log analytics workspace.

:::image type="content" source="./media/operation-event-logs-portal.jpg" lightbox="./media/operation-event-logs-portal.jpg" alt-text="Screenshot of Azure Cloud HSM operation event logs in the Azure portal.":::

**Azure CLI / PowerShell**: You can query Azure Cloud HSM operation event logs using Azure CLI and PowerShell. In this example, you update ResourceGroupName and WorkspaceName.

:::image type="content" source="./media/operation-event-logs-cli.png" lightbox="./media/operation-event-logs-cli.png" alt-text="Screenshot of Azure Cloud HSM operation event logs in the command line interface.":::

```bash
$workspaceId = az monitor log-analytics workspace show --resource-group <ResourceGroupName> --workspace-name <WorkspaceName> --query customerId -o tsv
az monitor log-analytics query -w $workspaceId --analytics-query "CloudHsmServiceOperationAuditLogs | take 10"
```

> [!NOTE]
> If you are getting the error message "\<Subscription\> is not registered to use microsoft.insights", your Azure subscription is not registered to use the `Microsoft.Insights` resource provider. To resolve this issue, you need to register with the `Microsoft.Insights` provider in your subscription.  Ensure that the registration state is registered. If it's still registering, you may need to wait a few moments and check again.
>
> You can register `Microsoft.Insights` by running the following commands:
>
> **Azure CLI**
>
> ```bash
> az provider register --namespace Microsoft.Insights
> az provider show --namespace Microsoft.Insights --query "registrationState" --output table
> ```
>
> **Azure PowerShell**
> 
> ```bash
> Register-AzResourceProvider -ProviderNamespace Microsoft.Insights
> Get-AzResourceProvider -ProviderNamespace Microsoft.Insights | Select-Object ProviderNamespace, > RegistrationState
> ```

## Querying operation event logs

The following commands can be used to retrieve specific HSM operation events recorded in operation logs. To list all operational events, just execute ‘CloudHsmServiceOperationAuditLogs’. For details on other operations that can be queried, refer to the glossary of terms, which provides a comprehensive list of recorded operations.

### Login and session events

```bash
// Find Login and Session Events
CloudHsmServiceOperationAuditLogs
| where OperationName in ("CN_LOGIN", "CN_AUTHORIZE_SESSION")
| project OperationName, MemberId, CallerIpAddress, TimeGenerated
```

### Create and delete user events

```bash
// Find Create and Delete User Events 
CloudHsmServiceOperationAuditLogs
| where OperationName in ("CN_CREATE_USER", "CN_DELETE_USER")
| project OperationName, MemberId, CallerIpAddress, TimeGenerated
```

### Key create events

```bash
// Find Key Creation Events
CloudHsmServiceOperationAuditLogs
| where OperationName in ("CN_GENERATE_KEY", "CN_GENERATE_KEY_PAIR")
| project OperationName, MemberId, CallerIpAddress, TimeGenerated
```

### Key delete events

```bash
// Find Delete Key Events
CloudHsmServiceOperationAuditLogs
| where OperationName == "CN_TOMBSTONE_OBJECT"
| project OperationName, MemberId, CallerIpAddress, TimeGenerated
```

## Glossary of terms

The following terms are used to describe HSM operation events.

### azcloudhsm_util

| Operation Name              | Command Name       | Description                                                   |
|-----------------------------|--------------------|---------------------------------------------------------------|
| CN_LOGIN                    | loginHSM           | Login to the HSM.                                             |
| CN_LOGOUT                   | logoutHSM          | Logout of the HSM.                                            |
| CN_GENERATE_KEY             | genSymKey          | Generates a Symmetric key.                                     |
| CN_GENERATE_KEY_PAIR        | genRSAKeyPair      | Generates an RSA Key Pair.                                     |
|                             | genECCKeyPair      | Generates an ECC Key Pair.                                     |
| CN_SHARE_OBJECT             | shareKey           | Shares/unshares an existing key with other users.              |
| CN_TOMBSTONE_OBJECT         | deleteKey          | Deletes a key.                                                |
| CN_FIND_OBJECTS_FROM_INDEX  | findSingleKey      | Finds a single key.                                           |
| CN_FIND_OBJECTS_USING_COUNT | findKey            | Finds a key.                                                  |
| CN_GET_OBJECT_INFO          | getKeyInfo         | Gets key info about shared users/sessions.                    |
| HASH_SINGLE_CALL            | sign               | Generates a signature. (ME_PKCS_PKCS1v15_CRT_ENCRYPT)         |
| HASH_SINGLE_CALL            | verify             | Verifies a signature. (ME_PKCS_PKCS1v15_DECRYPT)              |
| CN_LIST_TOKENS              | listTokens         | Lists all tokens in the current partition.                    |
| CN_GET_TOKEN                | getToken           | Gets a token.                                                 |
| CN_CREATE_USER              | createUser         | Creates a user.                                               |
| CN_DELETE_USER              | deleteUser         | Deletes a user.                                               |
| CN_LIST_USERS               | listUsers          | Lists users.                                                  |
| CN_CHANGE_PSWD              | changePswd         | Changes a password.                                           |
| CN_MODIFY_OBJECT            | setAttribute       | Sets an attribute of an object.                               |
|                             | getAttribute       | Gets an attribute of an object.                               |
| CN_TOKEN_INFO               | getHSMInfo         | Gets the HSM information.                                     |
| CN_PARTITION_INFO           | getPartitionInfo   | Gets the partition information.                               |
| ---                         | getClusterInfo     | Not recorded.                                                 |
| ---                         | server             | Not recorded.                                                 |

### azcloudhsm_mgmt

| Operation Name              | Command Name       | Description                                                   |
|-----------------------------|--------------------|---------------------------------------------------------------|
| CN_LOGIN                    | loginHSM           | Login to the HSM.                                             |
| CN_LOGOUT                   | logoutHSM          | Logout of the HSM.                                            |
| CN_GENERATE_KEY             | genSymKey          | Generates a Symmetric key.                                     |
| CN_GENERATE_KEY_PAIR        | genRSAKeyPair      | Generates an RSA Key Pair.                                     |
|                             | genECCKeyPair      | Generates an ECC Key Pair.                                     |
| CN_SHARE_OBJECT             | shareKey           | Shares/unshares an existing key with other users.              |
| CN_TOMBSTONE_OBJECT         | deleteKey          | Deletes a key.                                                |
| CN_FIND_OBJECTS_FROM_INDEX  | findSingleKey      | Finds a single key.                                           |
| CN_FIND_OBJECTS_USING_COUNT | findKey            | Finds a key.                                                  |
| CN_GET_OBJECT_INFO          | getKeyInfo         | Gets key info about shared users/sessions.                    |
| HASH_SINGLE_CALL            | sign               | Generates a signature. (ME_PKCS_PKCS1v15_CRT_ENCRYPT)         |
| HASH_SINGLE_CALL            | verify             | Verifies a signature. (ME_PKCS_PKCS1v15_DECRYPT)              |
| CN_LIST_TOKENS              | listTokens         | Lists all tokens in the current partition.                    |
| CN_GET_TOKEN                | getToken           | Gets a token.                                                 |
| CN_CREATE_USER              | createUser         | Creates a user.                                               |
| CN_DELETE_USER              | deleteUser         | Deletes a user.                                               |
| CN_LIST_USERS               | listUsers          | Lists users.                                                  |
| CN_CHANGE_PSWD              | changePswd         | Changes a password.                                           |
| CN_MODIFY_OBJECT            | setAttribute       | Sets an attribute of an object.                               |
|                             | getAttribute       | Gets an attribute of an object.                               |
| CN_TOKEN_INFO               | getHSMInfo         | Gets the HSM information.                                     |
| CN_PARTITION_INFO           | getPartitionInfo   | Gets the partition information.                               |
| ---                         | getClusterInfo     | Not recorded.                                                 |
| ---                         | server             | Not recorded.                                                 |

### Backup and Restore

| Operation Name | Command Name | Description |
|----------------|--------------|-------------|
|                | /backup      |             |
|                | /restore     |             |

## Next steps

- [Azure Cloud HSM overview](overview.md)
- [Tutorial: Deploy Azure Cloud HSM](tutorial-deploy-cloud-hsm.md)
