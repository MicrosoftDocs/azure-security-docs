---
title: Defender for Cloud Planning and Operations Guide
description: This document helps you prepare to adopt Defender for Cloud and raises related considerations regarding your daily operations.
ms.topic: concept-article
ms.date: 05/16/2024
#customer intent: As a reader, I want to understand the planning and operations considerations for adopting Defender for Cloud and how it fits into my organization's security requirements and cloud management model. I also want to learn about the security roles, access controls, security policies, data collection, and storage in Defender for Cloud.
---

# Planning and operations guide

This guide helps IT professionals, architects, security analysts, and cloud administrators prepare to use Defender for Cloud.

## Planning guide

This guide details how Defender for Cloud fits into your organization's security requirements and cloud management model. It's important to understand how different individuals or teams in your organization use this service to meet their needs in secure development and operations, monitoring, governance, and incident response. While planning to use Defender for Cloud, make sure to consider the following ideas:

- Security roles and access controls
- Managing security policies and recommendations
- Data collection and storage
- Onboarding non-Azure resources
- Ongoing security monitoring and remediation
- Incident response

In the upcoming sections, you'll learn how to implement each of these ideas and apply specific recommendations based on the requirements of your organization.

> [!NOTE]
> See [Defender for Cloud common questions](faq-general.yml) for a list of common questions that you can use during the designing and planning phase.

## Security roles and access controls

Depending on the size and structure of your organization, multiple individuals and teams might use Defender for Cloud to perform different security-related tasks. The following graphic presents examples of fictitious personas along with their respective roles and security responsibilities:

![Conceptual image that shows various people and the roles that they fill in an organization.](./media/defender-for-cloud-planning-and-operations-guide/defender-for-cloud-planning-and-operations-guide-fig01-new.png)


Defender for Cloud enables these individuals to meet these various responsibilities. For example:

| Persona         | Responsibilities                                                                                                          | Required Azure role-based access control roles                                                      |
|-----------------|----------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------|
| Workload Owner  | - Manage a cloud workload and its related resources.  <br> - Implement and maintain protections per company policy.        | Resource Group Owner/Contributor                                                                     |
| CISO/CIO        | - Responsible for all aspects of company security. <br> - Understand security posture across workloads. <br> - Stay informed of major attacks and risks. | Subscription Owner/Contributor or Security Admin                                                    |
| IT Security     | - Set company security policies. <br> - Monitor policy compliance. <br> - Generate reports for leadership or auditors.     | Subscription Owner/Contributor or Security Admin                                                    |
| Security Operations | - Monitor and respond to security alerts. <br> - Escalate to Cloud Workload Owner or IT Security Analyst.                  | - Reader or Security Reader (to view alerts) <br> - Owner/Contributor or Security Admin (to dismiss alerts) |
| Security Analyst | - Investigate attacks. <br> - Work with Cloud Workload Owner to apply remediation.                                         | - Reader (to view alerts) <br> - Owner/Contributor (to dismiss alerts) <br> - May require workspace access |


Defender for Cloud uses [Azure role-based access control (Azure Role-based access control)](/azure/role-based-access-control/role-assignments-portal), which provides [built-in roles](/azure/role-based-access-control/built-in-roles) that can be assigned to users, groups, and services in Azure. It displays only the information related to resources that the logged-in user can access based on their assigned role as Owner, Contributor, or Reader to a resource's subscription or resource group. In addition to the roles mentioned above, there are also the following Defender for Cloud-specific roles:

- **Security reader**: A user who can view, but not edit, Defender for Cloud configurations, including recommendations, alerts, policy, and health configurations.

- **Security admin**: Same as security reader but they can also update the security policy, and dismiss alerts and recommendations.

In granting these roles it is important to considert that only subscription Owners/Contributors and Security Admins can edit a security policy and only subscription and resource group Owners and Contributors can apply security recommendations for a resource.

Once you understand who needs access to Defender for Cloud and what they'll do, then you can properly configure Azure role-based access control.

> [!NOTE]
> We recommend that you assign the least privileged role needed for users to complete their tasks. For example, users who only need to view information about the security state of resources but not take action, such as applying recommendations or editing policies, should be assigned the Reader role.

## Manage security policies and recommendations

A security policy defines the desired configuration of your workloads and helps ensure compliance with company or regulatory security requirements. In Defender for Cloud, you can define policies for your Azure subscriptions, and tailor them to the type of workload or the sensitivity of data in your organization.

Defender for Cloud policies contain the following components:

- [Data collection](monitoring-components.md): Agent provisioning and data collection settings

- [Security policy](tutorial-security-policy.md): An [Azure policy](/azure/governance/policy/overview) that determines which controls are monitored and recommended by Defender for Cloud. You can also use an Azure policy to create new definitions, define more policies, and assign policies across management groups.

- [Email notifications](configure-email-notifications.md): Security contacts and notification settings
- [Pricing tier](defender-for-cloud-introduction.md#protect-cloud-workloads): Your choice of plan—either with or without Microsoft Defender plans—determines which Defender for Cloud features are available for your resources. Defender plans enable advanced capabilities such as threat protection and vulnerability management. You can configure pricing tiers at the subscription or workspace level, including through the API.

> [!NOTE]
> Specify a security contact to ensure that Azure can reach the right person in your organization in the event of a security incident. See [Provide security contact details in Defender for Cloud](configure-email-notifications.md) for more information on how to enable this recommendation.

### Security policy definitions and recommendations

Defender for Cloud automatically creates a default security policy for each of your Azure subscriptions. You can edit the policy in Defender for Cloud or use Azure Policy to create new definitions, define more policies, and assign policies across management groups. Management groups can represent the entire organization or a business unit within the organization and you can monitor policy compliance across these management groups.

Before configuring security policies, review each of the [security recommendations](review-security-recommendations.md):

- See if these policies are appropriate for your various subscriptions and resource groups.

- Understand what actions address the security recommendations.

- Determine who in your organization is responsible for monitoring and remediating new recommendations.

## Data collection and storage

Defender for Cloud uses the Log Analytics agent and the Azure Monitor Agent to collect security data from your virtual machines. [Data collected](monitoring-components.md) from this agent is stored in your Log Analytics workspaces.

### Agent

When automatic provisioning is enabled in the security policy, the [data collection agent](monitoring-components.md) is installed on all supported Azure VMs and any new supported VMs that are created. If the VM or computer already has the Log Analytics agent installed, Defender for Cloud uses the current installed agent. The agent's process is designed to be non-invasive and have minimal effect on VM performance.

If at some point you want to disable Data Collection, you can turn it off in the security policy. However, if you want to uninstall the agent, you must do so manually, because the Log Analytics agent might be used by other Azure management and monitoring services.

### Workspace

A workspace is an Azure resource that serves as a container for data. You or other members of your organization might use multiple workspaces to manage different sets of data that are collected from your IT infrastructure.

Data collected from the Log Analytics agent can be stored in an existing Log Analytics workspace associated with your Azure subscription or in a new workspace.

You can browse the Azure portal for a list of your Log Analytics workspaces, including any created by Defender for Cloud. A related resource group is created for new workspaces. Resources are created according to this naming convention:

- Workspace: *DefaultWorkspace-[subscription-ID]-[geo]*

- Resource Group: *DefaultResourceGroup-[geo]*

For workspaces created by Defender for Cloud, data is retained for 30 days. For existing workspaces, retention is based on the workspace pricing tier.

If your agent reports to a workspace other than the **default** workspace, any Defender for Cloud [Defender plans](defender-for-cloud-introduction.md#protect-cloud-workloads) that you've enabled on the subscription should be enabled on the workspace as well.

> [!NOTE]
> Microsoft makes strong commitments to protect the privacy and security of this data. Microsoft adheres to strict compliance and security guidelines—from coding to operating a service. See [Defender for Cloud Data Security](data-security.md) for more information about data handling and privacy.

## Onboard non-Azure resources

Defender for Cloud can monitor the security posture of your non-Azure computers once you onboard them. See [Onboard non-Azure computers](quickstart-onboard-machines.md) for more information on how to onboard non-Azure resources.

## Ongoing security monitoring and remediation

After the initial configuration and application of Defender for Cloud recommendations, the next step is to consider operational processes.

The Overview screen provides a unified view of the security situation across all of your connected resources. This example shows an environment with many issues to resolve:

![Defender for Cloud's overview page.](./media/overview-page/overview.png)

> [!NOTE]
> Defender for Cloud passively monitors your deployments and provides recommendations based on the security policies you enabled. It does not interfere with your normal operational procedures.

When you first opt in to use Defender for Cloud for your current Azure environment, make sure that you review all recommendations on the **Recommendations** screen.

The threat intelligence option allows you to identify security threats against the environment, such as computers that are part of a botnet. We recommend reviewing the threat intelligence as part of your daily security operations.

### Monitor for new or changed resources

Most Azure environments are dynamic, with resources regularly being created, spun up or down, reconfigured, and changed. Defender for Cloud helps ensure that you have visibility into the security state of these new resources.

When you add new resources (VMs, SQL DBs) to your Azure environment, Defender for Cloud automatically discovers these resources and begins to monitor their security, including PaaS web roles and worker roles. If Data Collection is enabled in the [Security Policy](tutorial-security-policy.md), more monitoring capabilities are enabled automatically for your virtual machines.

You should also regularly monitor existing resources for configuration changes that could create security risks and alerts, as well as drift from recommended baselines.

### Harden access and applications

Consider expanding this section to clarify that hardening is not a binary state, but an ongoing, risk-based process.As part of your security operations, adopt preventative measures to restrict access to VMs, and control the applications that are running on VMs. By locking down inbound traffic to your Azure VMs, you're reducing the exposure to attacks, and at the same time providing easy access to connect to VMs when needed. Use the [just-in-time VM access](just-in-time-access-usage.yml) feature to harden access to your VMs.

## Incident response

Defender for Cloud detects and alerts you to threats in real-time. Organizations should monitor for new security alerts and take action to investigate or remediate the attack. See [How Defender for Cloud detects and responds to threats](alerts-overview.md#detect-threats) for more information on how threat protection works.

Although we can't create your Incident Response plan, we'll use Microsoft Azure Security Response in the Cloud lifecycle as the foundation for incident response stages. The stages of incident response in the cloud lifecycle are:

![Stages of the incident response in the cloud lifecycle.](./media/defender-for-cloud-planning-and-operations-guide/defender-for-cloud-planning-and-operations-guide-fig5-1.png)


> [!NOTE]
> You can use the National Institute of Standards and Technology (NIST) [Computer Security Incident Handling Guide](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf) as a reference to assist you building your own Incident Response plan.

You can use Defender for Cloud alerts during the following stages of your plan:

- **Detect**: Identify a suspicious activity in one or more resources.

- **Assess**: Perform the initial assessment to get more information about the suspicious activity.

- **Diagnose**: Use the remediation steps to address the issue.

Each security alert provides information that can be used to better understand the nature of the attack and suggest possible mitigations. Some alerts also provide links to either more information or to other sources of information within Azure. You can use the information provided for further research and to begin mitigation. You can also search security-related data that is stored in your workspace.

The following example shows a suspicious RDP activity taking place:

![A suspicious activity report while it is happening.](./media/defender-for-cloud-planning-and-operations-guide/defender-for-cloud-planning-and-operations-guide-fig5-ga.png)

This page shows the details regarding the time of the attack, the source hostname, and the target VM. It also gives recommendation steps. In some circumstances, the source information of the attack might be empty. See [Missing Source Information in Defender for Cloud alerts](/archive/blogs/azuresecurity/missing-source-information-in-azure-security-center-alerts) for more information about this type of behavior.

Once you identify the compromised system, you can run a previously created [workflow automation](workflow-automation.yml). Workflow automations are a collection of procedures that can be executed from Defender for Cloud once triggered by an alert.

> [!NOTE]
> See [Managing and responding to security alerts in Defender for Cloud](managing-and-responding-alerts.yml) for more information on how to use Defender for Cloud capabilities to assist you during your Incident Response process.

## Learn more

In this document, you learned how to plan for Defender for Cloud adoption. Learn more at the following links:

- [Managing and responding to security alerts in Defender for Cloud](managing-and-responding-alerts.yml) - Learn how Defender for Cloud can assist in your Incident Response process.
- [Monitoring partner solutions with Defender for Cloud](./partner-integration.md) - Learn how to monitor the health status of your partner solutions.
- [Defender for Cloud common questions](faq-general.yml) - Find frequently asked questions about using the service.
- [Azure Security blog](/archive/blogs/azuresecurity/) - Read blog posts about Azure security and compliance.
