---
title: Azure Key Vault service limits
description: Review the service limits for Azure Key Vault, including transaction throughput, API request limits, and Managed HSM capacity.
ms.service: azure-key-vault
ms.subservice: general
ms.topic: reference
ms.date: 07/20/2025
author: msmbaldwin
ms.author: mbaldwin
---
# Azure Key Vault service limits

Azure Key Vault service supports two resource types: Vaults and Managed HSMs. The following two sections describe the service limits for each of them respectively.

## Key Vault

The following limits apply to standard Key Vault operations including transaction rate limits for keys, secrets, and certificates. Azure Key Vault includes limits on transaction throughput and API requests to ensure service reliability and availability. These limits are designed to help identify applications that may negatively impact other Key Vault customers while still allowing you to meet your operational requirements. For information on handling throttling when these limits are exceeded, see [Azure Key Vault throttling guidance](overview-throttling.md).

[!INCLUDE [key-vault-limits](~/reusable-content/ce-skilling/azure/includes/key-vault/key-vault-service-limits.md)]

## Key Vault: Managed HSM

Managed HSM provides dedicated HSM instances with different limits than standard Key Vault vaults. For detailed information about performance characteristics and capacity planning for your Managed HSM workloads, see [Azure Managed HSM scaling guidance](../managed-hsm/scaling-guidance.md).

[!INCLUDE [key-vault-limits](~/reusable-content/ce-skilling/azure/includes/key-vault/managed-hsm-service-limits.md)]
