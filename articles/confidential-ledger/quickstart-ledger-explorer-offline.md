---
title: Inspect Azure Confidential Ledger data with Ledger Explorer (Offline)
description: Learn how to load exported Azure Confidential Ledger data into Ledger Explorer (Offline) and inspect transactions, governance history, and cryptographic proofs.
author: shubhras
ms.author: shubhras
ms.service: azure-confidential-ledger
ms.topic: quickstart
ms.date: 04/23/2026
ai-usage: ai-assisted
---

# Quickstart: Inspect Azure Confidential Ledger data with Ledger Explorer (Offline)

In this quickstart, you load exported Azure Confidential Ledger data into the offline Ledger Explorer and inspect transactions, governance history, and cryptographic proofs. The offline Ledger Explorer is a web-based forensics tool designed for high-fidelity verification and trust validation, primarily for auditing and compliance purposes.

The offline Ledger Explorer reads ledger data directly from exported backup files and runs entirely in your browser. No data is sent to any external server.

> [!NOTE]
> This article covers the standalone offline Ledger Explorer for exported data. To verify transactions directly from your Azure portal ledger resource, see [Use the Azure portal ledger explorer to verify transactions](quickstart-ledger-explorer.md).

[!INCLUDE [quickstarts-free-trial-note](~/reusable-content/ce-skilling/azure/includes/quickstarts-free-trial-note.md)]

## Prerequisites

- An Azure subscription - [create one for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- An Azure Confidential Ledger instance. If you don't have one, see [Quickstart: Create a confidential ledger using the Azure portal](quickstart-portal.md).
- Exported ledger backup files stored in Azure Blob Storage. To export your ledger data, use the backup functionality available through the Azure Confidential Ledger REST API or the Azure portal.
- A SAS (Shared Access Signature) URL that provides read access to the exported backup files.

## Open Ledger Explorer and load your data

1. Go to the [Ledger Explorer web experience](https://microsoft.github.io/Azure-Ledger-Explorer/).
1. In the connection dialog, paste the SAS URL that points to your exported ledger backup files.
1. Select **Load** to import the ledger data into the explorer.

The explorer parses the exported files and presents the ledger data in several views. This process might take a few moments depending on the size of your ledger.

## Browse and inspect transactions

1. Select **Transactions** to view an ordered list of all transactions written to the ledger. Each entry represents an immutable ledger write.
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

This verification confirms that the transaction data hasn't been tampered with since it was committed to the ledger. For more information about how receipt verification works, see [Azure Confidential Ledger write transaction receipts](write-transaction-receipts.md).

## Clean up resources

The offline Ledger Explorer runs entirely in your browser. No server-side resources are created. To clear any locally cached data, go to **Configuration** in the explorer and clear the browser cache.

If you created Azure resources as part of this quickstart and no longer need them, delete the resource group to avoid incurring further charges.

## Related content

In this quickstart, you loaded exported Azure Confidential Ledger data into the offline Ledger Explorer and inspected transactions, governance history, and cryptographic proofs.

To learn more, see the following resources:

- [Use the Azure portal ledger explorer to verify transactions](quickstart-ledger-explorer.md)
- [Azure Confidential Ledger write transaction receipts](write-transaction-receipts.md)
- [Verify Azure Confidential Ledger write transaction receipts](verify-write-transaction-receipts.md)
- [Overview of Microsoft Azure confidential ledger](overview.md)
- [Provide feedback by taking a quick survey](https://aka.ms/ledger-explorer-survey)
