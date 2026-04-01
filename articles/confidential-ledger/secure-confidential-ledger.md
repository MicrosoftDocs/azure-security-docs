---
title: Secure your Azure confidential ledger deployment
description: Learn how to secure Azure confidential ledger, with best practices for authentication, data integrity, and access controls.
author: msmbaldwin
ms.author: mbaldwin
ms.service: security
ms.topic: best-practice
ms.custom: horz-security
ms.date: 04/01/2026
ai-usage: ai-assisted
---

# Secure your Azure confidential ledger deployment

Azure confidential ledger (ACL) is a highly secure immutable data store for managing sensitive data records with traceability, auditability, and integrity. The service runs on hardware-backed secure enclaves within Azure Confidential Computing, providing a tamper-protected and tamper-evident solution for critical data that requires strong integrity guarantees.

This article provides guidance on how to best secure your Azure confidential ledger deployment.

[!INCLUDE [Security horizontal Zero Trust statement](~/reusable-content/ce-skilling/azure/includes/security/zero-trust-security-horizontal.md)]

## Network security

Azure confidential ledger runs exclusively on hardware-backed secure enclaves, providing an isolated runtime environment to protect your data from external threats. The service utilizes secure communication channels and a minimalistic Trusted Computing Base (TCB) to maintain data integrity.

- **Verify ledger authenticity before exchanging data**: Always authenticate the ledger nodes before establishing a connection to ensure you're communicating with legitimate ACL nodes. This verification helps prevent sophisticated man-in-the-middle attacks and ensures your data is flowing to the intended location. For more details, see [Authenticating Azure confidential ledger nodes](authenticate-ledger-nodes.md).

- **Understand TLS connection security**: Azure confidential ledger uses the TLS 1.3 protocol for client connections, with connections terminating inside hardware-backed security enclaves. This approach prevents man-in-the-middle attacks, as neither Azure administrators nor anyone else can access data inside the secure enclave. For more information, see the [Azure confidential ledger FAQ](faq.yml).

- **Verify node quotes to establish trust**: Use the `verify_quote.sh` script (installed with the CCF Python package) to verify that the nodes your applications connect to are running trusted code within legitimate Intel SGX enclaves. This verification confirms the cryptographic hash of the node's identity public key matches the SGX report data. For more details, see [Establish trust on Azure confidential ledger](verify-node-quotes.md).

- **Retrieve the service identity certificate from the identity service**: Always download the service identity certificate from the dedicated identity service endpoint (`https://identity.confidential-ledger.core.azure.com`), which runs separately from the ledger itself. Don't hardcode or permanently cache this certificate, because it can be rotated during disaster recovery or maintenance events. For retrieval instructions and code examples, see [Authenticating Azure confidential ledger nodes](authenticate-ledger-nodes.md).

## Identity and access management

Azure confidential ledger supports robust identity and access management options. Unlike other Azure services, ACL user management is localized within the ledger itself, providing an additional layer of security by reducing the Trusted Computing Base and eliminating reliance on external authorization systems.

- **Use Microsoft Entra ID for authentication**: Microsoft Entra ID is the recommended authentication method for Azure confidential ledger. This approach ensures that your directory credentials are never exposed to the confidential ledger service and leverages Microsoft's secure identity platform. For more information, see [Azure confidential ledger authentication with Microsoft Entra ID](authentication-azure-ad.md).

- **Register your application properly with Microsoft Entra ID**: To use Microsoft Entra authentication, register your application with the appropriate configuration settings, including redirect URIs and permissions. A properly registered application establishes a trust relationship between your app and the Microsoft identity platform. See [How to register an Azure confidential ledger application with Microsoft Entra ID](register-application.md) for detailed steps.

- **Use certificate-based authentication for additional security**: For high-security scenarios or non-interactive applications, implement certificate-based authentication. This approach uses cryptographic certificates to establish identity rather than username/password combinations, enhancing security. Ensure you create certificates using secure methods with appropriate key lengths and algorithms. See [Creating a client certificate](create-client-certificate.md) for guidance.

- **Secure on-behalf-of authentication flows**: If your application acts as a mediator between users and the confidential ledger (such as a web service accessing ACL on behalf of signed-in users), implement proper on-behalf-of authentication using the OAuth2 token exchange flow. This scenario requires careful configuration with Microsoft Entra ID and might require administrator consent depending on your tenant settings. For more information, see [Azure confidential ledger authentication with Microsoft Entra ID](authentication-azure-ad.md).

- **Apply least privilege principles with role-based controls**: Assign users the minimum necessary permissions by using ACL's built-in roles: Reader (read-only), Contributor (read and write), and Administrator (read, write, and manage users). This approach limits potential damage if account credentials are compromised. For more details, see [Manage Microsoft Entra token-based users](manage-azure-ad-token-based-users.md) or [Manage certificate-based users](manage-certificate-based-users.md).

- **Carefully manage Administrator accounts**: Limit the number of users with Administrator role to only those who absolutely need to manage other users or perform administrative functions. Remember that Microsoft cannot help manage users once a ledger is created, so maintain proper access to administrator accounts. For more information, see the [Azure confidential ledger FAQ](faq.yml).

- **Implement proper user lifecycle management**: Regularly review and audit user access to your ledger, removing access for users who no longer require it. When adding new users, follow the principle of least privilege by starting with Reader access and only elevating privileges when necessary. For implementation details, consult [Manage Microsoft Entra token-based users](manage-azure-ad-token-based-users.md).

- **Review UDF authentication for advanced scenarios**: If you use advanced user defined functions (UDFs), be aware that they support custom authentication and authorization mechanisms. Ensure that any custom endpoints enforce appropriate access controls and don't bypass the ledger's built-in role-based security. For more information, see [Advanced user defined functions in Azure confidential ledger](user-defined-endpoints.md).

## Data protection

Azure confidential ledger is designed to provide strong data integrity and confidentiality guarantees through its immutable, append-only architecture, hardware-backed enclaves, and cryptographic verification mechanisms.

- **Choose the appropriate ledger type for your data sensitivity**: When creating a confidential ledger, select between a private (encrypted) or public (plaintext) ledger type based on your security requirements. Private ledgers encrypt data using three levels of encryption (ledger secrets, ledger secret wrapping key, and recovery key shares). Public ledgers store data in plaintext, suitable for scenarios that require transparency. This decision cannot be changed after ledger creation. For more information, see the [Azure confidential ledger overview](overview.md).

- **Verify transaction integrity using receipts**: For critical transactions, retrieve and store transaction receipts, which contain cryptographic proofs that the transaction was correctly committed to the ledger. These receipts enable you to verify transaction integrity without maintaining copies of the ledger data. For implementation details, see [Azure Confidential Ledger write transaction receipts](write-transaction-receipts.md).

- **Implement receipt verification for high-value transactions**: Establish a process to verify transaction receipts for high-value or critical data, ensuring that your records have been properly committed to the ledger and haven't been tampered with. This verification can be performed using the available verification utilities or by implementing the verification algorithm directly. For details, see [Verify Azure Confidential Ledger write transaction receipts](verify-write-transaction-receipts.md).

- **Use collection IDs for efficient data organization**: Organize your ledger data using collection IDs (also known as subledger IDs) to improve data management and query efficiency. This approach helps maintain logical separation between different types of data while facilitating easier retrieval of specific datasets. For more information, see [Azure confidential ledger overview](overview.md).

- **Consider integrating with other Azure data stores for comprehensive protection**: For relational or blob data requiring end-to-end integrity guarantees, integrate your data stores with Azure Confidential Ledger. Use the Azure SQL database's ledger feature with ACL as a Trusted Digest store, or use the Azure Marketplace application to protect blob storage integrity. See [Microsoft Azure confidential ledger overview](overview.md) for integration scenarios.

- **Use Azure Key Vault for key management**: When integrating Azure confidential ledger with other Azure services, use Azure Key Vault to manage cryptographic keys and secrets used in your application's authentication and data protection workflows. Key Vault provides its own data residency and compliance characteristics. For more information, see [Azure Key Vault overview](/azure/key-vault/general/overview).

## Logging and monitoring

Comprehensive logging and monitoring enable detection of suspicious activities and support compliance with audit requirements.

- **Enable diagnostic logging with Azure Monitor**: If your organization requires operational visibility into ledger activity, enable Azure Monitor diagnostic logs. Configure your Log Analytics workspace to store and analyze diagnostic data according to your retention and compliance policies. For more information, see [Azure Monitor overview](/azure/azure-monitor/overview).

- **Use transaction receipts for audit trails**: Azure confidential ledger provides built-in auditability through its append-only ledger and cryptographic receipts. Use these capabilities to create audit trails for regulatory compliance and forensic analysis. Each transaction produces a Merkle tree-based proof that can be independently verified. For details, see [Azure Confidential Ledger write transaction receipts](write-transaction-receipts.md).

- **Monitor instance health**: Azure confidential ledger uses continuous monitoring to observe instance health and automatically initiate recovery processes when the health of the confidential instance falls below a specified threshold. Understand this behavior to complement it with your own monitoring practices.

- **Store security event data in the ledger**: Consider using Azure confidential ledger as a tamper-proof repository for critical security event data, such as Microsoft Defender for Cloud alerts or operational IT security events. The immutable, append-only nature of the ledger ensures that security records can't be modified or deleted by an attacker who gains access to your environment. For more information, see [Azure confidential ledger overview](overview.md).

## Compliance and governance

Regular review of compliance posture and governance policies ensures your Azure confidential ledger deployment adheres to security standards and organizational requirements.

- **Understand data residency and replication**: Azure confidential ledger automatically replicates data to Azure regional pairs for disaster recovery. You can't modify or disable this replication behavior. Ensure that both the primary and paired regions meet your compliance and regulatory requirements before creating a ledger. For more information, see [Data residency for Azure Confidential Ledger](data-residency.md).

- **Review integrated service compliance**: Azure confidential ledger can integrate with Azure Key Vault for key management, Azure SQL Database for ledger-backed integrity, and Azure Blob Storage for digest protection. Each integrated service has its own compliance posture and data residency characteristics. Review the compliance documentation for each service you use alongside Azure confidential ledger.

- **Understand the immutability guarantee**: Azure confidential ledger provides tamper-evident, append-only storage that's ideal for meeting regulatory requirements around data integrity and audit trails. This property makes the ledger suitable for compliance scenarios where records must be protected from modification or deletion.

- **Plan for service limits**: Each subscription is limited to two standard SKU confidential ledger instances, with additional limits on collection IDs and request rates. Factor these limits into your governance planning and request increases from the Azure Confidential Ledger team if needed. For current limits, see [Azure confidential ledger overview](overview.md).

## Backup and recovery

Azure confidential ledger is designed with resiliency features to ensure high availability and business continuity.

- **Understand the resiliency architecture**: Azure confidential ledger nodes are deployed across Azure Availability Zones (AZ) to provide resilience against zone failures. The network automatically self-heals during zone-wide outages, helping maintain service availability. For more information, see [Azure confidential ledger architecture](architecture.md).

- **Plan for business continuity**: Azure confidential ledger automatically replicates ledger files to secondary storage periodically to support disaster recovery. In the event of a region-wide outage, Microsoft might initiate a failover to the paired region to restore service availability. For more details, see [Data residency for Azure Confidential Ledger](data-residency.md).

- **Be aware of deletion implications**: Azure confidential ledger uses "hard delete" when a ledger is deleted, meaning your data isn't recoverable after deletion. Carefully consider any decision to delete a confidential ledger, as this action is permanent. For more information, see [Azure confidential ledger overview](overview.md).

## Next steps

- [Overview of Microsoft Azure confidential ledger](overview.md)
- [Azure confidential ledger architecture](architecture.md)
- [Authenticating Azure confidential ledger nodes](authenticate-ledger-nodes.md)
- [Register an ACL app with Microsoft Entra ID](register-application.md)
- [Data residency for Azure Confidential Ledger](data-residency.md)
- [User defined functions in Azure confidential ledger](server-side-programming.md)