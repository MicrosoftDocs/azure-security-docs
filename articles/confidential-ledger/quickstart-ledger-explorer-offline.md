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

The web-based offline ledger explorer is a **forensics-style investigation tool** that allows you to inspect Azure Confidential Ledger data directly from exported ledger files. It's designed for **high-fidelity verification and trust validation**, primarily for auditing and compliance purposes.

> [!NOTE]
> This article covers the standalone offline Ledger Explorer for exported data. To verify transactions directly in the Azure portal, see [Use the Azure portal ledger explorer to verify transactions](quickstart-ledger-explorer.md).

## Prerequisites

Before you begin, make sure you have:

- An Azure Confidential Ledger instance.
- Exported ledger backup files.
- Access to the exported files using a SAS token.
- Access to the [Ledger Explorer web experience](https://microsoft.github.io/Azure-Ledger-Explorer/).

## Open the web-based explorer

1. Open the [Ledger Explorer web experience](https://microsoft.github.io/Azure-Ledger-Explorer/).
1. Provide the SAS URL that points to your exported ledger backup files.
1. Load the ledger files into the explorer.

Ledger Explorer reads the ledger data **directly from the exported files**, preserving the original trust guarantees of the ledger.

## Explore ledger data

Once the ledger is loaded, Ledger Explorer presents multiple views that expose different layers of the ledger.

### Transaction list

- Displays an ordered list of all transactions written to the ledger.
- Each entry represents an immutable ledger write.
- Select a transaction to view detailed information.

### Transaction details

For a selected transaction, you can inspect:

- The full transaction payload.
- Cryptographic proof data.
- Merkle tree proof that links the transaction into the ledger state.

These proofs are derived directly from the ledger data and enable independent verification without relying on any external indexing service.

### Public tables

Public tables expose application-level data written into the ledger.

- View business data recorded in the ledger.
- Validate that ledger contents align with expected system behavior.

### Governance tables

Governance tables expose **historical governance operations**, including:

- Users added or removed.
- Role assignments and role changes.
- Access mapping changes over time.

This historical governance visibility is written permanently to the ledger and isn't available in the Azure portal.

### Internal tables

Internal tables contain low-level bookkeeping data used to maintain ledger correctness.

- Intended for deep technical or forensic investigation.
- Not required for most validation scenarios.
- Exposed to provide maximum transparency.

## Related content

- [Use the Azure portal ledger explorer to verify transactions](quickstart-ledger-explorer.md)
- [Azure Confidential Ledger write transaction receipts](write-transaction-receipts.md)
- [Verify Azure Confidential Ledger write transaction receipts](verify-write-transaction-receipts.md)
- [Overview of Microsoft Azure confidential ledger](overview.md)
- [Provide feedback by taking a quick survey](https://aka.ms/ledger-explorer-survey)
