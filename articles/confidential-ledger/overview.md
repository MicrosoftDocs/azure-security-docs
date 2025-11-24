---
title: Azure Confidential Ledger Overview
description: An overview of Azure confidential ledger, which is a highly secure service for managing sensitive data records with traceability, auditability, and integrity.
services: confidential-ledger
author: msmbaldwin
ms.service: azure-confidential-ledger
ms.topic: overview
ms.date: 04/14/2025
ms.author: shusinha


---
# Azure confidential ledger

In the era of digital transformation, data integrity is paramount. As businesses increasingly rely on data-driven decisions, the accuracy and security of their data sources become critical.

Azure confidential ledger is a highly secure immutable data store for managing sensitive data records. The service epitomizes our commitment to secure, reliable, and immutable data storage.

Confidential ledger offers an auditable data store with unique data integrity advantages. These advantages include immutability, tamper-proofing, and append-only operations. Confidential ledger combines cryptographic techniques and blockchain technology.

These features are ideal when critical metadata records must have their integrity protected, such as for regulatory compliance and archival purposes. Data stored in confidential ledger remains privacy enhanced and protected from insider threats within an organization, including the cloud provider. It's also beneficial as a repository of audit trails or records that need to be protected and selectively shared with certain personas (for instance, auditors).

Confidential ledger can protect existing databases and applications by acting as a point-in-time source of truth for digests and hashes. Each transaction in confidential ledger provides cryptographic proofs in verification scenarios. For example, your Azure SQL data can be integrity protected further, where table digests or logs can be stored in confidential ledger.

For more information, you can learn about [protecting data source integrity with Azure confidential ledger](https://www.youtube.com/watch?v=lJSn46id-64) or watch an [Azure confidential ledger demo](https://www.youtube.com/watch?v=Cg0-5moftP0). You can also read a recent blog on how [Azure hardware security is protected via Azure confidential ledger](https://azure.microsoft.com/blog/microsoft-azure-confidential-ledger-enhancing-customer-trust-in-azures-hardware-supply-chain/).

:::image type="content" source="./media/use-cases.png" alt-text="Diagram of use cases for Azure confidential ledger." lightbox="media/use-cases.png":::

## What to store

Here are a few examples of things you can store in your confidential ledger instance:

- Records relating to your business transactions (for example, money transfers or confidential document edits).
- Updates to trusted assets (for example, core applications or contracts).
- Administrative and control changes (for example, granting access permissions).
- Operational IT and security events (for example, Microsoft Defender for Cloud alerts).

## Use cases

- I have *relational data* that requires end to end data integrity guarantees: Store your data in the [Azure SQL Database ledger feature](/sql/relational-databases/security/ledger/ledger-overview) and turn on confidential ledger as your Trusted Digest store.
- I have *blob data* that needs end to end integrity: Store your data in Azure Blob Storage and configure the [Azure Marketplace application backed by confidential ledger](https://azuremarketplace.microsoft.com/marketplace/apps/azureconfidentialledger.acl-blob-storage?tab=Overview) to [store signatures and verify against](/azure/confidential-ledger/create-blob-managed-app?tabs=azure-portal).
- I have *system records* that need integrity protection with verifiability: Store your records in confidential ledger directly. For example, have all your development records go to one confidential ledger instance and have your production logs go to another instance. When you want to audit, only selectively share confidential ledger transactions with the auditor.
- I have *confidential transactional data* that needs confidentiality and integrity protection: Store your critical confidential data's application records in confidential ledger directly.

## Enable data integrity for data sources

SQL databases and storage systems are foundational to enterprise data architecture. Confidential ledger enhances these systems by providing an extra layer of integrity protection. [For SQL databases](/sql/relational-databases/security/ledger/ledger-overview), confidential ledger can act as an external ledger where changes and transactions are recorded and verified, which adds a new dimension of security and trust.

[For Blob Storage](/azure/confidential-ledger/create-blob-managed-app), confidential ledger enhances security features by providing an immutable log of storage operations. This log is valuable for regulatory compliance and archival purposes where data integrity over time is crucial.

## How it works

Confidential ledger runs exclusively on hardware-backed secure enclaves. This heavily monitored and isolated runtime environment keeps potential attacks at bay. Confidential ledger also runs on a minimalistic Trusted Computing Base (TCB). The TCB ensures that no one⁠, not even Microsoft⁠, is "above" the ledger.

As the name suggests, the confidential ledger service uses the [Azure confidential computing platform](/azure/confidential-computing) and the [Confidential Consortium Framework](https://www.microsoft.com/research/project/confidential-consortium-framework) to provide a high-integrity solution that's tamper protected and tamper evident. One ledger spans across three or more identical instances. Each instance runs in a dedicated, fully attested hardware-backed enclave. The integrity of the confidential ledger instance is maintained through a consensus-based blockchain.

## Key features

Confidential ledger exposes a REST interface, which makes it easier to integrate with new or existing applications. Also, SDKs in popular languages like .NET, Java, Python, and JavaScript are provided to help with the integration.

Confidential ledger supports collection ID for easy data management. Grouping data by using collection IDs is a great way to manage and query data efficiently. It allows for easy identification and retrieval of specific datasets. This method can significantly enhance data organization and make operations like searching and updating more streamlined.

Each transaction on the confidential ledger instance has an associated receipt that records the Merkle tree data structure, which is used to [verify the transaction's integrity](/azure/confidential-ledger/verify-write-transaction-receipts). Read more about how you can [verify transaction receipts](/azure/confidential-ledger/write-transaction-receipts).

## Data storage in confidential ledger

Confidential ledger data is written in blocks that are chained together and stored in Azure-backed file storage. Transaction data can either be stored encrypted (for example, private ledger type) or in plain text (for example, public ledger type) depending on your needs.

Administrators can create and manage confidential ledger with administrative APIs (control plane), for example, to delete a resource or move it across resource groups. Confidential ledger provides functional APIs (data plane) for data operations such as CREATE, UPDATE, PUT, and GET.

## Ledger security

Confidential ledger supports both Microsoft Entra ID and certificate-based credentials for AuthN with custom role-based access control (RBAC) for AuthZ. Unlike other Azure services, user management is localized. In other words, users are stored and managed within the ledger by using the functional APIs. This design reduces the TCB and eliminates the need to rely on external authorization systems, such as Azure RBAC.

Confidential ledger uses the TLS 1.3 protocol to establish client connection and exchange data. The connection terminates inside the hardware-backed security enclaves (Intel SGX enclaves), which prevents a man-in-the-middle attack.

Applications are encouraged to verify the authenticity of the ledger nodes by [authenticating the ledger nodes](/azure/confidential-ledger/authenticate-ledger-nodes) to establish trust before exchanging data. This process ensures that the ledger nodes are genuine and not malicious.

## Resiliency and business continuity

Confidential ledger nodes are deployed across Azure availability zones to provide resiliency. The network can self-heal during zone-wide outages. To ensure business continuity, the files in the confidential ledger instance are automatically replicated to a secondary storage account periodically. When a disaster happens, these files are used for recovery. Continuous monitoring is used to observe and automatically initiate recovery processes when the health of the confidential instance falls below a specified threshold.

Data is automatically replicated to Azure regional pairs for disaster recovery. For information about data residency considerations, see [Data residency for Azure confidential ledger](data-residency.md).

## Constraints

- After a confidential ledger instance is created, you can't change the ledger type (private or public).
- Confidential ledger deletion leads to a "hard delete," so your data can't be recovered after deletion.
- Confidential ledger names must be globally unique. Ledgers with the same name aren't allowed, no matter their type.

## Terminology

| Term | Definition |
|--|--|
| ACL | Azure confidential ledger. |
| Ledger | An immutable append-only record of transactions (also known as a blockchain). |
| Commit | A confirmation that a transaction was appended to the confidential ledger instance. |
| Receipt | Proof that the confidential ledger instance processed a transaction. |

## Related content

- [Azure confidential ledger architecture](architecture.md)
- [Quickstart: Azure portal](quickstart-portal.md)
- [Quickstart: Python](quickstart-python.md)
- [Quickstart: Azure Resource Manager (ARM) template](quickstart-template.md)
