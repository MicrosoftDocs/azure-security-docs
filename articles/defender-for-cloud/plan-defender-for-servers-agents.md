---
title: Plan Defender for Servers agents and extensions deployment
description: Plan for agent deployment to protect Azure, AWS, GCP, and on-premises servers with Microsoft Defender for Servers.
ms.topic: concept-article
ms.author: dacurwin
author: dcurwin
ms.date: 06/25/2024
#customer intent: As a reader, I want to understand how to plan the deployment of Defender for Servers agents and extensions.
---

# Plan agents, extensions, and Azure Arc for Defender for Servers

This article helps you plan your agents, extensions, and Azure Arc resources for your Microsoft Defender for Servers deployment.

Defender for Servers is one of the paid plans provided by [Microsoft Defender for Cloud](defender-for-cloud-introduction.md).

## Before you begin

This article is the *fifth* article in the Defender for Servers planning guide. Before you begin, review the earlier articles:

1. [Start planning your deployment](plan-defender-for-servers.md).
1. [Understand where your data is stored and Log Analytics workspace requirements](plan-defender-for-servers-data-workspace.md).
1. [Review Defender for Servers access roles](plan-defender-for-servers-roles.md).
1. [Select a plan for Defender for Servers](plan-defender-for-servers-select-plan.md).

## Review Azure Arc requirements

Azure Arc helps you onboard Amazon Web Services (AWS), Google Cloud Platform (GCP), and on-premises machines to Azure. Defender for Cloud uses Azure Arc to protect non-Azure machines.

### Foundational cloud security posture management

The free foundational cloud security posture management (CSPM) features for AWS and GCP machines don't require Azure Arc. For full functionality, we recommend that you *do* have Azure Arc running on AWS or GCP machines.

Azure Arc onboarding is required for on-premises machines.

### Defender for Servers plan

To use Defender for Servers, all AWS, GCP, and on-premises machines should be Azure Arc-enabled.

You can onboard the Azure Arc agent to your AWS or GCP servers automatically with the AWS or GCP multicloud connector.

### Plan for Azure Arc deployment

To plan for Azure Arc deployment:

1. Review the Azure Arc [planning recommendations](/azure/azure-arc/servers/plan-at-scale-deployment) and [deployment prerequisites](/azure/azure-arc/servers/prerequisites).
1. Open the [network ports for Azure Arc](support-matrix-defender-for-servers.md#network-requirements) in your firewall.
1. Azure Arc installs the Connected Machine agent to connect to and manage machines that are hosted outside of Azure. Review the following information:

    - The [agent components and data collected from machines](/azure/azure-arc/servers/agent-overview#agent-resources).
    - [Network and internet access](/azure/azure-arc/servers/network-requirements) for the agent.
    - [Connection options](/azure/azure-arc/servers/deployment-options) for the agent.

## Log Analytics agent and Azure Monitor agent

> [!NOTE]
> As the Log Analytics agent is set to retire in August 2024 and as part of the Defender for Cloud [updated strategy](upcoming-changes.md#defender-for-cloud-plan-and-strategy-for-the-log-analytics-agent-deprecation), all **Defender for Servers** features and capabilities will be provided either through [Microsoft Defender for Endpoint integration](integration-defender-for-endpoint.md) or [agentless scanning](concept-agentless-data-collection.md), without dependency on either Log Analytics agent (MMA) or Azure Monitor agent (AMA). As a result, the shared autoprovisioning process for both agents will be adjusted accordingly For more information about this change, see [this announcement](upcoming-changes.md#defender-for-cloud-plan-and-strategy-for-the-log-analytics-agent-deprecation).

Defender for Cloud uses the Log Analytics agent and the Azure Monitor agent to collect information from compute resources. Then, it sends the data to a Log Analytics workspace for more analysis. Review the [differences and recommendations for both agents](/azure/azure-monitor/agents/agents-overview).

The following table describes the agents that are used in Defender for Servers:

Feature | Log Analytics agent | Azure Monitor agent
--- | --- | ---
Foundational CSPM recommendations (free) that depend on the agent: [OS baseline recommendation](apply-security-baseline.md) (Azure VMs) | :::image type="icon" source="./media/icons/yes-icon.png" :::| :::image type="icon" source="./media/icons/yes-icon.png" :::<br/><br/> With the Azure Monitor agent, the Azure Policy [guest configuration extension](/azure/virtual-machines/extensions/guest-configuration) is used.
Foundational CSPM: [System updates recommendations](recommendations-reference-compute.md) (Azure VMs) | :::image type="icon" source="./media/icons/yes-icon.png" ::: | Not yet available.
Foundational CSPM: [Antimalware/endpoint protection recommendations](endpoint-protection-recommendations-technical.md) (Azure VMs) | :::image type="icon" source="./media/icons/yes-icon.png" ::: | :::image type="icon" source="./media/icons/yes-icon.png" :::
Attack detection at the OS level and network layer, including fileless attack detection<br/><br/> Plan 1 relies on Defender for Endpoint capabilities for attack detection. | :::image type="icon" source="./media/icons/yes-icon.png" :::<br/><br/> Plan 2| :::image type="icon" source="./media/icons/yes-icon.png" :::<br/><br/> Plan 2
File integrity monitoring (Plan 2 only)  | :::image type="icon" source="./media/icons/yes-icon.png" ::: | :::image type="icon" source="./media/icons/yes-icon.png" :::

## Qualys extension

The Qualys extension is available in Defender for Servers Plan 2. The extension is deployed if you want to use Qualys for vulnerability assessment.

Here's more information:

- The Qualys extension sends metadata for analysis to one of two Qualys datacenter regions, depending on your Azure region.

  - If you’re operating within a European Azure region, data processing occurs at the Qualys European data center.
  - For other regions, data processing occurs at the US data center.

- To use Qualys on a machine, the extension must be installed and the machine must be able to communicate with the relevant network endpoint:
  - Europe datacenter: `https://qagpublic.qg2.apps.qualys.eu`
  - US datacenter: `https://qagpublic.qg3.apps.qualys.com`

## Guest configuration extension

The extension performs audit and configuration operations inside VMs.

- If you're using the Azure Monitor Agent, Defender for Cloud uses this extension to analyze operating system security baseline settings on Windows and Linux machines.
- Although Azure Arc-enabled servers and the guest configuration extension are free, more costs might apply if you use guest configuration policies on Azure Arc servers outside the scope of Defender for Cloud.

Learn more about the Azure Policy [guest configuration extension](/azure/virtual-machines/extensions/guest-configuration).

## Defender for Endpoint extensions

When you enable Defender for Servers, Defender for Cloud automatically deploys a Defender for Endpoint extension. The extension is a management interface that runs a script inside the operating system to deploy and integrate the Defender for Endpoint sensor on the machine.

- Windows machines extension: `MDE.Windows`
- Linux machines extension: `MDE.Linux`
- Machines must meet [minimum requirements](/microsoft-365/security/defender-endpoint/minimum-requirements).
- Some Windows Server versions have [specific requirements](/microsoft-365/security/defender-endpoint/configure-server-endpoints).

Most Defender for Endpoint services can be reached through `*.endpoint.security.microsoft.com` or through the Defender for Endpoint service tags. Make sure you are [connected to the Defender for Endpoint service and know the requirements for automatic updates and other features.](/defender-endpoint/configure-environment).

## Verify operating system support

Before you deploy Defender for Servers, verify operating system support for agents and extensions:

- Verify that your [operating systems are supported](/microsoft-365/security/defender-endpoint/minimum-requirements) by Defender for Endpoint.
- [Check requirements](/azure/azure-arc/servers/prerequisites) for the Azure Arc Connect Machine agent.
- Check operating system support for the [Log Analytics agent](/azure/azure-monitor/agents/log-analytics-agent#supported-operating-systems) and [Azure Monitor agent](/azure/azure-monitor/agents/agents-overview).

## Review agent provisioning

When you enable plans in Defender for Cloud, including Defender for Servers, you can choose to automatically provision some agents that are relevant for Defender for Servers:

- Log Analytics agent and Azure Monitor agent for Azure VMs
- Log Analytics agent and Azure Monitor agent for Azure Arc VMs
- Qualys agent
- Guest configuration agent

When you enable Defender for Servers Plan 1 or Plan 2, the Defender for Endpoint extension is automatically provisioned on all supported machines in the subscription.

## Provisioning considerations

The following table describes provisioning considerations to be aware of:

Provisioning | Details
--- | ---
Defender for Endpoint sensor |  If machines are running Microsoft Antimalware, also known as System Center Endpoint Protection (SCEP), the Windows extension automatically removes it from the machine.<br/><br/> If you deploy on a machine that already has the legacy Microsoft Monitoring agent (MMA) Defender for Endpoint sensor running, after the Defender for Cloud and Defender for Endpoint unified solution is successfully installed, the extension stops and it disables the legacy sensor. The change is transparent and the machine’s protection history is preserved.
AWS and GCP machines | Configure automatic provisioning when you set up the AWS or GCP connector.
Manual installation | If you don't want Defender for Cloud to provision the Log Analytics agent and Azure Monitor agent, you can install agents manually.<br/><br/> You can connect the agent to the default Defender for Cloud workspace or to a custom workspace.<br/><br/> The workspace must have the *SecurityCenterFree* (for free foundational CSPM) or *Security* solution enabled (Defender for Servers Plan 2).
[Log Analytics agent running directly](faq-data-collection-agents.yml#what-if-a-log-analytics-agent-is-directly-installed-on-the-machine-but-not-as-an-extension--direct-agent--) | If a Windows VM has the Log Analytics agent running but not as a VM extension, Defender for Cloud installs the extension. The agent reports to the Defender for Cloud workspace and to the existing agent workspace. <br/><br/> On Linux VMs, multi-homing isn't supported. If an existing agent exists, the Log Analytics agent isn't automatically provisioned.
[Operations Manager agent](faq-data-collection-agents.yml#what-if-a-system-center-operations-manager-agent-is-already-installed-on-my-vm-) | The Log Analytics agent can work side by side with the Operations Manager agent. The agents share common runtime libraries that are updated when the Log Analytics agent is deployed.
Removing the Log Analytics extension | If you remove the Log Analytics extension, Defender for Cloud can't collect security data and recommendations, resulting in missing alerts. Within 24 hours, Defender for Cloud determines that the extension is missing and reinstalls it.

## When to opt out of auto provisioning

You might want to opt out of automatic provisioning in the circumstances that are described in the following table:

Situation | Relevant agent | Details
--- | --- | ---
You have critical VMs that shouldn't have agents installed | Log Analytics agent, Azure Monitor agent | Automatic provisioning is for an entire subscription. You can't opt out for specific machines.
You're running the System Center Operations Manager agent version 2012 with Operations Manager 2012 | Log Analytics agent | With this configuration, don't turn on automatic provisioning; management capabilities might be lost.
You want to configure a custom workspace | Log Analytics agent, Azure Monitor agent | You have two options with a custom workspace:<br/><br/> - Opt out of automatic provisioning when you first set up Defender for Cloud. Then, configure provisioning on your custom workspace.<br/><br/>- Let automatic provisioning run to install the Log Analytics agents on machines. Set a custom workspace, and then reconfigure existing VMs with the new workspace setting.

## Next steps

After working through these planning steps, you can start deployment:

- [Enable plans in Defender for Servers](enable-enhanced-security.md)
- [Connect on-premises machines](quickstart-onboard-machines.md) to Azure.
- [Connect AWS accounts](quickstart-onboard-aws.md) to Defender for Cloud.
- [Connect GCP projects](quickstart-onboard-gcp.md) to Defender for Cloud.
- Learn about [scaling your Defender for Server deployment](plan-defender-for-servers-scale.md).
