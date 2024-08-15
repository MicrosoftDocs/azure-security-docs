---
title: Prepare for retirement of the Log Analytics agent 
description: Learn how to prepare for the deprecation of the Log Analytics (MMA) agent in Microsoft Defender for Cloud.
ms.topic: how-to
ms.date: 03/13/2024
# customer intent: As a user, I want to understand how to prepare for the retirement of the Log Analytics agent in Microsoft Defender for Cloud.
---

# Prepare for retirement of the Log Analytics agent

The Log Analytics agent, also known as the Microsoft Monitoring Agent (MMA), [is retiring in August 2024](https://techcommunity.microsoft.com/t5/microsoft-defender-for-cloud/microsoft-defender-for-cloud-strategy-and-plan-towards-log/ba-p/3883341). As a result, the Defender for Servers and Defender for SQL on machines plans in Microsoft Defender for Cloud will be updated, and features that rely on the Log Analytics agent will be redesigned.

This article summarizes plans for agent retirement.

## Preparing Defender for Servers

The Defender for Servers plan uses the Log Analytics agent in general availability (GA) and in AMA for [some features](plan-defender-for-servers-agents.md) (in preview). Here's what's happening with these features going forward:

To simplify onboarding, all Defender for Servers security features and capabilities will be provided with a single agent ([Microsoft Defender for Endpoint](integration-defender-for-endpoint.md)), complemented by [agentless machine scanning](concept-agentless-data-collection.md), without any dependency on Log Analytics agent or AMA.

- Defender for Servers features, which are based on AMA, are currently in preview and won’t be released in GA.  
- Features in preview that rely on AMA remain supported until an alternate version of the feature is provided, which will rely on the Defender for Endpoint integration or the agentless machine scanning feature.
- By enabling the Defender for Endpoint integration and agentless machine scanning feature before the deprecation takes place, your Defender for Servers deployment will be up to date and supported.

### Feature functionality

The following table summarizes how Defender for Servers features will be provided. Most features are already generally available using Defender for Endpoint integration or agentless machine scanning. The rest of the features will either be available in GA by the time the MMA is retired, or will be deprecated.

| Feature | Current support | New support | New experience status |
|----|----|----|----|
| Defender for Endpoint integration for down-level Windows machines (Windows Server 2016/2012 R2)  | Legacy Defender for Endpoint sensor, based on the Log Analytics agent | [Unified agent integration](/microsoft-365/security/defender-endpoint/configure-server-endpoints) | - Functionality with the MDE unified agent is GA.<br/>- Functionality with the legacy Defender for Endpoint sensor using the Log Analytics agent will be deprecated in August 2024.|
| OS-level threat detection | Log Analytics agent | Defender for Endpoint agent integration | Functionality with the Defender for Endpoint agent is GA. |
| Adaptive application controls | Log Analytics agent (GA), AMA (Preview) | --- | The adaptive application control feature is set to be deprecated in August 2024. |
| Endpoint protection discovery recommendations | Recommendations that are available through the Foundational Cloud Security Posture Management (CSPM) plan and Defender for Servers, using the Log Analytics agent (GA), AMA (Preview) | Agentless machine scanning | - Functionality with agentless machine scanning has been released to preview in early 2024 as part of Defender for Servers Plan 2 and the Defender CSPM plan.<br/>- Azure VMs, Google Cloud Platform (GCP) instances, and Amazon Web Services (AWS) instances are supported. On-premises machines are not supported.|
| Missing OS update recommendation | Recommendations available in the Foundational CSPM and Defender for Servers plans using the Log Analytics agent.  | Integration with Update Manager, Microsoft | New recommendations based on Azure Update Manager integration [are GA](release-notes-archive.md#two-recommendations-related-to-missing-operating-system-os-updates-were-released-to-ga), with no agent dependencies. |
| OS misconfigurations (Microsoft Cloud Security Benchmark) | Recommendations that are  available through the Foundational CSPM and Defender for Servers plans using the Log Analytics agent, Guest Configuration extension (Preview). | Guest Configuration extension, as part of Defender for Servers Plan 2.| - Functionality based on Guest Configuration extension will be released to GA in September 2024<br/>- For Defender for Cloud customers only: functionality with the Log Analytics agent will be deprecated in November 2024.<br/>- Support of this feature for Docker-hub and Azure Virtual Machine Scale Sets will be deprecated in Aug 2024.|
| File integrity monitoring | Log Analytics agent, AMA (Preview) | Defender for Endpoint agent integration | Functionality with the Defender for Endpoint agent will be available in August 2024.<br/>- For Defender for Cloud customers only: functionality with the Log Analytics agent will be deprecated in November 2024.<br/>- Functionality with AMA will deprecate when the Defender for Endpoint integration is released.|

### The 500-MB benefit for data ingestion

To preserve the 500 MB of free data ingestion allowance for the [supported data types](faq-defender-for-servers.yml#is-the-500-mb-of-free-data-ingestion-allowance-applied-per-workspace-or-per-machine-), you need to migrate from MMA to AMA.

> [!NOTE]
>
> - The benefit is granted to every AMA machine that is part of a subscription with Defender for Servers plan 2 enabled.
>
> - The benefit is granted to the workspace the machine is reporting to.
>
> - The security solution should be installed on the related Workspace. Learn more about how to perform it [here](https://techcommunity.microsoft.com/t5/microsoft-defender-for-cloud/how-to-configure-security-events-collection-with-azure-monitor/ba-p/3770719).
>
> - If the machine is reporting to more than one workspace, the benefit will be granted to only one of them.

Learn more about how to [deploy AMA](/azure/azure-monitor/vm/monitor-virtual-machine-agent).

For SQL servers on machines, we recommend to [migrate to SQL server-targeted Azure Monitoring Agent's (AMA) autoprovisioning process](defender-for-sql-autoprovisioning.md).

### Changes to legacy Defender for Servers Plan 2 onboarding via Log Analytics agent
The legacy approach to onboard servers to Defender for Servers Plan 2 based on the Log Analytics agent and using Log analytics workspaces is set for retirement as well:
- The onboarding experience for [onboarding new non-Azure machines](quickstart-onboard-machines.md) to Defender for Servers using Log Analytics agents and workspaces is removed from the **Inventory** and **Getting started** blades in the Defender for Cloud portal.
- To avoid losing security coverage on the affected machines connected to a Log Analytics Workspace, with the Agent retirement:
- If you onboarded non-Azure servers (both on-premises and multi-cloud) using the [legacy approach](quickstart-onboard-machines.md), you should now connect these machines via Azure Arc-enabled servers to Defender for Servers Plan 2 Azure subscriptions and connectors. [Learn more](/azure/azure-arc/servers/deployment-options) about deploying Arc machines at scale.
    
  - If you used the legacy approach to enable Defender for Servers Plan 2 on selected Azure VMs, we recommend enabling Defender for Servers Plan 2 on the Azure subscriptions for these machines. You can then exclude individual machines from the Defender for Servers coverage using the Defender for Servers [per-resource configuration](tutorial-enable-servers-plan.md).
    
This is a summary of the required action for each of the servers onboarded to Defender for Servers Plan 2 through the legacy approach:

 

|Machine type |Action required to preserve security coverage|
| -------- | -------- |
|On premise servers| [Onboarded to Arc ](/azure/azure-arc/servers/deployment-options) and connected to a subscription with Defender for Servers Plan 2 |
|Azure Virtual machines|Connect to subscription with Defender for Servers Plan 2|
|Multi Cloud Servers |Connect to [multicloud connector](/azure/defender-for-cloud/quickstart-onboard-aws) with Azure Arc provisioning and Defender for Servers plan 2|


### Endpoint protection recommendations experience - changes and migration guidance

Endpoint discovery and recommendations are currently provided by the Defender for Cloud Foundational CSPM and the Defender for Servers plans using the Log Analytics agent in GA, or in preview via the AMA. This experience will be replaced by security recommendations that are gathered using agentless machine scanning.

Endpoint protection recommendations are constructed in two stages. The first stage is [discovery](#endpoint-detection-and-response-solution---discovery) of an endpoint detection and response solution. The second is [assessment](#endpoint-detection-and-response-solution---configuration-assessment) of the solution’s configuration. The following tables provide details of the current and new experiences for each stage.

Learn how to [manage the new endpoint detection and response recommendations (agentless)](endpoint-detection-response.md).

#### Endpoint detection and response solution - discovery

| Area | Current experience (based on AMA/MMA)| New experience (based on agentless machine scanning) |
|----|----|----|
|**What's needed to classify a resource as healthy?** | An anti-virus is in place. | An endpoint detection and response solution is in place. |
| **What's needed to get the recommendation?** | Log Analytics agent | Agentless machine scanning |
| **What plans are supported?** | - Foundational CSPM (free)<br/>- Defender for Servers Plan 1 and Plan 2 |- Defender CSPM<br/>- Defender for Servers Plan 2 |
|**What fix is available?** | Install Microsoft anti-malware. | Install Defender for Endpoint on selected machines/subscriptions. |

#### Endpoint detection and response solution - configuration assessment

| Area | Current experience (based on AMA/MMA)| New experience (based on agentless machine scanning) |
|----|----|----|
| Resources are classified as unhealthy if one or more of the security checks aren’t healthy. | Three security checks:<br/>- Real time protection is off<br/>- Signatures are out of date.<br/>- Both quick scan and full scan aren't run for seven days. | Three security checks:<br/>- Anti-virus is off or partially configured<br/>- Signatures are out of date<br/>- Both quick scan and full scan aren't run for seven days. |
| Prerequisites to get the recommendation | An anti-malware solution in place | An endpoint detection and response solution in place. |

#### Which recommendations are being deprecated?

The following table summarizes the timetable for recommendations being deprecated and replaced.

| Recommendation | Agent | Supported resources | Deprecation date | Replacement recommendation |
|----|----|----|----|----|
| [Endpoint protection should be installed on your machines](https://ms.portal.azure.com/#view/Microsoft_Azure_Security/GenericRecommendationDetailsBlade/assessmentKey/4fb67663-9ab9-475d-b026-8c544cced439) (public) | MMA/AMA | Azure & non-Azure (Windows & Linux) | July 2024 | [New agentless recommendation](upcoming-changes.md#changes-in-endpoint-protection-recommendations) |
| [Endpoint protection health issues should be resolved on your machines](https://ms.portal.azure.com/#view/Microsoft_Azure_Security/GenericRecommendationDetailsBlade/assessmentKey/37a3689a-818e-4a0e-82ac-b1392b9bb000) (public)| MMA/AMA | Azure  (Windows) | July 2024 | [New agentless recommendation](upcoming-changes.md#changes-in-endpoint-protection-recommendations) |
| [Endpoint protection health failures on virtual machine scale sets should be resolved](https://ms.portal.azure.com/#view/Microsoft_Azure_Security/GenericRecommendationDetailsBlade/assessmentKey/e71020c2-860c-3235-cd39-04f3f8c936d2) | MMA | Azure Virtual Machine Scale Sets |  August 2024 | No replacement |
| [Endpoint protection solution should be installed on virtual machine scale sets](https://ms.portal.azure.com/#view/Microsoft_Azure_Security/GenericRecommendationDetailsBlade/assessmentKey/21300918-b2e3-0346-785f-c77ff57d243b) | MMA | Azure Virtual Machine Scale Sets |  August 2024 | No replacement |
| [Endpoint protection solution should be on machines](https://ms.portal.azure.com/#view/Microsoft_Azure_Security/GenericRecommendationDetailsBlade/assessmentKey/383cf3bc-fdf9-4a02-120a-3e7e36c6bfee) | MMA | Non-Azure resources (Windows)| August 2024 | No replacement |
| [Install endpoint protection solution on your machines](https://ms.portal.azure.com/#view/Microsoft_Azure_Security/GenericRecommendationDetailsBlade/assessmentKey/83f577bd-a1b6-b7e1-0891-12ca19d1e6df) | MMA | Azure and non-Azure (Windows) | August 2024 | [New agentless recommendation](upcoming-changes.md#changes-in-endpoint-protection-recommendations) |
| [Endpoint protection health issues on machines should be resolved](https://ms.portal.azure.com/#view/Microsoft_Azure_Security/GenericRecommendationDetailsBlade/assessmentKey/3bcd234d-c9c7-c2a2-89e0-c01f419c1a8a)  | MMA | Azure and non-Azure (Windows and Linux) | August 2024 | [New agentless recommendation](upcoming-changes.md#changes-in-endpoint-protection-recommendations). |

The [new recommendations](upcoming-changes.md#changes-in-endpoint-protection-recommendations) experience based on agentless machine scanning support both Windows and Linux OS across multicloud machines.

#### How will the replacement work?

- Current recommendations provided by the Log Analytics Agent or the AMA will be deprecated over time.  
- Some of these existing recommendations will be replaced by new recommendations based on agentless machine scanning.
- Recommendations currently in GA remain in place until the Log Analytics agent retires.
- Recommendations that are currently in preview will be replaced when the new recommendation is available in preview.

#### What's happening with secure score?

- Recommendations that are currently in GA will continue to affect secure score.  
- Current and upcoming new recommendations are located under the same Microsoft Cloud Security Benchmark control, ensuring that there’s no duplicate impact on secure score.

#### How do I prepare for the new recommendations?

- Ensure that [agentless machine scanning is enabled](enable-agentless-scanning-vms.md) as part of Defender for Servers Plan 2 or Defender CSPM.
- If suitable for your environment, for best experience we recommend that you remove deprecated recommendations when the replacement GA recommendation becomes available. To do that, disable the recommendation in the [built-in Defender for Cloud initiative in Azure Policy](policy-reference.md).

### File Integrity Monitoring experience - changes and migration guidance

Microsoft Defender for Servers Plan 2 now offers a new File Integrity Monitoring (FIM) solution powered by Microsoft Defender for Endpoint (MDE) integration. Once FIM powered by MDE is public, the FIM powered by AMA experience in the Defender for Cloud portal will be removed. In November, FIM powered by MMA will be deprecated.

#### Migration from FIM over AMA

If you currently use FIM over AMA:

- Onboarding new subscriptions or servers to FIM based on AMA and the change tracking extension, as well as viewing changes, will no longer be available through the Defender for Cloud portal beginning May 30.
- If you want to continue consuming FIM events collected by AMA, you can manually connect to the relevant workspace and view changes in the Change Tracking table with the following query:

    ```kusto
    ConfigurationChange

    | where TimeGenerated > ago(14d)

    | where ConfigChangeType in ('Registry', 'Files') 

    | summarize count() by Computer, ConfigChangeType
    ```

- If you want to continue onboarding new scopes or configure monitoring rules, you can manually use [Data Connection Rules](/azure/azure-monitor/essentials/data-collection-rule-overview) to configure or customize various aspects of data collection.
- Microsoft Defender for Cloud recommends disabling FIM over AMA, and onboarding your environment to the new FIM version based on Defender for Endpoint upon release.

#### Disabling FIM over AMA

To disable FIM over AMA, remove the Azure Change Tracking solution. For more information, see [Remove ChangeTracking solution](/azure/automation/change-tracking/remove-feature#remove-changetracking-solution).

Alternatively, you can remove the related file change tracking Data collection rules (DCR). For more information, see [Remove-AzDataCollectionRuleAssociation](/powershell/module/az.monitor/remove-azdatacollectionruleassociation) or [Remove-AzDataCollectionRule](/powershell/module/az.monitor/remove-azdatacollectionrule).

After you disable the file events collection using one of the methods above:

- New events will stop being collected on the selected scope.
- The historical events that already were collected remain stored in the relevant workspace under the *ConfigurationChange* table in the **Change Tracking** section. These events will remain available in the relevant workspace according to the retention period defined in this workspace. For more information, see [How retention and archiving work](/azure/azure-monitor/logs/data-retention-archive#how-retention-and-archiving-work).

#### Migration from FIM over Log Analytics Agent (MMA)

If you currently use FIM over the Log Analytics Agent (MMA):

- File Integrity Monitoring based on Log Analytics Agent (MMA) will be deprecated at the end of November 2024.

- Microsoft Defender for Cloud recommends disabling FIM over MMA, and onboarding your environment to the new FIM version based on Defender for Endpoint upon release.

#### Disabling FIM over MMA

To disable FIM over MMA, remove the Azure Change Tracking solution. For more information, see [Remove ChangeTracking solution](/azure/automation/change-tracking/remove-feature#remove-changetracking-solution).

After you disable the file events collection:

- New events will stop being collected on the selected scope.
- The historical events that already were collected remain stored in the relevant workspace under the *ConfigurationChange* table in the **Change Tracking** section. These events will remain available in the relevant workspace according to the retention period defined in this workspace. For more information, see [How retention and archiving work](/azure/azure-monitor/logs/data-retention-archive#how-retention-and-archiving-work).

## Preparing Defender for SQL on Machines

You can learn more about the [Defender for SQL Server on machines Log Analytics agent's deprecation plan](upcoming-changes.md#defender-for-sql-server-on-machines).

If you're using the current Log Analytics agent/Azure Monitor agent autoprovisioning process, you should migrate to the new Azure Monitoring Agent for SQL Server on machines autoprovisioning process. The migration process is seamless and provides continuous protection for all machines.

### Migrate to the SQL server-targeted AMA autoprovisioning process

1. Sign in to the [Azure portal](https://portal.azure.com).
1. Search for and select **Microsoft Defender for Cloud**.
1. In the Defender for Cloud menu, select **Environment settings**.
1. Select the relevant subscription.
1. Under the Databases plan, select **Action required**.

   :::image type="content" source="media/prepare-deprecation-log-analytics-mma-agent/select-action-required.png" alt-text="Screenshot that shows where to select Action required." lightbox="media/prepare-deprecation-log-analytics-mma-agent/select-action-required.png":::

1. In the pop-up window, select **Enable**.

    :::image type="content" source="media/prepare-deprecation-log-analytics-mma-agent/select-enable-sql.png" alt-text="Screenshot that shows selecting enable from popup window." lightbox="media/prepare-deprecation-log-analytics-mma-agent/select-enable-sql.png":::

1. Select **Save**.

Once the SQL server-targeted AMA autoprovisioning process is enabled, you should disable the Log Analytics agent/Azure Monitor agent autoprovisioning process and uninstall the MMA on all SQL servers:

To disable the Log Analytics agent:

1. Sign in to the [Azure portal](https://portal.azure.com).
1. Search for and select **Microsoft Defender for Cloud**.
1. In the Defender for Cloud menu, select **Environment settings**.
1. Select the relevant subscription.
1. Under the Database plan, select **Settings**.
1. Toggle the Log Analytics agent to **Off**.

    :::image type="content" source="media/prepare-deprecation-log-analytics-mma-agent/toggle-log-analytics-off.png" alt-text="Screenshot that shows toggling Log Analytics to Off." lightbox="media/prepare-deprecation-log-analytics-mma-agent/toggle-log-analytics-off.png":::

1. Select **Continue**.
1. Select **Save**.

## Migration planning

We recommend you plan agent migration in accordance with your business requirements. The table summarizes our guidance.

| **Are you using Defender for Servers?** | **Are these Defender for Servers features required in GA: file integrity monitoring, endpoint protection recommendations, security baseline recommendations?** | **Are you using  Defender for SQL servers on machines or AMA log collection?** |  **Migration plan** |
|----|----|----|----|
| Yes | Yes | No | 1. Enable [Defender for Endpoint integration](enable-defender-for-endpoint.md) and [agentless machine scanning](enable-agentless-scanning-vms.md).<br/>2. Wait for GA of all features with the alternative's platform (you can use preview version earlier).<br/>3. Once features are GA, disable the [Log Analytics agent](defender-for-sql-autoprovisioning.md#disable-the-log-analytics-agentazure-monitor-agent).|
| No | --- | No | You can remove the Log Analytics agent now. |
| No | --- | Yes | 1. You can [migrate to SQL autoprovisioning for AMA](defender-for-sql-autoprovisioning.md) now.<br/>2. [Disable](defender-for-sql-autoprovisioning.md#disable-the-log-analytics-agentazure-monitor-agent) Log Analytics/Azure Monitor Agent. |
| Yes | Yes | Yes | 1. Enable [Defender for Endpoint integration](enable-defender-for-endpoint.md) and [agentless machine scanning](enable-agentless-scanning-vms.md).<br/>2. You can use the Log Analytics agent and AMA side-by-side to get all features in GA. [Learn more](auto-deploy-azure-monitoring-agent.md#impact-of-running-with-both-the-log-analytics-and-azure-monitor-agents) about running agents side-by-side.<br>3. Migrate to [SQL autoprovisioning for AMA](defender-for-sql-autoprovisioning.md) in Defender for SQL on machines. Alternatively, start the migration from Log Analytics agent to AMA in April 2024.<br/>4. Once the migration is finished, [disable](defender-for-sql-autoprovisioning.md#disable-the-log-analytics-agentazure-monitor-agent) the Log Analytics agent. |
| Yes | No | Yes | 1. Enable [Defender for Endpoint integration](enable-defender-for-endpoint.md) and [agentless machine scanning](enable-agentless-scanning-vms.md).<br/>2. You can migrate to [SQL autoprovisioning for AMA](defender-for-sql-autoprovisioning.md) in Defender for SQL on machines now.<br/>3. [Disable](defender-for-sql-autoprovisioning.md#disable-the-log-analytics-agentazure-monitor-agent) the Log Analytics agent. |

## Next step

> [!div class="nextstepaction"]
> [Upcoming changes to the Defender for Cloud plan and strategy for the Log Analytics agent deprecation](upcoming-changes.md#defender-for-cloud-plan-and-strategy-for-the-log-analytics-agent-deprecation)
