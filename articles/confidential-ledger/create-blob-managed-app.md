---
title: Create a Managed Application to Store Blob Digests
description: Learn how to create a managed application that stores blob digests to Azure confidential ledger.
author: pallabpaul
ms.author: pallabpaul
ms.date: 10/26/2023
ms.service: azure-confidential-ledger
ms.topic: overview
ms.custom: sfi-ropc-nochange, sfi-image-nochange
---

# Create a managed application to store blob digests

## Prerequisites

- An Azure Storage account.
- The [Azure CLI](/cli/azure/install-azure-cli) (optional).
- The Python version that's [supported by the Azure SDK for Python](https://github.com/Azure/azure-sdk-for-python#prerequisites) (optional).

## Overview

You can use the Azure Blob Storage digest backed by Azure confidential ledger managed application to help ensure that the blobs within a blob container are trusted and not tampered with. After the application is connected to a storage account, it tracks all blobs that are added to every container in the storage account in real time. It also calculates and stores the digests in Azure confidential ledger. You can perform audits at any time to check the validity of the blobs and to ensure that the blob container isn't tampered with.

## Deploy the managed application

To find the managed application in Azure Marketplace, see [Blob Storage digests backed by confidential ledger (preview)](https://azuremarketplace.microsoft.com/marketplace/apps/azureconfidentialledger.acl-blob-storage?tab=Overview).

### Resources to be created

After the required fields are filled and the application is deployed, the following resources are created under a managed resource group:

- [Confidential ledger](overview.md)
- [Azure Service Bus queue](/azure/service-bus-messaging/service-bus-messaging-overview) with [sessions](/azure/service-bus-messaging/message-sessions) enabled
- [Storage account](/azure/storage/common/storage-account-overview) (Publisher owned storage account used to store digest logic and audit history)
- [Function app](/azure/azure-functions/functions-overview)
- [Application Insights](/azure/azure-monitor/app/app-insights-overview)

## Connect a storage account to the managed application

After the managed application is successfully deployed, you can connect it to an Azure Storage account to enable the processing and recording of Blob Container digests into confidential ledger.

The managed application currently supports the following types of storage accounts:

- **Standard General Purpose v2 (GPv2) Storage accounts**, including the accounts configured for hot, cool, or archive tiers.
- **Azure Data Lake Storage Gen2 (ADLS Gen2)** accounts with **Hierarchical Namespace (HNS)** enabled.
- **Storage accounts with either public or private endpoint configurations**, if appropriate network and identity permissions are granted.
- **Standard or Premium performance tiers**, if the **Blob service** is enabled.

Only the Blob service is supported. Other services (for example, File, Table, or Queue) aren't applicable for use with the managed application.

### Create a topic and event subscription for the storage account

The managed application uses a Service Bus queue to track and record all **Create Blob** and **Delete Blob** events. You use the Queue created in the managed resource group by the managed application and add it as an Event Subscriber for any storage account for which you're creating blobs. Also, ensure that the `System Topic Name` associated with the storage account that you're tracking is assigned `Azure Service Bus Data Sender` for the Service Bus queue created by the managed app.

### [Azure portal](#tab/azure-portal)

:::image type="content" source="./media/managed-application/managed-app-event-topic-inline.png" alt-text="Screenshot of the Azure portal in a web browser that shows how to set up a service bus role." lightbox="./media/managed-application/managed-app-event-topic-enhanced.png":::

:::image type="content" source="./media/managed-application/managed-app-event-subscription-inline.png" alt-text="Screenshot of the Azure portal in a web browser that shows how to set up a storage event subscription." lightbox="./media/managed-application/managed-app-event-subscription-enhanced.png":::

On the Azure portal, you can go to the storage account for which you want to start creating blob digests and go to the **Events** pane. There you can create an Event Subscription and connect it to the Service Bus queue endpoint. Be sure to mark the **Managed identity type** as **System Assigned**.

:::image type="content" source="./media/managed-application/managed-app-event-session-id-inline.png" alt-text="Screenshot of the Azure portal in a web browser that shows how to set up a storage event subscription session ID." lightbox="./media/managed-application/managed-app-event-session-id-enhanced.png":::

The queue uses sessions to maintain ordering across multiple storage accounts so that you also need to go to the **Delivery Properties** tab to enter a unique session ID for this event subscription.

### [CLI](#tab/cli-or-sdk)

#### Create the event topic

```azurecli
az eventgrid system-topic create \
  --resource-group {resource_group} \
  --name {sample_topic_name} \
  --location {location} \
  --topic-type microsoft.storage.storageaccounts \
  --source /subscriptions/{subscription}/resourceGroups/{resource_group}/providers/Microsoft.Storage/storageAccounts/{storage_account_name} \
  --identity SystemAssigned
```

- `resource-group`: Resource group of where the topic should be created.
- `name`: Name of the topic to be created.
- `location`: Location of the topic to be created.
- `source`: Resource ID of the storage account to create the topic for.

#### Create the event subscription

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

- `name`: Name of subscription to be created.
- `system-topic-name`: Name of the topic for which the subscription is being created (should be the same as the newly created topic).
- `resource-group`: Resource group of where the subscription should be created.
- `delivery-attribute-mapping`: Mapping for the required `sessionId` field. Enter a unique sessionId
- `endpoint`: Resource ID of the service bus queue that's subscribing to the storage account topic.

---

### Add required role to storage account

The managed application requires the `Storage Blob Data Owner` role to read and create hashes for each blob. This role is required to be added so that the digest is calculated correctly.

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

- `assignee-object-id`: Object identifier (OID) of the Azure Function created with the managed application. You can find it on the **Identity** pane.
- `scope`: Resource ID of the storage account for which to create the role.

---

> [!NOTE]
> You can connect multiple storage accounts to a single managed application instance. We currently recommend a maximum of 10 storage accounts that contain high-usage blob containers.

## Add blobs and digest creation

After the storage account is properly connected to the managed application, you can start to add blobs to the containers within the storage account. The blobs are tracked in real time. Digests are calculated and stored in confidential ledger.

### Transaction and block tables

All blob creation events are tracked in internal tables stored within the managed application.

:::image type="content" source="./media/managed-application/managed-app-transaction-table-inline.png" alt-text="Screenshot of the Azure portal in a web browser that shows the transaction table where blob hashes are stored." lightbox="./media/managed-application/managed-app-transaction-table-enhanced.png":::

The transaction table holds information about each blob and a unique hash that's generated by using a combination of the blob's metadata or contents.

:::image type="content" source="./media/managed-application/managed-app-block-table-inline.png" alt-text="Screenshot of the Azure portal in a web browser that shows the block table where digest information is stored." lightbox="./media/managed-application/managed-app-block-table-enhanced.png":::

The block table holds information related to every digest that was created for the blob container. The associated transaction ID for the digest is stored in confidential ledger.

### Digest settings

:::image type="content" source="./media/managed-application/managed-app-digest-settings-inline.png" alt-text="Screenshot of managed app provisioning that displays the digest settings." lightbox="./media/managed-application/managed-app-digest-settings-enhanced.png":::

There are a few digest settings that can be selected when you create the managed application. You can choose the `Hashing Algorithm` that's used to create the digests whether it's `MD5` or `SHA256`. You can also choose the number of blobs that are contained within each digest or the `Digest Size`. The Digest Size ranges from `1-16` and is the number of blobs that are hashed together within each block. Lastly, you can select the `Hash Contents` and what is hashed when each digest is created. It can be the `File Contents + Metadata` of each blob or just the `File Contents`.

### View the digest on confidential ledger

You can view the digests that are stored directly in confidential ledger by going to the **Ledger Explorer** pane.

:::image type="content" source="./media/managed-application/managed-app-acl-ledger-explorer-inline.png" alt-text="Screenshot of the Azure portal in a web browser that shows the confidential ledger explorer with digest transactions." lightbox="./media/managed-application/managed-app-acl-ledger-explorer-enhanced.png":::

## Perform an audit

To check the validity of the blobs that were added to a container to ensure that they weren't tampered with, you can run an audit at any point. The audit replays every blob creation event and recalculates the digests with the blobs that are stored in the container during the audit. It then compares the recalculated digests with the digests stored in confidential ledger. The audit provides a report that displays all digest comparisons and whether or not the blob container was tampered with.

### Trigger an audit

You can trigger an audit by including the following message to the Service Bus queue associated with your managed application:

```json
{
    "eventType": "PerformAudit",
    "storageAccount": "<storage_account_name>",
    "blobContainer": "<blob_container_name>"
}
```

### [Azure portal](#tab/azure-portal)

:::image type="content" source="./media/managed-application/managed-app-queue-trigger-audit-inline.png" alt-text="Screenshot os the Azure portal in a web browser that shows how to trigger an audit by adding a message to the queue." lightbox="./media/managed-application/managed-app-queue-trigger-audit-enhanced.png":::

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

### View audit results

:::image type="content" source="./media/managed-application/managed-app-audit-record-inline.png" alt-text="Screenshot of the Azure portal in a web browser, showing a sample audit record with matching digests." lightbox="./media/managed-application/managed-app-audit-record-enhanced.png":::

After an audit is performed successfully, you can find the results of the audit under a container named `<managed-application-name>-audit-records` found within the respective storage account. The results contain the recalculated digest, the digest retrieved from confidential ledger, and whether or not the blobs were tampered with.

:::image type="content" source="./media/managed-application/managed-app-email-alert-inline.png" alt-text="Screenshot of managed app provisioning that displays the audit email alert settings." lightbox="./media/managed-application/managed-app-email-alert-enhanced.png":::

When you create the managed application, if you opt in for email alerts you get an email sent to your email address during an `Audit Failure` or an `Audit Success and Failure` depending on the option that you select.

### Enable user tracking audits

You can track which users uploaded or deleted blobs by using diagnostic logs sent to the internal storage account created by the managed app.

#### 1. Enable diagnostic settings

1. Go to your Storage account and select **Monitoring** > **Diagnostic settings**.
1. Add a new setting for the **blob** service.
1. Enable:
   - `Storage Write`
   - `Storage Delete`
1. Set the destination to the internal archive Storage account created by the managed app.

#### 2. Send the following message to the Service Bus queue (with a session ID)

After you see the diagnostic logs flowing into the Storage account, you can add the `getUsers` field when an audit event is triggered:

```json
{
"eventType": "PerformAudit",
"storageAccount": "<storage_account_name>",
"blobContainer": "<blob_container_name>",
"getUsers": true
}
```

#### 3. Check the audit results

After an audit is successfully processed, each entry includes the user who performed the operation along with their OID if found:

```json
"user": {
  "upn": "user@example.com",
  "oid": "<object-id>"
}
```

## Logging and errors

You can find error logs under a container named `<managed-application-name>-error-logs` found within the respective storage account. If a blob creation event or audit process fails, the cause of the failure is recorded and stored in this container. If there are any questions about the error logs or application functionality, contact the Azure Confidential Ledger Support team provided in the managed application details.

## Clean up managed application

You can delete the managed application to clean up and remove all associated resources. Deleting the managed application stops all blob transactions from being tracked and stops all digests from being created. Audit reports remain valid for the blobs that were added before the deletion.

## More resources

For more information about managed applications and the deployed resources, see the following links:

- [Managed applications](/azure/azure-resource-manager/managed-applications/overview)
- [Azure Service Bus queue sessions](/azure/service-bus-messaging/message-sessions)
- [Azure Storage events](/azure/storage/blobs/storage-blob-event-overview)

## Related content

- [Overview of Azure confidential ledger](overview.md)
