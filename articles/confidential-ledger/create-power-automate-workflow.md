---
title: Data Ingress and Egress from Azure confidential ledger using Power Automate Connector
description: Learn to ingress and egress data from Azure confidential ledger using a Power Automate connector
author: ryazhang
ms.author: ryazhang
ms.date: 10/21/2025
ms.service: azure-confidential-ledger
ms.topic: overview
ms.custom: sfi-ropc-nochange, sfi-image-nochange
---

# Create a Power Automate workflow using Azure confidential ledger Connector
## Prerequisites
- An Azure confidential ledger Instance - to create an instance, follow the steps in [Create an Azure confidential ledger instance](./quickstart-portal.md)
- [A Power Automate Premium user license](https://learn.microsoft.com/power-platform/admin/power-automate-licensing/types?tabs=power-automate-premium%2Cpower-automate-process%2Cconnector-types)
- [Azure CLI](/cli/azure/install-azure-cli) (optional)

## Overview
The Azure confidential ledger is now available as a connector in Power Automate. This connector can be used in a workflow to read from and write data into a ledger instance. This guide walks you through the steps to create a sample workflow that adds an entry to a ledger instance and store the transaction ID into Cosmos DB.

## Locate the Azure confidential ledger connector
Search for the Azure confidential ledger connector in the Power Automate connectors list and create a workflow using the available actions.

:::image type="content" source="./media/power-automate/how-to-find-acl-connector.png" alt-text="Screenshot of the Power Automate connector list with Azure confidential ledger highlighted." lightbox="./media/power-automate/how-to-find-acl-connector.png":::

## Initial set-up
Before creating actions in Power Automate, ensure you have the necessary permissions and access to the Azure confidential ledger instance. You need to configure authentication and authorization settings to allow Power Automate to interact with your confidential ledger.

Due to limitations in Power Automate, Azure confidential ledger only supports Microsoft Entra ID token-based user authentication.

The connector performs read and write operations on-behalf of a user or service principal that has appropriate permission in the ledger. Currently it supports only Microsoft Entra ID token-based authentication. A Microsoft Entra ID principal is a unique Object ID. Use the following command to get the Object ID of the principal. It's used in the subsequent step to grant appropriate permission on the ledger.

```bash
az ad user show --id user@example.com --query id --output tsv
```

Depending on the workflow requirement, assign either a Reader or Contributor role to the principal in the ledger. Follow the steps available here to assign a role.
To validate change, we can go to Azure portal -> Azure confidential ledger instance -> Operations -> Manage Users (Preview).
:::image type="content" source="./media/power-automate/role-assignment.png" alt-text="Screenshot of the Role assignments page in Azure portal for Azure confidential ledger." lightbox="./media/power-automate/role-assignment.png":::

## Use Azure confidential ledger Connector in a workflow

1. Sign in to the [Power Automate](https://flow.microsoft.com/) platform.
2. Select "Create" from the left Navigation pane
3. Choose either "Automated cloud flow" or "Instant cloud flow."
4. Select "Create" to start building the flow.
5. In the flow editor, select "New step" to add an action.
6. Search for "Azure confidential Ledger" in the action search bar and select a desired action such as "Add Entry to Ledger."
7. Create a connection by providing the necessary details.
8. Configure the action by providing the necessary details like the ledger URL, entry data, etc.
9. Select "Save" to save the changes.
10. Test your flow to ensure it works as expected.

## Supported Actions

The Azure confidential ledger connector supports the following actions that allows it to interact with a confidential ledger instance.

### Create Ledger Entry
Write a ledger entry.

**Operation ID**: `CreateLedgerEntry`

**Parameters:**
- **Ledger Name**: The name of your Azure confidential ledger instance
- **Collection ID** (optional): The collection where you want to add the entry
- **Entry Contents**: The data to be stored in the ledger entry (string format)

**Returns:**
- **Collection ID**: The collection where the entry was stored
- **Transaction ID**: Unique identifier for the transaction (returned in response header `x-ms-ccf-transaction-id`)
:::image type="content" source="./media/power-automate/write-ledger-entry.png" alt-text="Screenshot of the Power Automate workflow showing the Write Ledger Entry action." lightbox="./media/power-automate/write-ledger-entry.png":::

### Get Ledger Entry
Get a ledger entry by its transaction ID.

**Operation ID**: `GetLedgerEntry`

**Parameters:**
- **Ledger Name**: The name of your Azure confidential ledger instance
- **Transaction ID**: The transaction ID of the entry to retrieve
- **Collection ID** (optional): The collection ID from which to fetch the value

**Returns:**
- **State**: Query state (Loading or Ready)
- **Entry**: The ledger entry data (available only if state is "Ready")
  - **Contents**: Contents of the ledger entry
  - **Collection ID**: The collection ID corresponding to the entry
  - **Transaction ID**: The transaction ID

:::image type="content" source="./media/power-automate/get-ledger-entry-by-txid.png" alt-text="Screenshot of the Power Automate workflow showing the Get Ledger Entry by Transaction ID action." lightbox="./media/power-automate/get-ledger-entry-by-txid.png":::

### Get Current Ledger Entry
Get the current (most recent) ledger entry from a collection

**Operation ID**: `GetCurrentLedgerEntry`

**Parameters:**
- **Ledger Name**: The name of your Azure confidential ledger instance
- **Collection ID** (optional): The collection ID corresponding to the entry

**Returns:**
- **Contents**: Contents of the most recent ledger entry
- **Collection ID**: The collection ID corresponding to the entry
- **Transaction ID**: The transaction ID of the current entry

:::image type="content" source="./media/power-automate/get-current-ledger-entry.png" alt-text="Screenshot of the Power Automate workflow showing the Get Current Ledger Entry action." lightbox="./media/power-automate/get-current-ledger-entry.png":::

### List Ledger Entries
Get ledger entries by collection and range.

**Operation ID**: `ListLedgerEntries`

**Parameters:**
- **Ledger Name**: The name of your Azure confidential ledger instance
- **Collection ID** (optional): Only entries in the specified collection return
- **From Transaction ID** (optional): Starting Transaction ID in the range
- **To Transaction ID** (optional): Ending Transaction ID in the range

**Returns:**
- **State**: Query state (Loading or Ready)
- **Entries**: A collection of entries within the specified Transaction ID range
- **Next Link**: A continuation link to retrieve the remaining entries.

:::image type="content" source="./media/power-automate/get-ledger-id-range.png" alt-text="Screenshot of the Power Automate workflow showing the Get Ledger ID Range action." lightbox="./media/power-automate/get-ledger-id-range.png":::

### Get Receipt
Get a cryptographic receipt for a transaction by transaction ID.

**Operation ID**: `GetReceipt`

**Parameters:**
- **Ledger Name**: The name of your Azure confidential ledger instance
- **Transaction ID**: Transaction ID corresponding to the receipt

**Returns:**
- **State**: Query state (Loading or Ready)
- **Transaction ID**: The transaction ID
- **Receipt**: Cryptographic receipt containing:
  - **Node ID**: Identifier of the node that processed the transaction
  - **Signature**: Digital signature
  - **Proof**: Cryptographic proof elements
  - **Certificate**: Node certificate

:::image type="content" source="./media/power-automate/get-ledger-receipt.png" alt-text="Screenshot of the Power Automate workflow showing the Get Receipt action." lightbox="./media/power-automate/get-ledger-receipt.png":::

### Get Transaction Status
Get the status of a transaction by transaction ID.

**Operation ID**: `GetTransactionStatus`

**Parameters:**
- **Ledger Name**: The name of your Azure confidential ledger instance
- **Transaction ID**: The transaction ID to check

**Returns:**
- **State**: Transaction state (Committed or Pending)
- **Transaction ID**: The transaction ID

:::image type="content" source="./media/power-automate/get-ledger-tx-status.png" alt-text="Screenshot of the Power Automate workflow showing the Get Transaction Status action." lightbox="./media/power-automate/get-ledger-tx-status.png":::

## Example Workflow: Add an entry and store the Transaction ID

The following section demonstrates how to use the connector to write a ledger entry


### Scenario
Create a workflow that:
1. Adds a new entry to the confidential ledger, and,
2. Stores the transaction ID in Azure Cosmos DB for reference

### Workflow Steps

1. **Trigger**: Choose your preferred trigger (manual, scheduled, or event-based)

2. **Create Ledger Entry Action**:
   - **Ledger Name**: `your-ledger-name` (not the full URL, just the name)
   - **Collection ID**: `audit-logs` (optional - leave empty for default collection)
   - **Entry Contents**: 
     ```
     {"content": "entry_data_here"}
     ```

3. **Parse JSON Action** (to extract transaction ID from headers):
   - Use `outputs('Create_Ledger_Entry')['headers']['x-ms-ccf-transaction-id']` to get the transaction ID

:::image type="content" source="./media/power-automate/pa-example-1.png" alt-text="Screenshot of the Power Automate workflow showing the Create Ledger Entry action." lightbox="./media/power-automate/pa-example-1.png":::

4. **Store in Cosmos DB Action**:
   - Use the parsed transaction ID from the previous step
   - Store it along with relevant metadata for future reference
   - For detailed information about the Cosmos DB connector, see [Azure Cosmos DB connector documentation](https://learn.microsoft.com/connectors/documentdb/)
:::image type="content" source="./media/power-automate/pa-example-2.png" alt-text="Screenshot of the Power Automate workflow showing the Cosmos DB action." lightbox="./media/power-automate/pa-example-2.png":::

### Example Entry Content Formats

The entry content must be a string with a specific JSON structure containing a "content" field. Here are some common patterns:

**JSON as String**:
```
{"content": "{\"event\": \"user_login\", \"userId\": \"12345\", \"timestamp\": \"@{utcNow()}\"}"}
```

**Plain Text Content**:
```
{"content": "User login event for user @{variables('userId')} at @{utcNow()}"}
```

**Base64 Encoded Data**:
```
{"content": "@{base64(variables('binaryData'))}"}
```

## Next Steps

- Learn more about [Azure confidential Ledger concepts](overview.md)
- Explore [authentication options](authentication-azure-ad.md) for advanced scenarios
- Review [best practices](secure-confidential-ledger.md) for production workflows


