---
title: Verify ledger integrity and inspect entries in Microsoft's Signing Transparency Ledger
titleSuffix: Azure Confidential Ledger
description: Learn how to verify the integrity of Microsoft's Signing Transparency Ledger and inspect recorded entries by using the ledger explorer or CCF scripts.
services: confidential-ledger
author: msmbaldwin
ms.service: azure-confidential-ledger
ms.topic: how-to
ai-usage: ai-assisted
ms.date: 06/10/2026
ms.author: mbaldwin
---

# Verify ledger integrity and inspect entries in Microsoft's Signing Transparency Ledger

This article shows you how to verify the integrity of Microsoft's Signing Transparency Ledger (MST) and inspect recorded entries by using the ledger explorer or CCF scripts. The steps confirm that the ledger's transaction history is intact and let you search for and review the entries an integrated service has registered.

Verifying an individual signature or transparent statement from a service is a separate, auditor-side workflow that uses a receipt issued by MST when a payload is registered. For background, see [Transparent statements](microsoft-signing-transparency-concepts.md#transparent-statements) and [Receipts and inclusion proofs](microsoft-signing-transparency-concepts.md#receipts-and-inclusion-proofs).

## Prerequisites

- Internet access to reach the MST ledger explorer
- Familiarity with [Signing Transparency concepts](microsoft-signing-transparency-concepts.md)
- (Optional) Python with CCF utilities installed for manual verification

## Access the MST ledger explorer

The MST standalone ledger explorer is a web interface for interacting with the MST ledger and verifying software artifact transparency. It's available at <https://aka.ms/ledgerexplorer>.

> [!NOTE]
> The ledger explorer is also open source. You can run it locally from the [scitt-ccf-ledger explorer source](https://github.com/microsoft/scitt-ccf-ledger) to verify signatures independently, without relying on the Microsoft-hosted instance.

> [!NOTE]
> Alternatively, you can manually download ledger files and use CCF scripts for verification. For more information, see [Option 2: Manual download and CCF scripts](#option-2-manual-download-and-ccf-scripts).

## Option 1: Verify MAA payloads by using the ledger explorer

This example demonstrates verification by using Microsoft Azure Attestation (MAA) payloads. The same process applies to other services with different identifiers.

### Step 1: Open the ledger explorer

1. Go to <https://aka.ms/ledgerexplorer>.
1. Go to the **Files** page.
1. If files are already present, go to **Configuration** and select **Clear all data**.

### Step 2: Import ledger data

1. Select **Add files** and select the **Microsoft's Signing Transparency** tab.
1. Enter your MST ledger domain name:
   - US datacenter: `esrp-cts-cp.confidential-ledger.azure.com`
   - European datacenter: `esrp-cts-db.confidential-ledger.azure.com`
   
   > [!NOTE]
   > These two datacenters are different ledgers; the data doesn't overlap between them.

1. Select **Import**.
1. Select the latest file to download all ledger files up to and including the selected file.

### Step 3: Verify ledger integrity

1. Open the **Ledger Verification** page.
1. Select **Start Verification**.
1. The explorer parses entries and verifies data integrity.
1. Wait for the verification process to complete.

### Step 4: Search for payloads

1. Open the **Tables** page.
1. Choose **public:scitt.entry** in the navigation pane.
1. Use the free text filter to search for the identifier of the service you want to inspect. For the list of service identifiers, see [Service identifiers for supported services](microsoft-signing-transparency-concepts.md#service-identifiers-for-supported-services).
1. The tool filters and displays all submissions matching your identifier.

### Step 5: Review payload details

1. Select an entry to view its details.
1. Examine the payload contents, including:
   - Signing event metadata
   - Countersignature
   - Merkle proof of inclusion
1. Verify the entry hasn't been tampered with.

### Explore additional information

The ledger explorer provides access to additional tables for deeper investigation:

- **public:ccf.gov.scitt.configuration**: View acceptance policies.
- **public:ccf.gov.proposals**: See operator actions.
- **public:ccf.gov.members.info**: Check member identities.
- **public:ccf.gov.nodes.info**: View node information.

## Option 2: Manual download and CCF scripts

For advanced users or automated verification, you can manually download ledger files and use CCF scripts.

### Prerequisites

- Python installed on your system
- CCF audit Python library installed

### Step 1: Install CCF utilities

Follow the documentation at the [CCF audit Python library](https://microsoft.github.io/CCF/main/audit/python_library.html) to install the necessary tools.

### Step 2: Download ledger files

1. Construct the URL by appending `ledger-files-` to your ledger domain:
   - Example: `ledger-files-esrp-cts-cp.confidential-ledger.azure.com`

1. Download files one by one from the `ledger` directory to your local machine.

> [!NOTE]
> No automated download script is provided. You must write your own script if you need to download files in bulk.

### Step 3: Verify ledger integrity

Run the following command to verify the ledger's integrity:

```bash
read_ledger.py /path/to/ledger/dir
```

The tool confirms when ledger verification is complete.

### Step 4: Parse and search entries

Use CCF tools to parse the ledger and search for specific entries by using EKU or Subject identifiers relevant to your service. For the list of service identifiers, see [Service identifiers for supported services](microsoft-signing-transparency-concepts.md#service-identifiers-for-supported-services).

## Understanding verification results

A successful verification confirms that:

- The signature is recorded in the immutable ledger.
- The entry hasn't been tampered with since recording.
- The cryptographic [proof of inclusion](microsoft-signing-transparency-concepts.md#receipts-and-inclusion-proofs) is valid.
- The signing event follows the service's trust policy.

If you encounter issues, see [Troubleshoot Microsoft's Signing Transparency Ledger](microsoft-signing-transparency-troubleshoot.md).

## Next steps

- [Learn about Signing Transparency concepts](microsoft-signing-transparency-concepts.md)
- [Usage: payloads, claims, and auditing](microsoft-signing-transparency-usage.md)
- [About Microsoft's Signing Transparency Ledger](about-microsoft-signing-transparency-ledger.md)
- [Explore Azure confidential ledger](overview.md)
- [Connect directly to MST using SCITT client tools](https://github.com/microsoft/scitt-ccf-ledger/blob/main/docs/clients.md) — retrieve entries and receipts, verify receipts, and call the public MST API directly.
