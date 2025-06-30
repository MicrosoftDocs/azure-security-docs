---
title: Defender for Cloud Planning and Operations Guide
description: This document helps you plan for adopting Defender for Cloud and considerations regarding daily operations.
ms.topic: concept-article
ms.date: 05/16/2024
#customer intent: As a reader, I want to understand the planning and operational considerations for adopting Defender for Cloud and how it aligns with my organization's security requirements and cloud management model. I also want to learn about the security roles, access controls, security policies, data collection, and storage.
---

# Planning and Operations Guide

This guide is for IT professionals, IT architects, security analysts, and cloud administrators planning to use Microsoft Defender for Cloud.

## Planning Guide

This guide outlines how Defender for Cloud fits into your organization's security strategy and cloud management model. Understanding how individuals and teams interact with the service helps support secure development, monitoring, governance, and incident response.

Key areas to consider:
- Security roles and access controls
- Security policies and recommendations
- Data collection and storage
- Onboarding non-Azure resources
- Ongoing security monitoring
- Incident response

The following sections explain how to plan for each area.

> [!NOTE]
> Read [Defender for Cloud common questions](faq-general.yml) for FAQs relevant during the planning phase.

## Security Roles and Access Controls

Depending on your organization’s size and structure, different people and teams may use Defender for Cloud for various security tasks.

>[!NOTE]
Give users the minimum permissions required to do their job effectively.

The diagram below shows example roles and what each person is responsible for:

:::image type="content" source="./media/defender-for-cloud-planning-and-operations-guide/defender-for-cloud-planning-and-operations-guide-fig01-new.png" alt-text="Conceptual image that shows various people and the roles that they fill in an organization.":::

Defender for Cloud helps the following individuals to meet various responsibilities:

**Jeff (Workload Owner)**

- Manages a cloud workload and its resources

- Implements protections according to company policy

**Ellen (CISO/CIO)**

- Oversees all aspects of organizational security

- Aims to understand the company’s overall security status across cloud workloads

- Monitors major attacks and high-risk issues affecting the organization

**David (IT Security)**

- Defines security policies and monitors compliance

- Monitors compliance with policies

- Generates reports for leadership or auditors

**Judy (Security Operations)**

- Monitors and responds to security alerts at any time

- Escalates security alerts to the Cloud Workload Owner or IT Security Analysts

**Sam (Security Analyst)**

- Investigate attacks

- Collaborates with the Cloud Workload Owner to implement remediation

Defender for Cloud uses [Azure role-based access control (Azure Role-based access control)](/azure/role-based-access-control/role-assignments-portal), which provides [built-in roles](/azure/role-based-access-control/built-in-roles) that can be assigned to users, groups, and services in Azure.

In Defender for Cloud, users are assigned the role of Owner, Contributor, or Reader to the subscription or resource group that a resource belongs to, which means they can only see information related to resources they have access to.

In addition to the roles mentioned above, there are two additional roles:

- **Security Reader**: Can view Defender for Cloud configurations, including recommendations, alerts, policies, and health status, but cannot make changes.

- **Security Admin**: Has all the permissions of a **Security Reader**, plus the ability to update security policies and dismiss recommendations and alerts.

The roles shown earlier are typically assigned the following Azure Role-based Access Control roles:

**Jeff (Workload Owner)**

- Resource Group Owner or Contributor

**Ellen (CISO/CIO)**

- Subscription Owner, Contributor or Security Admin

**David (IT Security)**

- Subscription Owner, Contributor or Security Admin

**Judy (Security Operations)**

- Subscription Reader or Security Reader (to view alerts)

- Subscription Owner, Contributor or Security Admin (required to dismiss alerts)

**Sam (Security Analyst)**

- Subscription Reader (to view alerts)

- Subscription Owner or Contributor (required to dismiss alerts)

- Workspace access may be required

> [!NOTE]
Only subscription Owners, Contributors, and Security Admins can edit security policies.
Only subscription and resource group Owners and Contributors can apply security recommendations.

When planning access to Defender for Cloud, first figure out who needs access and what actions they need to take. Then, assign them the appropriate role.

> [!TIP]
It's best to give each user only the access they need to do their job - nothing more.

## Security Policies and Recommendations

A security policy defines how your workloads should be configured to meet company and regulatory security requirements. In Defender for Cloud, you can create custom policies for your Azure subscriptions, adjusting them based on workload type or data sensitivity.

Defender for Cloud policies include the following components:

- [Data collection](monitoring-components.md):  Controls whether the monitoring agent is automatically installed and how security data is collected from your resources.

- [Security policy](tutorial-security-policy.md): An [Azure Policy](/azure/governance/policy/overview) that tells Defender for Cloud what to monitor and recommend. You can create and assign policies to organize and manage security rules across different parts of your organization.

- [Email notifications](configure-email-notifications.md): Configure security contact details and alert settings
  
- [Pricing tier](defender-for-cloud-introduction.md#protect-cloud-workloads): Determines available features based on selected Defender plan, set per subscription or workspace using API.

> [!NOTE]
> Specifying a security contact ensures that Azure can reach the right person in your organization if a security incident occurs. Read [Provide security contact details in Defender for Cloud](configure-email-notifications.md) for more information on how to enable this recommendation.

### Policy Definitions and Recommendations

A default security policy is automatically created for each of your Azure subscriptions in Defender for Cloud.
You can edit this policy or use it as a base to create new definitions, add more policies, and assign them across different parts of your organization.
Management groups can represent the entire organization or a specific business unit. You can track policy compliance across these groups.

Before configuring security policies, review each of the [security recommendations](review-security-recommendations.md):

- Decide whether each recommendation applies to your subscriptions and resource groups

- Understand the actions needed to meet each recommendation

- Determine who in your organization is responsible for monitoring and remediating new recommendations.

## Data Collection and Storage

Defender for Cloud uses both the Log Analytics agent and the Azure Monitor Agent (AMA) to collect security data from your virtual machines. The [data collected](monitoring-components.md) is stored in your Log Analytics workspace.

### Agent

When automatic provisioning is enabled in the security policy, the [data collection agent](monitoring-components.md) is installed on all supported Azure VMs, both existing and newly created. 

If the VM or computer already has the Log Analytics agent installed, Defender for Cloud uses the existing agent. 

The agent is designed to have minimal impact on VM performance.

#### Disabling the Data Collection

To disable the data collection, you must turn it off in the security policy.

Turning off data collection in the security policy does not automatically uninstall the agent. You can manually uninstall the agent if needed.

> [!NOTE]
The Log Analytics agent might be used by other Azure management and monitoring services, so turning off data collection in Defender for Cloud does not remove the agent automatically.

### Workspace

A workspace in Azure is where your collected data lives. Your organization might have several workspaces to manage different sets of data from your infrastructure.

Data gathered by the Log Analytics agent is stored in either an existing workspace linked to your Azure subscription or a new workspace.

You can view all your Log Analytics workspaces in the Azure portal, including those created by Defender for Cloud. When a new workspace is created, a related resource group is also created. They follow this naming pattern:

- Workspace: DefaultWorkspace-[subscription-ID]-[geo]

- Resource Group: DefaultResourceGroup-[geo]

Data in workspaces created by Defender for Cloud is kept for 30 days. Other workspaces keep data based on their pricing plan. You can choose to use an existing workspace if you want.

If your agent sends data to a workspace other than the **default**, make sure the [Defender plans](defender-for-cloud-introduction.md#protect-cloud-workloads) you’ve enabled on your subscription are also turned on for that workspace.

> [!NOTE]
> Microsoft takes your data privacy and security seriously, with strong protections throughout the process. To learn more, read [Defender for Cloud Data Security](data-security.md).

## Onboard Non-Azure Resources

Defender for Cloud can also monitor non-Azure machines, but you need to onboard them first. For help, see [Onboard non-Azure computers](quickstart-onboard-machines.md).

## Ongoing Security Monitoring

After initial configuration and application of Defender for Cloud recommendations, the next step is to consider Defender for Cloud operational processes.

The Overview page gives you a complete look at security across your Azure resources and any non-Azure machines you’ve connected. Here’s an example showing an environment with several problems to fix:

:::image type="content" source="./media/overview-page/overview.png" alt-text="Screenshot of Defender for Cloud's overview page." lightbox="./media/overview-page/overview.png":::

> [!NOTE]
> Defender for Cloud works quietly in the background and **doesn’t interrupt** your usual work. It provides recommendations based on the security policies you have set.

Once you start using Defender for Cloud for your current Azure environment, make sure that you review all recommendations via the **Recommendations** page.

> [!TIP]
> Make it a habit to check the Recommendations page regularly.
> Also, use the threat intelligence feature daily to spot threats like botnets or hacked machines.

### Monitor Resource Changes

With most Azure environments being dynamic, Defender for Cloud helps ensure that you have visibility into the security state of all newly created or modified resources.

Defender for Cloud automatically discovers new resources such as VMs, SQL DBs in your Azure environment, and begins to monitor their security, including PaaS web roles and worker roles. If Data Collection is turned on in your [Security Policy](tutorial-security-policy.md), Defender for Cloud will enable extra monitoring features for your virtual machines.

It’s also important to regularly check existing resources for any changes that could create security risks, cause them to drift from recommended settings, or trigger alerts.

### Harden Access and Applications

A good security practice is to limit who can access your VMs and control which applications run on them.
By restricting inbound traffic, you reduce the chance of attacks but still allow authorized users to connect when needed.
Use [just-in-time VM access](just-in-time-access-usage.yml) to restrict access when necessary.

## Incident Response

Defender for Cloud detects and alerts about threats in real time. Security teams should monitor these alerts and respond promptly. Learn more in [How Defender for Cloud detects and responds to threats](alerts-overview.md#detect-threats).

While this guide does not provide a full Incident Response plan, Microsoft’s Azure Security Response lifecycle is a useful framework. The stages include:

:::image type="content" source="./media/defender-for-cloud-planning-and-operations-guide/defender-for-cloud-planning-and-operations-guide-fig5-1.png" alt-text="Stages of the incident response in the cloud lifecycle.":::

> [!NOTE]
> Consider the National Institute of Standards and Technology (NIST) [Computer Security Incident Handling Guide](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf) when designing your response processes.

Use Defender for Cloud alerts during these phases:

- **Detect**: Identify suspicious activities

- **Assess**: Investigate and collect data to understand the issue

- **Diagnose**: Create and apply remediation procedures

Each alert provides information to help analyze the attack and suggests mitigation. Some alerts link to further resources within Azure.

The following example shows a suspicious RDP activity taking place:

:::image type="content" source="./media/defender-for-cloud-planning-and-operations-guide/defender-for-cloud-planning-and-operations-guide-fig5-ga.png" alt-text="Screenshot of a suspicious activity report while it is taking place.":::

This report shows details such as attack time, source hostname, target VM, and recommendations. If source details are missing, see [Missing Source Information in Defender for Cloud alerts](/archive/blogs/azuresecurity/missing-source-information-in-azure-security-center-alerts).

Once you find the compromised system, run a [workflow automation](workflow-automation.yml) that was previously created. Workflow automations are triggered procedures that help respond quickly.

> [!NOTE]
> For more on alerts and incident response, read [Managing and responding to security alerts in Defender for Cloud](managing-and-responding-alerts.yml) 

## Next Steps

Now you know how to plan for using Defender for Cloud. To learn more, check out:

- [Managing and responding to security alerts in Defender for Cloud](managing-and-responding-alerts.yml)
- [Monitoring partner solutions with Defender for Cloud](./partner-integration.md) - Learn how to monitor the health status of your partner solutions.
- [Defender for Cloud common questions](faq-general.yml) - Find frequently asked questions about using the service.
- [Azure Security blog](/archive/blogs/azuresecurity/) - Read blog posts about Azure security and compliance.
