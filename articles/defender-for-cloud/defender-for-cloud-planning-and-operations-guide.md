---
title: Defender for Cloud Planning and Operations Guide
description: This document helps you to plan before adopting Defender for Cloud and covers daily operations considerations.
ms.topic: concept-article
ms.date: 05/16/2024
#customer intent: As a reader, I want to understand the planning and operations considerations for adopting Defender for Cloud and how it fits into my organization's security requirements and cloud management model. I also want to learn about the security roles, access controls, security policies, data collection, and storage in Defender for Cloud.
---

# Planning and operations guide

This guide is for information technology (IT) professionals, IT architects, information security analysts, and cloud administrators and provides the background for how Defender for Cloud fits into your organization's security requirements and cloud management model. It's important to understand how different individuals or teams in your organization use the service to meet secure development and operations, monitoring, governance, and incident response needs. 

Key areas to consider when planning to use Defender for Cloud:

- Security roles and access controls
- Security policies and recommendations
- Data collection and storage
- Onboarding non-Azure resources
- Ongoing security monitoring
- Incident response

In the next section, you'll learn how to plan for each of these areas and apply recommendations based on your requirements.

> [!NOTE]
> Read [Defender for Cloud common questions](faq-general.yml) for more info that can be useful during the designing and planning phase.

## Security roles and access controls
<!--I had several concerns/comments about this section. 1. Why do we have a content-heavy diagram and then repeat most of the content again beneath it? This is redundant and overloads the users with text. 2. Not all the content in the diagram is in the written version below, meaning some of the content isn't accessible (screen readers, etc), nor is it included in the alt text). While I have written out the missing content below so it will be complete and accessible, ideally we could simplify the graphic. 3. There are some typos/formatting problems with the image, please see corrected one attached to submission email. 4. The list of names is repeated twice, separating the title and responsibilities from the assigned roles. This places a cognitive load on the user to remember which responsibility lines up with which role. I have synthesized both sections together so that it's clear how responsibilities and roles correlate and turned this into a table, hopefully making this more scannable as well. 5. I would consider presenting the content in a table format. Overall, I would want to sit with the lead on this project and discuss how we could have the diagram and content complement each other, with less repetition and more scannability. If KPIs were important for this article, I'd suggest AB testing different options.-->

Depending on the size and structure of your organization, multiple individuals and teams might use Defender for Cloud to perform different security-related tasks. Defender for Cloud uses [Azure role-based access control](/azure/role-based-access-control/role-assignments-portal), which provides [built-in roles](/azure/role-based-access-control/built-in-roles) that can be assigned to users, groups, and services in Azure. When a user opens Defender for Cloud, they'll only see information related to resources they have access to. They can be assigned the role of **Owner**, **Contributor**, or **Reader** to the subscription or resource group that a resource belongs to.<!--made roles bold so they'd match the roles beneath-->

In addition to these roles, there are two more roles specific to Defender for Cloud:

- **Security Reader**: a user that belongs to this role is able to view only Defender for Cloud configurations, which include recommendations, alerts, policy, and health, but they won't be able to make changes.

- **Security Admin**: same as Security Reader, but they can also update the security policy and dismiss recommendations and alerts.

Some other important information to consider:

- Only subscription Owners/Contributors and Security Admins can edit a security policy.

- Only subscription and resource group Owners and Contributors can apply security recommendations for a resource.

When planning access control, make sure you understand who in your organization needs access to Defender for Cloud according to the tasks they need to perform. Then you can configure Azure Role-based access control properly.

> [!NOTE]
> We recommend that you assign the least permissive role needed for users to complete their tasks. For example, users who only need to view security information but not edit policies should be assigned the Reader role.<!-simplified slightly so the example would be clearer. I'd confirm with PM in real life.-->

### Example of assigning roles in an organization
The following diagram shows a made-up organization with various personas, each with their respective roles and security responsibilities: <!--replaced "fictitious" with "made-up," in case that term was too advanced (in keeping with the Microsoft style guide's preference for simpler language when possible).-->

:::image type="content" source="./media/defender-for-cloud-planning-and-operations-guide/defender-for-cloud-planning-and-operations-guide-fig01-new.png" alt-text="Conceptual image that shows various people and the roles that they fill in an organization.":::

See how Defender for Cloud enables these individuals to meet their various responsibilities by assigning them different levels of access, according to their Azure Role-based access control roles:

| Name (Title) | Responsibilities | Azure Role |
| ------------- |-------------|-------------|
| **Jeff<br>(Workload Owner)**|<ul><li>Manages a cloud workload and its related resources (often in DevOps role)</li><li>Responsible for implementing and maintaining protections in accordance with company security policy</li><li>In small organizations, also defines policy and monitors alerts</ul>|Resource Group Owner/Contributor
| **Ellen<br>(CISO/CIO)**|<ul><li>Responsible for all aspects of security for the company</li><li>Wants to understand the company's security posture across cloud workloads</li><li>Needs to be informed of major attacks and risks</u>|Subscription Owner/Contributor or Security Admin
|**David<br>(IT Security)**|<ul><li>Sets company security policies to ensure the appropriate protections are in place</li><li>Monitors compliance with policies</li><li>Generates reports for leadership or auditors</u>|Subscription Owner/Contributor or Security Admin
|**Judy<br>(Security Operations)**|<ul><li>Monitors and responds to security alerts 24/7</li><li>Escalates to Cloud Workload Owner or IT Security Analyst</li><li>Sometimes performed by a Managed Security Provider</u>|Subscription Reader or Security Reader to view alerts<br><br>Subscription Owner/Contributor or Security Admin required to dismiss alerts|
|**Sam<br>(Security Analyst)**|<ul><li>Investigates attacks</li><li>Works with Cloud Workload Owner to apply remediation</u>|Subscription Reader to view alerts<br><br>Subscription Owner/Contributor required to dismiss alerts<br><br>Access to the workspace might be required|


## Security policies and recommendations

A security policy defines the desired configuration of your workloads and helps ensure compliance with company or regulatory security requirements. In Defender for Cloud, you can define policies for your Azure subscriptions, which can be tailored to the type of workload or the sensitivity of data.

Defender for Cloud policies contain the following components:

- [Data collection](monitoring-components.md): agent provisioning and data collection settings

- [Security policy](tutorial-security-policy.md): an [Azure Policy](/azure/governance/policy/overview) that determines which controls are monitored and recommended by Defender for Cloud. You can also use Azure Policy to create new definitions, define more policies, and assign policies across management groups.

- [Email notifications](configure-email-notifications.md): security contacts and notification settings

- [Pricing tier](defender-for-cloud-introduction.md#protect-cloud-workloads): with or without Microsoft Defender for Cloud's Defender plans, which determine which Defender for Cloud features are available for resources in scope (can be specified for subscriptions and workspaces using the API).<!--the "with or without" sounds awkard at the front. I would ask PM to clarify what they would like to say and rephrase, if possible.-->

> [!NOTE]
> Specifying a security contact ensures that Azure can reach the right person in your organization if a security incident occurs. Read [Provide security contact details in Defender for Cloud](configure-email-notifications.md) for more information on how to enable this recommendation.<!--hyperlink goes to article called "Configure email notifications for alerts and attack paths." I would check with PM if the hyperlink text is correct, because ideally they should match-->

### Security policies definitions and recommendations

Defender for Cloud automatically creates a default security policy for each of your Azure subscriptions. You can edit the policy in Defender for Cloud or use Azure Policy to create new definitions, define more policies, and assign policies across management groups. Management groups can represent the entire organization or a business unit within the organization. You can monitor policy compliance across these management groups.

Before configuring security policies, review each of the [security recommendations](review-security-recommendations.md):

- See if these policies are appropriate for your various subscriptions and resource groups.

- Understand what actions address the security recommendations.

- Determine who in your organization is responsible for monitoring and remediating new recommendations.

## Data collection and storage

Defender for Cloud uses the Log Analytics agent and the Azure Monitor Agent to collect security data from your virtual machines (VMs). [Data collected](monitoring-components.md) from this agent is stored in your Log Analytics workspaces.<!--There were two hyperlinks leading to the same article. In general it's bad practice to have neighboring links go to the same article, so for the purposes of this test I deleted the second one. In real life, I would ask the PM; perhaps they had meant to link to a different article entirely.-->

### Agent

When automatic provisioning is enabled in the security policy, the data collection agent is installed on all supported Azure VMs and any new supported VMs that are created. If the VM or computer already has the Log Analytics agent installed, Defender for Cloud uses the current installed agent. The agent's process is designed to be non-invasive and have minimal effect on VM performance.

If at some point you want to disable Data Collection, you can turn it off in the security policy. However, because the Log Analytics agent might be used by other Azure management and monitoring services, the agent won't be uninstalled automatically when you turn off data collection in Defender for Cloud. You can manually uninstall the agent if needed.

### Workspace

A workspace is an Azure resource that serves as a container for data. You or other members of your organization might use multiple workspaces to manage different sets of data that is collected from all or portions of your IT infrastructure.

Data collected from the Log Analytics agent can be stored in an existing Log Analytics workspace associated with your Azure subscription or in a new workspace.

In the Azure portal, you can browse to see a list of your Log Analytics workspaces, including any created by Defender for Cloud. A related resource group is created for new workspaces. Resources are created according to this naming convention:

- Workspace: *DefaultWorkspace-[subscription-ID]-[geo]*

- Resource Group: *DefaultResourceGroup-[geo]*

For workspaces created by Defender for Cloud, data is retained for 30 days. For existing workspaces, retention is based on the workspace pricing tier. <!--I have removed the following line because it didn't make sense in context. If actively working on this article, I would ask the lead to explain what they meant. "If you want, you can also use an existing workspace."-->

If your agent reports to a workspace other than the **default** workspace, any Defender for Cloud [Defender plans](defender-for-cloud-introduction.md#protect-cloud-workloads) that you've enabled on the subscription should also be enabled on the workspace.

> [!NOTE]
> Microsoft makes strong commitments to protect the privacy and security of this data. Microsoft adheres to strict compliance and security guidelines—from coding to operating a service. For more information about data handling and privacy, read [Defender for Cloud Data Security](data-security.md).

## Onboarding non-Azure resources

Defender for Cloud can monitor the security posture of your non-Azure computers, but you need to first onboard these resources. Read [Onboard non-Azure computers](quickstart-onboard-machines.md) for more information.

## Ongoing security monitoring

After initial configuration and application of Defender for Cloud recommendations, the next step is to consider Defender for Cloud operational processes.

The Defender for Cloud Overview provides a unified view of security across all your Azure resources and any non-Azure resources you've connected. This example shows an environment with many issues to resolve:

:::image type="content" source="./media/overview-page/overview.png" alt-text="Screenshot of Defender for Cloud's overview page." lightbox="./media/overview-page/overview.png":::

When you first opt in to use Defender for Cloud for your current Azure environment, make sure that you review all recommendations, which can be found on the **Recommendations** page.<!--I would ask if we could hyperlink the Recommendations page and shorten this sentence.-->

Plan to visit the threat intelligence option as part of your daily security operations. There you can identify security threats against the environment, such as identifying if a particular computer is part of a botnet.

> [!NOTE]
> Defender for Cloud doesn't interfere with your normal operational procedures. Defender for Cloud passively monitors your deployments and provides recommendations based on the security policies you enabled.

### Monitor for new or changed resources

Most Azure environments are dynamic, with resources regularly being created, spun up or down, reconfigured, and changed. Defender for Cloud helps ensure that you have visibility into the security state of these new resources.

When you add new resources (VMs, SQL DBs) to your Azure environment, Defender for Cloud automatically discovers these resources and begins to monitor their security, including PaaS web roles and worker roles. If Data Collection is enabled in the [Security Policy](tutorial-security-policy.md), more monitoring capabilities are enabled automatically for your virtual machines.

You should also regularly monitor existing resources for configuration changes that could have created security risks, drift from recommended baselines, and security alerts.

### Harden access and applications

As part of your security operations, you should also adopt preventative measures to restrict access to VMs and control the applications that are running on VMs. By locking down inbound traffic to your Azure VMs, you're reducing your exposure to attacks, and at the same time providing easy access to connect to VMs when needed. Use [just-in-time VM access](just-in-time-access-usage.yml) feature to harden access to your VMs.

## Incident response

Defender for Cloud detects and alerts you to threats as they occur. Organizations should monitor for new security alerts and take action as needed to investigate further or remediate the attack. For more information on how threat protection works, read [How Defender for Cloud detects and responds to threats](alerts-overview.md#detect-threats).

Although we can't create your incident response plan, we'll use Microsoft Azure Security Response in the Cloud lifecycle as the foundation for incident response stages. The stages of incident response in the cloud lifecycle are:

:::image type="content" source="./media/defender-for-cloud-planning-and-operations-guide/defender-for-cloud-planning-and-operations-guide-fig5-1.png" alt-text="Stages of the incident response in the cloud lifecycle: detect, assess, diagnose, stabilize, close.":::
<!--Added content to alt text so screen readers can read the words in the image-->
> [!NOTE]
> You can use the National Institute of Standards and Technology (NIST) [Computer Security Incident Handling Guide](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf) as a reference to assist you building your own.

You can use Defender for Cloud alerts during the following stages:

- **Detect**: identify a suspicious activity in one or more resources

- **Assess**: perform the initial assessment to obtain more information about the suspicious activity

- **Diagnose**: use the remediation steps to conduct the technical procedure to address the issue

Each security alert provides information that can be used to better understand the nature of the attack and suggest possible mitigations. Some alerts also provide links to either more information or to other sources of information within Azure. You can use the information provided for further research and to begin mitigation, and you can also search security-related data that is stored in your workspace.

The following example shows a suspicious Remote Desktop Protocol (RDP) activity taking place:

:::image type="content" source="./media/defender-for-cloud-planning-and-operations-guide/defender-for-cloud-planning-and-operations-guide-fig5-ga.png" alt-text="Screenshot of a suspicious activity report while it is taking place.":::

This page shows the details regarding the time that the attack took place, the source hostname, the target VM, and also gives recommendation steps. In some circumstances, the source information of the attack might be empty. Read [Missing Source Information in Defender for Cloud alerts](/archive/blogs/azuresecurity/missing-source-information-in-azure-security-center-alerts) for more information about this type of behavior.

Once you identify the compromised system, you can run a [workflow automation](workflow-automation.yml) that was previously created. Workflow automations are a collection of procedures that can be executed from Defender for Cloud once triggered by an alert.

> [!NOTE]
> Read [Managing and responding to security alerts in Defender for Cloud](managing-and-responding-alerts.yml) for more information on how to use Defender for Cloud capabilities to assist you during your incident response process.

## Next steps

In this document, you learned how to plan for Defender for Cloud adoption. Learn more about Defender for Cloud:

- [Managing and responding to security alerts in Defender for Cloud](managing-and-responding-alerts.yml)
- [Monitoring partner solutions with Defender for Cloud](./partner-integration.md): Learn how to monitor the health status of your partner solutions.
- [Defender for Cloud common questions](faq-general.yml): Find frequently asked questions about using the service.
- [Azure Security blog](/archive/blogs/azuresecurity/): Read blog posts about Azure security and compliance.
