---
title: Ledger Explorer concepts for Azure Confidential Ledger
description: Understand the Ledger Explorer tools for Azure Confidential Ledger, including the Azure portal experience and the offline forensics tool, and learn when to use each one.
author: shubhras
ms.author: shusinha
ms.service: azure-confidential-ledger
ms.topic: conceptual
ms.date: 04/23/2026
ai-usage: ai-assisted
---

# Ledger Explorer concepts

Azure Confidential Ledger provides two Ledger Explorer experiences for inspecting and verifying your ledger data. Both tools let you browse transactions and validate cryptographic proofs, but they serve different purposes and operate under different trust models.

This article explains what each tool is, what data it exposes, and when to use one versus the other. For step-by-step instructions, see the how-to guides linked in each section.

## Two Ledger Explorer experiences

| | Azure portal ledger explorer | Ledger Explorer (Offline) |
|---|---|---|
| **Access method** | Built into the Azure portal; works against a live ledger resource | Standalone web app; loads exported backup files via a SAS URL |
| **Connection** | Requires Azure authentication (Microsoft Entra ID) and a role assignment on the ledger resource | No Azure connection required; runs entirely in the browser |
| **Primary use case** | Day-to-day transaction browsing, creating ledger entries, and quick integrity checks | Forensic audits, regulatory investigations, and high-assurance verification |
| **Can create entries?** | Yes (with Contributor or Administrator role) | No (read-only) |
| **Data views** | Transactions and cryptographic proofs | Transactions, governance history, public tables, internal tables, and cryptographic proofs |
| **How-to guide** | [Use the Azure portal ledger explorer](ledger-explorer.md) | [Inspect ledger data with Ledger Explorer (Offline)](ledger-explorer-offline.md) |

## Trust differences between the two tools

Both tools rely on the same cryptographic foundations that Azure Confidential Ledger provides — immutable append-only storage, Merkle tree proofs, and hardware-backed enclaves. For details on the general trust model, see [Azure confidential ledger architecture](architecture.md) and [Authenticating Azure confidential ledger nodes](authenticate-ledger-nodes.md).

The key difference is **where the data comes from** and **what intermediaries are involved**:

- **Portal explorer**: Reads live data from your Azure Confidential Ledger resource through the Azure portal. Trust relies on your Azure authentication, the portal infrastructure, and the live ledger service.

- **Offline explorer**: Reads data directly from exported ledger backup files. No external database, indexer, or Azure service is involved at inspection time. You can independently validate ledger integrity and transaction history from the files alone, making this the **strongest trust model** available for ledger inspection.

Choose the offline explorer when trust, depth, and independent verifiability are more important than convenience or real-time access — for example, during forensic investigations, regulatory audits, or when verifying Microsoft service claims.

## Data views in Ledger Explorer

The Ledger Explorer tools expose several layers of your ledger data. For information about how data is structured at the API level (Transaction IDs, Collection IDs, Tags), see [Data organization in Azure Confidential Ledger](data-organization.md).

### Transactions

Both explorers display an ordered list of transactions written to the ledger. You can inspect individual transaction payloads and their associated cryptographic proof data. In the portal explorer, transactions are filtered by collection; in the offline explorer, you see all transactions from the exported backup.

### Governance history

The offline explorer exposes governance tables that show how the ledger itself has been governed over time, including:

- Users added or removed
- Role assignments and changes
- Access control modifications

This historical governance data is permanently recorded in the ledger. It isn't available in the portal explorer.

### Internal tables

The offline explorer also exposes low-level bookkeeping structures that support deep forensic analysis and advanced technical investigations. Most users don't need these tables, but they're available when full visibility into ledger internals is required.

### Public tables

The offline explorer shows application-level data written into the ledger. Use this view to validate that ledger contents align with expected system behavior.

## Cryptographic verification

Both explorers support cryptographic proof verification, which confirms that a transaction was correctly committed to the ledger and hasn't been tampered with. The verification process uses Merkle tree proofs and digital signatures from the ledger's signing nodes.

For a detailed explanation of how receipt verification works, see:

- [Azure Confidential Ledger write transaction receipts](write-transaction-receipts.md)
- [Verify Azure Confidential Ledger write transaction receipts](verify-write-transaction-receipts.md)

The portal explorer walks you through the verification steps visually (leaf node computation, root node computation, and signature verification). The offline explorer lets you inspect proof data directly from the exported files.

## Positioning and scope

Ledger Explorer is a **verification and forensics toolset**. It prioritizes integrity, depth, and cryptographic proof over convenience. <!-- TODO: [PM] Add public citation for usage expectations or remove --> The offline tool in particular is expected to be used infrequently but critically, by users who need the highest assurance that their ledger data is intact.

Other tools and APIs may provide higher-level, indexed, or search-oriented experiences for Azure Confidential Ledger data, but those come with different trust trade-offs. When **verifiable trust** is the primary goal, the Ledger Explorer tools — especially the offline experience — remain the reference approach. <!-- TODO: [PM] Add public citation or remove -->

## Related content

- [Use the Azure portal ledger explorer to verify transactions](ledger-explorer.md)
- [Inspect Azure Confidential Ledger data with Ledger Explorer (Offline)](ledger-explorer-offline.md)
- [Azure Confidential Ledger write transaction receipts](write-transaction-receipts.md)
- [Verify Azure Confidential Ledger write transaction receipts](verify-write-transaction-receipts.md)
- [Data organization in Azure Confidential Ledger](data-organization.md)
- [Overview of Microsoft Azure confidential ledger](overview.md)
