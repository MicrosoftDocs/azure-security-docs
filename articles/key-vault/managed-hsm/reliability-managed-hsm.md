---
title: Reliability in Azure Managed HSM
description: Find out about reliability in Azure Managed HSM, including availability zones and multi-region deployments.
author: anaharris-ms
ms.author: anaharris
ms.topic: reliability-article
ms.custom: subject-reliability, references_regions
ms.service: azure-key-vault
ms.date: 11/12/2024
#Customer intent: As an engineer responsible for business continuity, I want to understand the details of how Azure Managed HSM works from a reliability perspective and plan disaster recovery strategies in alignment with the exact processes that Azure services follow during different kinds of situations.

---

# Reliability in Azure Managed HSM

> This article describes reliability support in Azure Managed HSM, covering intra-regional resiliency via [availability zones](#availability-zone-support) and [multi-region deployments](#multi-region-support).
>
> Resiliency is a shared responsibility between you and Microsoft, and so this article also covers ways for you to create a resilient solution that meets your needs.

## Production deployment recommendations

TODO

## Reliability architecture overview

Azure Managed HSM provides redundancy by distributing HSM partitions across multiple nodes within a region. This ensures high availability and fault tolerance for cryptographic operations. For multi-region deployments, Managed HSM supports automated replication of key material, roles, and permissions between primary and extended regions.

## Transient faults

[!INCLUDE [Transient fault description](includes/reliability-transient-fault-description-include.md)]

Managed HSM handles transient faults by ensuring that cryptographic operations are retried automatically. Customers should implement retry logic in their applications to handle transient errors gracefully.

## Availability zone support

[!INCLUDE [AZ support description](includes/reliability-availability-zone-description-include.md)]

Azure Managed HSM supports availability zones in regions where they are available. Zone redundancy ensures that HSM partitions are distributed across multiple zones, providing resilience against zone-level failures.

### Region support

Zone-redundant Managed HSM resources can be deployed into any region that supports availability zones, except for specific regions where capacity limitations exist.

| Americas         | Europe               | Middle East   | Africa             | Asia Pacific   |
|------------------|----------------------|---------------|--------------------|----------------|
| US East          | France Central       | Qatar Central | South Africa North | Australia East |
| US East 2        | Germany West Central |               |                    | Japan East     |
| US West          | North Europe         |               |                    | Southeast Asia |
| Canada Central   | UK South             |               |                    | Central India  |

### Requirements

You must use the appropriate SKU to enable zone redundancy.

### Considerations

During an availability zone outage, Managed HSM continues to serve requests from other zones. However, some operations may experience latency due to failover.

### Cost

Zone redundancy incurs additional costs. For more information, see [Azure Managed HSM pricing](https://azure.microsoft.com/pricing/details/key-vault).

### Configure availability zone support

Zone redundancy can only be configured during the creation of a Managed HSM resource. Existing resources cannot be converted to use availability zones.

### Capacity planning and management

TODO

### Traffic routing between zones

Requests are automatically routed to the closest available zone. Traffic Manager ensures that requests are served from healthy zones during outages.

### Data replication between zones

Managed HSM replicates data synchronously across zones, ensuring consistency and minimizing data loss during zone failures.

### Zone-down experience

- **Detection and response**: Microsoft detects zone failures and initiates failover automatically.
- **Notification**: Customers can monitor Resource Health for zone failure notifications.
- **Active requests**: Active requests are retried automatically.
- **Expected data loss**: No data loss is expected due to synchronous replication.
- **Expected downtime**: Minimal downtime during failover.
- **Traffic rerouting**: Traffic is rerouted to healthy zones automatically.

### Failback

Failback is managed by Microsoft. When the affected zone recovers, traffic is rerouted, and normal operations resume.

### Testing for zone failures

You can simulate zone failures using Azure Chaos Studio to validate your application's resilience.

## Multi-region support

Azure Managed HSM supports multi-region replication, enabling customers to extend their HSM pools to an additional region for enhanced availability and disaster recovery.

### Region support

Multi-region replication supports a wide range of Azure regions. However, some regions may not be available as extended regions due to capacity limitations.

### Requirements

You must use the appropriate SKU to enable multi-region replication.

### Considerations

Write operations may take up to six minutes to replicate between regions. Customers should account for this delay in their applications.

### Cost

Multi-region replication incurs additional costs as a new HSM pool is created in the extended region. For more information, see [Azure Managed HSM pricing](https://azure.microsoft.com/pricing/details/key-vault).

### Configure multi-region support

To enable multi-region replication, use the Azure CLI command:

```azurecli-interactive
az keyvault region add --hsm-name "ContosoMHSM" --region "australiaeast"
```

### Capacity planning and management

TODO

### Traffic routing between regions

Requests are routed to the closest available region using Traffic Manager. During failover, requests are redirected to the healthy region.

### Data replication between regions

Managed HSM replicates data asynchronously between regions. Write operations are first applied in the primary region and then propagated to the extended region.

### Region-down experience

- **Detection and response**: Microsoft detects region failures and initiates failover automatically.
- **Notification**: Customers can monitor Service Health for region failure notifications.
- **Active requests**: Active requests are retried automatically.
- **Expected data loss**: Data not yet replicated to the extended region may be lost.
- **Expected downtime**: Minimal downtime during failover.
- **Traffic rerouting**: Traffic is rerouted to the healthy region automatically.

### Failback

Failback is managed by Microsoft. When the affected region recovers, traffic is rerouted, and normal operations resume.

### Testing for region failures

You can simulate region failures using Azure Chaos Studio to validate your application's resilience.

### Alternative multi-region approaches

For additional resiliency, customers can deploy separate Managed HSM resources in multiple regions and configure load balancing and failover policies.

## Backups

Managed HSM supports full backups and selective key restores. Backups are encrypted and stored in customer-designated storage accounts. Backups are region-specific and cannot be accessed across regions.

> For most solutions, you shouldn't rely exclusively on backups. Instead, use the other capabilities described in this guide to support your resiliency requirements. However, backups protect against some risks that other approaches don't. For more information, see [Azure Managed HSM backup and restore](backup-restore.md).

## Service-level agreement

> The service-level agreement (SLA) for Azure Managed HSM describes the expected availability of the service and the conditions that must be met to achieve that availability expectation. For more information, see [SLA for Azure Managed HSM](https://azure.microsoft.com/support/legal/sla/key-vault-managed-hsm/v1_0/).

## Related content

- [Reliability in Azure](https://learn.microsoft.com/azure/availability-zones/overview)
- [Azure Managed HSM backup and restore](backup-restore.md)
- [Azure Managed HSM multi-region replication](multi-region-replication.md)
- [Azure Managed HSM disaster recovery guide](disaster-recovery-guide.md)



