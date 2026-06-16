---
title: About Azure confidential ledger
description: Learn about Azure confidential ledger, a workload of Confidential Computing Ledger — a customer-managed, append-only ledger for tamper-evident records of sensitive transactions.
services: confidential-ledger
author: msmbaldwin
ms.service: azure-confidential-ledger
ms.topic: overview
ms.date: 04/23/2026
ms.author: mbaldwin
ai-usage: ai-assisted
---

# About Azure confidential ledger

Azure confidential ledger is a workload of [Confidential Computing Ledger](index.yml). It's a customer-managed, append-only ledger you create in your own subscription and write records to directly. Azure confidential ledger offers an auditable data store with unique data integrity advantages, including immutability, tamper-evident records, and append-only operations. It combines cryptographic techniques and blockchain technology built on the shared Azure confidential ledger platform.

These features are ideal when critical metadata records must have their integrity protected, such as for regulatory compliance and archival purposes. Data stored in Azure confidential ledger remains privacy enhanced and protected from insider threats within an organization, including the cloud provider. It's also beneficial as a repository of audit trails or records that need to be protected and selectively shared with certain personas (for instance, auditors).

Azure confidential ledger can protect existing databases and applications by acting as a point-in-time source of truth for digests and hashes. Each transaction provides cryptographic proofs in verification scenarios. For example, your Azure SQL data can be integrity protected further, where table digests or logs can be stored in Azure confidential ledger.

For more information, you can learn about [protecting data source integrity with Azure confidential ledger](https://www.youtube.com/watch?v=lJSn46id-64) or watch an [Azure confidential ledger demo](https://www.youtube.com/watch?v=Cg0-5moftP0). You can also read a recent blog on how [Azure hardware security is protected via Azure confidential ledger](https://azure.microsoft.com/blog/microsoft-azure-confidential-ledger-enhancing-customer-trust-in-azures-hardware-supply-chain/).

:::image type="content" source="./media/use-cases.png" alt-text="Diagram of use cases for Azure confidential ledger." lightbox="media/use-cases.png":::

## What to store

Here are a few examples of things you can store in an Azure confidential ledger instance:

- Records relating to your business transactions (for example, money transfers or confidential document edits).
- Updates to trusted assets (for example, core applications or contracts).
- Administrative and control changes (for example, granting access permissions).
- Operational IT and security events (for example, Microsoft Defender for Cloud alerts).

## Use cases

- I have *relational data* that requires end-to-end data integrity guarantees: Store your data in the [Azure SQL Database ledger feature](/sql/relational-databases/security/ledger/ledger-overview) and turn on Azure confidential ledger as your Trusted Digest store.
- I have *blob data* that needs end-to-end integrity: Store your data in Azure Blob Storage and configure the [Azure Marketplace application backed by Azure confidential ledger](https://azuremarketplace.microsoft.com/marketplace/apps/azureconfidentialledger.acl-blob-storage?tab=Overview) to [store signatures and verify against](create-blob-managed-app.md?tabs=azure-portal).
- I have *system records* that need integrity protection with verifiability: Store your records in Azure confidential ledger directly. For example, have all your development records go to one instance and have your production logs go to another instance. When you want to audit, only selectively share transactions with the auditor.
- I have *confidential transactional data* that needs confidentiality and integrity protection: Store your critical confidential data's application records in Azure confidential ledger directly.

## Enable data integrity for data sources

SQL databases and storage systems are foundational to enterprise data architecture. Azure confidential ledger enhances these systems by providing an extra layer of integrity protection. [For SQL databases](/sql/relational-databases/security/ledger/ledger-overview), Azure confidential ledger can act as an external ledger where changes and transactions are recorded and verified, which adds a new dimension of security and trust.

[For Blob Storage](create-blob-managed-app.md), Azure confidential ledger enhances security features by providing an immutable log of storage operations. This log is valuable for regulatory compliance and archival purposes where data integrity over time is crucial.

## Key features

Azure confidential ledger exposes a REST interface, which makes it easier to integrate with new or existing applications. SDKs in popular languages like .NET, Java, Python, and JavaScript help with the integration.

Azure confidential ledger supports collection ID for easy data management. Grouping data by using collection IDs is a great way to manage and query data efficiently. It allows for easy identification and retrieval of specific datasets. This method can significantly enhance data organization and make operations like searching and updating more streamlined.

Each transaction on the ledger has an associated receipt that records the Merkle tree data structure, which is used to [verify the transaction's integrity](verify-write-transaction-receipts.md). Read more about how you can [verify transaction receipts](write-transaction-receipts.md).

## Confidential computing foundations

Azure confidential ledger runs exclusively on hardware-backed secure enclaves. This heavily monitored and isolated runtime environment keeps potential attacks at bay. The service runs on a minimal Trusted Computing Base (TCB) that ensures no one — not even Microsoft — is "above" the ledger.

As the name suggests, the service uses the [Azure confidential computing platform](/azure/confidential-computing) and the [Confidential Consortium Framework](https://www.microsoft.com/research/project/confidential-consortium-framework) to provide a high-integrity solution that's tamper protected and tamper evident. One ledger spans three or more identical instances. Each instance runs in a dedicated, fully attested hardware-backed enclave. The integrity of each ledger is maintained through a consensus-based blockchain.

These confidential computing foundations are shared with the other Confidential Computing Ledger workload, [Microsoft's Signing Transparency Ledger](about-microsoft-signing-transparency-ledger.md). For a detailed view of the underlying components, see [Architecture](architecture.md).

## Data storage

Azure confidential ledger data is written in blocks that are chained together and stored in Azure-backed file storage. Transaction data can either be stored encrypted (for example, private ledger type) or in plain text (for example, public ledger type) depending on your needs.

Administrators can create and manage Azure confidential ledger instances with administrative APIs (control plane), for example, to delete a resource or move it across resource groups. Azure confidential ledger provides functional APIs (data plane) for data operations such as CREATE, UPDATE, PUT, and GET.

## Ledger security

Azure confidential ledger supports both Microsoft Entra ID and certificate-based credentials for authentication, with custom role-based access control (RBAC) for authorization. Unlike other Azure services, user management is localized. In other words, users are stored and managed within the ledger by using the functional APIs. This design reduces the TCB and eliminates the need to rely on external authorization systems, such as Azure RBAC.

Azure confidential ledger uses the TLS 1.3 protocol to establish client connections and exchange data. The connection terminates inside the hardware-backed security enclaves (Intel SGX enclaves), which prevents man-in-the-middle attacks.

Applications are encouraged to verify the authenticity of the ledger nodes by [authenticating the ledger nodes](authenticate-ledger-nodes.md) to establish trust before exchanging data. This process ensures that the ledger nodes are genuine and not malicious.

For broader security guidance that applies to both Confidential Computing Ledger workloads, see [Secure Confidential Computing Ledger](secure-confidential-ledger.md).

## Resiliency and business continuity

Azure confidential ledger nodes are deployed across Azure availability zones to provide resiliency. The network can self-heal during zone-wide outages. To ensure business continuity, the files in the ledger instance are automatically replicated to a secondary storage account periodically. When a disaster happens, these files are used for recovery. Continuous monitoring is used to observe and automatically initiate recovery processes when the health of the ledger instance falls below a specified threshold.

Data is automatically replicated to Azure regional pairs for disaster recovery. For information about data residency considerations, see [Data residency for Azure confidential ledger](data-residency.md).

## Limitations

| Resource | Limit |
|--|--|
| Number of ledgers per subscription | 2 standard SKU ledgers |
| Number of collection IDs per ledger | 50,000 |
| Create entry | 1800 requests per second, 1800 transactions per second |
| Get current entry | 3600 requests per second |
| Get entry | 2500 requests per second |
| Get receipt | 2400 requests per second |
| List entries | 3300 requests per second |

> [!NOTE]
> To request higher limits or discuss limitations, reach out to the Azure confidential ledger team.

## Constraints

- After an Azure confidential ledger instance is created, you can't change the ledger type (private or public).
- Azure confidential ledger deletion leads to a "hard delete," so your data can't be recovered after deletion.
- Azure confidential ledger names must be globally unique. Ledgers with the same name aren't allowed, no matter their type.

## Terminology

| Term | Definition |
|--|--|
| Confidential Computing Ledger | The umbrella service that hosts both ledger workloads on a shared confidential computing platform. |
| Azure confidential ledger | A workload of Confidential Computing Ledger: a customer-managed, append-only ledger you create in your own subscription. |
| Microsoft's Signing Transparency Ledger (MST) | A workload of Confidential Computing Ledger: a Microsoft-managed ledger of software signing events. |
| Ledger | An immutable append-only record of transactions (also known as a blockchain). |
| Commit | A confirmation that a transaction was appended to the ledger. |
| Receipt | Proof that the ledger processed a transaction. |
| CCF | Confidential Consortium Framework — the open-source framework that backs Confidential Computing Ledger. |
| TCB | Trusted Computing Base — the minimal set of components whose integrity the ledger relies on. |

## Related content

- [Architecture](architecture.md)
- [Azure portal quickstart](quickstart-portal.md)
- [Python quickstart](quickstart-python.md)
- [Use the Azure portal ledger explorer to verify transactions](ledger-explorer.md)
- [Inspect ledger data with Ledger Explorer (Offline)](ledger-explorer-offline.md)
- [Secure Confidential Computing Ledger](secure-confidential-ledger.md)
- [About Microsoft's Signing Transparency Ledger](about-microsoft-signing-transparency-ledger.md)
