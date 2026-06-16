---
title: Inspect Azure confidential ledger data with Ledger Explorer (Offline)
description: Learn how to use the Ledger Explorer (Offline) web experience to load Azure confidential ledger backup data or local ledger files and inspect transactions, governance history, and cryptographic proofs.
author: shubhras
ms.author: mbaldwin
ms.service: azure-confidential-ledger
ms.topic: how-to
ms.date: 04/23/2026
ai-usage: ai-assisted
---

# Inspect Azure confidential ledger data with Ledger Explorer (Offline)

This article shows you how to use the [Ledger Explorer (Offline)](https://microsoft.github.io/Azure-Ledger-Explorer/) web experience to inspect transactions, governance history, and cryptographic proofs from your Azure confidential ledger data. Ledger Explorer (Offline) runs entirely in your browser — no data is sent to any external server.

Ledger Explorer (Offline) provides two ways to load your ledger data:

- **Azure Ledger Backup** — Connect to exported ledger backup files stored in Azure Blob Storage using a SAS URL.
- **Audit Ledger Files** — Upload local CCF ledger files (`.committed` files) directly from your machine for local audit.

For background on how Ledger Explorer (Offline) compares to the Azure portal ledger explorer, see [Ledger Explorer concepts](ledger-explorer-concepts.md).

> [!NOTE]
> This article covers the Ledger Explorer (Offline) web experience. To verify transactions directly from your Azure portal ledger resource, see [Use the Azure portal ledger explorer to verify transactions](ledger-explorer.md).

## Prerequisites

For the **Azure Ledger Backup** option:

- An Azure subscription - [create one for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- An Azure confidential ledger instance. If you don't have one, see [Quickstart: Create a confidential ledger using the Azure portal](quickstart-portal.md).
- Exported ledger backup files stored in Azure Blob Storage. To export your ledger data, use the backup functionality available through the Azure confidential ledger REST API or the Azure portal.
- A SAS (Shared Access Signature) URL that provides read access to the exported backup files.

For the **Audit Ledger Files** option:

- CCF ledger files in `.committed` format from your local system.

## Load data from Azure Ledger Backup

Use this option when you have exported ledger backup files stored in Azure Blob Storage.

1. Go to [Ledger Explorer (Offline)](https://microsoft.github.io/Azure-Ledger-Explorer/).
1. Select **Azure Ledger Back Up**.

   <!-- TODO: [PM] Add screenshot of the Azure Ledger Backup landing page -->

1. Paste the SAS URL that points to your exported ledger backup files.
1. Select **Load** to import the ledger data into the explorer.

The explorer parses the exported files and presents the ledger data in several views. This process might take a few moments depending on the size of your ledger.

## Load data from local ledger files

Use this option when you have CCF ledger files on your local machine and want to audit them without connecting to Azure.

1. Go to [Ledger Explorer (Offline)](https://microsoft.github.io/Azure-Ledger-Explorer/).
1. Select **Audit Ledger Files**.

   <!-- TODO: [PM] Add screenshot of the Audit Ledger Files landing page -->

1. Upload your `.committed` files using one of these methods:
   - Drag and drop files into the upload area.
   - Select the upload button to browse and select files.
   - You can upload multiple files at once.

Your ledger files must meet these requirements:

- **File extension**: Only `.committed` files are accepted.
- **Naming format**: Files must follow the pattern `ledger_<start>-<end>.committed` (for example, `ledger_1-18.committed`, `ledger_19-25.committed`).
- **Sequence**: File ranges must start at 1, with no gaps or overlaps between files.

After upload, the files are validated, parsed, and stored locally in your browser for fast searching. No data leaves your machine.

## Browse and inspect transactions

After loading data (from either method), the explorer presents the ledger data in several views.

1. Select **Transactions** to view an ordered list of all transactions written to the ledger. Each entry represents an immutable ledger write.

   <!-- TODO: [PM] Add screenshot of the transactions list view -->

1. Select a transaction row to open its details. For the selected transaction, you can inspect:
   - The full transaction payload.
   - Cryptographic proof data.
   - The Merkle tree proof that links the transaction into the ledger state.

These proofs are derived directly from the ledger data and enable independent verification without relying on any external indexing service.

## Inspect governance history

1. Select **Governance tables** to view historical governance operations recorded in the ledger. Governance tables expose:
   - Users added or removed.
   - Role assignments and role changes.
   - Access mapping changes over time.

This historical governance visibility is written permanently to the ledger and isn't available in the Azure portal. Use this view to audit who had access to the ledger and when changes were made.

## Review public and internal tables

1. Select **Public tables** to view application-level data written into the ledger. Use this view to validate that ledger contents align with expected system behavior.

1. Select **Internal tables** to view low-level bookkeeping data used to maintain ledger correctness. Internal tables are intended for deep technical or forensic investigation and aren't required for most validation scenarios.

## Verify cryptographic proofs

For any transaction, you can verify the cryptographic proof chain to confirm that the transaction was correctly committed to the ledger:

1. Select a transaction from the transaction list.
1. Open the proof details for that transaction.
1. Verify that the Merkle tree proof links the transaction to the ledger root. The explorer displays the proof path and the computed root hash.

This verification confirms that the transaction data hasn't been tampered with since it was committed to the ledger. For more information about how receipt verification works, see [Azure confidential ledger write transaction receipts](write-transaction-receipts.md).

## Related content

- [Ledger Explorer concepts](ledger-explorer-concepts.md)
- [Use the Azure portal ledger explorer to verify transactions](ledger-explorer.md)
- [Azure confidential ledger write transaction receipts](write-transaction-receipts.md)
- [Verify Azure confidential ledger write transaction receipts](verify-write-transaction-receipts.md)
- [Overview of Microsoft Azure confidential ledger](overview.md)
- [Ledger Explorer (Offline) on GitHub](https://microsoft.github.io/Azure-Ledger-Explorer/)
- [Provide feedback by taking a quick survey](https://aka.ms/ledger-explorer-survey)
