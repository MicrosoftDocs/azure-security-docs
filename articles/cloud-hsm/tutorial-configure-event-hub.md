---
title: Tutorial - Configure Event Hub for Azure Cloud HSM
description: Learn how to configure Azure Event Hubs as a destination for Azure Cloud HSM operation event logs for real-time streaming and downstream processing.
author: keithp
manager: keithp
ms.service: azure-cloud-hsm
ms.topic: tutorial
ms.date: 04/08/2026
ms.author: keithp

#Customer Intent: As an IT pro, I want to stream Azure Cloud HSM operation logs to Event Hub for real-time processing and integration with downstream systems.

---

# Tutorial: Configure Event Hub for Azure Cloud HSM

If you configured operation event logging for Azure Cloud HSM, you already have a working diagnostic setting on your Cloud HSM cluster that routes `HsmServiceOperations` logs to Storage and Log Analytics. Adding Event Hub is simply adding a third destination to that same diagnostic setting or creating a new one that targets Event Hub.

Azure Monitor diagnostic settings support multiple destinations simultaneously. Since your Log Analytics pipeline already proves that log emission from Cloud HSM is working, Event Hub becomes another sink receiving the same `HsmServiceOperations` category.

In this tutorial, you:

> [!div class="checklist"]
>
> - Create an Event Hub namespace and event hub for Cloud HSM logs.
> - Configure authorization rules with least-privilege permissions.
> - Update diagnostic settings to stream logs to Event Hub.
> - Verify that Event Hub receives Cloud HSM operation events.

## Prerequisites

- An Azure Cloud HSM resource that's deployed and activated. For more information, see the [Azure Cloud HSM onboarding guide](onboarding-guide.md).
- Diagnostic settings emitting operation event logs to Storage and Log Analytics. For more information, see [Configure and query operation event logging for Azure Cloud HSM](tutorial-operation-event-logging.md).
- `Contributor` or `Monitoring Contributor` role on the Cloud HSM resource group.

## Verify the logs resource group exists

Event Hub should deploy into the same resource group that contains your storage account and Log Analytics workspace for operational event logging. If you didn't set up operational event logging, first follow the guidance in [Configure and query operation event logging for Azure Cloud HSM](tutorial-operation-event-logging.md).

Verify that your targeted resource group exists:

```azurepowershell
az group show --name "<resource-group>" --query "{name:name, location:location}" --output table
```

## Create an Event Hubs namespace

The namespace is the container that holds one or more event hubs. Use the **Standard** tier, which is required for diagnostic settings integration:

```azurepowershell
az eventhubs namespace create `
  --name "<eventhub-namespace>" `
  --resource-group "<resource-group>" `
  --location "<location>" `
  --sku Standard `
  --capacity 1 `
  --enable-auto-inflate false
```

Key options:

- `--sku Standard`: The Basic tier doesn't support diagnostic settings as a destination.
- `--capacity 1`: One throughput unit (1 MB/s ingress, 2 MB/s egress) is sufficient for HSM audit logs.
- `--enable-auto-inflate false`: HSM log volume is low, so auto-inflate isn't necessary.

## Create an event hub inside the namespace

Create an event hub (topic) to receive the Cloud HSM logs:

```azurepowershell
az eventhubs eventhub create `
  --name "cloudhsm-logs" `
  --namespace-name "<eventhub-namespace>" `
  --resource-group "<resource-group>" `
  --partition-count 2 `
  --retention-time-in-hours 168 `
  --cleanup-policy Delete
```

Key options:

- `--partition-count 2`: Two partitions are sufficient for HSM audit log throughput.
- `--retention-time-in-hours 168`: Keep messages for 7 days (168 hours, which is the maximum for Standard tier).
- `--cleanup-policy Delete`: Delete messages after the retention period expires.

## Create a consumer group

Create a dedicated consumer group for downstream processing. Reserve the default `$Default` group for other uses:

```azurepowershell
az eventhubs eventhub consumer-group create `
  --name "azure-cloud-hsm" `
  --namespace-name "<eventhub-namespace>" `
  --eventhub-name "cloudhsm-logs" `
  --resource-group "<resource-group>"
```

## Create an authorization rule

Diagnostic settings need **Send** permission to push logs into the event hub. Create a shared access policy with only the required permission:

```azurepowershell
az eventhubs namespace authorization-rule create `
  --name "DiagnosticSettingsSendRule" `
  --namespace-name "<eventhub-namespace>" `
  --resource-group "<resource-group>" `
  --rights Send
```

> [!NOTE]
> This rule grants only `Send` permission, not `Listen` or `Manage`. Follow the principle of least privilege. Your downstream consumers (such as Azure Functions or Stream Analytics) should use a separate rule with `Listen` permission.

## Get the authorization rule resource ID

Retrieve the authorization rule resource ID for use in the diagnostic setting:

```azurepowershell
$authRuleId = az eventhubs namespace authorization-rule show `
  --name "DiagnosticSettingsSendRule" `
  --namespace-name "<eventhub-namespace>" `
  --resource-group "<resource-group>" `
  --query id --output tsv

Write-Host "Auth Rule ID: $authRuleId"
```

Save this value for use in the next step.

## Update the diagnostic setting to add Event Hub

You have two options for adding Event Hub as a destination:

### Option A: Update the existing diagnostic setting (recommended)

This approach updates your existing diagnostic setting to add Event Hub while keeping Storage and Log Analytics:

```azurepowershell
# Set your resource group variables
$hsmResourceGroup = "<resource-group>"
$logsResourceGroup = "<resource-group>"

# Find the HSM cluster name (auto-generated during deployment)
$hsmClusterName = az resource list `
  --resource-group $hsmResourceGroup `
  --resource-type Microsoft.HardwareSecurityModules/cloudHsmClusters `
  --query "[0].name" --output tsv
Write-Host "HSM Cluster: $hsmClusterName"

# Get the HSM cluster resource ID
$hsmResourceId = az resource show `
  --resource-group $hsmResourceGroup `
  --resource-type Microsoft.HardwareSecurityModules/cloudHsmClusters `
  --name $hsmClusterName `
  --query id --output tsv

# Get your existing storage account ID
$storageAccountId = az storage account list `
  --resource-group $logsResourceGroup `
  --query "[0].id" --output tsv

# Get your existing Log Analytics workspace ID
$workspaceId = az monitor log-analytics workspace list `
  --resource-group $logsResourceGroup `
  --query "[0].id" --output tsv

# Get the Event Hub auth rule ID
$authRuleId = az eventhubs namespace authorization-rule show `
  --name "DiagnosticSettingsSendRule" `
  --namespace-name "<eventhub-namespace>" `
  --resource-group $logsResourceGroup `
  --query id --output tsv

# Update the diagnostic setting with all three destinations
az monitor diagnostic-settings create `
  --name "<diagnostic-setting-name>" `
  --resource $hsmResourceId `
  --storage-account $storageAccountId `
  --workspace $workspaceId `
  --event-hub "cloudhsm-logs" `
  --event-hub-rule $authRuleId `
  --logs '[{\"category\":\"HsmServiceOperations\",\"enabled\":true}]'
```

> [!IMPORTANT]
> The `az monitor diagnostic-settings create` command performs an upsert operation. If the name matches an existing setting, it replaces the setting entirely. You must include `--storage-account` and `--workspace` again, or those destinations are removed.

### Option B: Create a separate diagnostic setting for Event Hub only

If you prefer to keep your existing setting unchanged and add a second one:

```azurepowershell
# Find the HSM cluster name (if you don't already have it from Option A)
$hsmClusterName = az resource list `
  --resource-group "<resource-group>" `
  --resource-type Microsoft.HardwareSecurityModules/cloudHsmClusters `
  --query "[0].name" --output tsv

$hsmResourceId = az resource show `
  --resource-group "<resource-group>" `
  --resource-type Microsoft.HardwareSecurityModules/cloudHsmClusters `
  --name $hsmClusterName `
  --query id --output tsv

$authRuleId = az eventhubs namespace authorization-rule show `
  --name "DiagnosticSettingsSendRule" `
  --namespace-name "<eventhub-namespace>" `
  --resource-group "<resource-group>" `
  --query id --output tsv

az monitor diagnostic-settings create `
  --name "chsm-eventhub-diagnostic-setting" `
  --resource $hsmResourceId `
  --event-hub "cloudhsm-logs" `
  --event-hub-rule $authRuleId `
  --logs '[{\"category\":\"HsmServiceOperations\",\"enabled\":true}]'
```

> [!NOTE]
> Azure supports up to five diagnostic settings per resource. A second setting is valid and keeps concerns separated.

## Verify Event Hub is receiving messages

After you configure the diagnostic setting, verify that Event Hub is receiving Cloud HSM logs.

### Check the diagnostic setting in the portal

1. In the Azure portal, go to your Cloud HSM cluster.
1. Under **Monitoring**, select **Diagnostic settings**.
1. Confirm that Event Hub is listed as a destination.

### Check Event Hub metrics

Run the following command to check incoming messages over the last hour:

```azurepowershell
# Get your subscription ID
$subId = az account show --query id --output tsv

# Check incoming messages (last 1 hour)
az monitor metrics list `
  --resource "/subscriptions/$subId/resourceGroups/<resource-group>/providers/Microsoft.EventHub/namespaces/<eventhub-namespace>" `
  --metric "SuccessfulRequests" `
  --interval PT1H `
  --output table
```

### Peek at messages (optional)

If you want to read a few messages to confirm content, create a **Listen** rule:

```azurepowershell
# Create a Listen rule for your consumer
az eventhubs namespace authorization-rule create `
  --name "ConsumerListenRule" `
  --namespace-name "<eventhub-namespace>" `
  --resource-group "<resource-group>" `
  --rights Listen

# Get the connection string
az eventhubs namespace authorization-rule keys list `
  --name "ConsumerListenRule" `
  --namespace-name "<eventhub-namespace>" `
  --resource-group "<resource-group>" `
  --query primaryConnectionString --output tsv
```

You can use this connection string with Azure Event Hub Explorer, the VS Code Event Hub extension, or a Python script to peek at messages.

## Related content

- [Configure and query operation event logging for Azure Cloud HSM](tutorial-operation-event-logging.md)
- [Azure Event Hubs documentation](/azure/event-hubs/event-hubs-about)
- [Diagnostic settings in Azure Monitor](/azure/azure-monitor/essentials/diagnostic-settings)
- [Azure Monitor diagnostic log schema](/azure/azure-monitor/essentials/resource-logs-schema)
