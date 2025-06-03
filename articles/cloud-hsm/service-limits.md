---
title: Azure Cloud HSM Service Limits
description: Get detailed information on service limits for Azure Cloud HSM, including object limits and transaction limits for various cryptographic operations.
author: keithp
manager: davinune
ms.service: azure-cloud-hsm
ms.topic: reference
ms.date: 03/20/2025
ms.author: keithp
---

# Azure Cloud HSM service limits

This article describes service limits for the resource type `microsoft.hardwaresecuritymodules/cloudHsmClusters` in Azure Cloud HSM.

## Object limits

The following table describes the limits for the number of objects that you can create in Azure Cloud HSM. The limits are per Cloud HSM instance. Key types are Rivest-Shamir-Adleman (RSA), elliptic curve (EC), and Advanced Encryption Standard (AES).

> [!IMPORTANT]
> Key storage limits might vary if you're storing a mixed set of key types. Azure Cloud HSM supports a maximum of 3,200 key handles, irrespective of key type and size.

| Key type | Limit   |
| ------------- | -------------- |
| RSA keys  | Maximum of 1,600 RSA keys. Each key consumes two handles. |
| EC keys  | Maximum of 1,600 EC keys. Each key consumes two handles. |
| AES keys  | Maximum of 3,200 AES keys. Each key consumes one handle. |

## Transaction limits

The following tables describe the transaction limits for various cryptographic operations, measured in the number of operations per second per Cloud HSM instance.

Each Azure Cloud HSM instance constitutes three load-balanced hardware security module (HSM) partitions. The throughput limits are a function of underlying hardware capacity allocated for each partition. The following tables show maximum throughput with at least one partition available. Actual throughput might be up to three times higher if all three partitions are available.

Throughput limits noted in the tables assume that you use a single key to achieve maximum throughput. For example, if you use a single RSA-2048 key, the maximum throughput is 1,100 sign operations. If you use 1,100 keys with one transaction per second each, they won't achieve the same throughput.
  
### RSA key operations

The following table describes the number of operations per second for RSA key operations, categorized by key size.

| Operation | 2,048-bit   | 3,072-bit | 4,096-bit |
| ------------- | -------------- | ------------ |------------- |
| Create key | 1 | 1 | 1 |
| Encrypt | 12,000 | 8,800 | 5500 |
| Decrypt | 1,100 | 360 | 160 |
| Wrap | 12,000 | 9,200 | 5,700 |
| Unwrap | 1,100 | 360 | 160 |
| Sign | 1,100 | 360 | 160 |
| Verify | 12,000 | 9,200 | 5,700 |

### EC key operations

The following table describes the number of operations per second for EC key operations, categorized by key type.

| Operation | P-256 | P-256K | P-384 | P-521 | ED25519 |
| ------------- | ----------| ---------- | --------- | --------- | ----------- |
| Create key | 1 | 1 | 1 | 1 | 1 |
| Sign | 330 | 330 | 200 | 70 | 420 |
| Verify | 170 | 170 | 100 | 35 | 420 |

### AES key operations

The following table describes the number of operations per second for AES key operations, categorized by key size. In the table:

- Encrypt and decrypt operations assume a 4-KB packet size.
- Throughput limits for encrypt and decrypt apply to the AES-CBC and AES-GCM algorithms.
- Throughput limits for wrap and unwrap apply to the AES-KW algorithm.

| Operation | 128-bit | 192-bit | 256-bit |
| ------------- | ------------| ----------- | ----------- |
| Create key | 1 | 1 | 1 |
| Encrypt | 10,000 | 10,000 | 10,000 |
| Decrypt | 10,000 | 10,000 | 10,000 |
| Wrap | 10,000 | 10,000 | 10,000 |
| Unwrap | 10,000 | 10,000 | 10,000 |

## Related content

- [Azure Cloud HSM overview](overview.md)
- [Deploy Azure Cloud HSM by using the Azure portal](quickstart-portal.md)
- [Deploy Azure Cloud HSM by using Azure PowerShell](quickstart-powershell.md)
- [Azure Cloud HSM FAQs](faq.yml)
