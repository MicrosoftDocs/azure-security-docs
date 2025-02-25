---
title: Microsoft Azure confidential ledger overview
description: An overview of Azure confidential ledger, a highly secure service for managing sensitive data records with traceability, auditability, and integrity
services: confidential-ledger
author: msmbaldwin
ms.service: azure-confidential-ledger
ms.topic: overview
ms.date: 01/30/2024
ms.author: mbaldwin

---
# Microsoft Azure confidential ledger

Microsoft Azure confidential ledger (ACL) is a highly secure immutable data store for managing sensitive data records.  This auditable data store offers unique data integrity advantages, including immutability, tamper-proofing, and append-only operations through the private blockchain architecture. 

These features are ideal when critical metadata records must not be modified and kept intact, such as for regulatory compliance and archival purposes. Data logged in the ledger remains privacy enhanced and protected from insider threats within an organization or even the cloud provider.  It is also beneficial as a repository of audit trails or records that need to be protected and selectively shared with certain personas (i.e. auditors).

The confidential ledger can integrity protect existing databases and applications by acting as a point-in-time source of truth for digests and signatures. Each transaction in the confidential ledger provides cryptographic proofs in verification scenarios.   For example, your Azure SQL data can be integrity protected.  Please see a recent blog on how [Azure’s hardware security is protected via ACL](https://azure.microsoft.com/en-us/blog/microsoft-azure-confidential-ledger-enhancing-customer-trust-in-azures-hardware-supply-chain/).

For more information, you can learn about [protecting data source integrity with Azure confidential ledger](https://www.youtube.com/watch?v=lJSn46id-64) or watch a product demo [Azure confidential ledger demo](https://www.youtube.com/watch?v=Cg0-5moftP0).

# What to store 
Here are a few examples of things you can store on your ledger:

- Records relating to your business transactions (for example, money transfers or confidential document edits).
- Updates to trusted assets (for example, core applications or contracts).
- Administrative and control changes (for example, granting access permissions).
- Operational IT and security events (for example, Microsoft Defender for Cloud alerts)

## Use Cases 
- I have *relational data* that requires end to end data integrity guarantees - Store your data in [Azure SQL database's ledger feature](https://learn.microsoft.com/en-us/sql/relational-databases/security/ledger/ledger-overview?view=sql-server-ver16) and turn on Azure Confidential Ledger as your Trusted Digest store.
- I have *blob data* that needs end to end integrity - Store your data in immutable blob storage and configure the [Azure Marketplace application backed by Confidential Ledger](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/azureconfidentialledger.acl-blob-storage?tab=Overview) to store signatures and verify against.
- I have *system records* that need integrity protection with verifiability - Store your records in the confidential ledger directly. Use different ledger instances for your purposes!
- I have *confidential transactional data* that needs confidentiality and integrity protection - Store your critical confidential data's application records in the confidential ledger directly.


# How it works
The confidential ledger runs exclusively on hardware-backed secure enclaves, a heavily monitored and isolated runtime environment, which keeps potential attacks at bay. Furthermore, Azure confidential ledger runs on a minimalistic Trusted Computing Base (TCB), which ensures that no one⁠—not even Microsoft⁠—is "above" the ledger.

As its name suggests, Azure confidential ledger utilizes the [Azure Confidential Computing platform](/azure/confidential-computing) and the [Confidential Consortium Framework](https://www.microsoft.com/en-us/research/project/confidential-consortium-framework) to provide a high integrity solution that is tamper-protected and evident. One ledger spans across three or more identical instances, each of which run in a dedicated, fully attested hardware-backed enclave. The ledger's integrity is maintained through a consensus-based blockchain.


## Key Features

The confidential ledger is exposed through REST APIs, which can be integrated into new or existing applications. Administrators can manage the confidential ledger with Administrative APIs (Control Plane). The confidential ledger can also be called directly by application code through Functional APIs (Data Plane). The Administrative APIs support basic operations such as create, update, get and, delete. The Functional APIs allow direct interaction with your instantiated ledger and include operations such as put and get data.

## Ledger security

The ledger APIs support certificate-based authentication process with owner roles as well as Microsoft Entra ID based authentication and also role-based access (for example, owner, reader, and contributor).

The data to the ledger is sent through TLS 1.3 connection and the TLS 1.3 connection terminates inside the hardware backed security enclaves (Intel® SGX enclaves), ensuring that no one can intercept the connection between a customer's client and the confidential ledger server nodes.

### Ledger data storage

Confidential ledgers are created as blocks in blob storage containers belonging to an Azure Storage account. Transaction data can either be stored encrypted or in plaintext depending on your needs.

Administrators can manage the confidential ledger with Administrative APIs (Control Plane), and the confidential ledger can be called directly by your application code through Functional APIs (Data Plane). The Administrative APIs support basic operations such as create, update, get and, delete.

The Functional APIs allow direct interaction with your instantiated confidential ledger and include operations such as put and get data.

## BCDR 
<insert copy>

## Performance 
<insert copy>
  
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
