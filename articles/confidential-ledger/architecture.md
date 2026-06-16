---
title: Architecture
description: Architecture of Azure confidential ledger, the shared confidential computing platform that underlies Confidential Computing Ledger workloads.
services: confidential-ledger
author: msmbaldwin
ms.service: azure-confidential-ledger
ms.topic: concept-article
ms.date: 04/14/2025
ms.author: mbaldwin
ai-usage: ai-assisted
---

# Architecture

This article describes the architecture of the confidential computing platform that underlies both [Confidential Computing Ledger](index.yml) workloads: [Azure confidential ledger](overview.md) and [Microsoft's Signing Transparency Ledger](about-microsoft-signing-transparency-ledger.md).

The platform is a REST API service that lets clients interact with the ledger through administrative and functional API calls. When data is recorded to the ledger, it's sent to permissioned blockchain nodes that are replicas backed by hardware-based secure enclaves. The replicas follow a consensus protocol. Clients can also retrieve receipts for data that was committed to the ledger.

## Architecture diagram

This image provides an architectural overview of Azure confidential ledger, showing users interacting with the cloud APIs for a created ledger.

:::image type="content" source="./media/architecture-overview.png" alt-text="Architectural overview of Azure confidential ledger, showing client interaction with cloud APIs and the underlying enclave-backed replica nodes.":::

## Next steps

- [About Azure confidential ledger](overview.md)
- [About Microsoft's Signing Transparency Ledger](about-microsoft-signing-transparency-ledger.md)
- [Data residency for Azure confidential ledger](data-residency.md)
- [Authenticate Azure confidential ledger nodes](authenticate-ledger-nodes.md)
- [Azure confidential ledger write transaction receipts](write-transaction-receipts.md)
