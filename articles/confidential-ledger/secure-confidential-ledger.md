---
title: Secure your Azure confidential ledger deployment
description: Learn how to secure Azure confidential ledger, with best practices for authentication, data integrity, and access controls.
author: msmbaldwin
ms.author: mbaldwin
ms.service: security
ms.topic: conceptual
ms.custom: horz-security
ms.date: 05/15/2023
---

# Secure your Azure confidential ledger deployment

Azure confidential ledger (ACL) is a highly secure immutable data store for managing sensitive data records with traceability, auditability, and integrity. The service runs on hardware-backed secure enclaves within Azure Confidential Computing, providing a tamper-protected and tamper-evident solution for critical data that requires strong integrity guarantees.

This article provides guidance on how to best secure your Azure confidential ledger deployment.

## Network security

Azure confidential ledger runs exclusively on hardware-backed secure enclaves, providing an isolated runtime environment to protect your data from external threats. The service utilizes secure communication channels and a minimalistic Trusted Computing Base (TCB) to maintain data integrity.

- **Verify ledger authenticity before exchanging data**: Always authenticate the ledger nodes before establishing a connection to ensure you're communicating with legitimate ACL nodes. This verification helps prevent sophisticated man-in-the-middle attacks and ensures your data is flowing to the intended location. For more details, see [Authenticating Azure confidential ledger nodes](authenticate-ledger-nodes.md).

- **Understand TLS connection security**: Azure confidential ledger supports TLS 1.2 and 1.3 protocols for client connections, with connections terminating inside hardware-backed security enclaves. This approach prevents man-in-the-middle attacks, as neither Azure administrators nor anyone else can access data inside the secure enclave. For more information, see the [Azure confidential ledger FAQ](faq.yml).

- **Verify node quotes to establish trust**: Use the `verify_quote.sh` script (installed with the CCF Python package) to verify that the nodes your applications connect to are running trusted code within legitimate Intel SGX enclaves. This verification confirms the cryptographic hash of the node's identity public key matches the SGX report data. For more details, see [Establish trust on Azure confidential ledger](verify-node-quotes.md).

## Identity management

Azure confidential ledger supports robust identity and access management options, allowing you to implement secure authentication methods appropriate for your workload.

- **Use Microsoft Entra ID for authentication**: Microsoft Entra ID is the recommended authentication method for Azure confidential ledger. This approach ensures that your directory credentials are never exposed to the confidential ledger service and leverages Microsoft's secure identity platform. For more information, see [Azure confidential ledger authentication with Microsoft Entra ID](authentication-azure-ad.md).

- **Register your application properly with Microsoft Entra ID**: To use Microsoft Entra authentication, register your application with the appropriate configuration settings, including redirect URIs and permissions. A properly registered application establishes a trust relationship between your app and the Microsoft identity platform. See [How to register an Azure confidential ledger application with Microsoft Entra ID](register-application.md) for detailed steps.

- **Use certificate-based authentication for additional security**: For high-security scenarios or non-interactive applications, implement certificate-based authentication. This approach uses cryptographic certificates to establish identity rather than username/password combinations, enhancing security. Ensure you create certificates using secure methods with appropriate key lengths and algorithms. See [Creating a client certificate](create-client-certificate.md) for guidance.

## Privileged access

Managing access rights is critical to maintaining the security posture of your Azure confidential ledger. Unlike other Azure services, ACL user management is localized within the ledger itself, providing an additional layer of security.

- **Apply least privilege principles with role-based controls**: Assign users the minimum necessary permissions by using ACL's built-in roles: Reader (read-only), Contributor (read and write), and Administrator (read, write, and manage users). This approach limits potential damage if account credentials are compromised. For more details, see [Manage Microsoft Entra token-based users](manage-azure-ad-token-based-users.md) or [Manage certificate-based users](manage-certificate-based-users.md).

- **Carefully manage Administrator accounts**: Limit the number of users with Administrator role to only those who absolutely need to manage other users or perform administrative functions. Remember that Microsoft cannot help manage users once a ledger is created, so maintain proper access to administrator accounts. For more information, see the [Azure confidential ledger FAQ](faq.yml).

- **Implement proper user lifecycle management**: Regularly review and audit user access to your ledger, removing access for users who no longer require it. When adding new users, follow the principle of least privilege by starting with Reader access and only elevating privileges when necessary. For implementation details, consult [Manage Microsoft Entra token-based users](manage-azure-ad-token-based-users.md).

## Data integrity and protection

Azure confidential ledger is designed to provide strong data integrity guarantees through its immutable, append-only architecture and cryptographic verification mechanisms.

- **Choose the appropriate ledger type for your data sensitivity**: When creating a confidential ledger, select between a private (encrypted) or public (plaintext) ledger type based on your security requirements. Private ledgers offer enhanced confidentiality through encryption, while public ledgers provide transparency. This decision cannot be changed after ledger creation. For more information, see the [Azure confidential ledger overview](overview.md).

- **Verify transaction integrity using receipts**: For critical transactions, retrieve and store transaction receipts, which contain cryptographic proofs that the transaction was correctly committed to the ledger. These receipts enable you to verify transaction integrity without maintaining copies of the ledger data. For implementation details, see [Azure Confidential Ledger write transaction receipts](write-transaction-receipts.md).

- **Implement receipt verification for high-value transactions**: Establish a process to verify transaction receipts for high-value or critical data, ensuring that your records have been properly committed to the ledger and haven't been tampered with. This verification can be performed using the available verification utilities or by implementing the verification algorithm directly. For details, see [Verify Azure Confidential Ledger write transaction receipts](verify-write-transaction-receipts.md).

- **Use collection IDs for efficient data organization**: Organize your ledger data using collection IDs (also known as subledger IDs) to improve data management and query efficiency. This approach helps maintain logical separation between different types of data while facilitating easier retrieval of specific datasets. For more information, see [Azure confidential ledger overview](overview.md).

- **Consider integrating with other Azure data stores for comprehensive protection**: For relational or blob data requiring end-to-end integrity guarantees, integrate your data stores with Azure Confidential Ledger. Use the Azure SQL database's ledger feature with ACL as a Trusted Digest store, or use the Azure Marketplace application to protect blob storage integrity. See [Microsoft Azure confidential ledger overview](overview.md) for integration scenarios.

## Resiliency and monitoring

Azure confidential ledger is designed with resiliency features to ensure high availability and business continuity.

- **Understand the resiliency architecture**: Azure confidential ledger nodes are deployed across Azure Availability Zones (AZ) to provide resilience against zone failures. The network automatically self-heals during zone-wide outages, helping maintain service availability. For more information, see [Azure confidential ledger architecture](architecture.md).

- **Plan for business continuity**: Azure confidential ledger automatically replicates ledger files to secondary storage periodically to support disaster recovery. Understand that continuous monitoring is used to observe instance health and automatically initiate recovery processes when needed. For more details, see [Azure confidential ledger overview](overview.md).

- **Be aware of deletion implications**: Azure confidential ledger uses "hard delete" when a ledger is deleted, meaning your data will not be recoverable after deletion. Carefully consider any decision to delete a confidential ledger, as this action is permanent. For more information, see [Azure confidential ledger overview](overview.md).

## Next steps

- [Overview of Microsoft Azure confidential ledger](overview.md)
- [Azure confidential ledger architecture](architecture.md)
- [Register the Azure confidential ledger resource provider](register-ledger-resource-provider.md)
- [Register an ACL app with Microsoft Entra ID](register-application.md)
