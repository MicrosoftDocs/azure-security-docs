---
title: Tutorial - Configure Event Hubs for Azure Cloud HSM
description: Learn how to configure Azure Event Hubs as a destination for Azure Cloud HSM operation event logs for real-time streaming and downstream processing.
author: keithp
manager: keithp
ms.service: azure-cloud-hsm
ms.topic: tutorial
ms.date: 04/08/2026
ms.author: keithp
ai-usage: ai-assisted

#Customer Intent: As an IT pro, I want to stream Azure Cloud HSM operation logs to Event Hubs for real-time processing and integration with downstream systems.

---

# Tutorial: Configure Event Hubs for Azure Cloud HSM

If you configured operation event logging for Azure Cloud HSM, you already have a working diagnostic setting on your Cloud HSM cluster that routes `HsmServiceOperations` logs to Storage and Log Analytics. To add Event Hubs as a destination, add a third destination to that same diagnostic setting or create a new one that targets Event Hubs.

Azure Monitor diagnostic settings support multiple destinations simultaneously. Because your Log Analytics pipeline already proves that log emission from Cloud HSM is working, Event Hubs becomes another destination receiving the same `HsmServiceOperations` category.

In this tutorial, you:

> [!div class="checklist"]
>
> - Create an Event Hub namespace and event hub for Cloud HSM logs.
> - Configure authorization rules with least-privilege permissions.
> - Update diagnostic settings to stream logs to Event Hubs.
> - Verify that Event Hubs receives Cloud HSM operation events.

## Prerequisites

- A deployed and activated Azure Cloud HSM resource. For more information, see the [Azure Cloud HSM onboarding guide](onboarding-guide.md).
- Diagnostic settings emitting operation event logs to Storage and Log Analytics. For more information, see [Configure and query operation event logging for Azure Cloud HSM](tutorial-operation-event-logging.md).
- `Contributor` or `Monitoring Contributor` role on the Cloud HSM resource group.

## Verify the logs resource group exists

Event Hubs should deploy into the same resource group that contains your storage account and Log Analytics workspace for operational event logging. If you didn't set up operational event logging, first follow the guidance in [Configure and query operation event logging for Azure Cloud HSM](tutorial-operation-event-logging.md).

Verify that your targeted resource group exists:

# [Azure CLI](#tab/azure-cli)

```azurecli
az group show --name "<resource-group>" --query "{name:name, location:location}" --output table
```

# [Azure PowerShell](#tab/azure-powershell)

```azurepowershell
Get-AzResourceGroup -Name "<resource-group>" | Select-Object ResourceGroupName, Location
```

---

## Create an Event Hubs namespace

The namespace is the container that holds one or more event hubs. Use the **Standard** tier, which is required for diagnostic settings integration.

# [Azure CLI](#tab/azure-cli)

```azurecli
az eventhubs namespace create \
  --name "<eventhub-namespace>" \
  --resource-group "<resource-group>" \
  --location "<location>" \
  --sku Standard \
  --capacity 1 \
  --enable-auto-inflate false
```

# [Azure PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzEventHubNamespace `
  -Name "<eventhub-namespace>" `
  -ResourceGroupName "<resource-group>" `
  -Location "<location>" `
  -SkuName Standard `
  -SkuCapacity 1 `
  -EnableAutoInflate:$false
```

---

Key options:

- **Standard SKU**: The Basic tier doesn't support diagnostic settings as a destination.
- **Capacity 1**: One throughput unit (1-MB/s ingress, 2-MB/s egress) is sufficient for HSM audit logs.
- **Autoinflate disabled**: HSM log volume is low, so autoinflate isn't necessary.

## Create an event hub inside the namespace

To receive the Cloud HSM logs, create an event hub inside the namespace.

# [Azure CLI](#tab/azure-cli)

```azurecli
az eventhubs eventhub create \
  --name "cloudhsm-logs" \
  --namespace-name "<eventhub-namespace>" \
  --resource-group "<resource-group>" \
  --partition-count 2 \
  --retention-time-in-hours 168 \
  --cleanup-policy Delete
```

# [Azure PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzEventHub `
  -Name "cloudhsm-logs" `
  -NamespaceName "<eventhub-namespace>" `
  -ResourceGroupName "<resource-group>" `
  -PartitionCount 2 `
  -RetentionTimeInHour 168 `
  -CleanupPolicy Delete
```

---

Key options:

- **Partition count 2**: Two partitions are sufficient for HSM audit log throughput.
- **Retention time 168 hours**: Keep messages for seven days (the maximum for Standard tier).
- **Cleanup policy Delete**: Delete messages after the retention period expires.

## Create a consumer group

Create a dedicated consumer group for downstream processing. Reserve the default `$Default` group for other uses.

# [Azure CLI](#tab/azure-cli)

```azurecli
az eventhubs eventhub consumer-group create \
  --name "azure-cloud-hsm" \
  --namespace-name "<eventhub-namespace>" \
  --eventhub-name "cloudhsm-logs" \
  --resource-group "<resource-group>"
```

# [Azure PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzEventHubConsumerGroup `
  -Name "azure-cloud-hsm" `
  -NamespaceName "<eventhub-namespace>" `
  -EventHubName "cloudhsm-logs" `
  -ResourceGroupName "<resource-group>"
```

---

## Create an authorization rule

Diagnostic settings need **Send** permission to push logs into the event hub. Create a shared access policy with only the required permission.

# [Azure CLI](#tab/azure-cli)

```azurecli
az eventhubs namespace authorization-rule create \
  --name "DiagnosticSettingsSendRule" \
  --namespace-name "<eventhub-namespace>" \
  --resource-group "<resource-group>" \
  --rights Send
```

# [Azure PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzEventHubAuthorizationRule `
  -Name "DiagnosticSettingsSendRule" `
  -NamespaceName "<eventhub-namespace>" `
  -ResourceGroupName "<resource-group>" `
  -Rights @("Send")
```

---

> [!NOTE]
> This rule grants only `Send` permission, not `Listen`, or `Manage`. Follow the principle of least privilege. Your downstream consumers (such as Azure Functions or Stream Analytics) should use a separate rule with `Listen` permission.

## Get the authorization rule resource ID

Retrieve the authorization rule resource ID for use in the diagnostic setting.

# [Azure CLI](#tab/azure-cli)

```azurecli
authRuleId=$(az eventhubs namespace authorization-rule show \
  --name "DiagnosticSettingsSendRule" \
  --namespace-name "<eventhub-namespace>" \
  --resource-group "<resource-group>" \
  --query id --output tsv)

echo "Auth Rule ID: $authRuleId"
```

# [Azure PowerShell](#tab/azure-powershell)

```azurepowershell
$authRule = Get-AzEventHubAuthorizationRule `
  -Name "DiagnosticSettingsSendRule" `
  -NamespaceName "<eventhub-namespace>" `
  -ResourceGroupName "<resource-group>"

$authRuleId = $authRule.Id
Write-Host "Auth Rule ID: $authRuleId"
```

---

Save this value for use in the next step.

## Update the diagnostic setting to add Event Hubs

You have two options for adding Event Hubs as a destination:

### Option A: Update the existing diagnostic setting (recommended)

This approach updates your existing diagnostic setting to add Event Hubs while keeping Storage and Log Analytics.

# [Azure CLI](#tab/azure-cli)

```azurecli
# Set your resource group variables
hsmResourceGroup="<resource-group>"
logsResourceGroup="<resource-group>"

# Find the HSM cluster name (auto-generated during deployment)
hsmClusterName=$(az resource list \
  --resource-group $hsmResourceGroup \
  --resource-type Microsoft.HardwareSecurityModules/cloudHsmClusters \
  --query "[0].name" --output tsv)
echo "HSM Cluster: $hsmClusterName"

# Get the HSM cluster resource ID
hsmResourceId=$(az resource show \
  --resource-group $hsmResourceGroup \
  --resource-type Microsoft.HardwareSecurityModules/cloudHsmClusters \
  --name $hsmClusterName \
  --query id --output tsv)

# Get your existing storage account ID
storageAccountId=$(az storage account list \
  --resource-group $logsResourceGroup \
  --query "[0].id" --output tsv)

# Get your existing Log Analytics workspace ID
workspaceId=$(az monitor log-analytics workspace list \
  --resource-group $logsResourceGroup \
  --query "[0].id" --output tsv)

# Get the Event Hub auth rule ID
authRuleId=$(az eventhubs namespace authorization-rule show \
  --name "DiagnosticSettingsSendRule" \
  --namespace-name "<eventhub-namespace>" \
  --resource-group $logsResourceGroup \
  --query id --output tsv)

# Update the diagnostic setting with all three destinations
az monitor diagnostic-settings create \
  --name "<diagnostic-setting-name>" \
  --resource $hsmResourceId \
  --storage-account $storageAccountId \
  --workspace $workspaceId \
  --event-hub "cloudhsm-logs" \
  --event-hub-rule $authRuleId \
  --logs '[{"category":"HsmServiceOperations","enabled":true}]'
```

# [Azure PowerShell](#tab/azure-powershell)

```azurepowershell
# Set your resource group variables
$hsmResourceGroup = "<resource-group>"
$logsResourceGroup = "<resource-group>"

# Find the HSM cluster
$hsmCluster = Get-AzResource `
  -ResourceGroupName $hsmResourceGroup `
  -ResourceType Microsoft.HardwareSecurityModules/cloudHsmClusters
$hsmClusterName = $hsmCluster.Name
Write-Host "HSM Cluster: $hsmClusterName"

# Get your existing storage account
$storageAccount = Get-AzStorageAccount -ResourceGroupName $logsResourceGroup | Select-Object -First 1

# Get your existing Log Analytics workspace
$workspace = Get-AzOperationalInsightsWorkspace -ResourceGroupName $logsResourceGroup | Select-Object -First 1

# Get the Event Hub auth rule ID
$authRule = Get-AzEventHubAuthorizationRule `
  -Name "DiagnosticSettingsSendRule" `
  -NamespaceName "<eventhub-namespace>" `
  -ResourceGroupName $logsResourceGroup

# Create the diagnostic setting with all three destinations
$log = New-AzDiagnosticSettingLogSettingsObject -Category HsmServiceOperations -Enabled $true

New-AzDiagnosticSetting `
  -Name "<diagnostic-setting-name>" `
  -ResourceId $hsmCluster.ResourceId `
  -StorageAccountId $storageAccount.Id `
  -WorkspaceId $workspace.ResourceId `
  -EventHubName "cloudhsm-logs" `
  -EventHubAuthorizationRuleId $authRule.Id `
  -Log $log
```

---

> [!IMPORTANT]
> Both CLI and PowerShell commands replace the entire diagnostic setting if the name matches an existing one. You must include the storage account and workspace again, or those destinations are removed.

### Option B: Create a separate diagnostic setting for Event Hubs only

If you prefer to keep your existing setting unchanged and add a second one,

# [Azure CLI](#tab/azure-cli)

```azurecli
# Find the HSM cluster name
hsmClusterName=$(az resource list \
  --resource-group "<resource-group>" \
  --resource-type Microsoft.HardwareSecurityModules/cloudHsmClusters \
  --query "[0].name" --output tsv)

hsmResourceId=$(az resource show \
  --resource-group "<resource-group>" \
  --resource-type Microsoft.HardwareSecurityModules/cloudHsmClusters \
  --name $hsmClusterName \
  --query id --output tsv)

authRuleId=$(az eventhubs namespace authorization-rule show \
  --name "DiagnosticSettingsSendRule" \
  --namespace-name "<eventhub-namespace>" \
  --resource-group "<resource-group>" \
  --query id --output tsv)

az monitor diagnostic-settings create \
  --name "chsm-eventhub-diagnostic-setting" \
  --resource $hsmResourceId \
  --event-hub "cloudhsm-logs" \
  --event-hub-rule $authRuleId \
  --logs '[{"category":"HsmServiceOperations","enabled":true}]'
```

# [Azure PowerShell](#tab/azure-powershell)

```azurepowershell
# Find the HSM cluster
$hsmCluster = Get-AzResource `
  -ResourceGroupName "<resource-group>" `
  -ResourceType Microsoft.HardwareSecurityModules/cloudHsmClusters

# Get the Event Hub auth rule ID
$authRule = Get-AzEventHubAuthorizationRule `
  -Name "DiagnosticSettingsSendRule" `
  -NamespaceName "<eventhub-namespace>" `
  -ResourceGroupName "<resource-group>"

# Create the diagnostic setting for Event Hub only
$log = New-AzDiagnosticSettingLogSettingsObject -Category HsmServiceOperations -Enabled $true

New-AzDiagnosticSetting `
  -Name "chsm-eventhub-diagnostic-setting" `
  -ResourceId $hsmCluster.ResourceId `
  -EventHubName "cloudhsm-logs" `
  -EventHubAuthorizationRuleId $authRule.Id `
  -Log $log
```

---

> [!NOTE]
> Azure supports up to five diagnostic settings per resource. A second setting is valid and keeps concerns separated.

## Verify Event Hubs is receiving messages

After you configure the diagnostic setting, verify that Event Hubs is receiving Cloud HSM logs.

### Check the diagnostic setting in the portal

1. In the Azure portal, go to your Cloud HSM cluster.
1. Under **Monitoring**, select **Diagnostic settings**.
1. Confirm that Event Hubs is listed as a destination.

### Check Event Hubs metrics

Run the following command to check incoming messages over the last hour:

# [Azure CLI](#tab/azure-cli)

```azurecli
# Get your subscription ID
subId=$(az account show --query id --output tsv)

# Check incoming messages (last 1 hour)
az monitor metrics list \
  --resource "/subscriptions/$subId/resourceGroups/<resource-group>/providers/Microsoft.EventHub/namespaces/<eventhub-namespace>" \
  --metric "SuccessfulRequests" \
  --interval PT1H \
  --output table
```

# [Azure PowerShell](#tab/azure-powershell)

```azurepowershell
# Get the Event Hub namespace
$namespace = Get-AzEventHubNamespace `
  -ResourceGroupName "<resource-group>" `
  -NamespaceName "<eventhub-namespace>"

# Check incoming messages (last 1 hour)
Get-AzMetric `
  -ResourceId $namespace.Id `
  -MetricName "SuccessfulRequests" `
  -TimeGrain 01:00:00 `
  -StartTime (Get-Date).AddHours(-1)
```

---

### View messages (optional)

If you want to read a few messages to confirm content, create a **Listen** rule:

# [Azure CLI](#tab/azure-cli)

```azurecli
# Create a Listen rule for your consumer
az eventhubs namespace authorization-rule create \
  --name "ConsumerListenRule" \
  --namespace-name "<eventhub-namespace>" \
  --resource-group "<resource-group>" \
  --rights Listen

# Get the connection string
az eventhubs namespace authorization-rule keys list \
  --name "ConsumerListenRule" \
  --namespace-name "<eventhub-namespace>" \
  --resource-group "<resource-group>" \
  --query primaryConnectionString --output tsv
```

# [Azure PowerShell](#tab/azure-powershell)

```azurepowershell
# Create a Listen rule for your consumer
New-AzEventHubAuthorizationRule `
  -Name "ConsumerListenRule" `
  -NamespaceName "<eventhub-namespace>" `
  -ResourceGroupName "<resource-group>" `
  -Rights @("Listen")

# Get the connection string
$keys = Get-AzEventHubKey `
  -Name "ConsumerListenRule" `
  -NamespaceName "<eventhub-namespace>" `
  -ResourceGroupName "<resource-group>"

$keys.PrimaryConnectionString
```

---

You can use this connection string with Azure Event Hubs Explorer, the Visual Studio Code Event Hubs extension, or a Python script to view messages.

## Related content

- [Configure and query operation event logging for Azure Cloud HSM](tutorial-operation-event-logging.md)
- [Azure Event Hubs documentation](/azure/event-hubs/event-hubs-about)
- [Diagnostic settings in Azure Monitor](/azure/azure-monitor/essentials/diagnostic-settings)
- [Azure Monitor diagnostic log schema](/azure/azure-monitor/essentials/resource-logs-schema)
