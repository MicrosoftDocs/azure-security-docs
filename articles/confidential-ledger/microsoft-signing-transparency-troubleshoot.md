---
title: Troubleshoot Microsoft's Signing Transparency Ledger in Azure confidential ledger
description: Find solutions to common issues when verifying signatures in Microsoft's Signing Transparency Ledger.
services: confidential-ledger
author: msmbaldwin
ms.service: azure-confidential-ledger
ms.topic: troubleshooting
ai-usage: ai-assisted
ms.date: 06/10/2026
ms.author: mbaldwin
---

# Troubleshoot Microsoft's Signing Transparency Ledger

This article helps you diagnose and resolve common issues when working with Microsoft's Signing Transparency Ledger.

## SCITT SCRAPI status codes

For the table of HTTP status codes returned by the transparency service in response to SCRAPI requests, see [SCITT SCRAPI status codes](microsoft-signing-transparency-concepts.md#scitt-scrapi-status-codes) in the concepts article.

## Recent signing events don't appear in the ledger

If you don't see a recent signing event in the ledger explorer, or the latest submissions are missing from query results, this is expected behavior. Publicly accessible ledger data synchronizes at regular intervals and might not contain the very latest submissions immediately after they're recorded.

Wait a few minutes and re-import the ledger data in the explorer. The synchronization process typically completes within minutes, but can take up to 24 hours in some cases.

## Search returns no results for expected entries

If a search in the ledger explorer returns no results, verify:

1. **Identifier**: You're using the correct EKU or Subject identifier for your service. See [EKU and Subject identifiers](microsoft-signing-transparency-concepts.md#eku-and-subject-identifiers).
1. **Ledger domain**: You're connected to the correct datacenter. For datacenter URLs, see [Step 2: Import ledger data](microsoft-signing-transparency-verify-signatures.md#step-2-import-ledger-data).
1. **Search syntax**: Your search query uses the correct format in the explorer's free text filter.

## Verification process fails with errors

If the ledger verification process fails when using the ledger explorer or CCF scripts:

1. **Re-download ledger files**: Delete existing ledger files and download them again from the ledger domain.
1. **Update CCF utilities**: Install the latest [CCF audit Python library](https://microsoft.github.io/CCF/main/audit/python_library.html).
1. **Verify file completeness**: Confirm you downloaded all ledger files, not just a subset. Verification requires the complete chain.
1. **Check dependencies**: Confirm all required Python packages and system dependencies are installed.

## Clear ledger explorer data

To start fresh or clear cached data, go to **Configuration** in the ledger explorer and select **Clear all data**. The explorer runs in your browser, so clearing data only affects your local cache; the verification process is read-only and doesn't require deleting any Azure resources.

## Next steps

- [Verify ledger integrity and inspect entries](microsoft-signing-transparency-verify-signatures.md)
- [Microsoft's Signing Transparency Ledger concepts](microsoft-signing-transparency-concepts.md)
- [Usage: payloads, claims, and auditing](microsoft-signing-transparency-usage.md)
- [About Microsoft's Signing Transparency Ledger](about-microsoft-signing-transparency-ledger.md)
