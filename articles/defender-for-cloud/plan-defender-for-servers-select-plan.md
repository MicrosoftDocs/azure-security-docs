---
title: Select a Defender for Servers plan
description: This article helps you understand which Defender for Servers plan to deploy in Microsoft Defender for Cloud.
ms.topic: concept-article
ms.author: dacurwin
author: dcurwin
ms.date: 02/23/2025
#customer intent: As a reader, I want to understand which Defender for Servers plan I want to deploy.
---

# Select a Defender for Servers plan and deployment scope

This article helps you understand which [Defender for Servers plan](defender-for-servers-overview.md) to deploy in Microsoft Defender for Cloud.

## Before you begin

This article is the *third* in the Defender for Servers planning guide. Before you begin, review the earlier articles:

1. Start [planning your deployment](plan-defender-for-servers.md).
1. Review [Defender for Servers access roles](plan-defender-for-servers-roles.md).

## Review plans

Defender for Servers offers two paid plans:

- **Defender for Servers Plan 1** is entry-level, and focuses on the endpoint detection and response (EDR) capabilities provided by the Defender for Endpoint integration with Defender for Cloud.
- **Defender for Servers Plan 2** provides the same features as Plan 1, and more:

  - [Agentless scanning](concept-agentless-data-collection.md) for machine posture scanning, vulnerability assessment, threat protection, malware scanning, and secrets scanning.
  - [Compliance assessment](regulatory-compliance-dashboard.md) against various regulatory standards. Available with Defender for Servers Plan 2 or any other paid plan.
  - Capabilities provided by [premium Microsoft Defender Vulnerability Management](/defender-vulnerability-management/defender-vulnerability-management-capabilities).
  - [A free data ingestion benefit](data-ingestion-benefit.md) for specific data types.
  - [OS configuration assessment](operating-system-misconfiguration.md) against compute security baselines in the Microsoft Cloud Security Benchmark.
  - [OS updates assessment](enable-periodic-system-updates.md) with Azure Updates integrated into Defender for Servers.
  - [File integrity monitoring](file-integrity-monitoring-overview.md) to examine files and registries for changes that might indicate an attack.
  - [Just-in-time machine access](just-in-time-access-overview.md) to lock down machine ports and reduce attack surfaces.
  - [Network map](protect-network-resources.md) to get a geographical view of network recommendations.

For a full list, review [Defender for Servers plan features](defender-for-servers-overview.md#plan-protection-features).

## Decide on deployment scope

We recommend enabling Defender for Servers at the subscription level, but you can enable and disable Defender for Servers plans at the resource level if you need deployment granularity.

**Scope** | **Plan 1** | **Plan 2**
--- | --- | ---
**Enable for Azure subscription** | Yes | Yes
**Enable for resource** | Yes | No
**Disable for resource** | Yes | Yes

- Plan 1 can be enabled and disabled at resource level.
- Plan 2 can't be enabled at the resource level, but you can disable the plan at the resource level.

Here are some use case examples to help you decide on Defender for Servers deployment scope.

**Use case** | **Enabled in subscription** | **Details** | **Method**
--- | --- | --- | ---
**Turn on for a subscription** | Yes | We recommend this option. | Turn on in the portal.<br/><br/> You can also turn off the plan for an entire subscription in the portal.
**Turn on Plan 1 for multiple machines** | No | You can use a script or policy to enable Plan 1 for a group of machines without turning on the plan for an entire subscription. | In the [script](https://github.com/Azure/Microsoft-Defender-for-Cloud/tree/main/Powershell%20scripts/Defender%20for%20Servers%20on%20resource%20level), specify the relevant machines using a resource tag or resource group. Then follow the on-screen instruction.<br/><br/>With the [policy](/azure/governance/policy/samples/built-in-policies#security-center---granular-pricing), create the assignment on a resource group, or specify the relevant machines using a resource tag. The tag is customer specific.
**Turn on Plan 1 for multiple machines** | Yes | If Defender for Servers Plan 2 is enabled in a subscription, you can use a script or policy assignment to downgrade a group of machines to Defender for Servers Plan 1. | In the [script](https://github.com/Azure/Microsoft-Defender-for-Cloud/tree/main/Powershell%20scripts/Defender%20for%20Servers%20on%20resource%20level), specify the relevant machines using a resource tag or resource group. Then follow the on-screen instruction.<br/><br/>With the [policy](/azure/governance/policy/samples/built-in-policies#security-center---granular-pricing), create the assignment on a resource group, or specify the relevant machines using a resource tag. The tag is customer specific.
**Turn on Plan 1 for individual machines** | No | When Defender for Servers isn't enabled in a subscription, you can use the API to turn on Plan 1 for individual machines. | Use the Azure Microsoft Security [Pricings operation group](/rest/api/defenderforcloud-composite/pricings?view=rest-defenderforcloud-composite-latest&preserve-view=true).<br/><br/>In [Update Pricings](/rest/api/defenderforcloud-composite/pricings/update?view=rest-defenderforcloud-composite-latest&tabs=HTTP&preserve-view=true#update-pricing-on-resource-(example-for-virtualmachines-plan)), use a PUT request to set the pricingTier property to *standard* and the subplan to **P1**.<br/><br/> The pricingTier property indicates whether the plan is enabled on the selected scope.
**Turn on Plan 1 for individual machines** | Yes | When Defender for Servers Plan 2 is enabled in a subscription, you can use the API to turn on Plan 1, instead of Plan 2, for individual machines in the subscription. | Use the Azure Microsoft Security [Pricings operation group](/rest/api/defenderforcloud-composite/pricings?view=rest-defenderforcloud-composite-latest&preserve-view=true).<br/><br/>In [Update Pricings](/rest/api/defenderforcloud-composite/pricings/update?view=rest-defenderforcloud-composite-latest&tabs=HTTP&preserve-view=true#update-pricing-on-resource-(example-for-virtualmachines-plan)), use a PUT request to set the pricingTier property to *standard* and the subplan to **P1**.<br/><br/> The pricingTier property indicates whether the plan is enabled on the selected scope.
**Turn off a plan for multiple machines** | Yes/No | Regardless of whether a plan is turned on or off in a subscription, you can turn off the plan for a group of machines. | Use the [script](https://github.com/Azure/Microsoft-Defender-for-Cloud/tree/main/Powershell%20scripts/Defender%20for%20Servers%20on%20resource%20level) or [policy](/azure/governance/policy/samples/built-in-policies#security-center---granular-pricing) to specify the relevant machines using a resource tag or resource group.
**Turn off a plan for specific machines** | Yes/No | Regardless of whether a plan is turned on or off in a subscription, you can turn off a plan for a specific machine. | In [Update Pricings](/rest/api/defenderforcloud-composite/pricings/update?view=rest-defenderforcloud-composite-latest&tabs=HTTP&preserve-view=true#update-pricing-on-resource-(example-for-virtualmachines-plan)), use a PUT request to set the pricingTier property to *free* and the subplan to **P1**.
**Delete the plan configuration on individual machines** | Yes/No | Remove the configuration from a machine to make the subscription-wide setting effective. | In Update Pricings, use a  [Delete](/rest/api/defenderforcloud-composite/pricings/delete?view=rest-defenderforcloud-composite-latest&tabs=HTTP&preserve-view=true) request to remove the configuration.
**Delete the plan on multiple resources** | | Remove the configuration from a group of resources to make the the subscription-wide setting effective.|  In the [script](https://github.com/Azure/Microsoft-Defender-for-Cloud/tree/main/Powershell%20scripts/Defender%20for%20Servers%20on%20resource%20level), specify the relevant machines using a resource group or tag. Then follow the on-screen instructions.

[Learn more](tutorial-enable-servers-plan.md) about how to deploy the plan on a subscription and on specific resources.

## Workspace considerations

Defender for Servers needs a Log Analytics workspace when:

- You deploy Defender for Servers Plan 2 and you want to take advantage of free daily ingestion for specific data types. [Learn more](data-ingestion-benefit.md).
- You deploy Defender for Servers Plan 2 and you're using file integrity monitoring. [Learn more](file-integrity-monitoring-overview.md).

## Azure ARC onboarding

We recommend that you onboard machine in non-Azure clouds and on-premises to Azure as Azure Arc-enabled VMs. Enabling as Azure Arc VMs allows machines to take full advantage of Defender for Servers features. Azure Arc-enabled machines have the Azure Arc Connected Machine agent installed on them.

- When you use the Defender for Cloud multicloud connector to connect to [Amazon Web Service (AWS) accounts](quickstart-onboard-aws.md) and [Google Cloud Platform (GCP) projects](quickstart-onboard-gcp.md), you can automatically onboard the Azure Arc agent to AWS or GCP servers.
- We recommend that you [onboard on-premises machines as Azure Arc-enabled](quickstart-onboard-machines.md).
- Although you onboard [on-premises machines by directly installing the Defender for Endpoint agent](onboard-machines-with-defender-for-endpoint.md) instead of onboarding machines with Azure Arc, Defender for Servers Plan functionality remains available. For Defender for Servers Plan 2, in addition to Plan 1 features, only the premium Defender Vulnerability Management features are available.

Before you deploy Azure Arc:

- [Review a full list](/azure/azure-arc/servers/prerequisites#supported-operating-systems) of operating systems supported by Azure Arc.
- Review the Azure Arc [planning recommendations](/azure/azure-arc/servers/plan-at-scale-deployment) and [deployment prerequisites](/azure/azure-arc/servers/prerequisites).
- [Review networking requirements](/azure/azure-arc/servers/arc-gateway) for the Connected Machine agent.
- Open the [network ports for Azure Arc](support-matrix-defender-for-servers.md#network-requirements) in your firewall.
- Review requirements for the Connected Machine agent:
  - [Agent components and data collected from machines](/azure/azure-arc/servers/agent-overview#agent-resources).
  - [Network and internet access](/azure/azure-arc/servers/network-requirements) for the agent.
  - [Connection options](/azure/azure-arc/servers/deployment-options) for the agent.

## Next steps

[Understand how data is collected](plan-defender-for-servers-agents.md) to Azure.
