---
title: Create a Power Automate workflow using Azure Confidential Ledger Connector
description: Learn to create a Power Automate workflow using Azure Confidential Ledger Connector
author: ryazhang
ms.author: ryazhang
ms.date: 10/21/2025
ms.service: azure-confidential-ledger
ms.topic: overview
ms.custom: sfi-ropc-nochange, sfi-image-nochange
---


# Create a Power Automate workflow using Azure Confidential Ledger Connector

## Prerequisites

- An Azure Confidential ledger Instance - to create an instance, follow the steps in [Create an Azure Confidential Ledger instance](create-confidential-ledger-instance.md)
- [A Power Automate Premium user license](https://learn.microsoft.com/en-us/power-platform/admin/power-automate-licensing/types?tabs=power-automate-premium%2Cpower-automate-process%2Cconnector-types)
- [Azure CLI](/cli/azure/install-azure-cli) (optional)

## Overview

The Azure confidential ledger now available as a connector in Power Automate. You can use this connector to create workflows that interact with your confidential ledger instance. This guide will walk you through the steps to create a simple workflow that adds an entry to your confidential ledger then store the transcation id to cosmos db. 

## How to Find the Azure Confidential Ledger Connector in Power Automate

Simply search for "Azure Confidential Ledger" in the Power Automate connector list, then you can use the available actions to create your workflow. 
:::image type="content" source="./media/power-automate/how_to_find_acl_connector.png" alt-text="Screenshot of the Power Automate connector list with Azure Confidential Ledger highlighted." lightbox="./media/power-automate/how_to_find_acl_connector.png":::

## Set up before creating an actions in Power Automate

Before creating actions in Power Automate, ensure you have the necessary permissions and access to the Azure Confidential Ledger instance. You may need to configure authentication and authorization settings to allow Power Automate to interact with your confidential ledger.

Due to limitations in Power Automate, Azure Confidential Ledger only supports Azure AD token-based user authentication. 

To find user object id by email login into the Power Automate Platform, you can use the Azure CLI command below:
```bash
az ad user show --id user@example.com --query id --output tsv
```

Next, we need assign the correct role to the user. The available roles are Contributor, Administrator, and Reader. To proceed, please follow the steps outlined [here](manage-azure-ad-token-based-users.md)
To validate change we can go to azure Portal -> Azure Confidential Ledger instance -> Operations -> Manage Users (Preview).
:::image type="content" source="./media/power-automate/role_assignment.png" alt-text="Screenshot of the Role assignments page in Azure Portal for Azure Confidential Ledger." lightbox="./media/power-automate/role_assignment.png":::

## Create a Power Automate Workflow and Use Azure Confidential Ledger Connector

1. Log in to the [Power Automate](https://flow.microsoft.com/) platform.
2. Click on "Create" from the left navigation pane.
3. Choose "Automated cloud flow" or "Instant cloud flow" based on your requirements.
4. Click on "Create" to start building your flow.
5. In the flow editor, click on "New step" to add an action.
6. Search for "Azure Confidential Ledger" in the action search bar and select the desired action (e.g., "Add Entry to Ledger").
7. Create connections if you haven't already done so by providing the necessary authentication details.
8. Configure the action by providing the necessary details such as the ledger instance URL, entry data, etc.
9. Once you have configured all the actions, click on "Save" to  save your flow.
10. Test your flow to ensure it works as expected.

## Azure Confidential Ledger Connector supported actions

The Azure Confidential Ledger connector provides several actions that allow you to interact with your confidential ledger instance from Power Automate workflows. Below are the supported actions based on the connector specification:

### Create Ledger Entry
Writes a ledger entry to the confidential ledger.

**Operation ID**: `CreateLedgerEntry`

**Parameters:**
- **Ledger Name**: The name of your Azure Confidential Ledger instance
- **Collection ID** (optional): The collection where you want to add the entry
- **Entry Contents**: The data to be stored in the ledger entry (string format)

**Returns:**
- **Collection ID**: The collection where the entry was stored
- **Transaction ID**: Unique identifier for the transaction (returned in response header `x-ms-ccf-transaction-id`)
:::image type="content" source="./media/power-automate/write-ledger-entry.png" alt-text="Screenshot of the Power Automate workflow showing the Write Ledger Entry action." lightbox="./media/power-automate/write-ledger-entry.png":::

### Get Ledger Entry
Gets a specific ledger entry by transaction ID.

**Operation ID**: `GetLedgerEntry`

**Parameters:**
- **Ledger Name**: The name of your Azure Confidential Ledger instance
- **Transaction ID**: The transaction ID of the entry to retrieve
- **Collection ID** (optional): The collection from which to fetch the value

**Returns:**
- **State**: Query state (Loading or Ready)
- **Entry**: The ledger entry data (available only if state is "Ready")
  - **Contents**: Contents of the ledger entry
  - **Collection ID**: The collection containing the entry
  - **Transaction ID**: The transaction ID

:::image type="content" source="./media/power-automate/get-ledger-entry-by-tx-id.png" alt-text="Screenshot of the Power Automate workflow showing the Get Ledger Entry by Transaction ID action." lightbox="./media/power-automate/get-ledger-entry-by-tx-id.png":::

### Get Current Ledger Entry
Gets the current (most recent) ledger entry from a collection.

**Operation ID**: `GetCurrentLedgerEntry`

**Parameters:**
- **Ledger Name**: The name of your Azure Confidential Ledger instance
- **Collection ID** (optional): The collection from which to fetch the current entry

**Returns:**
- **Contents**: Contents of the most recent ledger entry
- **Collection ID**: The collection containing the entry
- **Transaction ID**: The transaction ID of the current entry

:::image type="content" source="./media/power-automate/get-current-ledger-entry.png" alt-text="Screenshot of the Power Automate workflow showing the Get Current Ledger Entry action." lightbox="./media/power-automate/get-current-ledger-entry.png":::

### List Ledger Entries
Gets ledger entries by collection and range.

**Operation ID**: `ListLedgerEntries`

**Parameters:**
- **Ledger Name**: The name of your Azure Confidential Ledger instance
- **Collection ID** (optional): Only entries in the specified collection will be returned
- **From Transaction ID** (optional): Specify the first transaction ID in a range
- **To Transaction ID** (optional): Specify the last transaction ID in a range

**Returns:**
- **State**: Query state (Loading or Ready)
- **Entries**: Array of ledger entries
- **Next Link**: Path to retrieve the next page of results (for pagination)

:::image type="content" source="./media/power-automate/get-ledger-id-range.png" alt-text="Screenshot of the Power Automate workflow showing the Get Ledger ID Range action." lightbox="./media/power-automate/get-ledger-id-range.png":::

### Get Receipt
Gets a cryptographic receipt for a transaction by transaction ID.

**Operation ID**: `GetReceipt`

**Parameters:**
- **Ledger Name**: The name of your Azure Confidential Ledger instance
- **Transaction ID**: The transaction ID for which to get the receipt

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
Gets the status of a transaction by transaction ID.

**Operation ID**: `GetTransactionStatus`

**Parameters:**
- **Ledger Name**: The name of your Azure Confidential Ledger instance
- **Transaction ID**: The transaction ID to check

**Returns:**
- **State**: Transaction state (Committed or Pending)
- **Transaction ID**: The transaction ID

:::image type="content" source="./media/power-automate/get-ledger-tx-status.png" alt-text="Screenshot of the Power Automate workflow showing the Get Transaction Status action." lightbox="./media/power-automate/get-ledger-tx-status.png":::

## Example Workflow: Adding an Entry and Storing Transaction ID

Here's a practical example of how to use the Azure Confidential Ledger connector in a Power Automate workflow:

### Scenario
Create a workflow that:
1. Adds a new entry to the confidential ledger
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

:::image type="content" source="./media/power-automate/PA_example_1.png" alt-text="Screenshot of the Power Automate workflow showing the Create Ledger Entry action." lightbox="./media/power-automate/PA_example_1.png":::

4. **Store in Cosmos DB Action**:
   - Use the parsed transaction ID from the previous step
   - Store it along with relevant metadata for future reference
   - For detailed information about the Cosmos DB connector, see [Azure Cosmos DB connector documentation](https://learn.microsoft.com/en-us/connectors/documentdb/)
:::image type="content" source="./media/power-automate/PA_example_2.png" alt-text="Screenshot of the Power Automate workflow showing the Cosmos DB action." lightbox="./media/power-automate/PA_example_2.png":::

### Example Entry Content Formats

The entry contents must be a string with a specific JSON structure containing a "content" field. Here are some common patterns:

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

This workflow ensures that all important events are immutably recorded in the confidential ledger while maintaining easy access to transaction references through Cosmos DB.

## Next Steps

- Learn more about [Azure Confidential Ledger concepts](overview.md)
- Explore [authentication options](authentication-azure-ad.md) for advanced scenarios
- Review [best practices](secure-confidential-ledger.md) for production workflows


