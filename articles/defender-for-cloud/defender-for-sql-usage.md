---
title: Enable Microsoft Defender for SQL Servers on Machines
description: Learn how to protect your Microsoft SQL Servers on Azure VMs, on-premises, and in hybrid and multicloud environments with Microsoft Defender for Cloud.
ms.topic: how-to
ms.author: elkrieger
author: Elazark
ms.date: 04/27/2025
#customer intent: As a security administrator, I want to enable Defender for SQL servers on machines so that I can protect my SQL servers in various environments.
---

# Enable Defender for SQL Servers on Machines

> [!IMPORTANT]
> This article applies to commercial clouds. If you're using Government clouds, see the [Enable Defender for SQL servers on Machines government](defender-for-sql-usage-gov.md) article.

The Defender for SQL Servers on Machines plan is one of the Defender for Databases plans in Microsoft Defender for Cloud. Use Defender for SQL Servers on Machines to protect SQL virtual machines (VM) and Azure Arc SQL Server instances.

> [!IMPORTANT]
> The Defender for SQL Servers on Machines plan is undergoing a transition to the new agent architecture. For more information, see [Defender for SQL Servers on Machines plan transition](release-notes.md#update-to-defender-for-sql-servers-on-machines-plan).

## Prerequisites

- **Subscription permissions**: To deploy the plan on a subscription, including Azure Policy, you need **Subscription Owner** permissions.

- **SQL Server instance permissions**: SQL Server service accounts must be a member of the **sysadmin** fixed server role on each SQL Server instance, which is the default setting. Learn more about the [SQL Server service account requirement](/sql/sql-server/azure-arc/configure-least-privilege?view=sql-server-ver16). 

- **Supported Resources**: 
    - [SQL virtual machines](/azure/azure-sql/virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview?view=azuresql), and [Azure Arc SQL Server instances](/sql/sql-server/azure-arc/overview?view=sql-server-ver16) are supported.
    - On-premises machines must be [onboarded to Arc and registered as Azure Arc SQL Server instances](/azure/azure-arc/servers/learn/quick-enable-hybrid-vm).

**Communication**: Allow outbound HTTPS traffic on Transmission Control Protocol (TCP) port 443 using Transport Layer Security (TLS) to `*.<region>.arcdataservices.com` URL. Learn more about [URL requirements](/azure/azure-arc/servers/network-requirements#urls?tabs=azure-cloud).
  
- **Extensions**: Ensure these extensions aren't blocked in your environment. Learn more about [restricting extensions installation on Windows VMs](/azure/virtual-machines/extensions/extensions-rmpolicy-howto-ps).
    - **Defender for SQL (IaaS and Arc)**
        - Publisher: Microsoft.Azure.AzureDefenderForSQL
        - Type: AdvancedThreatProtection.Windows
    - **SQL IaaS Extension (IaaS)**
        - Publisher: Microsoft.SqlServer.Management
        - Type: SqlIaaSAgent
    - **SQL IaaS Extension (Arc)**
        - Publisher: Microsoft.AzureData
        - Type: WindowsAgent.SqlServer

- **Supported SQL Server versions** - SQL Server 2012 (11.x) and later versions.

- **Supported operating systems**- Windows Server 2012 R2 and later versions.

## Enable the plan

### Enable the plan on an Azure subscription

To enable the Defender for SQL servers on machines plan, you need to enable the Defender for Databases plan on your subscription. The Defender for SQL servers on machines plan is included in the Defender for Databases plan.

1. Sign in to the [Azure portal](https://portal.azure.com/).

1. Navigate to **Microsoft Defender for Cloud** > **Environment settings**.

1. Select the relevant subscription.

1. On the Defender plans page, locate the Databases plan and select **Select types**.

    :::image type="content" source="media/tutorial-enabledatabases-plan/select-types.png" alt-text="Screenshot that shows you where to select types on the Defender plans page." lightbox="media/tutorial-enabledatabases-plan/select-types.png":::

1. In the Resource types selection window, toggle the **SQL Servers on Machines** plan to **On**.

    :::image type="content" source="media/defender-for-sql-usage/sql-toggle-on.png" alt-text="Screenshot that shows where to toggle the Defender for SQL servers on machines, to on." lightbox="media/defender-for-sql-usage/sql-toggle-on.png":::

1. Select **Continue** > **Save**.

### Enable the plan on an Amazon Web Services (AWS) or Google Cloud Platform (GCP) subscription

To enable the Defender for SQL servers on machines plan, you need to enable the Defender for Databases plan on your subscription. The Defender for SQL servers on machines plan is included in the Defender for Databases plan.

1. Sign in to the [Azure portal](https://portal.azure.com/).

1. Search for and select **Microsoft Defender for Cloud**.

1. Select **Environment settings**.

1. Select the relevant AWS or GCP subscription.

1. On the Defender plans page, locate the Databases plan and select **Settings**.

1. In the SQL Servers on machines section, toggle the SQL Servers on machines plan to **On**.

    :::image type="content" source="media/defender-for-sql-usage/enable-on-aws.png" alt-text="Screenshot that shows where to locate the on button for Defender for SQL Servers on machines is located." lightbox="media/defender-for-sql-usage/enable-on-aws.png":::

1. Select **Save**.

## Enable the plan at the SQL Server resource level

Although we recommend enabling the plan for an entire Azure subscription, you might need to enable Defender for SQL on machines on specific machines only. To do this, [disable the plan on the subscription](disable-sql-on-machines.md) and follow instructions below on the resource level. 

1. In the Azure portal, search for and select:
    - **Azure Arc** > **Data services** > **SQL Server instances**.
    <br> 
    or
    - **SQL virtual machines**.

1. Select the relevant SQL Server instance. 

1. Locate the security menu and select **Microsoft Defender for Cloud**.

    :::image type="content" source="media/defender-for-sql-usage/select-defender-for-cloud.png" alt-text="Screenshot that shows where to locate Defender for Cloud under the security section." lightbox="media/defender-for-sql-usage/select-defender-for-cloud.png":::

1. Select **Enable Microsoft Defender for SQL servers on Machines**.

    :::image type="content" source="media/defender-for-sql-usage/enable-resource-level.png" alt-text="Screenshot that shows where to enable Defender for SQL servers on machines." lightbox="media/defender-for-sql-usage/enable-resource-level.png":::

## Verify that your machines are protected

> [!IMPORTANT]
>Don't skip this step, as it's important to verify your deployment is protected.

Depending on your environment, it can take a few hours to discover and protect SQL instances. As a final step, you should [verify that all machines are protected](verify-machine-protection.md).

## Related content

- [Verify that all machines are protected](verify-machine-protection.md)
- [Troubleshoot Defender for SQL on machines configuration](troubleshoot-sql-machines-guide.md)
- [Enable Microsoft Defender for SQL Servers on Machines at scale](enable-defender-sql-at-scale.md)
- [Disable Defender for SQL Servers on Machines](disable-sql-on-machines.md)
