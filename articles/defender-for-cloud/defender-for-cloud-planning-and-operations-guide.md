---
title: Defender for Cloud Planning and Operations Guide
description: This document helps you to plan before adopting Defender for Cloud and considerations regarding daily operations.
ms.topic: concept-article
ms.date: 05/16/2024
#customer intent: As a reader, I want to understand the planning and operations considerations for adopting Defender for Cloud and how it fits into my organization's security requirements and cloud management model. I also want to learn about the security roles, access controls, security policies, data collection, and storage in Defender for Cloud.
---

# Planning and operations guide

This guide is for information technology (IT) professionals, IT architects, information security analysts, and cloud administrators planning to use Defender for Cloud.

## Planning guide

This guide provides the background for how Defender for Cloud fits into your organization's security requirements and cloud management model. It's important to understand how different individuals or teams within your organization use the service to meet secure development and operational requirements, and for monitoring, governance, and incident response needs. The key areas to consider when planning to use Defender for Cloud are:

- Security roles and access controls
- Security policies and recommendations
- Data collection and storage
- Onboarding non-Azure resources
- Ongoing security monitoring
- Incident response

In the next section, you'll learn how to plan for each of these areas and apply those recommendations based on your requirements.

> [!NOTE]
> Read [Defender for Cloud common questions](faq-general.yml) for a list of common questions that can also help during the designing and planning phase.

## Security roles and access controls

Depending on the size and structure of your organization, multiple individuals and teams might use Defender for Cloud for various security-related tasks. The following diagram provides examples of fictitious personas and their respective roles and security responsibilities:

:::image type="content" source="./media/defender-for-cloud-planning-and-operations-guide/defender-for-cloud-planning-and-operations-guide-fig01-new.png" alt-text="Conceptual image that shows various people and the roles that they fill in an organization.":::

Defender for Cloud enables these individuals to meet these various responsibilities. For example:

**Jeff (Cloud workload owner)**

- Manages a cloud workload and its related resources.

- Responsible for implementing and maintaining protections in accordance with company security policy.

**Ellen (CISO/CIO)**

- Responsible for all aspects of security for the company.

- Wants to understand the company's security posture across cloud workloads.

- Needs to be informed of major attacks and risks.

**David (IT security)**

- Sets company security policies to ensure the appropriate protections are in place.

- Monitors compliance with policies.

- Generates reports for leadership or auditors.

**Judy (Security operations)**

- Monitors and responds to security alerts at any time.

- Escalates to the Cloud Workload owner or an IT Security Analyst.

**Sam (Security analyst)**

- Investigates attacks.

- Works with the Cloud Workload Owner to apply remediation.

Defender for Cloud uses [Azure role-based access control (Azure Role-based access control)](/azure/role-based-access-control/role-assignments-portal), which provides [built-in roles](/azure/role-based-access-control/built-in-roles) that can be assigned to users, groups, and services in Azure. Each user is assigned the role of owner, contributor or reader for the subscription or resource group a resource belongs to. When a user opens Defender for Cloud, they only see information related to resources they have access to. 

In addition to these roles, there are two roles that are specific to Defender for Cloud:

- **Security reader**: A user with this role can view Defender for Cloud configurations only, which includes recommendations, alerts, policy, and health, but can't make changes to the configurations.

- **Security admin**: A user with this role has the same access as the security reader role, but can also update the security policy, and dismiss recommendations and alerts.

The personas explained in the previous diagram need these Azure role-based access control roles:

**Jeff (Cloud workload owner)**

- Resource group owner and contributor.

**Ellen (CISO/CIO)**

- Subscription owner and contributor or security admin.

**David (IT security)**

- Subscription owner and contributor or security admin.

**Judy (Security operations)**

- Subscription reader or security reader to view alerts.

- Subscription owner and contributor or security admin required to dismiss alerts.

**Sam (Security analyst)**

- Subscription reader to view alerts.

- Subscription owner and contributor required to dismiss alerts.

- Access to the workspace might be required.

Some other important information to consider:

- Only subscription owners and contributors, and security admins can edit a security policy.

- Only subscription and resource group owners and contributors can apply security recommendations for a resource.

When you plan role-based access control for Defender for Cloud, make sure you understand who needs access and what tasks they need to perform, so you can configure roles correctly.

> [!NOTE]
> For all users, we recommend that you assign the least permissive role needed to complete their tasks. For example, users who only need to view information about the security state of resources but not take action, such as applying recommendations or editing policies, must be assigned the reader role.

## Security policies and recommendations

A security policy defines the desired configuration of your workloads and helps ensure compliance with company or regulatory security requirements. In Defender for Cloud, you can define policies for your Azure subscriptions that are tailored to the type of workload or the sensitivity of data.

Defender for Cloud policies contain the following components:

- [Data collection](monitoring-components.md): Agent provisioning and data collection settings.

- [Security policy](tutorial-security-policy.md): An [Azure Policy](/azure/governance/policy/overview) that determines which controls are monitored and recommended by Defender for Cloud. You can also use Azure Policy to create new definitions, define more policies, and assign policies across management groups.

- [Email notifications](configure-email-notifications.md): Security contacts and notification settings.
- [Pricing tier](defender-for-cloud-introduction.md#protect-cloud-workloads): This determines which Defender for Cloud features are available for resources in scope (can be specified for subscriptions and workspaces using the API) and is relevant both with or without Defender for Cloud's Defender plans. (**Comment: I would ask for clarification on this sentence. It's unclear to me**)

> [!NOTE]
> Specifying a security contact ensures that Azure can reach the right person in your organization if a security incident occurs. Read [Provide security contact details in Defender for Cloud](configure-email-notifications.md) for more information on how to enable this recommendation.

### Security policy definitions and recommendations

Defender for Cloud automatically creates a default security policy for each of your Azure subscriptions. You can edit the policy in Defender for Cloud or use Azure Policy to create new definitions, define more policies, and assign policies across management groups. Management groups can represent the entire organization or a business unit within the organization. You can monitor policy compliance across these management groups.

Before configuring security policies, review each of the [security recommendations](review-security-recommendations.md):

- Verify whether these policies are appropriate for your various subscriptions and resource groups.

- Understand which actions address the security recommendations.

- Determine who in your organization is responsible for monitoring and remediating new recommendations.

## Data collection and storage

Defender for Cloud uses the Log Analytics agent and the Azure Monitor agent to collect security data from your virtual machines. [Data collected](monitoring-components.md) from this agent is stored in your Log Analytics workspaces.

### Agent

When automatic provisioning is enabled in the security policy, the [data collection agent](monitoring-components.md) is installed on all supported Azure virtual machines (VMs) and any new VMs that are created and supported. If the VM or computer already has the Log Analytics agent installed, Defender for Cloud uses the current installed agent. The agent's process is designed to be non-invasive and have minimal effect on VM performance.

If you want to disable data collection, you can turn it off at any time in the security policy. However, because the Log Analytics agent might be used by other Azure management and monitoring services, the agent won't be uninstalled automatically when you turn off data collection in Defender for Cloud. You can manually uninstall the agent, if needed.

### Workspace

A workspace is an Azure resource that serves as a container for data. You or other members of your organization might use multiple workspaces to manage different sets of data that is collected from all or portions of your IT infrastructure.

Data collected from the Log Analytics agent can be stored in an existing Log Analytics workspace associated with your Azure subscription or a new workspace.

In the Azure portal, you can browse to see a list of your Log Analytics workspaces, including any created by Defender for Cloud. A related resource group is created for new workspaces. Resources are created according to this naming convention:

- Workspace: *DefaultWorkspace-[subscription-ID]-[geo]*

- Resource Group: *DefaultResourceGroup-[geo]*

For workspaces created by Defender for Cloud, data is retained for 30 days. For existing workspaces, retention is based on the workspace pricing tier. You can also use an existing workspace, if necessary.

If your agent reports to a workspace other than the **default** workspace, any Defender for Cloud [Defender plans](defender-for-cloud-introduction.md#protect-cloud-workloads) that you've enabled on the subscription must also be enabled on the workspace.

> [!NOTE]
> Microsoft makes strong commitments to protect the privacy and security of this data. Microsoft adheres to strict compliance and security guidelines—from coding to operating a service. For more information about data handling and privacy, read [Defender for Cloud Data Security](data-security.md).

## Onboard non-Azure resources

Defender for Cloud can monitor the security posture of your non-Azure computers but you need to first onboard these resources. Read [Onboard non-Azure computers](quickstart-onboard-machines.md) for more information on how to onboard non-Azure resources.

## Ongoing security monitoring

Once you've completed the initial configuration and applied Defender for Cloud recommendations, the next step is to focus on its operational processes.

The Defender for Cloud Overview provides a unified view of security across all your Azure resources and any non-Azure resources you've connected. This example shows an environment with many issues that need resolving:

:::image type="content" source="./media/overview-page/overview.png" alt-text="Screenshot of Defender for Cloud's overview page." lightbox="./media/overview-page/overview.png":::

> [!NOTE]
> Defender for Cloud doesn't interfere with your normal operational procedures. Defender for Cloud passively monitors your deployments and provides recommendations based on the security policies you've enabled.

When you first opt in to use Defender for Cloud for your current Azure environment, make sure that you review all recommendations shown in the **Recommendations** page.

Plan to visit the threat intelligence option as part of your daily security operations. There you can identify security threats against the environment, such as determining whether a particular computer is part of a botnet.

### Monitor for new or changed resources

Most Azure environments are dynamic, with resources regularly being created, spun up or down (**Comment: this is not terminology that I've used before. I would ask if this is OK to use, is it too colloquial, can it be translated easily for localization**), reconfigured, and changed. Defender for Cloud helps ensure that you have visibility into the security state of these new resources.

When you add new resources (VMs, SQL DBs) to your Azure environment, Defender for Cloud automatically discovers these resources and begins to monitor their security, including PaaS web roles and worker roles. If Data Collection is enabled in the [Security Policy](tutorial-security-policy.md), more monitoring capabilities are enabled automatically for your virtual machines.

Make sure that you regularly monitor existing resources for configuration changes that might create security risks or drift from recommended baselines, and for security alerts.

### Harden access and applications

As part of your security operations, you must adopt preventative measures to restrict access to VMs and control the applications that are running on VMs. By locking down inbound traffic to your Azure VMs, you're reducing the exposure to attacks and at the same time providing easy access to connect to VMs when needed. Use the [just-in-time VM access](just-in-time-access-usage.yml) access feature to harden the access to your VMs.

## Incident response

Defender for Cloud detects and alerts you to threats as they occur. Organizations must continuously monitor for new security alerts and take appropriate action to investigate or remediate potential attacks. For more information on how Defender for Cloud threat protection works, read [How Defender for Cloud detects and responds to threats](alerts-overview.md#detect-threats).

Although we can't create your Incident Response plan, we'll use Microsoft Azure Security Response in the Cloud lifecycle as the foundation for incident response stages. The stages of incident response in the cloud lifecycle are:

:::image type="content" source="./media/defender-for-cloud-planning-and-operations-guide/defender-for-cloud-planning-and-operations-guide-fig5-1.png" alt-text="Stages of the incident response in the cloud lifecycle.":::

> [!NOTE]
> You can use the National Institute of Standards and Technology (NIST) [Computer Security Incident Handling Guide](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf) as a reference to assist you building your own.

You can use Defender for Cloud alerts during the following stages:

- **Detect**: Identify a suspicious activity in one or more resources.

- **Assess**: Perform the initial assessment to obtain more information about the suspicious activity.

- **Diagnose**: Use the remediation steps to conduct the technical procedure to address the issue.

Each security alert provides information that can be used to better understand the nature of the attack and suggest possible mitigations. Some alerts also provide links to either more information or to other information sources within Azure. You can use the information provided for further research and to begin mitigation, and you can also search security-related data that is stored in your workspace.

The following example shows a suspicious RDP activity taking place:

:::image type="content" source="./media/defender-for-cloud-planning-and-operations-guide/defender-for-cloud-planning-and-operations-guide-fig5-ga.png" alt-text="Screenshot of a suspicious activity report while it is taking place.":::

This page shows the details regarding the time that the attack took place, the source hostname, and the target VM. The page also provides recommended remediation steps. In some circumstances, the source information of the attack might be empty. Read [Missing Source Information in Defender for Cloud alerts](/archive/blogs/azuresecurity/missing-source-information-in-azure-security-center-alerts) for more information about this type of behavior.

Once you identify the compromised system, you can run an existing [workflow automation](workflow-automation.yml). Workflow automations are a collection of procedures that can be executed from Defender for Cloud once triggered by an alert.

> [!NOTE]
> Read [Managing and responding to security alerts in Defender for Cloud](managing-and-responding-alerts.yml) for more information on how to use Defender for Cloud capabilities to assist you during your Incident Response process.

## Next steps

In this document, you learned how to plan for Defender for Cloud adoption. Learn more about Defender for Cloud:

- [Managing and responding to security alerts in Defender for Cloud](managing-and-responding-alerts.yml)
- [Monitoring partner solutions with Defender for Cloud](./partner-integration.md): Learn how to monitor the health status of your partner solutions.
- [Defender for Cloud common questions](faq-general.yml): Find frequently asked questions about using the service.
- [Azure Security blog](/archive/blogs/azuresecurity/): Read blog posts about Azure security and compliance.
