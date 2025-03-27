---
title: Azure Cloud HSM Service Limits
description: Detailed information on service limits for Azure Cloud HSM, including object limits and transaction limits for various cryptographic operations.
author: keithp
manager: davinune
ms.service: azure-cloud-hsm
ms.topic: reference
ms.date: 03/20/2025
ms.author: keithp
---

# Azure Cloud HSM service limits

Resource type: `microsoft.hardwaresecuritymodules/cloudHsmClusters`.

This section describes service limits for resource type `cloudHsmClusters`.

## Object limits

The following table describes the limits for the number of objects that can be created in Azure Cloud HSM. The limits are per Cloud HSM instance.

> [!IMPORTANT]
> Key storage limits might vary if storing a mixed set of key types. Azure Cloud HSM supports a maximum of 3200 key handles, irrespective of key type and size.

| **Key Type** | **Limits**   |
| ------------- | -------------- |
| **RSA keys**  | Maximum of 1600 RSA keys (each key consumes two handles). | 
| **EC keys**  | Maximum of 1600 EC keys (each key consumes two handles). | 
| **AES keys**  | Maximum of 3200 AES keys (each key consumes one handle). | 

## Transaction limits

The following tables describe the transaction limits for various cryptographic operations, measured in the number of operations per second per Cloud HSM instance.

- Each Azure Cloud HSM instance constitutes three load balanced HSM partitions. The throughput limits are a function of underlying hardware capacity allocated for each partition. The following tables show maximum throughput with at least one partition available. Actual throughput may be up to 3x higher if all three partitions are available.
- Throughput limits noted assume that one single key is being used to achieve maximum throughput. For example, if a single RSA-2048 key is used, the maximum throughput is 1100 sign operations. If you use 1100 different keys with one transaction per second each, they will not achieve the same throughput.
  
### RSA key operations (number of operations per second per Cloud HSM instance)

The following table describes the number of operations per second for RSA key operations, categorized by key size.

| **Operation** | **2048-bit**   | **3072-bit** | **4096-bit** |
| ------------- | -------------- | ------------ |------------- |
| Create Key | 1 | 1 | 1 |
| Encrypt | 12000 | 8800 | 5500 |
| Decrypt | 1100 | 360 | 160 |
| Wrap | 12000 | 9200 | 5700 |
| Unwrap | 1100 | 360 | 160 |
| Sign | 1100 | 360 | 160 |
| Verify | 12000 | 9200 | 5700 |
### EC key operations (number of operations per second per Cloud HSM instance)

The following table describes the number of operations per second for EC key operations, categorized by key type.

| **Operation** | **P-256** | **P-256K** | **P-384** | **P-521** | **ED25519** |
| ------------- | ----------| ---------- | --------- | --------- | ----------- |
| Create Key | 1 | 1 | 1 | 1 | 1 |
| Sign | 330 | 330 | 200 | 70 | 420 |
| Verify | 170 | 170 | 100 | 35 | 420 |

### AES key operations (number of operations per second per Cloud HSM instance)

The following table describes the number of operations per second for AES key operations, categorized by key size. 

- Encrypt and Decrypt operations assume a 4KB packet size.
- Throughput limits for Encrypt/Decrypt apply to AES-CBC and AES-GCM algorithms.
- Throughput limits for Wrap/Unwrap apply to AES-KW algorithm.

| **Operation** | **128-bit** | **192-bit** | **256-bit** |
| ------------- | ------------| ----------- | ----------- | 
| Create Key | 1 | 1 | 1 |
| Encrypt | 10000 | 10000 | 10000 |
| Decrypt | 10000 | 10000 | 10000 |
| Wrap | 10000 | 10000 | 10000 |
| Unwrap | 10000 | 10000 | 10000 |

## Next steps
- [Azure Cloud HSM overview](overview.md)
- [Deploying Azure Cloud HSM](tutorial-deploy-cloud-hsm.md)
- [Azure Cloud HSM FAQs](faq.yml)
