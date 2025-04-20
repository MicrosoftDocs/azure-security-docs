---
title: Defender for Cloud Planning and Operations Guide
description: This document helps you to plan before adopting Defender for Cloud and considerations regarding daily operations.
ms.topic: concept-article
ms.date: 05/16/2024
#customer intent: As a reader, I want to understand the planning and operations considerations for adopting Defender for Cloud and how it fits into my organization's security requirements and cloud management model. I also want to learn about the security roles, access controls, security policies, data collection, and storage in Defender for Cloud.
---

# Planning and Operations Guide

This guide is intended for IT professionals, architects, information security analysts, and cloud administrators planning to adopt Microsoft Defender for Cloud.

Microsoft Defender for Cloud helps organizations strengthen their security posture across cloud and hybrid environments. This guide outlines key areas to consider during planning and deployment:

- Security Roles and Access Controls
- Security Policies and Recommendations
- Data Collection and Storage
- Onboarding non-Azure resources
- Ongoing Security Monitoring
- Incident Response

Understanding how various roles in your organization interact with Defender for Cloud is essential for proper configuration and effective operations.

> [!NOTE]
> Read [Defender for Cloud common questions](faq-general.yml) for a list of common questions that can also be useful during the designing and planning phase.

## Security Roles and Access Controls
Defender for Cloud uses [Azure role-based access control (Azure Role-based access control)](/azure/role-based-access-control/role-assignments-portal), which provides [built-in roles](/azure/role-based-access-control/built-in-roles) that can be assigned to users, groups, and services in Azure.

Depending on the size and structure of your organization, multiple individuals and teams might use Defender for Cloud to perform different security-related tasks. The diagram below illustrates sample users roles and their associated security responsibilities.
:::image type="content" source="./media/defender-for-cloud-planning-and-operations-guide/defender-for-cloud-planning-and-operations-guide-fig01-new.png" alt-text="Conceptual image that shows various people and the roles that they fill in an organization.":::

### Built-in roles
|Role | Description |
|-----| ----------- |
|Owner| Full access to resources including management permissions.|
|Contributor | Full access to manage resources, excluding access management.|
|Reader | View-only access to resources.|
|Security Reader | View Defender for Cloud configurations, recommendations, and alerts.|
|Security Admin | Includes Security Reader permissions plus ability to modify policies and dismiss alerts.

### Example personas
|Persona | Role Assignment Recommendations |
|--------| ------------------------------- |
|Jeff (Workload Owner)| Resource Group Owner or Contributor|
|Ellen (CISO/CIO) | Subscription Owner, Contributor, or Security Admin|
|David (IT Security) | Subscription Owner, Contributor, or Security Admin|
|Judy (Security Operations) | Security Reader or Subscription Reader; Security Admin if dismissing alerts|
|Sam (Security Analyst) | Reader for investigation; Contributor to take remediation actions |

 
[!IMPORTANT]

- Only subscription Owners/Contributors and Security Admins can edit a security policy.
- Only subscription and resource group Owners and Contributors can apply security recommendations for a resource.
- When planning access control for Microsoft Defender for Cloud using Azure Role-Based Access Control (RBAC), first identify who in your organization needs access and what tasks they need to perform. This understanding helps you assign roles appropriately and configure RBAC effectively.
- Assign users the least permissive role required to perform their tasks. For example, users who only need to view the security state of resources — without applying recommendations or modifying policies — should be assigned the Reader role.


## Security Policies and Recommendations

A security policy defines the desired configuration of your workloads and ensures compliance with organization and regulatory standards. Defender for Cloud provides default policies that you can customize or extend using Azure Policy.

### Key policy components

- [Data collection](monitoring-components.md): Configures agent provisioning and data collection settings.
- [Security policy](tutorial-security-policy.md): Specifies which controls are monitored and recommended.
- [Email notifications](configure-email-notifications.md): Sets security contacts and notification preferences.
- [Pricing tier](defender-for-cloud-introduction.md#protect-cloud-workloads): Determines feature availability. Plans can be configured per subscription or workspace using the API.

> [!NOTE]
> Specifying a security contact ensures that Azure can reach the right person in your organization if a security incident occurs. Read [Provide security contact details in Defender for Cloud](configure-email-notifications.md) for more information on how to enable this recommendation.

### Security policies definitions and recommendations

Defender for Cloud automatically creates a default security policy for each of your Azure subscriptions. You can edit the policy in Defender for Cloud or use Azure Policy to create new definitions, define more policies, and assign policies across management groups. Management groups can represent the entire organization or a business unit within the organization. You can monitor policy compliance across these management groups.

Before configuring security policies, review each of the [security recommendations](review-security-recommendations.md):

- See if these policies are appropriate for your various subscriptions and resource groups.

- Understand what actions address the security recommendations.

- Determine who in your organization is responsible for monitoring and remediating new recommendations.

## Data Collection and Storage

Defender for Cloud uses the Log Analytics agent and the Azure Monitor Agent to collect security data from your virtual machines (VMs). [Data collected](monitoring-components.md) from this agent is stored in your Log Analytics workspaces.

### Agent

When automatic provisioning is enabled in the security policy, the [data collection agent](monitoring-components.md) is installed on all supported Azure VMs and any new supported VMs that are created. If the VM or computer already has the Log Analytics agent installed, Defender for Cloud uses the current installed agent. The agent's process is designed to be non-invasive and have minimal effect on VM performance.

If at some point you want to disable Data Collection, you can turn it off in the security policy. However, because the Log Analytics agent might be used by other Azure management and monitoring services, the agent won't be uninstalled automatically when you turn off data collection in Defender for Cloud. You can manually uninstall the agent if needed.

### Workspace

A workspace is an Azure resource that serves as a container for data. You or other members of your organization might use multiple workspaces to manage different sets of data that is collected from all or portions of your IT infrastructure.

Data collected from the Log Analytics agent can be stored in an existing Log Analytics workspace associated with your Azure subscription or a new workspace.

In the Azure portal, you can browse to see a list of your Log Analytics workspaces, including any created by Defender for Cloud. A related resource group is created for new workspaces. Resources are created according to this naming convention:

- Workspace: *DefaultWorkspace-[subscription-ID]-[geo]*

- Resource Group: *DefaultResourceGroup-[geo]*

For workspaces created by Defender for Cloud, data is retained for 30 days. For existing workspaces, retention is based on the workspace pricing tier. If you want, you can also use an existing workspace.

If your agent reports to a workspace other than the **default** workspace, any Defender for Cloud [Defender plans](defender-for-cloud-introduction.md#protect-cloud-workloads) that you've enabled on the subscription should also be enabled on the workspace.

> [!NOTE]
> Microsoft makes strong commitments to protect the privacy and security of this data. Microsoft adheres to strict compliance and security guidelines—from coding to operating a service. For more information about data handling and privacy, read [Defender for Cloud Data Security](data-security.md).

## Onboard Non-Azure Resources

Defender for Cloud can monitor the security posture of your non-Azure computers but you need to first onboard these resources. Read [Onboard non-Azure computers](quickstart-onboard-machines.md) for more information on how to onboard non-Azure resources.

## Ongoing Security Monitoring

After initial configuration and application of Defender for Cloud recommendations, the next step is considering Defender for Cloud operational processes.

The Defender for Cloud Overview provides a unified view of security across all your Azure resources and any non-Azure resources you've connected, as displayed in the example below.
:::image type="content" source="./media/overview-page/overview.png" alt-text="Screenshot of Defender for Cloud's overview page." lightbox="./media/overview-page/overview.png":::

> [!NOTE]
> Defender for Cloud does not interfere with your normal operational procedures. It passively monitors your deployments and provides recommendations based on the security policies you enable.

When you first opt in to use Defender for Cloud for your current Azure environment, make sure that you review all recommendations, which can be done in the **Recommendations** page.

Plan to visit the threat intelligence option as part of your daily security operations. There you can identify security threats against the environment, such as identify if a particular computer is part of a botnet.

## Monitor New or Updated Resources

Most Azure environments are dynamic, with resources regularly being created and updated. Defender for Cloud helps ensure that you have visibility into the security state of these new resources.

When you add new resources—like VMs, SQL databases, or PaaS web and worker roles— Defender for Cloud automatically detects and monitors them for security. If Data Collection is enabled in the [Security Policy](tutorial-security-policy.md), more monitoring capabilities are enabled automatically for your virtual machines.

You should also regularly monitor existing resources for configuration changes that could have created security risks, drift from recommended baselines, and security alerts.

### Harden access and applications

As part of your security operations, it is important to implement preventative measures that restrict access to VMs and control the applications running on them. By limiting inbound traffic to your Azure VMs, you reduce their exposure to potential attacks while still allowing secure, on-demand access when needed. To strengthen VM access control, use the [just-in-time VM access](just-in-time-access-usage.yml) feature.

## Incident Response

Microsoft Defender for Cloud supports incident response workflows, aligned with the (NIST) [Computer Security Incident Handling Guide](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf)

Defender for Cloud detects and alerts you to threats as they occur. Organizations should monitor for new security alerts and take action as needed to investigate further or remediate the attack. For more information on how Defender for Cloud threat protection works, read [How Defender for Cloud detects and responds to threats](alerts-overview.md#detect-threats).

### Incident lifecycle stages
:::image type="content" source="./media/defender-for-cloud-planning-and-operations-guide/defender-for-cloud-planning-and-operations-guide-fig5-1.png" alt-text="Stages of the incident response in the cloud lifecycle.":::

1. **Detect:** Identify suspicious activity or threats.

2. **Assess:** Analyze alerts and gather supporting data.

3. **Diagnose:** Execute remediation steps.

### Tools and automation
- Security alerts
- Workflow automations

**Security alerts** 

Each Security Alert provides information that can be used to better understand the nature of the attack and suggest possible mitigations. Some alerts also provide links to either more information or to other sources of information within Azure. You can use the information provided for further research and to begin mitigation, and you can also search security-related data that is stored in your workspace.

The following example shows a suspicious RDP activity taking place:

:::image type="content" source="./media/defender-for-cloud-planning-and-operations-guide/defender-for-cloud-planning-and-operations-guide-fig5-ga.png" alt-text="Screenshot of a suspicious activity report while it is taking place.":::

This page provides the following details:
- Time of the attack
- The source hostname
-  The target VM
-  Recommended remediation steps.

In some circumstances, the source information of the attack might be empty. Read [Missing Source Information in Defender for Cloud alerts](/archive/blogs/azuresecurity/missing-source-information-in-azure-security-center-alerts) for more information about this type of behavior.

**Workflow automations**

Workflow automations are a collection of procedures that can be executed from Defender for Cloud once triggered by an alert. Once you identify the compromised system, you can run a [workflow automation](workflow-automation.yml) that was previously created. 

> [!NOTE]
> Read [Managing and responding to security alerts in Defender for Cloud](managing-and-responding-alerts.yml) for more information on how to use Defender for Cloud capabilities to assist you during your Incident Response process.

## Next steps

In this document, you learned how to plan for Defender for Cloud adoption. Learn more about Defender for Cloud:

- [Managing and responding to security alerts in Defender for Cloud](managing-and-responding-alerts.yml)
- [Monitoring partner solutions with Defender for Cloud](./partner-integration.md) - Learn how to monitor the health status of your partner solutions.
- [Defender for Cloud common questions](faq-general.yml) - Find frequently asked questions about using the service.
- [Azure Security blog](/archive/blogs/azuresecurity/) - Read blog posts about Azure security and compliance.
