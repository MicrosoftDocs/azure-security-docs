---
title: Defender for Cloud Planning and Operations Guide
description: This document helps you to plan before adopting Defender for Cloud and considerations regarding daily operations.
ms.topic: concept-article
ms.date: 05/16/2024
#customer intent: As a reader, I want to understand the planning and operations considerations for adopting Defender for Cloud and how it fits into my organization's security requirements and cloud management model. I also want to learn about the security roles, access controls, security policies, data collection, and storage in Defender for Cloud.
---

# Planning and operations guide

This guide is intended for the following professionals who plan to use Defender for Cloud:
-	Information Technology (IT) specialists 
-	IT architects 
-	Information security analysts 
-	Cloud administrators 



## Planning the implementation of Defender for Cloud

This guide provides the background for the way Defender for Cloud can fit into your organization security requirements and cloud management model. 

It is important to understand how different individuals or teams in your organization can use  Defender for Cloud to meet the needs of secure development and operations, monitoring, governance, and incident responses. The key areas to consider when planning to implement Defender for Cloud are:

- Security Roles and Access Controls – assigning security privileges to your organization professionals. 
-	Security Policies and Recommendations – configuring security policies for your resources and workloads.
-	Data Collection and Storage – managing the Data Collection Agent, which collects data, and the Log Analytics workspace, which stores your security data.  
-	Onboarding non-Azure resources – connecting non-Azure resource to Defender for Cloud.

In the next section, you will learn how to plan for each one of those areas and apply these guidelines based on your requirements.

> [!NOTE]
> For a list of common questions that can assist you during the designing and planning phase, see [Defender for Cloud common questions](faq-general.yml).

### Assigning Security roles and access controls

Depending on the size and structure of your organization, multiple individuals and teams may use Defender for Cloud to perform different security-related tasks. Each individual and team can be assigned different privileges according to their role and needs. 

Defender for Cloud uses [Azure role-based access control](/azure/role-based-access-control/role-assignments-portal), which provides [built-in roles](/azure/role-based-access-control/built-in-roles) that can be assigned to users, groups, and services in Azure. These roles are associated with a specific subscription or resource group. Therefore, when you want users to view or manage specific resources, you need to assign to them a role to the subscription or resource group that includes the required resources. In addition to Azure built-in roles, Defender for Cloud provides two additional Security Roles that you can assign to users.

The assigned roles determine what information the users will be able to view, and what actions they will be able to perform. After users are assigned roles, when they open Defender for Cloud, they can only see information that is related to the resources to which they have access, and they can only perform actions that are granted to them by their role. 

The standard built-in roles are: 
- **Owner** 
- **Contributor** 
- **Reader** 

In addition to these standard roles, there are two additional roles specific to Defender for Cloud:
- **Security Reader**: allows only to view Defender for Cloud configurations, which include recommendations, alerts, policy, and health. This role does not allow making any changes to these configurations.
- **Security Admin**: includes the Security Reader privileges, while also allowing the update of security policies, and the dismissal of recommendations and alerts.

When planning the role assignment, you should also take into consideration the following:
- Only subscription **Owners/Contributors** and **Security Admins** can edit a security policy.
- Only subscription and resource group **Owners** and **Contributors** can apply security recommendations for a resource.

When planning access control using the Azure Role-based access control for Defender for Cloud, make sure you understand who in your organization should have access to Defender for Cloud according to the tasks they will need to perform. This information will help you configure Azure Role-based access control appropriately.

> [!NOTE]
> We recommend that you assign the least permissive role needed for users to complete their tasks. For example, users who only need to view information about the security state of resources but not to take action, such as applying recommendations or editing policies, should be assigned the Reader role.

#### An example of analyzing and assigning security responsibilities and roles

In the following diagram, you have an example of fictitious personas and their respective roles and security responsibilities:

:::image type="content" source="./media/defender-for-cloud-planning-and-operations-guide/defender-for-cloud-planning-and-operations-guide-fig01-new.png" alt-text="Conceptual image that shows various people and the roles that they fill in an organization.":::

Defender for Cloud enables these individuals to meet their various responsibilities. For example:

**Jeff (Workload Owner)**

- Manages a cloud workload and its related resources.
  
- Responsible for implementing and maintaining protections in accordance with the company security policy.

**Ellen (CISO/CIO)**

- Responsible for the entire security aspects and needs of the company.
  
- Manages the company security posture across cloud workloads.
  
- Needs to be informed of major attacks and risks.

**David (IT Security)**

- Sets company security policies to ensure the appropriate protections are in place.

- Monitors compliance with policies.

- Generates reports for leadership or auditors.

**Judy (Security Operations)**

- Monitors and responds to security alerts at any time.

- Escalates to Cloud Workload Owner or IT Security Analyst.

**Sam (Security Analyst)**

- Investigates attacks.

- Works with Cloud Workload Owner to apply remediation.


The personas explained in the previous diagram need the following Azure Role-based access control roles:

**Jeff (Workload Owner)**

- Resource Group Owner/Contributor.

**Ellen (CISO/CIO)**

- Subscription Owner/Contributor or Security Admin.

**David (IT Security)**

- Subscription Owner/Contributor or Security Admin.

**Judy (Security Operations)**

- Subscription Reader or Security Reader to view alerts.

- Subscription Owner/Contributor or Security Admin to dismiss alerts.

**Sam (Security Analyst)**

- Subscription Reader to view alerts.

- Subscription Owner/Contributor to dismiss alerts.

- Access to the workspace might be required.


### Configuring security policies and recommendations

A Security Policy defines security standards and remediation recommendations for your resources and workloads. On a regular basis, Defender for Cloud assesses your resources and workloads against these security standards. If security or compliance issues are detected, Defender for Cloud provides recommendations that include practical steps to remediate these issues. 

Defender for Cloud automatically creates a default security policy for each of your Azure subscriptions. In Defender for Cloud, you can define or edit policies for your Azure subscriptions, which can be tailored to the specific type of workload or to the sensitivity of your data. You can also use Azure Policy page to create new definitions, set more policies, and assign policies across management groups. Management groups can represent the entire organization or a business unit within the organization. You can monitor policy compliance across these management groups.

Defenders for Cloud policies contain the following components:

- [Data collection](monitoring-components.md): agent provisioning and data collection settings.

- [Security policy](tutorial-security-policy.md): an [Azure Policy](/azure/governance/policy/overview) that determines which resources are monitored and assessed by Defender for Cloud.

- [Email notifications](configure-email-notifications.md): security contacts and notification settings.
> [!NOTE]
> Specifying a security contact ensures that Azure can reach the right person in your organization if a security incident occurs. To learn how to enable this feature, see [Provide security contact details in Defender for Cloud](configure-email-notifications.md).
  
- [Pricing tier](defender-for-cloud-introduction.md#protect-cloud-workloads): a pricing plan that determines which Defender for Cloud features are available for resources in your scope. It can be specified for subscriptions and workspaces using the API.

Before editing security policies, review each of the security recommendations that are included in the existing policies:
-	See if these recommendations are appropriate for your various subscriptions and resource groups.
-	Understand what actions are prompted by the security recommendations.
-	Determine who in your organization is responsible for monitoring and remediating new recommendations.


### Managing the collection and storage of security data

Defender for Cloud uses the Log Analytics Agent and the Azure Monitor Agent to collect security data from your virtual machines. [Data collected](monitoring-components.md) from these Agents is stored in your Log Analytics workspaces.

#### Data Collection Agent

When automatic provisioning is enabled in the security policy, the Log Analytics Agent, which is the [data collection agent](monitoring-components.md), is installed on all supported Azure VMs and any new supported VMs that are created. If the VM or computer already has the Log Analytics Agent installed, Defender for Cloud uses the current installed Agent. The Agent process is designed to be non-invasive, and it has minimal effect on VM performance.

If at some point you want to disable Data Collection, you can disable this feature in the security policy. However, because the Log Analytics Agent might be used by other Azure management and monitoring services, the Agent will not be uninstalled automatically when you turn off the Data Collection feature in Defender for Cloud. You can manually uninstall the Agent if needed.

#### Workspace Data

A workspace is an Azure resource that serves as a container for data. You or other members of your organization might use multiple workspaces to manage different sets of data that is collected from all or portions of your IT infrastructure.

Data collected from the Log Analytics Agent can be stored in an existing Log Analytics workspace associated with your Azure subscription or a new workspace.

In the Azure portal, you can browse to see a list of your Log Analytics workspaces, including any created by Defender for Cloud. A related resource group is created for new workspaces. Resources are created according to the following naming convention:

- Workspace: *DefaultWorkspace-[subscription-ID]-[geo]*

- Resource Group: *DefaultResourceGroup-[geo]*

For workspaces created by Defender for Cloud, data is retained for 30 days. For existing workspaces, retention is based on the workspace pricing tier. If you want, you can also use an existing workspace.

If your Log Analytics Agent reports to a workspace other than the **default** workspace, any [Defender for Cloud plans](defender-for-cloud-introduction.md#protect-cloud-workloads) that you have enabled on the subscription should also be enabled on the workspace.

> [!NOTE]
> Microsoft makes strong commitments to protect the privacy and security of your data. Microsoft adheres to strict compliance and security guidelines—from coding to operating a service. For more information about data handling and privacy, see [Defender for Cloud Data Security](data-security.md).

### Connecting your non-Azure resources to Defender for Cloud

Defender for Cloud can monitor the security posture of your non-Azure computers, but you first need to connect and onboard these resources. To learn how to onboard non-Azure resources, see 
[Onboard non-Azure computers](quickstart-onboard-machines.md). 

## Managing the operation of Defender for Cloud
After the initial configuration and implementation of Defender for Cloud, the next step is to plan your ongoing use of it. The key areas to consider when planning to use Defender for Cloud are:
-	Ongoing monitoring of the security state of your resources.
-	Monitoring new or changed resources.
-	Hardening access and applications.
-	Responding to security incidents. 

### Ongoing monitoring of the security state of your resources

The Defender for Cloud Overview page provides a unified view of the security state of all your Azure resources and non-Azure resources you have connected. The following example shows an environment with many security issues that need to be addressed:

:::image type="content" source="./media/overview-page/overview.png" alt-text="Screenshot of Defender for Cloud's overview page." lightbox="./media/overview-page/overview.png":::

> [!NOTE]
> Defender for Cloud does not interfere with your normal operational procedures. Defender for Cloud passively monitors your resources and provides recommendations based on the security policies you enabled.

When you first start to use Defender for Cloud for your current Azure environment, make sure that you review all security issues and remediation recommendations. The existing recommendations appear on the **Recommendation** page. 

As part of your daily security operations, it is recommended to visit the **Threat Intelligence** option. On this page you can find security threats against your environment, for example, if a particular connected computer is part of a botnet.


### Monitoring new or changed resources

Most Azure environments are dynamic, with resources regularly being created, spun up or down, reconfigured, and changed. Defender for Cloud helps ensure that you have visibility into the security state of these new and changed resources.

When you add new resources (VMs, SQL DBs) to your Azure environment, Defender for Cloud automatically discovers these resources and begins to monitor their security, including PaaS web roles and worker roles. If Data Collection is enabled in the [Security Policy](tutorial-security-policy.md), more monitoring capabilities are enabled automatically for your virtual machines. After you add a new resource, it is recommended to verify that the new resource is indeed monitored by Defender for Cloud.

You should also regularly monitor existing resources for configuration changes that could have created security risks, drift from recommended baselines, and security alerts.

### Hardening access and applications

As part of your security operations, you should also adopt preventative measures to restrict access to VMs, and control the applications that are running on VMs. By locking down inbound traffic to your Azure VMs, you are reducing the exposure to attacks, and at the same time providing easy access to connect to VMs when needed. 

 To harden access to your VMs, use the [just-in-time VM access](just-in-time-access-usage.yml) feature.

### Responding to security incidents

Defender for Cloud detects and alerts you to threats as they occur. Organizations should monitor new security alerts and take action as needed to investigate further or remediate the attack. For more information on how the Threat Protection feature works, see [How Defender for Cloud detects and responds to threats](alerts-overview.md#detect-threats).

Although we cannot create your Incident Response plan, we will use Microsoft Azure Security Response in the Cloud lifecycle as the foundation for the incident response stages. The stages of incident response in the cloud lifecycle are:

:::image type="content" source="./media/defender-for-cloud-planning-and-operations-guide/defender-for-cloud-planning-and-operations-guide-fig5-1.png" alt-text="Stages of the incident response in the cloud lifecycle.":::

> [!NOTE]
> You can use the National Institute of Standards and Technology (NIST) [Computer Security Incident Handling Guide](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf) as a reference that can assist you in building your own Incident Response plan.

You can use Defender for Cloud alerts during the following stages:

- **Detect**: identify a suspicious activity in one or more resources.

- **Assess**: perform the initial assessment to obtain more information about the suspicious activity.

- **Diagnose**: use the remediation steps to perform the technical procedure needed to address the issue.

Each Security Alert provides information that can be used to better understand the nature of the attack and the ways to remediate it. Some alerts also provide links to either more information or other sources of information within Azure. You can use the information provided for further research and remediation, and you can also search for and use security-related data that is stored in your workspace.

The following example shows a suspicious RDP activity:

:::image type="content" source="./media/defender-for-cloud-planning-and-operations-guide/defender-for-cloud-planning-and-operations-guide-fig5-ga.png" alt-text="Screenshot of a suspicious activity report while it is taking place.":::

This Incident page shows the following details:
-	The time of the attack  
-	The source hostname 
-	The target VM 
-	Recommendation steps 

In some circumstances, the source information of the attack might be empty. For more information about this type of behavior, see [Missing Source Information in Defender for Cloud alerts](/archive/blogs/azuresecurity/missing-source-information-in-azure-security-center-alerts).

Once you identify the compromised system, you can run a [workflow automation](workflow-automation.yml) that was previously created. Workflow automations are a collection of procedures that can be executed from Defender for Cloud once triggered by an alert.

> [!NOTE]
> For more information on how to use Defender for Cloud capabilities to assist you during your Incident Response process, see [Managing and responding to security alerts in Defender for Cloud](managing-and-responding-alerts.yml).

## Next steps

In this article, you learned how to plan the implementation and operation of Defender for Cloud. To learn more about Defender for Cloud, see:

- [Defender for Cloud common questions](faq-general.yml) - Find frequently asked questions about using Defender for Cloud.
- [Managing and responding to security alerts in Defender for Cloud](managing-and-responding-alerts.yml).
- [Monitoring partner solutions with Defender for Cloud](./partner-integration.md) - Learn how to monitor the health status of your partner solutions.
- [Azure Security blog](/archive/blogs/azuresecurity/) - Read blog posts about Azure security and compliance.
