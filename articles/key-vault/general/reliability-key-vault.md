---
title: Reliability in Azure Key Vault
description: Find out about reliability in Azure Key Vault, including availability zones and multi-region deployments.
author: msmbaldwin
ms.author: mbaldwin
ms.topic: reliability-article
ms.custom: subject-reliability, references_regions
ms.service: azure-key-vault
ms.date: 04/07/2025
#Customer intent: As an engineer responsible for business continuity, I want to understand the details of how Azure Key Vault works from a reliability perspective and plan disaster recovery strategies in alignment with the exact processes that Azure services follow during different kinds of situations. 
---

# Reliability in Azure Key Vault

> This article describes reliability support in Azure Key Vault, covering intra-regional resiliency via [availability zones](#availability-zone-support) and [multi-region deployments](#multi-region-support).
>
> Resiliency is a shared responsibility between you and Microsoft, and so this article also covers ways for you to create a resilient solution that meets your needs.

Azure Key Vault is designed to provide a secure service for managing your cryptographic keys and secrets that's highly available even during region-wide outages. This guide focuses on the reliability aspects of Azure Key Vault (excluding Managed HSM, which follows a different model).

> [!NOTE]
> This guide applies to vaults. Managed HSM pools use a different high availability and disaster recovery model; for more information, see [Managed HSM Disaster Recovery Guide](../managed-hsm/disaster-recovery-guide.md).

## Production deployment recommendations

- Enable soft-delete and purge protection on your key vault to protect against accidental or malicious deletion.
- Soft-delete allows recovery of deleted vaults and objects during a configurable retention period (7-90 days).
- Purge protection prevents permanent deletion until the retention period expires.
- For higher reliability, use Key Vault in regions that support automatic cross-region replication.
- Use Azure RBAC for access control to your key vault resources.
- Monitor your key vaults using Azure Monitor and diagnostic logs.

See [Secure your Azure Key Vault](secure-key-vault.md) for more information about how to secure your key vaults.

## Reliability architecture overview

By default, Azure Key Vault achieves redundancy by using multiple layers of redundancy to ensure that your keys and secrets remain available to your application even if individual components of the service fail.

Key Vault data is replicated both within the region and to the paired region for most Azure regions. This approach ensures high durability of your keys and secrets. Key Vault automatically handles component failures within a region by using alternate components, and it automatically routes requests to a secondary region if an entire region fails.

In regions with availability zones, if an availability zone is unavailable, Azure Key Vault automatically redirects your requests to another availability zone to ensure high availability.

## Transient faults

[!INCLUDE [Transient fault description](includes/reliability-transient-fault-description-include.md)]

Azure Key Vault is designed to handle transient faults transparently in most cases, but your application should still implement appropriate retry logic when making calls to Key Vault.

When interacting with Azure Key Vault, your application might encounter transient faults such as network timeouts, throttling due to rate limiting, or temporary service unavailability. To handle these scenarios, implement the following best practices:

- Use exponential backoff retry logic with jitter for Key Vault operations.
- Use the Azure SDK client libraries, which have built-in retry mechanisms.
- Be aware of Azure Key Vault service limits for requests per second to avoid throttling.
- Implement circuit-breaker patterns for handling extended service degradation.

## Availability zone support

[!INCLUDE [AZ support description](includes/reliability-availability-zone-description-include.md)]

Azure Key Vault uses availability zones in regions where they're available to provide higher availability and resilience against availability zone failures.

When using Key Vault in regions with availability zones, your Key Vault resources are automatically distributed across zones for high availability. If a zone becomes unavailable, Azure Key Vault continues to function without interruption by using the resources in the available zones.

### Region support

Zone-redundant Azure Key Vault can be deployed into any region that supports availability zones.

### Requirements

Zone redundancy is available for all tiers of Azure Key Vault. There are no special SKU requirements to enable zone redundancy.

### Considerations

During an availability zone outage, your Key Vault continues to operate and serve traffic. All operations remain available, though some operations might experience slightly increased latency.

### Cost

There is no additional cost for zone redundancy in Azure Key Vault. The pricing remains the same regardless of whether your key vault is deployed in a region with or without availability zones.

### Configure availability zone support

For Azure Key Vault, no special configuration is required to enable availability zone support. When you create a key vault in a region that supports availability zones, Azure Key Vault automatically distributes your resources across zones.

### Data replication between zones

When a client changes any data in your Azure Key Vault, that change is applied to all instances in all zones simultaneously. This approach is called *synchronous replication.* Synchronous replication ensures a high level of data consistency, which reduces the likelihood of data loss during a zone failure. Availability zones are located relatively close together, which means there's minimal effect on latency or throughput.

### Zone-down experience

- **Detection and response**: The Azure Key Vault platform is responsible for detecting a failure in an availability zone. You don't need to do anything to initiate a zone failover.
- **Notification**: To determine when a zone failure occurred, check Azure Service Health or Resource Health.
- **Active requests**: Any in-flight requests that were being processed by the unavailable zone might fail and should be retried by the client.
- **Expected data loss**: A zone failure isn't expected to cause any data loss due to the synchronous replication between zones.
- **Expected downtime**: A zone failure isn't expected to cause downtime to your Key Vault resources.
- **Traffic rerouting**: When a zone is unavailable, Azure Key Vault automatically detects the loss and routes new requests to the available zones.

### Failback

When the availability zone recovers, Azure Key Vault automatically restores operations to that zone and continues normal operation across all available zones.

### Testing for zone failures  

The Azure Key Vault platform manages traffic routing, failover, and failback for zone-redundant resources. You don't need to initiate anything. Because this feature is fully managed, you don't need to validate availability zone failure processes.

## Multi-region support

Azure Key Vault provides built-in multi-region support through automatic data replication to paired regions for most Azure regions.

The contents of your key vault are replicated both within the region and to the paired region, which is usually at least 150 miles away but within the same geography. This approach ensures high durability of your keys and secrets.

> [!IMPORTANT]
> Cross-region failover is not supported in the following regions:
>
> - Brazil South (which is paired to a region in another geography)
> - Brazil Southeast
> - West US 3
> - Any region that doesn't have a paired region
>
> For these regions, you'll need to implement your own backup and recovery strategy.

### Region support

Azure Key Vault's multi-region support relies on Azure region pairs. A key vault created in a primary region automatically replicates to its paired region.

For more information about Azure region pairs, see [Azure paired regions](/azure/reliability/cross-region-replication-azure).

### Requirements 

Multi-region support is automatically available for all Azure Key Vault tiers in supported regions. There are no special SKU requirements for enabling multi-region support.

### Considerations

While Key Vault automatically replicates data to paired regions, there are some limitations:

- During a region failover, your key vault is in read-only mode. The following operations are supported in read-only mode:
  - List certificates
  - Get certificates
  - List secrets
  - Get secrets
  - List keys
  - Get (properties of) keys
  - Encrypt
  - Decrypt
  - Wrap
  - Unwrap
  - Verify
  - Sign
  - Backup
- You won't be able to make changes to key vault properties, access policies, or firewall configurations during failover.
- In the event of a region failover, it may take a few minutes for the service to fail over. Requests made during this time before failover may fail.
- If you're using private link to connect to your key vault, it may take up to 20 minutes for the connection to be re-established in the event of a region failover.

### Cost

There is no additional charge for multi-region replication in Azure Key Vault. The standard Key Vault pricing applies.

### Configure multi-region support 

For Azure Key Vault, no special configuration is required to enable multi-region replication. When you create a key vault in a region with a paired region, Azure Key Vault automatically replicates your data to the paired region.

### Data replication between regions 

When a client changes any data in your Azure Key Vault, that change is applied to the primary region. At that point, the write is considered to be complete. Later, the data in the secondary region is automatically updated with the change. This approach is called *asynchronous replication.* Asynchronous replication ensures high performance and throughput. However, any data that wasn't replicated between regions could be lost if the primary region experiences a failure.

### Region-down experience 

- **Detection and response**: Azure Key Vault is responsible for detecting a failure in a region and automatically failing over to the secondary region.
- **Notification**: To determine when a region failure occurred, check Azure Service Health or Resource Health.
- **Active requests**: Any active requests to the unavailable region might fail and should be retried by the client.
- **Expected data loss**: You might lose some data during a region failure if that data hasn't yet synchronized to the paired region.
- **Expected downtime**: Your Key Vault resource might be unavailable for a few minutes during the region failover process.
- **Traffic rerouting**: When a region failover occurs, Key Vault automatically routes traffic to the secondary region.

### Failback

When the primary region recovers, Azure Key Vault automatically fails back to the primary region and resumes normal operations. After failback, all request types (including both read and write requests) become available.

### Testing for region failures  

The Azure Key Vault platform manages traffic routing, failover, and failback for multi-region resources. You don't need to initiate anything. Because this feature is fully managed, you don't need to validate region failure processes.

### Alternative multi-region approaches 

For regions without built-in multi-region support (Brazil South, Brazil Southeast, West US 3, and regions without pairs), you can implement your own backup and recovery strategy.

To back up and restore your Azure key vault to a region of your choice, complete the steps detailed in [Azure Key Vault backup](backup.md).

## Backups

Azure Key Vault provides the ability to back up secrets, keys, and certificates individually as encrypted blobs. These backups can be restored to any key vault within the same Azure subscription and Azure geography.

When you back up a key vault object, such as a secret, key, or certificate, the backup operation will download the object as an encrypted blob. This blob can't be decrypted outside of Azure. To get usable data from this blob, you must restore the blob into a key vault within the same Azure subscription and Azure geography.

> For most solutions, you shouldn't rely exclusively on backups. Instead, use the other capabilities described in this guide to support your resiliency requirements. However, backups protect against some risks that other approaches don't. For more information, see [Azure Key Vault backup](backup.md).

## Service-level agreement

> The service-level agreement (SLA) for Azure Key Vault describes the expected availability of the service, and the conditions that must be met to achieve that availability expectation. For more information, see [SLA for Azure Key Vault](https://azure.microsoft.com/support/legal/sla/key-vault/).

Azure Key Vault's SLA guarantees a high level of availability. The key points to understand about the Key Vault SLA include:

- Valid API requests to Key Vault will be processed successfully.
- The SLA covers both the basic management operations and key/secret operations.
- The SLA does not cover operations during periods of read-only mode due to region failover.

## Related content

- [Azure Key Vault availability and redundancy](disaster-recovery-guidance.md)
- [Azure Key Vault backup](backup.md)
- [Azure Key Vault soft-delete overview](./soft-delete-overview.md)
- [Reliability in Azure](/azure/reliability/overview)
