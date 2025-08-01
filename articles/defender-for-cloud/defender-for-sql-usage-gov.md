---
title: Enable Microsoft Defender for SQL Servers on Machines government
description: Learn how to protect your Microsoft SQL Servers on Azure VMs, on government clouds with Microsoft Defender for Cloud.
ms.topic: how-to
ms.author: elkrieger
author: Elazark
ms.date: 04/23/2025
#customer intent: As a security administrator, I want to enable Defender for SQL servers on machines so that I can protect my SQL servers in various environments.
---

# Enable Defender for SQL Servers on Machines government

> [!IMPORTANT]
> This article applies to government clouds. If you're using commercial clouds, see the [Enable Defender for SQL servers on Machines](defender-for-sql-usage.md) article.

The Defender for SQL Servers on Machines plan is one of the Defender for Databases plans in Microsoft Defender for Cloud. Use Defender for SQL Servers on Machines to protect SQL Server databases hosted on Azure VMs and Azure Arc-enabled VMs.

## Prerequisites

| Requirement | Details |
|-------------|---------|
| **Permissions** | To deploy the plan on a subscription including Azure Policy, you need **Subscription Owner** permissions. <br> <br> The Windows user on the SQL VM must have the **Sysadmin** role on the database. |
| **Multicloud machines** | Multicloud machines (AWS and GCP) must be onboarded as Azure Arc-enabled VMs. They can be automatically onboarded as Azure Arc machines when onboarded with the connector. <br> [Onboard your AWS connector](quickstart-onboard-aws.md) and automatically provision Azure Arc. <br> [Onboard your GCP connector](quickstart-onboard-gcp.md) and automatically provision Azure Arc. |
| **On-premises machines** | On-premises machines must be onboarded as Azure Arc-enabled VMs. [Onboard on-premises machines and install Azure Arc](/azure/azure-arc/servers/learn/quick-enable-hybrid-vm). |
| **Azure Arc** | Review Azure Arc deployment requirements <br> - [Plan and deploy Azure Arc-enabled servers](/azure/azure-arc/servers/plan-at-scale-deployment) <br> - [Connected Machine agent prerequisites](/azure/azure-arc/servers/prerequisites) <br> - [Connected Machine agent network requirements](/azure/azure-arc/servers/network-requirements) <br> - [Roles specific to SQL Server enabled by Azure Arc](/sql/relational-databases/security/authentication-access/server-level-roles#roles-specific-to-sql-server-enabled-by-azure-arc) |
| **Extensions**| Ensure these extensions aren't blocked in your environment. | 
| Defender for SQL (IaaS and Arc)| - Publisher: Microsoft.Azure.AzureDefenderForSQL<br>  - Type: AdvancedThreatProtection.Windows |
| SQL IaaS Extension (IaaS)| - Publisher: Microsoft.SqlServer.Management<br>  - Type: SqlIaaSAgent |
| SQL IaaS Extension (Arc)| - Publisher: Microsoft.AzureData<br>  - Type: WindowsAgent.SqlServer|
| AMA extension (IaaS and Arc) | - Publisher: Microsoft.Azure.Monitor<br>  - Type: AzureMonitorWindowsAgent |
| **Region requirement** | When you enable the plan, a resource group is created in the East US. Ensure this region isn't blocked in your environment. |
| **Resource naming conventions** | Defender for SQL uses the following naming convention when creating our resources: <br> - Data Collection Rule: `MicrosoftDefenderForSQL--dcr` <br> - DCRA: `/Microsoft.Insights/MicrosoftDefenderForSQL-RulesAssociation` <br> - Resource group: `DefaultResourceGroup-` <br> - Log analytics workspace: `D4SQL--` <br> - Defender for SQL uses *MicrosoftDefenderForSQL* as a *createdBy* database tag. <br><br> Ensure that Deny policies don't block this naming convention. |
| **Supported SQL Server Versions**| SQL Server 2012 or later is supported for SQL instances. |
| **Supported Operating Systems**| Windows Server 2012 R2 or later. |

## Enable the plan

1. In the Azure portal, search for and select **Microsoft Defender for Cloud**.

1. In the Defender for Cloud menu, select **Environment settings**.

1. Select the relevant subscription.

1. On the Defender plans page, locate the Databases plan and select **Select types**.

    :::image type="content" source="media/tutorial-enabledatabases-plan/select-types.png" alt-text="Screenshot that shows you where to select types on the Defender plans page." lightbox="media/tutorial-enabledatabases-plan/select-types.png":::

1. In the Resource types selection window, toggle the **SQL Servers on Machines** plan to **On**.

1. Select **Continue** > **Save**.

## Select a workspace

Select a Log Analytics workspace to work with the Defender for SQL on Machines plan.

1. In the **Defender plans** page, in **Databases**, **Monitoring Coverage** column select **Settings**.

1. In the **Azure Monitoring Agent for SQL Server on Machines** section, in the **Configurations** column select **Edit Configurations**.

1. In the **Autoprovisioning Configuration** page, select the **Default Workspace** or specify a **Custom Workspace**.

1. In SQL Server automatic registration, make sure that you leave the **Register Azure SQL Server instances by enabling SQL IaaS extension automatic registration** option enabled.

    :::image type="content" source="media/defender-for-sql-usage-gov/leave-enabled.png" alt-text="Screenshot that shows where to leave the register Azure SQL Server instances enabled." lightbox="media/defender-for-sql-usage-gov/leave-enabled.png":::

    Registration ensures that all SQL instances can be discovered and configured correctly.

1. Select **Apply**.

## Verify that your machines are protected

Depending on your environment, it can take a few hours to discover and protect SQL instances. As a final step, you should [verify that all machines are protected](verify-machine-protection-gov.md). Don't skip this step, as it's important to verify your deployment is protected.
