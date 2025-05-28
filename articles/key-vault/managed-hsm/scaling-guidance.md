---
title: Azure Managed HSM Scaling Guidance
description: Learn about capacity planning for Azure Key Vault Managed HSM, including benchmark performance numbers for cryptographic operations.
ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: concept-article
author: msmbaldwin
ms.author: mbaldwin
ms.date: 05/16/2023

---

# Azure Managed HSM Scaling Guidance

This document describes how to correctly plan capacity for Azure Managed HSM. 

## Common customer scenarios for capacity planning

### Customer Managed Keys Encryption at Rest

Azure services that use customer-managed keys perform a small number of get key, wrap, and unwrap operations, then cache the results while periodically checking access permissions. This means only a few calls are made per hour. Because of this behavior, it is very unlikely that you will reach the throughput limits described in this document when using Managed HSM for encryption at rest.

### Custom Applications

For customers using Managed HSM for custom applications, the benchmark numbers can be used as approximate expectations for architecting the custom application. Managed HSM does not throttle on cryptographic operation transactions, and uses the HSM hardware and CPUs to their fullest. The numbers should be used as guidance for capacity planning, though we recommend load testing and validating performance numbers. The numbers are an inherent limit of the HSM and its current firmware.

## Planning for transaction capacity for cryptographic operations

Each Managed HSM instance constitutes three load balanced HSM partitions. The throughput limits are a function of underlying hardware capacity allocated for each partition. The tables show internally measured approximate maximum throughput with at least one partition available. Actual throughput may be up to 3x higher if all three partitions are available. Partitions are occasionally taken down while healing failing hardware, patching servers, and during other service code updates. For capacity planning, it is safe to assume that 2 partitions will be available. If you need a guaranteed number of operations per second, capacity plan with only one partition available in mind.

The benchmark numbers were calculated from internal performance testing one key operation a time on our current hardware with its current firmware. Each key operation was run against a single-partition MHSM pool, using the same key for each request. The numbers shown are the average number of operations per second, sustained over 5 minutes.

These numbers assume that one single key is being used to achieve maximum throughput. For example, if a single RSA-2048 key is used, the maximum throughput is 900 sign operations. If you use 900 different keys with one transaction per second each, they will not be able to achieve the same throughput.

> [!NOTE]
> For accuracy, we recommend testing your application workloads to validate performance numbers and determine scaling needs.

### RSA key operations (number of operations per second per HSM instance)

| Operation | 2048-bit | 3072-bit | 4096-bit |
|-----------|----------|----------|----------|
| Create Key | 1 | 1 | 1 |
| Delete Key (soft-delete) | 10 | 10 | 10 |
| Purge Key | 10 | 10 | 10 |
| Backup Key | 10 | 10 | 10 |
| Restore Key | 10 | 10 | 10 |
| Get Key Information | 1100 | 1100 | 1100 |
| Encrypt | 2800 | 2700 | 2300 |
| Decrypt | 1100 | 360 | 160 |
| Wrap | 2200 | 1900 | 1900 |
| Unwrap | 1100 | 360 | 160 |
| Sign | 900 | 340 | 150 |
| Verify | 3400 | 3400 | 3700 |

### EC key operations (number of operations per second per HSM instance)

This table describes number of operations per second for each curve type.

| Operation | P-256 | P-256K | P-384 | P-521 |
|-----------|-------|--------|-------|-------|
| Create Key | 1 | 1 | 1 | 1 |
| Delete Key (soft-delete) | 10 | 10 | 10 | 10 |
| Purge Key | 10 | 10 | 10 | 10 |
| Backup Key | 10 | 10 | 10 | 10 |
| Restore Key | 10 | 10 | 10 | 10 |
| Get Key Information | 1100 | 1100 | 1100 | 1100 |
| Sign | 330 | 330 | 160 | 200 |
| Verify | 130 | 130 | 82 | 28 |

### AES key operations (number of operations per second per HSM instance)

Encrypt and Decrypt operations assume a 4-KB packet size.
Throughput limits for Encrypt/Decrypt apply to AES-CBC and AES-GCM algorithms.
Throughput limits for Wrap/Unwrap apply to AES-KW algorithm.

| Operation | 128-bit | 192-bit | 256-bit |
|-----------|---------|---------|---------|
| Create Key | 1 | 1 | 1 |
| Delete Key (soft-delete) | 10 | 10 | 10 |
| Purge Key | 10 | 10 | 10 |
| Backup Key | 10 | 10 | 10 |
| Restore Key | 10 | 10 | 10 |
| Get Key Information | 1100 | 1100 | 1100 |
| Encrypt | 2000 | 2500 | 2200 |
| Decrypt | 1800 | 1600 | 3200 |
| Wrap | 2100 | 2100 | 2100 |
| Unwrap | 1800 | 1900 | 2200 |

### Transaction limits for administrative operations (number of concurrent operations per HSM instance)

| Operation | Number of concurrent operations per HSM instance |
|-----------|------------------------------------------------|
| Full HSM Backup/Restore<br>(only one concurrent backup or restore operation per HSM instance supported) | 1 |

For Azure Managed HSM service limits, see [Service limits](service-limits.md).

