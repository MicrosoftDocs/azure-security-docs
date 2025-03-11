---
title: Microsoft Azure confidential ledger overview
description: An overview of Azure confidential ledger, a highly secure service for managing sensitive data records with traceability, auditability, and integrity
services: confidential-ledger
author: msmbaldwin
ms.service: azure-confidential-ledger
ms.topic: overview
ms.date: 01/30/2024
ms.author: shusinha


---
# Microsoft Azure confidential ledger

In the era of digital transformation, data integrity is paramount. As businesses increasingly rely on data-driven decisions, the accuracy and security of their data sources become critical.

Microsoft Azure confidential ledger (ACL) is a highly secure immutable data store for managing sensitive data records. The service epitomizes our commitment to secure, reliable, and immutable data storage. 

The confidential ledger offers an auditable data store with unique data integrity advantages, including immutability, tamper-proofing, and append-only operations by combining cryptographic techniques and blockchain technology. 

These features are ideal when critical metadata records must have their integrity protected, such as for regulatory compliance and archival purposes. Data stored in the ledger remains privacy enhanced and protected from insider threats within an organization including the cloud provider. It is also beneficial as a repository of audit trails or records that need to be protected and selectively shared with certain personas (for instance, auditors).

The confidential ledger can protect existing databases and applications by acting as a point-in-time source of truth for digests and hashes. Each transaction in the confidential ledger provides cryptographic proofs in verification scenarios. For example, your Azure SQL data can be integrity protected further, where table digests or logs can be stored in the confidential ledger.

For more information, you can learn about [protecting data source integrity with Azure confidential ledger](https://www.youtube.com/watch?v=lJSn46id-64) or watch a product demo [Azure confidential ledger demo](https://www.youtube.com/watch?v=Cg0-5moftP0). You can also read a recent blog on how [Azure’s hardware security is protected via ACL](https://azure.microsoft.com/blog/microsoft-azure-confidential-ledger-enhancing-customer-trust-in-azures-hardware-supply-chain/).

:::image type="content" source="./media/use-cases.png" alt-text="Diagram of use cases for Azure confidential ledger." lightbox="media/use-cases.png":::

## What to store

Here are a few examples of things you can store on your ledger:

- Records relating to your business transactions (for example, money transfers or confidential document edits).
- Updates to trusted assets (for example, core applications or contracts).
- Administrative and control changes (for example, granting access permissions).
- Operational IT and security events (for example, Microsoft Defender for Cloud alerts)

## Use cases

- I have *relational data* that requires end to end data integrity guarantees - Store your data in [Azure SQL database's ledger feature](/sql/relational-databases/security/ledger/ledger-overview) and turn on Azure Confidential Ledger as your Trusted Digest store.
- I have *blob data* that needs end to end integrity - Store your data in blob storage and configure the [Azure Marketplace application backed by Confidential Ledger](https://azuremarketplace.microsoft.com/marketplace/apps/azureconfidentialledger.acl-blob-storage?tab=Overview) to [store signatures and verify against](/azure/confidential-ledger/create-blob-managed-app?tabs=azure-portal).
- I have *system records* that need integrity protection with verifiability - Store your records in the confidential ledger directly. For instance, have all your development records go to one ledger instance and have your production logs go to another instance! When it is time to audit, only selectively share ledger transactions with the auditor.
- I have *confidential transactional data* that needs confidentiality and integrity protection - Store your critical confidential data's application records in the confidential ledger directly.

## Enabling data integrity for data sources
SQL databases and storage systems are foundational to enterprise data architecture. The confidential ledger enhances these systems by providing an additional layer of integrity protection. For SQL databases, ACL can act as an external ledger where changes and transactions are recorded and verified, adding a new dimension of security and trust.

For Azure Blob Storage, the confidential ledger enhances security features by providing an immutable log of storage operations, which is valuable for regulatory compliance and archival purposes where data integrity over time is crucial.

## How it works

The confidential ledger runs exclusively on hardware-backed secure enclaves, a heavily monitored and isolated runtime environment, which keeps potential attacks at bay. Furthermore, Azure confidential ledger runs on a minimalistic Trusted Computing Base (TCB), which ensures that no one⁠—not even Microsoft⁠—is "above" the ledger.

As its name suggests, Azure confidential ledger utilizes the [Azure Confidential Computing platform](/azure/confidential-computing) and the [Confidential Consortium Framework](https://www.microsoft.com/research/project/confidential-consortium-framework) to provide a high integrity solution that is tamper-protected and tamper-evident. One ledger spans across three or more identical instances, each of which run in a dedicated, fully attested hardware-backed enclave. The ledger's integrity is maintained through a consensus-based blockchain.

## Key features

The confidential ledger exposes a REST interface which makes it easier to integrate with new or existing applications. Also, SDKs in popular languages (.NET, Java, Python, and JavaScript) are provided to help with the integration.

Ledger supports Collection ID for easy data management. Grouping data using collection IDs is a great way to manage and query data efficiently. It allows for easy identification and retrieval of specific data sets. This method can significantly enhance data organization, making operations like searching and updating more streamlined.

Each transaction on the ledger has an associated receipt that records the merkle tree data structure, which is used to [verify the transaction's integrity](/azure/confidential-ledger/verify-write-transaction-receipts). Read more about how [transaction receipts can be verified](/azure/confidential-ledger/write-transaction-receipts).

## Data storage on the confidential ledger

The ledger data is written in blocks that are chained together and stored in Azure backed file storage. Transaction data can either be stored encrypted (for example, private ledger type) or in plain text (for example, public ledger type) depending on your needs.

Administrators can create and manage the confidential ledger with Administrative APIs (Control Plane) for example, delete a resource or move it across Resource Groups. The ledger provides Functional APIs (Data Plane) for data CRUD operations, such as create, update, put, and get. 

## Ledger security

The confidential ledger supports both Microsoft Entra ID and certificate-based credentials for AuthN with custom RBAC for AuthZ. Unlike other Azure services, user management is localized. In other words, users are stored and managed within the ledger using the functional APIs. This design reduces the Trusted Computing Base (TCB) and eliminates the need to rely on external authorization systems such as Azure RBAC.

The confidential ledger uses TLS 1.3 protocol to establish client connection and exchange data. The connection terminates inside the hardware backed security enclaves (Intel® SGX enclaves), thereby preventing a man-in-the-middle attack.

Applications are encouraged to verify the authenticity of the ledger nodes by [authenticating the ledger nodes](/azure/confidential-ledger/authenticate-ledger-nodes) to establish trust before exchanging data. This process ensures that the ledger nodes are genuine and not malicious.

## Resiliency and business continuity 

The confidential ledger nodes are deployed across Azure Availability Zones (AZ) to provide resiliency. The network can self-heal during zone-wide outages. To ensure business continuity, the ledger files are automatically replicated to a secondary storage account periodically. When a disaster happens, these files are used for recovery. Continuous monitoring is utilized to observe and automatically initiate recovery processes when the instance's health falls below a specified threshold.

## Constraints

- Once a confidential ledger is created, you cannot change the ledger type (private or public).
- Azure confidential ledger deletion leads to a "hard delete", so your data will not be recoverable after deletion.
- Azure confidential ledger names must be globally unique. Ledgers with the same name, irrespective of their type, are not allowed.

## Terminology

| Term | Definition |
|--|--|
| ACL | Azure confidential ledger |
| Ledger | An immutable append-only record of transactions (also known as a Blockchain) |
| Commit | A confirmation that a transaction was appended to the ledger. |
| Receipt | Proof that the ledger processed a transaction. |

## Next steps

- [Microsoft Azure confidential ledger architecture](architecture.md)
- [Quickstart: Azure portal](quickstart-portal.md)
- [Quickstart: Python](quickstart-python.md)
- [Quickstart: Azure Resource Manager (ARM) template](quickstart-template.md)
