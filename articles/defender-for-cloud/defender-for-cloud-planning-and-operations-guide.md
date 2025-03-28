---
title: Defender for Cloud Planning and Operations Guide
description: This document helps you plan before adopting Defender for Cloud and provides considerations regarding daily operations.
ms.topic: concept-article
ms.date: 05/16/2024
#customer intent: As a reader, I want to understand the planning and operations considerations for adopting Defender for Cloud and how it fits into my organization's security requirements and cloud management model. I also want to learn about the security roles, access controls, security policies, data collection, and storage in Defender for Cloud.
---

# Planning and operations guide

This guide is for information technology (IT) professionals, information security analysts, and cloud administrators planning to use Defender for Cloud.

## Planning guide

This guide explains how Defender for Cloud fits into your organization's security requirements and cloud management model. Understanding how different teams use the service to meet development, operations, monitoring, governance, and incident response needs is crucial. When planning to use Defender for Cloud, consider these key areas::

- Security roles and access controls
- Security policies and recommendations
- Data collection and storage
- Onboarding non-Azure resources
- Ongoing security monitoring
- Incident response

In the next section, you'll learn how to plan for each one of these areas and apply the recommendations based on your requirements.

> [!NOTE]
> Read [Defender for Cloud frequently asked questions (FAQ)](faq-general.yml), which can be helpful during the design and planning phase.

## Security roles and access controls

Depending on the size and structure of your organization, multiple individuals and teams might use Defender for Cloud to perform different security-related tasks. The following diagram illustrates how Defender for Cloud supports different roles in managing security responsibilities:

:::image type="content" source="./media/defender-for-cloud-planning-and-operations-guide/defender-for-cloud-planning-and-operations-guide-fig01-new.png" alt-text="Conceptual image that shows various people and the roles that they fill in an organization.":::

Defender for Cloud uses [Azure role-based access control (Azure RBAC)](/azure/role-based-access-control/role-assignments-portal), which provides [built-in roles](/azure/role-based-access-control/built-in-roles) that can be assigned to users, groups, and services in Azure. When a user opens Defender for Cloud, they only see information related to resources they have access to. This means that the user must be assigned the Owner, Contributor, or Reader role for the subscription or resource group to which a resource belongs. In addition to these roles, Defender for Cloud has two specific roles:

- **Security reader**: this role allows users to only view Defender for Cloud configurations, including recommendations, alerts, policy, and security health.

- **Security admin**: this role is similar to a Security reader, but it also allows the user to manage security policy and dismiss recommendations and alerts.

The personas from the diagram above need these Azure RBAC roles:

**Jeff (Workload Owner)**

- Resource Group Owner/Contributor.

**Ellen (CISO/CIO)**

- Subscription Owner/Contributor or Security Admin.

**David (IT Security)**

- Subscription Owner/Contributor or Security Admin.

**Judy (Security Operations)**

- Subscription Reader or Security Reader (to view alerts).

- Subscription Owner/Contributor or Security Admin (to dismiss alerts).

**Sam (Security Analyst)**

- Subscription Reader (to view alerts).

- Subscription Owner/Contributor (to dismiss alerts).

- May require workspace access.

> [!IMPORTANT]
> - Only subscription Owners/Contributors and Security Admins can edit a security policy.
> - Only subscription and resource group Owners and Contributors can apply security recommendations for a resource.

When planning access control using Azure RBAC control for Defender for Cloud, make sure you understand who in your organization needs access to Defender for Cloud and the tasks they'll perform. This helps you configure Azure RBAC properly.

> [!NOTE]
> Assign the least permissive role needed for tasks. For example, users who only need to view security information (without making changes) should be assigned the Reader Role.

## Security policies and recommendations

A security policy defines the desired configuration of your workloads and ensures compliance with company or regulatory security requirements. In Defender for Cloud, you can define policies for your Azure subscriptions based on workload or data sensitivity.

Defender for Cloud policies contain the following components:

- [Data collection](monitoring-components.md): settings for agent provisioning and data collection.

- [Security policy](tutorial-security-policy.md): an [Azure Policy](/azure/governance/policy/overview) that determines which controls are monitored and recommended by Defender for Cloud. Azure Policy can also be used to create new definitions, add policies, and assign policies across management groups.

- [Email notifications](configure-email-notifications.md): settings for security contacts and notification.
- [Pricing tier](defender-for-cloud-introduction.md#protect-cloud-workloads): Microsoft Defender for Cloud offers different plans, and each plan unlocks specific features for protecting resources. You can choose which plan to use for your resources (like subscriptions or workspaces) and specify that choice through the API..

> [!NOTE]
> Specifying a security contact ensures that Azure can reach the right person in your organization if a security incident occurs. For more details on how to enable these recommendations, refer to [Provide security contact details in Defender for Cloud](configure-email-notifications.md).

### Security policy definitions and recommendations

Defender for Cloud automatically creates a default security policy for your Azure subscriptions. You can edit this policy in Defender for Cloud, or use Azure Policy to create new definitions. Additionally, you can define more policies and assign policies across management groups. Management groups can represent either the entire organization or specific business units in it. Policy compliance can also be monitored across these management groups.

> [!TIP]
> Before configuring security policies, review each of the [security recommendations](review-security-recommendations.md):
> - Check if these policies are appropriate for your various subscriptions and resource groups.
> - Understand what actions are required to address the security recommendations.
> - Determine who in your organization is responsible for monitoring and remediating new recommendations.

## Data collection and storage

Defender for Cloud uses the Log Analytics agent and the Azure Monitor Agent to collect security data from your virtual machines (VM). This [data](monitoring-components.md) is stored in your Log Analytics workspaces.

### Agent

When automatic provisioning is enabled in the security policy, the [data collection agent](monitoring-components.md) is installed on all supported Azure VMs and on any new supported VMs that are created. If the VMs or computers already have the Log Analytics agent installed, Defender for Cloud uses the existing agent. The agent's design ensures that it is non-invasive and has minimal effect on VM performance.

If at some point you want to disable Data Collection, you can turn it off in the security policy. However, note that the Log Analytics agent might be used by other Azure management and monitoring services, and it won't be automatically uninstalled when you turn off data collection in Defender for Cloud. You can manually uninstall the agent if needed.

### Workspace

A workspace is an Azure resource that serves as a container for data. You or other members of your organization may use multiple workspaces to manage different sets of data collected from your IT infrastructure.

Data collected from the Log Analytics agent can be stored in an existing Log Analytics workspace associated with your Azure subscription or a new workspace.

In the Azure portal, you can view a list of your Log Analytics workspaces, including any created by Defender for Cloud. A related resource group is created for new workspaces. Resources are named according to this convention:

- Workspace: *DefaultWorkspace-[subscription-ID]-[geo]*

- Resource Group: *DefaultResourceGroup-[geo]*

For workspaces created by Defender for Cloud, data is retained for 30 days. For existing workspaces, retention depends on the workspace pricing tier. You can also use an existing workspace.

If your agent reports to a workspace other than the **default** workspace, ensure that any [Defender plans](defender-for-cloud-introduction.md#protect-cloud-workloads) on the subscription are also enabled on the workspace.

> [!NOTE]
> Microsoft is committed to protecting the privacy and security of your data. Microsoft follows strict compliance and security guidelines throughout the lifecycle of the service, from coding to operation. For more information about data handling and privacy, please refer to [Defender for Cloud Data Security](data-security.md).

## Onboard non-Azure resources

Defender for Cloud can monitor the security posture of your non-Azure computers, but you first need to onboard these resources. Read [Onboard non-Azure computers](quickstart-onboard-machines.md) for more information on how to onboard non-Azure resources.

## Ongoing security monitoring

After the initial configuration and application of Defender for Cloud recommendations, the next step is to consider Defender for Cloud operational processes.

The Defender for Cloud Overview provides a unified view of security across all your Azure resources and any non-Azure resources you've connected. This image shows an environment in Defender for Cloud with multiple issues to resolve:

:::image type="content" source="./media/overview-page/overview.png" alt-text="Screenshot of Defender for Cloud's overview page." lightbox="./media/overview-page/overview.png":::

> [!NOTE]
> Defender for Cloud doesn't interfere with your normal operational procedures. It passively monitors your deployments and provides recommendations based on the security policies you have enabled.

When you first opt in to use Defender for Cloud for your current Azure environment, make sure to review all recommendations, which can be done on the **Recommendations** page.

>[!TIP]
> Plan to visit the threat intelligence option as part of your daily security operations. There you can identify security threats against the environment, such as identify if a particular computer is part of a botnet.

### Monitor for new or changed resources

Most Azure environments are dynamic, with resources regularly being created, spun up or down, reconfigured, and changed. Defender for Cloud helps ensure that you have visibility into the security state of these new resources.

When you add new resources (virtual machines [VMs], structured query language [SQL], and databases [DBs]) to your Azure environment, Defender for Cloud automatically discovers these resources and begins to monitor their security, including PaaS web roles and worker roles. If Data Collection is enabled in the [Security Policy](tutorial-security-policy.md), more monitoring capabilities are automatically enabled for your virtual machines.

You should also regularly monitor existing resources for configuration changes that could introduce security risks, drift from recommended baselines, or security alerts.

### Harden access and applications

As part of your security operations, you should also adopt preventative measures to restrict access to VMs and control the applications that are running on them. By locking down inbound traffic to your Azure VMs, you reduce the exposure to attacks while still providing easy access when necessary. Use [just-in-time VM access](just-in-time-access-usage.yml) to harden access to your VMs.

## Incident response

Defender for Cloud detects and alerts you to threats as they occur. Organizations should monitor new security alerts and take action as needed to investigate further or remediate the attack. For more information on how Defender for Cloud threat protection works, read [How Defender for Cloud detects and responds to threats](alerts-overview.md#detect-threats).

Although we can't create your Incident Response plan, we will use Microsoft Azure Security Response in the Cloud lifecycle as the foundation for incident response stages. The stages are:

- **Detect** – identify a suspicious activity in one or more resources.

- **Assess** – perform the initial assessment to obtain more information about the suspicious activity.

- **Diagnose** – use the remediation steps to conduct the technical procedure to address the issue.

- **Stabilize** – implement corrective actions to prevent the issue from persisting or spreading.

- **Close** – complete the response process and document lessons learned.

The following diagram illustrates these five stages, showing how they flow from detection to resolution:

:::image type="content" source="./media/defender-for-cloud-planning-and-operations-guide/defender-for-cloud-planning-and-operations-guide-fig5-1.png" alt-text="Stages of the incident response in the cloud lifecycle.":::

> [!NOTE]
> You can use the National Institute of Standards and Technology (NIST) [Computer Security Incident Handling Guide](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf) as a reference for building your own.

Defender for Cloud alerts can be used during the detect, assess, and diagnose stages. Each alert provides valuable information to help you understand the nature of the attack, suggest mitigations, and offer links to additional Azure resources. This data can guide your research, support mitigation efforts, and help you search security-related information stored in your workspace.

The following example shows a suspicious RDP activity taking place:

:::image type="content" source="./media/defender-for-cloud-planning-and-operations-guide/defender-for-cloud-planning-and-operations-guide-fig5-ga.png" alt-text="Screenshot of a suspicious activity report while it is taking place.":::

This page provides details on the time the attack occurred, the source hostname, and the target VM. It also provides recommendation steps. In some circumstances, the source information of the attack might be empty. Read [Missing Source Information in Defender for Cloud alerts](/archive/blogs/azuresecurity/missing-source-information-in-azure-security-center-alerts) for more information about this kind of behavior.

Once you identify the compromised system, you can run a previously created [workflow automation](workflow-automation.yml). Workflow automations are procedures that run in Defender for Cloud when an alert is triggered.  

> [!NOTE]
> Read [Managing and responding to security alerts in Defender for Cloud](managing-and-responding-alerts.yml) for more information on how to use Defender for Cloud capabilities to help you during your Incident Response process.

## Next steps

You learned how to plan for Defender for Cloud adoption in this guide. Learn more about Defender for Cloud:

- [Managing and responding to security alerts in Defender for Cloud](managing-and-responding-alerts.yml)
- [Monitoring partner solutions with Defender for Cloud](./partner-integration.md) - Learn how to monitor the status of your partner solutions.
- [Defender for Cloud frequently asked questions](faq-general.yml) - Get answers to FAQs about using the service.
- [Azure Security blog](/archive/blogs/azuresecurity/) - Read blog posts about Azure security and compliance.
