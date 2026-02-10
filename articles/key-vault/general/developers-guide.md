---
title: Azure Key Vault developer's guide
description: Developers can use Azure Key Vault to manage cryptographic keys within the Microsoft Azure environment.
services: key-vault
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 01/08/2026
ms.author: mbaldwin
---
# Azure Key Vault developer's guide

Azure Key Vault is a cloud service that provides secure storage and management of cryptographic keys, secrets, and certificates. This guide helps developers integrate Key Vault into their applications.

## Overview

Azure Key Vault enables you to:

- **Secure storage**: Protect keys, secrets, and certificates without writing custom security code.
- **Simplified key management**: Centralize cryptographic operations and key lifecycle management.
- **Customer-owned keys**: Allow customers to manage their own keys while you focus on core application features.
- **External key management**: Use keys for signing and encryption while keeping them external to your application.

For general information on Azure Key Vault, see [About Azure Key Vault](overview.md).

## Developer scenarios

Common developer tasks with Key Vault include:

- **Store and retrieve secrets**: Manage connection strings, passwords, API keys, and SAS tokens securely. For more information, see [About secrets](../secrets/about-secrets.md).
- **Use keys for encryption and signing**: Perform cryptographic operations without exposing key material to your application. For more information, see [About keys](../keys/about-keys.md).
- **Manage certificates**: Automate certificate provisioning, renewal, and deployment for SSL/TLS. For more information, see [About certificates](../certificates/about-certificates.md).

## Public previews

Microsoft periodically releases public previews of new Key Vault features. To try preview features and provide feedback, contact the team at azurekeyvault@microsoft.com. For information about the latest features and updates, see [What's new in Azure Key Vault](whats-new.md).

## Create and manage key vaults

Key Vault uses a two-plane access model:

- **Control plane**: Manages the Key Vault resource itself (create, delete, update properties, assign access policies). Operations are managed through [Azure Resource Manager](/azure/azure-resource-manager/management/overview). For access control, see [Assign a Key Vault access policy](assign-access-policy.md).
- **Data plane**: Manages the data stored in Key Vault (keys, secrets, certificates). Access is controlled through [Azure RBAC with Key Vault](rbac-guide.md).

Use the predefined **Key Vault Contributor** role to grant management access to Key Vault resources. For more information on authentication and authorization, see [Authentication in Azure Key Vault](authentication.md).

### Network security

Reduce network exposure by configuring private endpoints, firewalls, or service endpoints. For comprehensive network security guidance including configuration options from most to least restrictive, see [Secure your Azure Key Vault: Network Security](secure-key-vault.md#network-security) and [Configure Azure Key Vault networking settings](network-security.md).     

### APIs and SDKs for key vault management

The following table lists SDKs and quickstarts for managing Key Vault resources (control plane operations). For the latest versions and installation instructions, see [Client libraries](client-libraries.md).

| Azure CLI | PowerShell | REST API | Resource Manager | .NET | Python | Java | JavaScript |  
|--|--|--|--|--|--|--|--|
|[Reference](/cli/azure/keyvault)<br>[Quickstart](quick-create-cli.md)|[Reference](/powershell/module/az.keyvault)<br>[Quickstart](quick-create-powershell.md)|[Reference](/rest/api/keyvault/)|[Reference](/azure/templates/microsoft.keyvault/vaults)<br>[Quickstart](./vault-create-template.md)|[Reference](/dotnet/api/microsoft.azure.management.keyvault)|[Reference](/python/api/azure-mgmt-keyvault/azure.mgmt.keyvault)|[Reference](/java/api/overview/azure/resourcemanager-keyvault-readme)|[Reference](/javascript/api/@azure/arm-keyvault)|

## Authenticate to Key Vault in code

Key Vault uses Microsoft Entra authentication, which requires a Microsoft Entra security principal to grant access. A Microsoft Entra security principal can be a user, an application service principal, a [managed identity for Azure resources](/entra/identity/managed-identities-azure-resources/overview), or a group of any of these types.

### Authentication best practices

For applications deployed to Azure, use managed identities to eliminate the need for storing credentials in code. For detailed authentication guidance and security principal recommendations for different environments (production, development, local), see [Authentication in Azure Key Vault](authentication.md) and [Secure your Azure Key Vault](secure-key-vault.md).

### Azure Identity client libraries

The preceding authentication scenarios are supported by the *Azure Identity client library* and integrated with Key Vault SDKs. You can use the Azure Identity client library across environments and platforms without changing your code. The library automatically retrieves authentication tokens from users who are signed in to Azure user through the Azure CLI, Visual Studio, Visual Studio Code, and other means. 

For more information about the Azure Identity client library, see:

| .NET | Python | Java | JavaScript |
|--|--|--|--|
|[Azure Identity SDK .NET](/dotnet/api/overview/azure/identity-readme)|[Azure Identity SDK Python](/python/api/overview/azure/identity-readme)|[Azure Identity SDK Java](/java/api/overview/azure/identity-readme)|[Azure Identity SDK JavaScript](/javascript/api/overview/azure/identity-readme)|     

> [!Note]
> We recommended [App Authentication library](/dotnet/api/overview/azure/service-to-service-authentication) for Key Vault .NET SDK version 3, but it's now deprecated. To migrate to Key Vault .NET SDK version 4, follow the [AppAuthentication to Azure.Identity migration guidance](/dotnet/api/overview/azure/app-auth-migration).

For tutorials on how to authenticate to Key Vault in applications, see:
- [Use Azure Key Vault with a virtual machine in .NET](./tutorial-net-virtual-machine.md)
- [Use Azure Key Vault with a virtual machine in Python](./tutorial-python-virtual-machine.md)
- [Use a managed identity to connect Key Vault to an Azure web app in .NET](./tutorial-net-create-vault-azure-web-app.md)

## Manage keys, certificates, and secrets

> [!Note]
> SDKs for .NET, Python, Java, JavaScript, PowerShell, and the Azure CLI are part of the Key Vault feature release process through public preview and general availability with Key Vault service team support. Other SDK clients for Key Vault are available, but they are built and supported by individual SDK teams over GitHub and released in their teams schedule. For the latest SDK versions and installation packages, see [Client libraries](client-libraries.md).

The data plane controls access to keys, certificates, and secrets. You can use [Azure RBAC with Key Vault](rbac-guide.md) for access control through the data plane.

### APIs and SDKs for keys

The following table lists SDKs and quickstarts for working with keys (data plane operations). For more information about keys, see [About keys](../keys/about-keys.md).

| Azure CLI | PowerShell | REST API | Resource Manager | .NET | Python | Java | JavaScript |  
|--|--|--|--|--|--|--|--|
|[Reference](/cli/azure/keyvault/key)<br>[Quickstart](../keys/quick-create-cli.md)|[Reference](/powershell/module/az.keyvault/)<br>[Quickstart](../keys/quick-create-powershell.md)|[Reference](/rest/api/keyvault/#key-operations)|[Reference](/azure/templates/microsoft.keyvault/vaults/keys)<br>[Quickstart](../keys/quick-create-template.md)|[Reference](/dotnet/api/azure.security.keyvault.keys)<br>[Quickstart](../keys/quick-create-net.md)|[Reference](/python/api/overview/azure/keyvault-keys-readme)<br>[Quickstart](../keys/quick-create-python.md)|[Reference](/java/api/overview/azure/security-keyvault-keys-readme)<br>[Quickstart](../keys/quick-create-java.md)|[Reference](/javascript/api/@azure/keyvault-keys/)<br>[Quickstart](../keys/quick-create-node.md)|

#### Other Libraries

##### Cryptography client for Key Vault and Managed HSM
This module provides a cryptography client for the [Azure Key Vault Keys client module for Go](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go/sdk/security/keyvault/azkeys).

> [!Note]
> This project is not supported by the Azure SDK team, but does align with the cryptography clients in other supported languages.

| Language | Reference |
|--|--|
|Go|[Reference](https://pkg.go.dev/github.com/heaths/azcrypto)|

### APIs and SDKs for certificates

The following table lists SDKs and quickstarts for working with certificates (data plane operations). For more information about certificates, see [About certificates](../certificates/about-certificates.md).

| Azure CLI | PowerShell | REST API | Resource Manager | .NET | Python | Java | JavaScript |  
|--|--|--|--|--|--|--|--|
|[Reference](/cli/azure/keyvault/certificate)<br>[Quickstart](../certificates/quick-create-cli.md)|[Reference](/powershell/module/az.keyvault)<br>[Quickstart](../certificates/quick-create-powershell.md)|[Reference](/rest/api/keyvault/#certificate-operations)|N/A|[Reference](/dotnet/api/azure.security.keyvault.certificates)<br>[Quickstart](../certificates/quick-create-net.md)|[Reference](/python/api/overview/azure/keyvault-certificates-readme)<br>[Quickstart](../certificates/quick-create-python.md)|[Reference](/java/api/overview/azure/security-keyvault-certificates-readme)<br>[Quickstart](../certificates/quick-create-java.md)|[Reference](/javascript/api/@azure/keyvault-certificates/)<br>[Quickstart](../certificates/quick-create-node.md)|

### APIs and SDKs for secrets

The following table lists SDKs and quickstarts for working with secrets (data plane operations). For more information about secrets, see [About secrets](../secrets/about-secrets.md).

| Azure CLI | PowerShell | REST API | Resource Manager | .NET | Python | Java | JavaScript |  
|--|--|--|--|--|--|--|--|
|[Reference](/cli/azure/keyvault/secret)<br>[Quickstart](../secrets/quick-create-cli.md)|[Reference](/powershell/module/az.keyvault/)<br>[Quickstart](../secrets/quick-create-powershell.md)|[Reference](/rest/api/keyvault/#secret-operations)|[Reference](/azure/templates/microsoft.keyvault/vaults/secrets)<br>[Quickstart](../secrets/quick-create-template.md)|[Reference](/dotnet/api/azure.security.keyvault.secrets)<br>[Quickstart](../secrets/quick-create-net.md)|[Reference](/python/api/overview/azure/keyvault-secrets-readme)<br>[Quickstart](../secrets/quick-create-python.md)|[Reference](/java/api/overview/azure/security-keyvault-secrets-readme)<br>[Quickstart](../secrets/quick-create-java.md)|[Reference](/javascript/api/@azure/keyvault-secrets/)<br>[Quickstart](../secrets/quick-create-node.md)|

### Usage of secrets
Use Azure Key Vault to store only secrets for your application. Examples of secrets that should be stored in Key Vault include:

- Client application secrets
- Connection strings
- Passwords
- Shared access keys
- SSH keys

Any secret-related information, like usernames and application IDs, can be stored as a tag in a secret. For any other sensitive configuration settings, you should use [Azure App Configuration](/azure/azure-app-configuration/overview).
 
For installation packages and source code, see [Client libraries](client-libraries.md).

## Use Key Vault in applications

To take advantage of the most recent features in Key Vault, we recommend that you use the available Key Vault SDKs for using secrets, certificates, and keys in your application. The Key Vault SDKs and REST API are updated as new features are released for the product, and they follow best practices and guidelines.

For basic scenarios, there are other libraries and integration solutions for simplified usage, with support provided by Microsoft partners or open-source communities.

For certificates, you can use:

- The Key Vault virtual machine (VM) extension, which provides automatic refresh of certificates stored in an Azure key vault. For more information, see: 
  - [Key Vault virtual machine extension for Windows](/azure/virtual-machines/extensions/key-vault-windows)
  - [Key Vault virtual machine extension for Linux](/azure/virtual-machines/extensions/key-vault-linux)
  - [Key Vault virtual machine extension for Azure Arc-enabled servers](/azure/azure-arc/servers/manage-vm-extensions#azure-key-vault-vm-extension)
- Azure App Service integration, which can import and automatically refresh certificates from Key Vault. For more information, see [Import a certificate from Key Vault](/azure/app-service/configure-ssl-certificate#import-a-certificate-from-key-vault).

For secrets, you can use:

- Key Vault secrets with App Service application settings. For more information, see [Use Key Vault references for App Service and Azure Functions](/azure/app-service/app-service-key-vault-references).
- Key Vault references with Azure App Configuration to streamline your application's access to configuration and secrets. For more information, see [Use Key Vault references in Azure App Configuration](/azure/azure-app-configuration/use-key-vault-references-dotnet-core).

## Code examples

For complete examples of using Key Vault with applications, see [Azure Key Vault code samples](https://azure.microsoft.com/resources/samples/?service=key-vault). 

## Task-specific guidance

The following articles and scenarios provide task-specific guidance for working with Azure Key Vault:

- To access a key vault, your client application needs to be able to access multiple endpoints for various functionalities. See [Accessing Key Vault behind a firewall](access-behind-firewall.md). 
- A cloud application running in an Azure VM needs a certificate. How do you get this certificate into this VM? See [Key Vault virtual machine extension for Windows](/azure/virtual-machines/extensions/key-vault-windows) or [Key Vault virtual machine extension for Linux](/azure/virtual-machines/extensions/key-vault-linux).
- To assign an access policy by using the Azure CLI, PowerShell, or the Azure portal, see [Assign a Key Vault access policy](assign-access-policy.md). 
- For guidance on the use and lifecycle of a key vault and various key vault objects with soft-delete enabled, see [Azure Key Vault recovery management with soft delete and purge protection](./key-vault-recovery.md).
- When you need to pass a secure value (like a password) as a parameter during deployment, you can store that value as a secret in a key vault and reference the value in other Resource Manager templates. See [Use Azure Key Vault to pass secure parameter values during deployment](/azure/azure-resource-manager/templates/key-vault-parameter).

## Integration with Key Vault

The following services and scenarios use or integrate with Key Vault:

- [Encryption at rest](/azure/security/fundamentals/encryption-atrest) allows the encoding (encryption) of data when it's persisted. Data encryption keys are often encrypted with a key encryption key in Azure Key Vault to further limit access.
- [Azure Information Protection](/azure/information-protection/plan-implement-tenant-key) allows you to manage your own tenant key. For example, instead of Microsoft managing your tenant key (the default), you can manage your own tenant key to comply with specific regulations that apply to your organization. Managing your own tenant key is also called *bring your own key (BYOK)*.
- [Azure Private Link](private-link-service.md) enables you to access Azure services (for example, Azure Key Vault, Azure Storage, and Azure Cosmos DB) and Azure-hosted customer/partner services over a private endpoint in your virtual network.
- Key Vault integration with [Azure Event Grid](/azure/event-grid/event-schema-key-vault) allows users to be notified when the status of a secret stored in Key Vault has changed. You can distribute new versions of secrets to applications or rotate near-expiration secrets to prevent outages.
- Protect your [Azure DevOps](/azure/devops/pipelines/release/azure-key-vault) secrets from unwanted access in Key Vault.
- Use secrets stored in Key Vault to [connect to Azure Storage from Azure Databricks](./integrate-databricks-blob-storage.md).
- Configure and run the Azure Key Vault provider for the [Secrets Store CSI driver](./key-vault-integrate-kubernetes.md) on Kubernetes. 

## Disaster recovery and business continuity

Key Vault provides built-in disaster recovery with automatic regional replication. For production deployments, enable soft delete and purge protection, and implement regular backups. For more information, see [Azure Key Vault availability and redundancy](disaster-recovery-guidance.md), [Azure Key Vault recovery management](key-vault-recovery.md), and [Azure Key Vault backup](backup.md).

## Performance and scalability

When developing applications that use Key Vault, consider the following performance and scalability best practices:

- **Service limits**: Key Vault has service limits for transactions per vault per region. Exceeding these limits results in throttling. For more information, see [Azure Key Vault service limits](service-limits.md).
- **Throttling guidance**: Implement retry logic with exponential backoff to handle throttling responses. For more information, see [Azure Key Vault throttling guidance](overview-throttling.md).
- **Caching**: Cache secrets and certificates in your application to reduce calls to Key Vault and improve performance.
- **Connection management**: Reuse HTTP connections to Key Vault when possible to reduce latency and improve performance.

## Monitoring and logging

Enable logging and monitoring for security, compliance, and troubleshooting. Configure diagnostic settings, Event Grid notifications, and alerts for critical events. For detailed guidance, see [Monitor Azure Key Vault](monitor-key-vault.md), [Azure Key Vault logging](logging.md), [Monitoring Key Vault with Azure Event Grid](event-grid-overview.md), and [Secure your Azure Key Vault: Logging and Threat Detection](secure-key-vault.md#logging-and-threat-detection).

## Common parameters and request patterns

When working with the Key Vault REST API, understanding common parameters and request/response patterns is helpful:

- **API versions**: Key Vault uses versioned APIs. Always specify the API version in your requests.
- **Authentication requests**: Understand how authentication tokens are acquired and used, including common request headers and response formats. For more information, see [Authentication, requests, and responses](authentication-requests-and-responses.md).
- **Error codes**: Familiarize yourself with common REST API error codes to handle failures gracefully. For more information, see [Azure Key Vault REST API error codes](rest-error-codes.md).

## Troubleshooting

For help resolving common issues:

- **Access denied errors**: Verify your authentication credentials and that your security principal has the necessary permissions through RBAC assignments. See [Azure RBAC for Key Vault data plane operations](rbac-guide.md).
- **Network connectivity**: If accessing Key Vault from behind a firewall, ensure required endpoints are accessible. See [Accessing Key Vault behind a firewall](access-behind-firewall.md).
- **Throttling**: If receiving 429 (Too Many Requests) responses, implement exponential backoff. See [Azure Key Vault throttling guidance](overview-throttling.md).

## Security best practices

For comprehensive security guidance including identity and access management, data protection, compliance, governance, and backup strategies, see [Secure your Azure Key Vault](secure-key-vault.md).

## Additional resources

### Key Vault concepts

- [Azure Key Vault basic concepts](basic-concepts.md) - Foundational concepts for working with Key Vault.
- [Azure Key Vault soft-delete overview](soft-delete-overview.md) - Recovery of deleted objects.
- [Azure Key Vault throttling guidance](overview-throttling.md) - Basic concepts and approach for your app.
- [Azure Key Vault security worlds and geographic boundaries](overview-security-worlds.md) - Regional and security relationships.
- [Azure Key Vault service limits](service-limits.md) - Transaction limits and other service restrictions.

### Management and operations

- [Monitor Azure Key Vault](monitor-key-vault.md) - Set up monitoring and diagnostics.
- [Azure Key Vault logging](logging.md) - Enable and analyze Key Vault logs.

### Community and support

- [Microsoft Q&A](/answers/products/) - Ask questions and get answers from the community.
- [Stack Overflow for Key Vault](https://stackoverflow.com/questions/tagged/azure-keyvault) - Technical Q&A from developers.
- [Azure Feedback](https://feedback.azure.com/d365community/forum/285c5ae0-f524-ec11-b6e6-000d3a4f0da0) - Submit feature requests and feedback.
