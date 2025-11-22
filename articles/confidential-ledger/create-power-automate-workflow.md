---
title: Data Ingress and Egress from Azure Confidential Ledger by using a Power Automate connector
description: Learn to ingress and egress data from Azure confidential ledger by using a Power Automate connector.
author: ryazhang
ms.author: ryazhang
ms.date: 10/21/2025
ms.service: azure-confidential-ledger
ms.topic: overview
ms.custom: sfi-ropc-nochange, sfi-image-nochange
---

# Data ingress and egress from Azure confidential ledger by using a Power Automate connector

A Power Automate connector is now available for interacting with Azure confidential ledger. This article explains how to build a workflow that adds an entry into a confidential ledger instance and saves the transaction ID in Azure Cosmos DB.

## Prerequisites

- An Azure confidential ledger instance. To create an instance, follow the steps in [Create an Azure confidential ledger instance](./quickstart-portal.md).
- A [Power Automate Premium user license](/power-platform/admin/power-automate-licensing/types?tabs=power-automate-premium%2Cpower-automate-process%2Cconnector-types).
- The [Azure CLI](/cli/azure/install-azure-cli) (optional).

## Locate the Azure confidential ledger connector

Search for the confidential ledger connector in the Power Automate connectors list. To create a workflow, use the available actions.

:::image type="content" source="./media/power-automate/how-to-find-acl-connector.png" alt-text="Screenshot of the Power Automate connector list that shows Azure confidential ledger." lightbox="./media/power-automate/how-to-find-acl-connector.png":::

## Initial setup

Before you create actions in Power Automate, ensure that you have the necessary permissions and access to the onfidential ledger instance. You need to configure authentication and authorization settings to allow Power Automate to interact with your ledger.

Currently, Power Automate supports only Microsoft Entra ID token-based user authentication with confidential ledger.

The connector performs read and write operations by using a user or service principal that has appropriate permission in the ledger. A service principal is identified by a unique object identifier (OID). Use the following command to get the OID. It's used in the following step to grant appropriate permission in the ledger.

```bash
az ad user show --id user@example.com --query id --output tsv
```

Depending on the workflow requirement, assign either a Reader or Contributor role to the service principal in the ledger. Follow these steps to assign a role.

To validate change, go to the Azure portal. Select the confidential ledger instance, and then select **Operations** > **Manage users (preview)**.
:::image type="content" source="./media/power-automate/role-assignment.png" alt-text="Screenshot of the Role Assignments tab for Azure confidential ledger." lightbox="./media/power-automate/role-assignment.png":::

## Use a confidential ledger connector in a workflow

1. Sign in to the [Power Automate](https://flow.microsoft.com/) platform.
1. On the left pane, select **Create**.
1. Choose either **Automated cloud flow** or **Instant cloud flow**.
1. Select **Create** to start building the flow.
1. In the flow editor, select **New step** to add an action.
1. Search for **Azure confidential ledger** in the action search bar. Select the action that you want, such as **Add Entry to Ledger**.
1. Configure the action by providing the necessary details like the ledger URL and entry data.
1. Select **Save** to save the changes.
1. Test your flow to ensure that it works as expected.

## Supported actions

The confidential ledger connector supports the following actions.

### Create a ledger entry

Write a ledger entry.

**Operation ID**: `CreateLedgerEntry`

**Parameters:**

- **Ledger Name**: The name of your confidential ledger instance.
- **Collection ID** (optional): The collection where you want to add the entry.
- **Entry Contents**: The data to be stored in the ledger entry (string format).

**Returns:**

- **Collection ID**: The collection where the entry was stored.
- **Transaction ID**: Unique identifier for the transaction (returned in the response header `x-ms-ccf-transaction-id`).
:::image type="content" source="./media/power-automate/write-ledger-entry.png" alt-text="Screenshot of the Power Automate workflow that shows the Write ledger entry action." lightbox="./media/power-automate/write-ledger-entry.png":::

### Get a ledger entry

Get a ledger entry by its transaction ID.

**Operation ID**: `GetLedgerEntry`

**Parameters:**

- **Ledger Name**: The name of your confidential ledger instance.
- **Transaction ID**: The transaction ID of the entry to retrieve.
- **Collection ID** (optional): The collection ID from which to fetch the value.

**Returns:**

- **State**: The query state is either **Loading** or **Ready**.
- **Entry**: The ledger entry data (available only if the state is **Ready**).
  - **Contents**: Contents of the ledger entry.
  - **Collection ID**: The collection ID to which the entries belong.
  - **Transaction ID**: The transaction ID.

:::image type="content" source="./media/power-automate/get-ledger-entry-by-transaction-id.png" alt-text="Screenshot of the Power Automate workflow that shows the Get Ledger Entry by Transaction ID action." lightbox="./media/power-automate/get-ledger-entry-by-transaction-id.png":::

### Get the current ledger entry

Get the most recent ledger entry from a collection.

**Operation ID**: `GetCurrentLedgerEntry`

**Parameters:**

- **Ledger Name**: The name of your confidential ledger instance.
- **Collection ID** (optional): The collection ID that corresponds to the entry.

**Returns:**

- **Contents**: Contents of the most recent ledger entry.
- **Collection ID**: The collection ID to which the entries belong.
- **Transaction ID**: The transaction ID of the current entry.

:::image type="content" source="./media/power-automate/get-current-ledger-entry.png" alt-text="Screenshot of the Power Automate workflow that shows the Get Current Ledger Entry action." lightbox="./media/power-automate/get-current-ledger-entry.png":::

### List ledger entries

Get ledger entries by collection and range.

**Operation ID**: `ListLedgerEntries`

**Parameters:**

- **Ledger Name**: The name of your confidential ledger instance.
- **Collection ID** (optional): The collection ID to which the entries belong.
- **From Transaction ID** (optional): Starting transaction ID in the range.
- **To Transaction ID** (optional): Ending transaction ID in the range.

**Returns:**

- **State**: The query state is either **Loading** or **Ready**.
- **Entries**: A collection of entries within the specified Transaction ID range.
- **Next Link**: A continuation link to retrieve the remaining entries.

:::image type="content" source="./media/power-automate/get-ledger-id-range.png" alt-text="Screenshot of the Power Automate workflow that shows the Get Ledger ID Range action." lightbox="./media/power-automate/get-ledger-id-range.png":::

### Get a receipt

Get a cryptographic receipt for a transaction by transaction ID.

**Operation ID**: `GetReceipt`

**Parameters:**

- **Ledger Name**: The name of your confidential ledger instance.
- **Transaction ID**: Transaction ID that corresponds to the receipt.

**Returns:**

- **State**: The query state is either **Loading** or **Ready**.
- **Transaction ID**: The transaction ID.
- **Receipt**: Cryptographic receipt containing:
  - **Node ID**: Identifier of the node that processed the transaction.
  - **Signature**: Digital signature.
  - **Proof**: Cryptographic proof elements.
  - **Certificate**: Node certificate.

:::image type="content" source="./media/power-automate/get-ledger-receipt.png" alt-text="Screenshot of the Power Automate workflow that shows the Get Receipt action." lightbox="./media/power-automate/get-ledger-receipt.png":::

### Get the transaction status

Get the status of a transaction by transaction ID.

**Operation ID**: `GetTransactionStatus`

**Parameters:**

- **Ledger Name**: The name of your confidential ledger instance.
- **Transaction ID**: The transaction ID to check.

**Returns:**

- **State**: Transaction state (**Committed** or **Pending**).
- **Transaction ID**: The transaction ID.

:::image type="content" source="./media/power-automate/get-ledger-transaction-status.png" alt-text="Screenshot of the Power Automate workflow that shows the Get Transaction Status action." lightbox="./media/power-automate/get-ledger-transaction-status.png":::

## Example workflow: Add an entry and store the transaction ID

The following section demonstrates how to use the connector to write a ledger entry.

### Scenario

Create a workflow that:

- Adds a new entry to confidential ledger.
- Stores the transaction ID in Azure Cosmos DB for reference.

### Workflow steps

1. **Trigger**: Choose your preferred trigger (manual, scheduled, or event-based).

2. **Create ledger entry action**:
   - **Ledger Name**: `your-ledger-name` (not the full URL, just the name).
   - **Collection ID**: `audit-logs` (optional: leave empty for default collection).
   - **Entry Contents**:

     ```
     {"content": "entry_data_here"}
     ```

3. **Parse JSON action** (to extract transaction ID from headers):
   - Use `outputs('Create_Ledger_Entry')['headers']['x-ms-ccf-transaction-id']` to get the transaction ID.

:::image type="content" source="./media/power-automate/power-automate-example-1.png" alt-text="Screenshot of the Power Automate workflow that shows the Create ledger entry action." lightbox="./media/power-automate/power-automate-example-1.png":::

4. **Store in Cosmos DB action**:
   - Use the parsed transaction ID from the previous step.
   - Store it along with relevant metadata for future reference.
   - For detailed information about the Cosmos DB connector, see [Azure Cosmos DB connector documentation](/connectors/documentdb/).
:::image type="content" source="./media/power-automate/power-automate-example-2.png" alt-text="Screenshot of the Power Automate workflow that shows the Cosmos DB action." lightbox="./media/power-automate/power-automate-example-2.png":::

### Example entry content formats

The entry content must be a string with a specific JSON structure that contains a "content" field. Here are some common patterns:

JSON as string:

```
{"content": "{\"event\": \"user_login\", \"oid\": \"12345\", \"timestamp\": \"@{utcNow()}\"}"}
```

Plain text content:

```
{"content": "User login event for user @{variables('oid')} at @{utcNow()}"}
```

Base64 encoded data:

```
{"content": "@{base64(variables('binaryData'))}"}
```

## Related content

- Learn more about [Azure confidential ledger concepts](overview.md).
- Explore [authentication options](authentication-azure-ad.md) for advanced scenarios.
- Review [best practices](secure-confidential-ledger.md) for production workflows.