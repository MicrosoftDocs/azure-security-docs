---
title: Ledger Explorer concepts for Azure confidential ledger
description: Understand the Ledger Explorer tools for Azure confidential ledger, including the Azure portal experience and the Ledger Explorer (Offline) web tool, and learn when to use each one.
author: shubhras
ms.author: shusinha
ms.service: azure-confidential-ledger
ms.topic: conceptual-article
ms.date: 04/23/2026
ai-usage: ai-assisted
---

# Ledger Explorer concepts

Azure confidential ledger provides two Ledger Explorer experiences for inspecting and verifying your ledger data. Both tools let you browse transactions and validate cryptographic proofs, but they serve different purposes and operate under different trust models.

This article explains what each tool is, what data it exposes, and when to use one versus the other. For step-by-step instructions, see the how-to guides linked in each section.

## Two Ledger Explorer experiences

| | Azure portal ledger explorer | Ledger Explorer (Offline) |
|---|---|---|
| **What it is** | Built into the Azure portal; works against a live ledger resource | Standalone web app ([Ledger Explorer (Offline)](https://microsoft.github.io/Azure-Ledger-Explorer/)); runs entirely in the browser |
| **Data sources** | Live ledger resource via Azure authentication | Two options: **Azure Ledger Backup** (SAS URL to exported backup files) or **Audit Ledger Files** (upload local `.committed` files) |
| **Connection** | Requires Microsoft Entra ID and a role assignment on the ledger resource | No Azure connection required at inspection time |
| **Primary use case** | Day-to-day transaction browsing, creating ledger entries, and quick integrity checks | Forensic audits, regulatory investigations, and high-assurance verification |
| **Can create entries?** | Yes (with Contributor or Administrator role) | No (read-only) |
| **Data views** | Transactions and cryptographic proofs | Transactions, governance history, public tables, internal tables, and cryptographic proofs |
| **How-to guide** | [Use the Azure portal ledger explorer](ledger-explorer.md) | [Inspect ledger data with Ledger Explorer (Offline)](ledger-explorer-offline.md) |

### Ledger Explorer (Offline) data loading options

The Ledger Explorer (Offline) offers two ways to load data:

- **Azure Ledger Backup**: Connect to exported ledger backup files stored in Azure Blob Storage using a SAS URL. Use this when you've created a backup of your Azure confidential ledger and want to inspect it independently of the live service.
- **Audit Ledger Files**: Upload CCF ledger files (`.committed` files) directly from your local machine. Use this for auditing any CCF-based ledger data locally without any Azure connection.

## Trust differences between the two tools

Both tools rely on the same cryptographic foundations that Azure confidential ledger provides — immutable append-only storage, Merkle tree proofs, and hardware-backed enclaves. For details on the general trust model, see [Azure confidential ledger architecture](architecture.md) and [Authenticating Azure confidential ledger nodes](authenticate-ledger-nodes.md).

The key difference is **where the data comes from** and **what intermediaries are involved**:

- **Portal explorer**: Reads live data from your Azure confidential ledger resource through the Azure portal. Trust relies on your Azure authentication, the portal infrastructure, and the live ledger service.

- **Ledger Explorer (Offline)**: Reads data directly from exported ledger backup files or uploaded local ledger files. No external database, indexer, or Azure service is involved at inspection time. You can independently validate ledger integrity and transaction history from the files alone, making this the **strongest trust model** available for ledger inspection.

Choose the Ledger Explorer (Offline) when trust, depth, and independent verifiability are more important than convenience or real-time access — for example, during forensic investigations, regulatory audits, or when verifying Microsoft service claims.

## Data views in Ledger Explorer

The Ledger Explorer tools expose several layers of your ledger data. For information about how data is structured at the API level (Transaction IDs, Collection IDs, Tags), see [Data organization in Azure confidential ledger](data-organization.md).

### Transactions

Both explorers display an ordered list of transactions written to the ledger. You can inspect individual transaction payloads and their associated cryptographic proof data. In the portal explorer, transactions are filtered by collection; in the Ledger Explorer (Offline), you see all transactions from the loaded data.

### Governance history

The Ledger Explorer (Offline) exposes governance tables that show how the ledger itself has been governed over time, including:

- Users added or removed
- Role assignments and changes
- Access control modifications

This historical governance data is permanently recorded in the ledger. It isn't available in the portal explorer.

### Internal tables

The Ledger Explorer (Offline) also exposes low-level bookkeeping structures that support deep forensic analysis and advanced technical investigations. Most users don't need these tables, but they're available when full visibility into ledger internals is required.

### Public tables

The Ledger Explorer (Offline) shows application-level data written into the ledger. Use this view to validate that ledger contents align with expected system behavior.

## Cryptographic verification

Both explorers support cryptographic proof verification, which confirms that a transaction was correctly committed to the ledger and hasn't been tampered with. The verification process uses Merkle tree proofs and digital signatures from the ledger's signing nodes.

For a detailed explanation of how receipt verification works, see:

- [Azure confidential ledger write transaction receipts](write-transaction-receipts.md)
- [Verify Azure confidential ledger write transaction receipts](verify-write-transaction-receipts.md)

The portal explorer walks you through the verification steps visually (leaf node computation, root node computation, and signature verification). The Ledger Explorer (Offline) lets you inspect proof data directly from the loaded files.

## Positioning and scope

Ledger Explorer is a **verification and forensics toolset**. It prioritizes integrity, depth, and cryptographic proof over convenience. <!-- TODO: [PM] Add public citation for usage expectations or remove --> The Ledger Explorer (Offline) in particular is expected to be used infrequently but critically, by users who need the highest assurance that their ledger data is intact.

Other tools and APIs may provide higher-level, indexed, or search-oriented experiences for Azure confidential ledger data, but those come with different trust trade-offs. When **verifiable trust** is the primary goal, the Ledger Explorer tools — especially the Ledger Explorer (Offline) — remain the reference approach. <!-- TODO: [PM] Add public citation or remove -->

## Related content

- [Use the Azure portal ledger explorer to verify transactions](ledger-explorer.md)
- [Inspect ledger data with Ledger Explorer (Offline)](ledger-explorer-offline.md)
- [Azure confidential ledger write transaction receipts](write-transaction-receipts.md)
- [Verify Azure confidential ledger write transaction receipts](verify-write-transaction-receipts.md)
- [Data organization in Azure confidential ledger](data-organization.md)
- [Overview of Microsoft Azure confidential ledger](overview.md)
