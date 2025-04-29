---
title: Create a managed application to store blob digests
description: Learn to create a managed application that stores blob digests to Azure Confidential Ledger
author: pallabpaul
ms.author: pallabpaul
ms.date: 10/26/2023
ms.service: azure-confidential-ledger
ms.topic: overview
---

# Create a Managed Application to Store Blob Digests

## Prerequisites

- An Azure Storage Account
- [Azure CLI](/cli/azure/install-azure-cli) (optional)
- Python version that is [supported by the Azure SDK for Python](https://github.com/Azure/azure-sdk-for-python#prerequisites) (optional)

## Overview

The **Blob Storage Digest Backed by Confidential Ledger** Managed Application can be used to guarantee that the blobs within a blob container are trusted and not tampered with. The application, once connected to a storage account, tracks all blobs being added to every container in the storage account in real time in addition to calculating and storing the digests into Azure Confidential Ledger. Audits can be performed at any time to check the validity of the blobs and to ensure that the blob container isn't tampered with.


## Deploying the managed application

The Managed Application can be found in the Azure Marketplace here: [Blob Storage Digests Backed by Confidential Ledger (preview)](https://azuremarketplace.microsoft.com/marketplace/apps/azureconfidentialledger.acl-blob-storage?tab=Overview).

### Resources to be created

Once the required fields are filled and the application is deployed, the following resources are created under a Managed Resource Group:

- [Confidential Ledger](overview.md)
- [Service Bus Queue](/azure/service-bus-messaging/service-bus-messaging-overview) with [Sessions](/azure/service-bus-messaging/message-sessions) enabled
- [Storage Account](/azure/storage/common/storage-account-overview) (Publisher owned storage account used to store digest logic and audit history)
- [Function App](/azure/azure-functions/functions-overview)
- [Application Insights](/azure/azure-monitor/app/app-insights-overview)

## Connecting a storage account to the managed application

Once a Managed Application is created, you're able to then connect the Managed Application to your Storage Account to start processing and recording Blob Container digests to Azure Confidential Ledger.

### Create a topic and event subscription for the storage account

The Managed Application uses an Azure Service Bus Queue to track and record all **Create Blob** and **Delete Blob** events. You will use the Queue created in the Managed Resource Group by the Managed Application and add it as an Event Subscriber for any storage account that you're creating blobs for. Also, ensure the `System Topic Name` associated with the storage account you are tracking is assigned the `Azure Service Bus Data Sender` for the Azure Service Bus Queue created by the managed app.

### [Azure portal](#tab/azure-portal)

:::image type="content" source="./media/managed-application/managed-app-event-topic-inline.png" alt-text="Screenshot of the Azure portal in a web browser, showing how to set up a service bus role." lightbox="./media/managed-application/managed-app-event-topic-enhanced.png":::

:::image type="content" source="./media/managed-application/managed-app-event-subscription-inline.png" alt-text="Screenshot of the Azure portal in a web browser, showing how to set up a storage event subscription." lightbox="./media/managed-application/managed-app-event-subscription-enhanced.png":::

On the Azure portal, you can navigate to the storage account that you would like to start creating blob digests for and go to the `Events` blade. There you can create an Event Subscription and connect it to the Azure Service Bus Queue Endpoint. Be sure to mark the `Managed identity type` as `System Assigned`.

:::image type="content" source="./media/managed-application/managed-app-event-session-id-inline.png" alt-text="Screenshot of the Azure portal in a web browser, showing how to set up a storage event subscription session ID." lightbox="./media/managed-application/managed-app-event-session-id-enhanced.png":::

The queue uses sessions to maintain ordering across multiple storage accounts so you will also need to navigate to the `Delivery Properties` tab and to enter a unique session ID for this event subscription.

### [CLI](#tab/cli-or-sdk)

**Creating the Event Topic:**

```azurecli
az eventgrid system-topic create \
  --resource-group {resource_group} \
  --name {sample_topic_name} \
  --location {location} \
  --topic-type microsoft.storage.storageaccounts \
  --source /subscriptions/{subscription}/resourceGroups/{resource_group}/providers/Microsoft.Storage/storageAccounts/{storage_account_name} \
  --identity SystemAssigned
```

`resource-group` - Resource Group of where Topic should be created

`name` - Name of Topic to be created

`location` - Location of Topic to be created

`source` - Resource ID of storage account to create Topic for

**Creating the Event Subscription:**

```azurecli
az eventgrid system-topic event-subscription create \
--name {sample_subscription_name} \
--system-topic-name {sample_topic_name} \
--resource-group {resource_group} \
--event-delivery-schema EventGridSchema \
--included-event-types Microsoft.Storage.BlobCreated \
--delivery-attribute-mapping sessionId static {sample_session_id} false \
--endpoint-type servicebusqueue \
--endpoint /subscriptions/{subscription}/resourceGroups/{managed_resource_group}/providers/Microsoft.ServiceBus/namespaces/{service_bus_namespace}/queues/{service_bus_queue}
```

`name` - Name of Subscription to be created

`system-topic-name` - Name of Topic the Subscription is being created for (Should be same as newly created topic)

`resource-group` - Resource Group of where Subscription should be created

`delivery-attribute-mapping` - Mapping for required sessionId field. Enter a unique sessionId

`endpoint` - Resource ID of the service bus queue that is subscribing to the storage account Topic

---

### Add required role to storage account

The Managed Application requires the `Storage Blob Data Owner` role to read and create hashes for each blob and this role is required to be added in order for the digest to be calculated correctly.

### [Azure portal](#tab/azure-portal)

:::image type="content" source="./media/managed-application/managed-app-managed-identity-inline.png" alt-text="Screenshot of the Azure portal in a web browser, showing how to set up a managed identity for the managed app." lightbox="./media/managed-application/managed-app-managed-identity-enhanced.png":::

### [CLI](#tab/cli-or-sdk)

```azurecli
az role assignment create \
--role "Storage Blob Data Owner" \
--assignee-object-id {function_oid} \
--assignee-principal-type ServicePrincipal\
--scope /subscriptions/{subscription}/resourceGroups/{resource_group}/providers/Microsoft.Storage/storageAccounts/{storage_account_name}
```

`assignee-object-id` - OID of the Azure Function created with the Managed Application. Can be found under the 'Identity' blade

`scope` - Resource ID of storage account to create the role for

---

> [!NOTE]
> Multiple storage accounts can be connected to a single Managed Application instance. We currently recommend a maximum of **10 storage accounts** that contain high usage blob containers.

## Adding blobs and digest creation

Once the storage account is properly connected to the Managed Application, blobs can start being added to containers within the storage account. The blobs are tracked in real-time and digests are calculated and stored in Azure Confidential Ledger.

### Transaction and block tables

All blob creation events are tracked in internal tables stored within the Managed Application.

:::image type="content" source="./media/managed-application/managed-app-transaction-table-inline.png" alt-text="Screenshot of the Azure portal in a web browser, showing the transaction table where blob hashes are stored." lightbox="./media/managed-application/managed-app-transaction-table-enhanced.png":::

The transaction table holds information about each blob and a unique hash that is generated using a combination of the blob's metadata and/ or contents.

:::image type="content" source="./media/managed-application/managed-app-block-table-inline.png" alt-text="Screenshot of the Azure portal in a web browser, showing the block table where digest information is stored." lightbox="./media/managed-application/managed-app-block-table-enhanced.png":::

The block table holds information related to every digest this is created for the blob container and the associated transaction ID for the digest is stored in Azure Confidential Ledger.

### Digest settings

:::image type="content" source="./media/managed-application/managed-app-digest-settings-inline.png" alt-text="Screenshot of managed app provisioning displaying the digest settings." lightbox="./media/managed-application/managed-app-digest-settings-enhanced.png":::

There are a few digest settings that can be selected when creating the managed application. You can choose the `Hashing Algorithm` being used to create the digests whether it be `MD5` or `SHA256`. You are also able to choose the number of blobs that are contained within each digest or the `Digest Size`. The Digest Size ranges from `1-16` and is the number of blobs that will be hashed together within each block. Lastly, you are able to select the `Hash Contents` and what will be hashed when creating each digest. This can be the `File Contents + Metadata` of each blob or just the `File Contents`.

### Viewing digest on Azure Confidential Ledger

You can view the digests being stored directly in Azure Confidential Ledger by navigating to the `Ledger Explorer` blade.

:::image type="content" source="./media/managed-application/managed-app-acl-ledger-explorer-inline.png" alt-text="Screenshot of the Azure portal in a web browser, showing the Azure Confidential Ledger explorer with digest transactions." lightbox="./media/managed-application/managed-app-acl-ledger-explorer-enhanced.png":::

## Performing an audit

If you ever want to check the validity of the blobs that are added to a container to ensure that they aren't tampered with, an audit can be run at any point in time. The audit replays every blob creation event and recalculates the digests with the blobs that are stored in the container during the audit. It then compares the recalculated digests with the digests stored in Azure Confidential and provides a report displaying all digest comparisons and whether or not the blob container is tampered with.

### Triggering an audit

An audit can be triggered by including the following message to the Service Bus Queue associated with your Managed Application:

```json
{
    "eventType": "PerformAudit",
    "storageAccount": "<storage_account_name>",
    "blobContainer": "<blob_container_name>"
}
```

### [Azure portal](#tab/azure-portal)

:::image type="content" source="./media/managed-application/managed-app-queue-trigger-audit-inline.png" alt-text="Screenshot of the Azure portal in a web browser, how to trigger an audit by adding a message to the queue." lightbox="./media/managed-application/managed-app-queue-trigger-audit-enhanced.png":::

Be sure to include a `Session ID` as the queue has sessions enabled.

### [Python SDK](#tab/cli-or-sdk)

```python
import json
import uuid
from azure.servicebus import ServiceBusClient, ServiceBusMessage

SERVICE_BUS_CONNECTION_STR = "<service_bus_connection_string>"
QUEUE_NAME = "<service_bus_queue_name>"
STORAGE_ACCOUNT_NAME = "<storage_account_name>"
BLOB_CONTAINER_NAME = "<blob_container_name>"
SESSION_ID = str(uuid.uuid4())

servicebus_client = ServiceBusClient.from_connection_string(conn_str=SERVICE_BUS_CONNECTION_STR, logging_enable=True)
sender = servicebus_client.get_queue_sender(queue_name=QUEUE_NAME)

message = {
    "eventType": "PerformAudit",
    "storageAccount": STORAGE_ACCOUNT_NAME,
    "blobContainer": BLOB_CONTAINER_NAME
}

message = ServiceBusMessage(json.dumps(message), session_id=SESSION_ID)
sender.send_messages(message)
```
---

### Viewing audit results

:::image type="content" source="./media/managed-application/managed-app-audit-record-inline.png" alt-text="Screenshot of the Azure portal in a web browser, showing a sample audit record with matching digests." lightbox="./media/managed-application/managed-app-audit-record-enhanced.png":::

Once an audit is performed successfully, the results of the audit can be found under a container named `<managed-application-name>-audit-records` found within the respective storage account. The results contain the recalculated digest, the digest retrieved from Azure Confidential Ledger and whether or not the blobs are tampered with.

:::image type="content" source="./media/managed-application/managed-app-email-alert-inline.png" alt-text="Screenshot of managed app provisioning displaying the audit email alert settings." lightbox="./media/managed-application/managed-app-email-alert-enhanced.png":::

When creating the managed application, if you opt in for email alerts you will get an email sent to your email during an `Audit Failure` or an `Audit Success and Failure` depending on what option is selected.

## Logging and errors

Error logs can be found under a container named `<managed-application-name>-error-logs` found within the respective storage account. If a blob creation event or audit process fails, the cause of the failure is recorded and stored in this container. If there are any questions about the error logs or application functionality, contact the Azure Confidential Ledger Support team provided in the Managed Application details.

## Clean up managed application

You can delete the Managed Application to clean up and remove all associated resources. Deleting the Managed Application stops all blob transactions from being tracked and stop all digests from being created. Audit reports remain valid for the blobs that were added before the deletion.

## More resources

For more information about managed applications and the deployed resources, see the following links:

- [Managed Applications](/azure/azure-resource-manager/managed-applications/overview)
- [Azure Service Queue Sessions](/azure/service-bus-messaging/message-sessions)
- [Azure Storage Events](/azure/storage/blobs/storage-blob-event-overview)

## Next steps

- [Overview of Microsoft Azure confidential ledger](overview.md)
