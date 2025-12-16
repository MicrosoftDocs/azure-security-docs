---
title: Secure your Azure Key Vault secrets
description: Learn how to secure Azure Key Vault secrets, with best practices specific to secrets management.
author: msmbaldwin
tags: azure-key-vault
ms.service: azure-key-vault
ms.subservice: secrets
ms.topic: conceptual
ms.custom: horz-security
ms.date: 11/10/2025
ms.author: mbaldwin
ai-usage: ai-assisted
# Customer intent: As a developer using Key Vault secrets, I want to implement secrets-specific security best practices.
---
# Secure your Azure Key Vault secrets

Azure Key Vault secrets store sensitive application credentials like passwords, connection strings, and access keys. This article provides security recommendations specific to secrets management.

> [!NOTE]
> This article focuses on security practices specific to Key Vault secrets. For comprehensive Key Vault security guidance including network security, identity and access management, and vault architecture, see [Secure your Azure Key Vault](../general/secure-key-vault.md).

## What to store as secrets

Azure Key Vault secrets are designed for storing service or application credentials. Store the following types of data as secrets:

- **Application credentials**: Client application secrets, API keys, service principal credentials
- **Connection strings**: Database connection strings, storage account connection strings
- **Passwords**: Service passwords, application passwords
- **Access keys**: Redis Cache keys, Azure Event Hubs keys, Azure Cosmos DB keys, Azure Storage keys
- **SSH keys**: Private SSH keys for secure shell access

> [!IMPORTANT]
> Do not store configuration data in Key Vault. IP addresses, service names, feature flags, and other configuration settings should be stored in [Azure App Configuration](/azure/azure-app-configuration/overview) rather than in Key Vault. Key Vault is optimized for cryptographic secrets, not general configuration management.

For more information about secrets, see [About Azure Key Vault secrets](about-secrets.md).

## Secrets storage format

When storing secrets in Key Vault, follow these formatting best practices:

- **Store compound credentials properly**: For credentials with multiple components (like username/password), store them as:
  - A properly formatted connection string, or
  - A JSON object containing the credential components
  
- **Use tags for metadata**: Store management information like rotation schedules, expiration dates, and ownership in secret tags rather than in the secret value itself.

- **Minimize secret size**: Keep secret values concise. Large payloads should be stored in Azure Storage with encryption, using a Key Vault key for encryption and a Key Vault secret for the storage access token.

## Secrets rotation

Secrets stored in application memory or configuration files persist for the entire application lifecycle, increasing exposure risk. Implement regular secret rotation to minimize compromise risk:

- **Rotate secrets regularly**: Rotate secrets at least every 60 days, or more frequently for high-security scenarios
- **Automate rotation**: Use Azure Key Vault's rotation capabilities to automate the rotation process
- **Use dual credentials**: For zero-downtime rotation, implement resources with two sets of authentication credentials

For more information about secrets rotation, see:
- [Automate secret rotation for resources with one set of authentication credentials](tutorial-rotation.md)
- [Automate secret rotation for resources with two sets of authentication credentials](tutorial-rotation-dual.md)

## Secrets caching and performance

Key Vault enforces service limits to prevent abuse. To optimize secrets access while maintaining security:

- **Cache secrets in memory**: Cache secrets in your application for at least 8 hours to reduce Key Vault API calls
- **Implement retry logic**: Use exponential back-off retry logic to handle transient failures and throttling
- **Refresh on rotation**: Update cached values when secrets are rotated to ensure applications use current credentials

For more information about throttling, see [Azure Key Vault throttling guidance](../general/overview-throttling.md).

## Secrets monitoring

Enable monitoring to track secret access patterns and detect potential security issues:

- **Enable Key Vault logging**: Log all secret access operations to detect unauthorized access attempts. See [Azure Key Vault logging](../general/logging.md)
- **Set up Event Grid notifications**: Monitor secret lifecycle events (created, updated, expired, near expiry) for automated workflows. See [Azure Key Vault as Event Grid source](/azure/event-grid/event-schema-key-vault)
- **Configure alerts**: Set up Azure Monitor alerts for suspicious access patterns or failed authentication attempts. See [Monitoring and alerting for Azure Key Vault](../general/alert.md)
- **Review access regularly**: Periodically audit who has access to secrets and remove unnecessary permissions

## Related security articles

- [Secure your Azure Key Vault](../general/secure-key-vault.md) - Comprehensive Key Vault security guidance
- [Secure your Azure Key Vault keys](../keys/secure-keys.md) - Security best practices for cryptographic keys
- [Secure your Azure Key Vault certificates](../certificates/secure-certificates.md) - Security best practices for certificates

## Next steps

- [About Azure Key Vault secrets](about-secrets.md)
- [Azure Key Vault developer's guide](../general/developers-guide.md)
- [Monitor Key Vault with Azure Monitor](../general/monitor-key-vault.md)

