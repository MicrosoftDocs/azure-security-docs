---
title: Data residency for Azure Confidential Ledger
description: Describes data residency posture for Azure Confidential Ledger.
services: confidential-ledger
author: tachou

ms.service: azure-confidential-ledger
ms.topic: conceptual
ms.date: 10/21/2025
ms.author: tachou
---

# Data residency for Azure Confidential Ledger

This article describes the data residency and resiliency behavior for Azure Confidential Ledger.

## About data residency for Azure Confidential Ledger

Azure Confidential Ledger uses geo-redundant storage (GRS) and Azure File Storage to ensure data durability and availability. The service leverages [Azure Regional Pairs](/azure/reliability/cross-region-replication-azure#azure-paired-regions) for data replication across all regions where the service is available.

When you create an Azure Confidential Ledger resource, your data is automatically replicated to the paired region to provide protection against regional outages. This replication behavior is built into the service and cannot be modified or disabled.

### Regional pairs and data replication

Azure Confidential Ledger stores and processes customer data using the following approach:

- **Primary region**: Your ledger data is stored in the primary region you select when creating the resource.
- **Paired region**: Data is automatically replicated to the Azure regional pair for disaster recovery and high availability purposes.

For example:

- If you create a ledger in West US, data is automatically replicated to East US.

To understand which regions are paired, see [Azure paired regions](/azure/reliability/cross-region-replication-azure#azure-paired-regions).

### Data residency considerations

Unlike some Azure services, Azure Confidential Ledger does not currently offer an option to restrict data to a single region. Data replication to the paired region is a core part of the service's reliability and disaster recovery design.

Organizations with strict data residency requirements should:

1. Review the regional pairs to understand where data will be replicated.
2. Ensure that both the primary and paired regions meet their compliance and regulatory requirements.
3. Select a primary region whose paired region aligns with their data sovereignty needs.

### Service resiliency

Azure Confidential Ledger is designed to be resilient to:

- **Zone-wide outages**: The service uses availability zones within a region where available.
- **Region-wide outages**: Automatic replication to the paired region ensures data durability and enables disaster recovery.

In the event of a region-wide outage affecting your primary region, Microsoft may initiate a failover to the paired region to restore service availability.

## Data stored by Azure Confidential Ledger

Azure Confidential Ledger stores the following types of data:

- **Ledger entries**: All append-only ledger transactions and data submitted by users.
- **Service metadata**: Configuration information and metadata required for ledger operation.

All data is encrypted at rest and in transit. The confidential nature of the ledger is maintained through hardware-backed trusted execution environments (TEEs), ensuring data integrity and confidentiality.

## Related services

Azure Confidential Ledger may integrate with other Azure services that have their own data residency policies:

- **Azure Storage**: Used for backing ledger data with GRS replication.
- **Azure Key Vault**: May be used for key management, which has its own data residency characteristics.
- **Azure Monitor**: If enabled, diagnostic logs may be stored according to your Log Analytics workspace configuration.

Review the data residency documentation for these services if you use them with Azure Confidential Ledger.

## Next steps

- Learn more about [Azure Confidential Ledger](/azure/confidential-ledger/overview)
- Learn more about [Azure data residency requirements](https://azure.microsoft.com/global-infrastructure/data-residency/)
- Understand [Azure paired regions](/azure/reliability/cross-region-replication-azure#azure-paired-regions)
