---
title: Defender for Cloud Planning and Operations Guide
description: This document helps you to plan before adopting Defender for Cloud and considerations regarding daily operations.
ms.topic: concept-article
ms.date: 05/16/2024
#customer intent: As a reader, I want to understand the planning and operations considerations for adopting Defender for Cloud and how it fits into my organization's security requirements and cloud management model. I also want to learn about the security roles, access controls, security policies, data collection, and storage in Defender for Cloud.
---

# Planning and operations guide

This guide is for for information technology (IT) professionals, IT architects, information security analysts, and cloud administrators planning to use Microsoft Defender for Cloud.

## Planning guide

This guide provides the background on how Defender for Cloud fits into your organization's security requirements and cloud management model. It's important to understand how different individuals or teams in your organization use the service to meet secure development, operations, monitoring, governance, and incident response needs.  

These are the key areas to consider when planning to use Microsoft Defender for Cloud:  

- Security Roles and Access Controls
- Security Policies and Recommendations
- Data Collection and Storage
- Onboarding non-Azure resources
- Ongoing Security Monitoring
- Incident Response

In the following sections, you'll learn how to plan for each one of those areas and apply those recommendations based on your requirements.

> [!NOTE]
> Read [Defender for Cloud common questions](faq-general.yml) for a list of common questions that can also be useful during the designing and planning phase.

## Security roles and access controls  

Microsoft Defender for Cloud uses [Azure role-based access control (Azure RBAC)](/azure/role-based-access-control/role-assignments-portal), which provides [built-in roles](/azure/role-based-access-control/built-in-roles) Built-in roles include Owner, Contributor, and Reader. Azure roles can be assigned to users, groups, and services in Azure. Defender for Cloud users can only see information related to resources to which they have access
### Specific Security Roles for Microsoft Defender for Cloud  

In addition to the standard Azure roles, Defender for Cloud includes the following roles:  

- **Security reader**: A user in this role has read-only access to Defender for Cloud. The user can view recommendations, alerts, security policies, and security states but can't make changes.

- **Security admin**: A user in this role has the same access as the Security Reader and can also update security policies and dismiss alerts and recommendations.

### Role Considerations  

- **Security Policy Editing:** Only subscription Owners, Contributors, and Security Admins can modify security policies.

- **Applying Security Recommendations:** Only subscription and resource group Owners and Contributors can implement security recommendations for a resource.

### Planning Access Control
When configuring Azure RBAC for Defender for Cloud, ensure you identify who in your organization requires access and understand the tasks they will perform. This allows you to properly assign roles and permissions.

> [!NOTE]
> We recommend that you assign the least permissive role needed for users to complete their tasks. For instance, assign the **Reader Role** to users who only need to view the resource security state without taking actions, such as applying recommendations or editing policies.

### Example Personas for Microsoft Defender for Cloud RBAC
Depending on the size and structure of your organization, different individuals and teams might use Microsoft Defender for Cloud to perform different security-related tasks. Below are examples of how specific roles align with typical tasks and the corresponding Azure Role-Based Access Control (RBAC) permissions. The following diagram displays the fictitious personas and their respective roles and security responsibilities:  

:::image type="content" source="./media/defender-for-cloud-planning-and-operations-guide/defender-for-cloud-planning-and-operations-guide-fig01-new.png" alt-text="Conceptual image that shows various people and the roles that they fill in an organization.":::


#### Jeff (Workload Owner)  
 
**Tasks and Requirements**
  - Manages a cloud workload and its related resource
  - Responsible for implementing and maintaining protections in accordance with company security policy (In small organizations would also define policy and monitor alerts).
   
**RBAC Role:** Resource Group Owner or Contributor

#### Ellen (CISO/CIO)

**Tasks and Requirements**  
- Responsible for all aspects of security for the company.
- Wants to understand the company's security posture across cloud workloads.
- Needs to be informed of major attacks and risks.
  
**RBAC Role:** Subscription Owner, Contributor, or Security Admin

#### David (IT Security)  

**Tasks and Requirements:**  
 
- Sets company security policies to ensure the appropriate protections are in place.
- Monitors compliance with policies.
- Generates reports for leadership or auditors.

**RBAC Role:** Subscription Owner, Contributor, or Security Admin.

#### Judy (Security Operations)  

**Tasks and Requirements:**

- Monitors and responds to security alerts at any time.
- Escalates to Cloud Workload Owner or IT Security Analyst (Sometimes performed by a Managed Security Provider).

**RBAC Roles**  
- Subscription Reader or Security Reader (required to view alerts).
- Subscription Owner, Contributor, or Security Admin (required to dismiss alerts).

#### Sam (Security Analyst) 

**Tasks and Requirements**  

- Investigates attacks.
- Works with Cloud Workload Owner to apply remediation.

**RBAC Roles (Access to the workspace might be required)**   
  
- Subscription Reader (required to view alerts)
- Subscription Owner or Contributor (required to dismiss alerts).

  
## Security policies and recommendations

A security policy defines the desired configuration of your workloads and helps ensure compliance with company or regulatory security requirements. In Microsoft Defender for Cloud, you can define policies for your Azure subscriptions, which can be tailored to the type of workload or the sensitivity of data.

Microsoft Defender for Cloud policies contain the following components:

- [Data collection](monitoring-components.md): Agent provisioning and data collection settings.
- [Security policy](tutorial-security-policy.md): An [Azure Policy](/azure/governance/policy/overview) that determines which controls are monitored and recommended by Microsoft Defender for Cloud. You can also use Azure Policies to create new definitions, define more policies, and assign policies across management groups.
- [Email notifications](configure-email-notifications.md): Configure email notifications for alerts and attack paths.
- [Pricing tier](defender-for-cloud-introduction.md#protect-cloud-workloads): Determines which Microsoft Defender for Cloud features are available for your resources. You can choose either the free tier or enable paid Defender plans for enhanced protection. Pricing can be configured at the subscription or workspace level using the API.

> [!NOTE]
> Specifying a security contact ensures that Azure can reach the right person in your organization if a security incident occurs. Read [Provide security contact details in Defender for Cloud](configure-email-notifications.md) for more information to learn how to enable security contacts.

## Security policies definitions and recommendations

Defender for Cloud automatically creates a default security policy for each of your Azure subscriptions. You can edit the policy in Defender for Cloud or use Azure Policy to create new definitions, define more policies, and assign policies across management groups. Management groups can represent the entire organization or a business unit within the organization. You can monitor policy compliance across these management groups.

Before configuring security policies, review each of the [security recommendations](review-security-recommendations.md):

- Determine if the policies suit your subscriptions and resource groups.
- Understand the actions needed to address recommendations.
- Assign specific individuals responsible for monitoring and remediating new security recommendations within your organization.

## Data collection and storage

Defender for Cloud uses Log Analytics and Azure Monitor agents to collect security data from your virtual machines. [Data collected](monitoring-components.md) from these agents is stored in your Log Analytics workspaces.

### Agents

When automatic provisioning is enabled in the security policy, the [data collection agent](monitoring-components.md) is automatically installed on all supported Azure VMs - both existing and newly created. If the VM or computer already has the Log Analytics agent installed, Defender for Cloud will use the current installed agent. The agent's process is designed to be non-invasive and have minimal effect on VM performance.

If at some point you want to disable Data Collection, you can turn it off in the security policy. However, note that disabling data collection does not automatically remove the Log Analytics agent, as other Azure services may still be using it. If necessary, you can manually uninstall the agent yourself.

### Workspaces

A workspace is an Azure resource that serves as a data container. You or other members of your organization might use multiple workspaces to manage different sets of data collected from all or portions of your IT infrastructure.

Data collected from the Log Analytics agent can be stored in an existing Log Analytics workspace associated with your Azure subscription or a new workspace.

In the Azure portal, you can browse to see a list of your Log Analytics workspaces, including any created by Microsoft Defender for Cloud. A related resource group is created for new workspaces. Resources are created according to this naming convention:

- Workspace: *DefaultWorkspace-[subscription-ID]-[geo]*

- Resource Group: *DefaultResourceGroup-[geo]*

For workspaces created by Defender for Cloud, data is retained for 30 days. For existing workspaces, retention is based on the workspace pricing tier. If you want, you can also use an existing workspace.

If your agent reports to a workspace other than the **default** workspace, any Defender for Cloud [Defender plans](defender-for-cloud-introduction.md#protect-cloud-workloads) that you've enabled on the subscription should also be enabled on the workspace.

> [!NOTE]
> Microsoft makes strong commitments to protect the privacy and security of this data. Microsoft adheres to strict compliance and security guidelines—from coding to operating a service. For more information about data handling and privacy, read [Defender for Cloud Data Security](data-security.md).

## Onboarding non-Azure resources

Defender for Cloud can monitor the security posture of your non-Azure computers but you need to first onboard these resources. Read [Onboard non-Azure computers](quickstart-onboard-machines.md) for more information on how to onboard non-Azure resources.

## Ongoing security monitoring

After the initial configuration and application of Defender for Cloud recommendations, the next step is considering Defender for Cloud operational processes.

The Defender for Cloud Overview provides a unified view of security across all your Azure resources and any non-Azure resources you've connected. This example shows an environment with many issues to resolve:

:::image type="content" source="./media/overview-page/overview.png" alt-text="Screenshot of Defender for Cloud's overview page." lightbox="./media/overview-page/overview.png":::

> [!NOTE]
> Defender for Cloud doesn't interfere with your normal operational procedures. Defender for Cloud passively monitors your deployments and provides recommendations based on the security policies you enabled.

When you first opt in to use Defender for Cloud for your current Azure environment, make sure that you review all recommendations, which can be done in the **Recommendations** page.

Plan to visit the threat intelligence option as part of your daily security operations. There you can identify security threats against the environment, such as identify if a particular computer is part of a botnet.

### Monitoring for new or changed resources

Most Azure environments are dynamic, with resources regularly being created, spun up or down, reconfigured, and changed. Defender for Cloud helps ensure that you have visibility into the security state of these new resources.

When adding new resources (VMs, SQL DBs) to your Azure environment, Defender for Cloud automatically discovers these resources and begins to monitor their security, including PaaS web roles and worker roles. If Data Collection is enabled in the [Security Policy](tutorial-security-policy.md), more monitoring capabilities are enabled automatically for your virtual machines.

In addition, you should regularly monitor existing resources for configuration changes that could have created security risks, drift from recommended baselines, and security alerts.

### Hardening access and applications

As part of your security operations, you should also adopt preventative measures to restrict access to VMs, and control the applications that are running on VMs. By locking down inbound traffic to your Azure VMs, you're reducing the exposure to attacks, and at the same time providing easy access to connect to VMs when needed. Use the [just-in-time VM access](just-in-time-access-usage.yml) access feature to hardening access to your VMs.

## Incident response

Defender for Cloud detects and alerts you to threats as they occur. Organizations should monitor for new security alerts and take action as needed to investigate further or remediate the attack. For more information on how Defender for Cloud threat protection works, read [How Defender for Cloud detects and responds to threats](alerts-overview.md#detect-threats).

Although we can't create your Incident Response plan, we'll use Microsoft Azure Security Response in the Cloud lifecycle as the foundation for incident response stages. The stages of incident response in the cloud lifecycle are:

:::image type="content" source="./media/defender-for-cloud-planning-and-operations-guide/defender-for-cloud-planning-and-operations-guide-fig5-1.png" alt-text="Stages of the incident response in the cloud lifecycle.":::

> [!NOTE]
> You can use the National Institute of Standards and Technology (NIST) [Computer Security Incident Handling Guide](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf) as a reference to assist you building your own.

You can use Defender for Cloud alerts during the following stages:

- **Detect**: Identify a suspicious activity in one or more resources.
- **Assess**: Perform an initial assessment to obtain more information about the suspicious activity.
- **Diagnose**: Follow remediation steps to conduct the technical procedure related to the issue.

Each Security Alert provides information that can be used to better understand the nature of the attack and suggest possible mitigations. Some alerts also provide links to either more information or to other sources of information within Azure. You can use the information provided for further research and to begin mitigation, and you can also search security-related data that is stored in your workspace.

The following example shows a suspicious RDP activity taking place:

:::image type="content" source="./media/defender-for-cloud-planning-and-operations-guide/defender-for-cloud-planning-and-operations-guide-fig5-ga.png" alt-text="Screenshot of a suspicious activity report while it is taking place.":::

The page above displays the details regarding the time that the attack took place, the source hostname, the target VM, and recommendation steps. In some circumstances, the source information of the attack might be empty. Read [Missing Source Information in Defender for Cloud alerts](/archive/blogs/azuresecurity/missing-source-information-in-azure-security-center-alerts) for more information about this type of behavior.

Once you identify the compromised system, you can run a [workflow automation](workflow-automation.yml) that was previously created. Workflow automations are a collection of procedures that can be executed from Defender for Cloud once triggered by an alert.

> [!NOTE]
> Read [Managing and responding to security alerts in Defender for Cloud](managing-and-responding-alerts.yml) for more information on how to use Defender for Cloud capabilities to assist you during your Incident Response process.

## Next steps

In this document, you learned how to plan for Defender for Cloud adoption. Learn more about Defender for Cloud:

- [Managing and responding to security alerts in Defender for Cloud](managing-and-responding-alerts.yml)
- [Monitoring partner solutions with Defender for Cloud](./partner-integration.md) - Learn how to monitor the health status of your partner solutions.
- [Defender for Cloud common questions](faq-general.yml) - Find frequently asked questions about using the service.
- [Azure Security blog](/archive/blogs/azuresecurity/) - Read blog posts about Azure security and compliance.
