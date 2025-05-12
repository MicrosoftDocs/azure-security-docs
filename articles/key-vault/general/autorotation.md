---
title: Understanding autorotation in Azure Key Vault
description: Learn about autorotation concepts for keys, secrets, and certificates in Azure Key Vault
services: key-vault
author: msmbaldwin
ms.author: mbaldwin
ms.service: azure-key-vault
ms.subservice: general
ms.topic: conceptual
ms.date: 05/08/2025
---

# Understanding autorotation in Azure Key Vault

Cryptographic assets like certificates, keys, and secrets have limited lifetimes. As a security best practice, these assets should be rotated regularly to reduce the risk of compromise and ensure compliance with security policies. Azure Key Vault provides automation capabilities for rotating these assets, helping organizations maintain strong security posture with minimal operational overhead.

## What is autorotation?

Autorotation is the process of automatically replacing cryptographic assets with new ones at predefined intervals or in response to specific events. In Azure Key Vault, autorotation capabilities vary depending on the asset type:

- **Keys**: Automatic generation of new key versions based on configured rotation policies
- **Secrets**: Event-triggered updates of secrets with integration to the systems that use them
- **Certificates**: Automatic renewal of certificates before they expire

## Benefits of autorotation

Implementing autorotation for your cryptographic assets provides several benefits:

- **Enhanced security**: Regular rotation of cryptographic material reduces the risk of compromise
- **Simplified compliance**: Meet regulatory and organizational requirements for cryptographic asset lifecycle management
- **Operational efficiency**: Reduce manual effort and the risk of human error in rotation processes
- **Reduced downtime**: Proactive rotation before expiry prevents service disruptions
- **Scalable management**: Automate rotation across multiple assets and services

## Autorotation for different asset types

### Key autorotation

Keys in Azure Key Vault can be configured with rotation policies that automatically generate new key versions at specified frequencies. This is useful for keys used as customer-managed keys (CMKs) in Azure services.

Key autorotation supports:
- Configurable rotation intervals (minimum of seven days)
- Near-expiry notifications via Event Grid
- On-demand rotation in addition to scheduled rotation
- Integration with Azure services using CMKs

[Learn how to configure key autorotation in Azure Key Vault](../keys/how-to-configure-key-rotation.md)

### Secret autorotation

Secrets in Azure Key Vault can be configured for autorotation using Azure functions triggered by Event Grid events. This is valuable for database credentials, API keys, and other sensitive information that requires regular updates.

Secret autorotation supports:
- Event-based triggering via Event Grid
- Integration with Azure Functions for custom rotation logic
- Near-expiry notifications for manual rotation reminders
- Updating dependent services with new secret values

Azure Key Vault supports two secret rotation scenarios:
- [Learn how to implement secret autorotation for resources with one set of authentication credentials](../secrets/tutorial-rotation.md)
- [Learn how to implement secret autorotation for resources with two sets of authentication credentials](../secrets/tutorial-rotation-dual.md)

### Certificate autorotation

Certificates stored in Azure Key Vault can be automatically renewed before they expire. Key Vault handles certificate renewal with integrated certificate authorities (CAs) or through self-signed certificate regeneration.

Certificate autorotation supports:
- Configuration of validity periods
- Automatic renewal at a specified percentage of lifetime or days before expiry
- Email notifications for certificate expiration
- Integration with certificate authorities like DigiCert and GlobalSign

[Learn how to configure certificate autorotation in Azure Key Vault](../certificates/tutorial-rotate-certificates.md)

## Best practices for autorotation

When implementing autorotation in Azure Key Vault, consider these best practices:

1. **Use versioning**: Ensure systems reference the latest version of a key, certificate, or secret automatically
2. **Implement proper access controls**: Use Azure RBAC to control who can configure rotation policies
3. **Monitor rotation events**: Set up notifications and alerts for successful and failed rotations
4. **Test rotation procedures**: Validate that dependent systems can handle rotated assets correctly
5. **Configure appropriate rotation frequencies**: Balance security requirements with operational considerations
6. **Document fallback procedures**: Have manual rotation processes documented for emergency scenarios
7. **Follow security best practices**: Implement comprehensive security measures as outlined in [Secure your Azure Key Vault](secure-key-vault.md)

## Common autorotation scenarios

### Customer-managed keys for Azure services

Many Azure services support encryption with customer-managed keys stored in Key Vault. When these keys are configured for autorotation, the services automatically use the latest key version for new encryption operations while maintaining access to data encrypted with previous versions.

### Database credentials

Database credentials stored as secrets in Key Vault can be automatically rotated with function apps that not only update the secret in Key Vault but also apply the new credentials to the database.

### API keys and service credentials

Autorotation of API keys and service credentials helps maintain security by limiting the lifetime of these sensitive assets. By implementing rotation with Azure Functions, you can update both Key Vault and the target services.

## Next steps

- [Configure cryptographic key autorotation in Azure Key Vault](../keys/how-to-configure-key-rotation.md)
- [Configure certificate autorotation in Key Vault](../certificates/tutorial-rotate-certificates.md)
- [Automate the rotation of a secret for resources with one set of authentication credentials](../secrets/tutorial-rotation.md)
- [About Azure Key Vault](overview.md)
- [Key management in Azure](/azure/security/fundamentals/key-management)
