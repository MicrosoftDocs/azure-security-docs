---
title: Planning and Operations
description: This document helps you plan before adopting Defender for Cloud and manage daily operations.
ms.topic: concept-article
ms.date: 05/16/2024
#customer intent: As a reader, I want to understand the planning and operations considerations for adopting Defender for Cloud and how it fits into my organization's security requirements and cloud management model. I also want to learn about the security roles, access controls, security policies, data collection, and storage in Defender for Cloud.
---

Microsoft Defender for Cloud monitors your cloud environment to detect vulnerabilities, enforce security policies, and provide recommendations. This article provides information for IT professionals, security architects, and cloud administrators to plan, deploy, and operate Defender for Cloud.

## Security roles and access control

Defender for Cloud uses [Azure role-based access control (RBAC)](/azure/role-based-access-control/overview) to manage permissions. Azure RBAC assigns built-in roles at the subscription or resource group level that determine what resources users can access and the actions they can perform. 

Defender for Cloud also includes two additional roles to manage access within the service:

- **Security Reader**: Can view Defender for Cloud configurations, including recommendations, alerts, policies, and health status, but cannot make changes

- **Security Admin**: Has all Security Reader permissions and can also update security policies and dismiss alerts or recommendations

The diagram below illustrates example key security personas and the roles they play in planning and operating Microsoft Defender for Cloud:

:::image type="content" source="./media/defender-for-cloud-planning-and-operations-guide/defender-for-cloud-planning-and-operations-guide-fig01-new.png" alt-text="Conceptual image that shows various people and the roles that they fill in an organization."::: 

This table shows each persona’s tasks and recommended RBAC roles.

| Persona | Responsibilities | Azure RBAC Role Assignments |
|---|---|---|
| **Jeff (Workload Owner)** | - Manage workload resources<br>- Implement and maintain protections per company policy | - Subscription Owner<br>- Subscription Contributor |
| **Ellen (CISO/CIO)** | - Oversee all security aspects<br>- Understand security posture<br>- Be informed of major attacks and risks | - Subscription Owner<br>- Contributor<br>- Security Admin |
| **David (IT Security)** | - Set security policies<br>- Monitor compliance<br>- Generate reports for leadership or auditors | - Subscription Owner<br>- Contributor<br>- Security Admin |
| **Judy (Security Operations)** | - Monitor and respond to security alerts<br>- Escalate as needed | - Subscription Reader<br>- Security Reader<br>- Subscription Owner<br>- Subscription Contributor<br>- Security Admin *(can dismiss alerts)* |
| **Sam (Security Analyst)** | - Investigate attacks<br>- Collaborate on remediation | - Subscription Reader *(can view alerts)*<br>- Subscription Owner<br>- Subscription Contributor *(can dismiss alerts)*<br>- May need workspace access |

> [!NOTE] 
> - Only subscription and resource group Owners and Contributors can apply security recommendations for a resource
> - Only subscription and resource group Owners and Contributors can apply security recommendations for a resource
> - It is recommended to assign the minimum permissions needed for users

## Security policies

Security policies define which security controls Defender for Cloud monitors and enforce across your resources. They help you maintain compliance with company standards and regulatory requirements.

Defender for Cloud includes a built-in default policy for each subscription, but every organization has unique risks and workloads. You should review, customize, and extend this policy to match your business and compliance needs. You can also create and assign multiple custom security policies tailored to different scopes.

A security policy typically covers:

- **Controls and standards:** The rules Defender for Cloud checks your resources against
- **Scopes:** Where the policy applies — individual subscriptions or management groups that represent teams, regions, or business units
- **Notifications:** Who gets security alerts and compliance updates
- **Pricing tiers:** Which Defender for Cloud features and protections are enabled for each subscription or workspace

Use [Azure Policy](tutorial-security-policy.md) to edit the default controls, add new ones, and assign policies at scale to keep your environment consistent and compliant.

> [!NOTE]
> Before adjusting your security policies, review your [security recommendations](review-security-recommendations.md) to confirm your settings address current risks and compliance gaps.

## Data collection and storage

Defender for Cloud uses the Log Analytics agent and the Azure Monitor Agent to collect security data from your virtual machines. [Data collected](monitoring-components.md) by these agents is stored in your Log Analytics workspaces.

### Agent

When you enable automatic provisioning in your security policy, Defender for Cloud installs the [data collection agent](monitoring-components.md) on all supported Azure VMs and any new supported VMs that you create. If a VM already has the Log Analytics agent, Defender for Cloud uses it. The agent’s process minimizes impact on performance and operates non-invasively.

If you disable Data Collection in your security policy later, Defender for Cloud won’t uninstall the agent automatically because other Azure services may still use it. You can manually uninstall it if needed.

### Workspace

A workspace is an Azure resource that serves as a container for collected data. Organizations often use multiple workspaces to manage different sets of data across their infrastructure.

Data from the Log Analytics agent can go to an existing Log Analytics workspace or a new one.

In the Azure portal, you can view all Log Analytics workspaces, including those created by Defender for Cloud. Related resource groups follow this naming convention:

- Workspace: *DefaultWorkspace-[subscription-ID]-[geo]*

- Resource Group: *DefaultResourceGroup-[geo]*

Workspaces created by Defender for Cloud retain data for 30 days by default. Retention for existing workspaces depends on the workspace pricing tier.

If your agent reports to a non-default workspace, any [Defender plans](defender-for-cloud-introduction.md#protect-cloud-workloads) enabled on the subscription should also be enabled on that workspace.

> [!NOTE]
> Microsoft makes strong commitments to protect the privacy and security of this data. To learn more, see [Defender for Cloud Data Security](data-security.md).

### Onboarding non-Azure resources

Defender for Cloud can monitor the security posture of your non-Azure machines once you onboard them. See [Onboard non-Azure computers](quickstart-onboard-machines.md) for guidance.

## Ongoing security monitoring

After you configure Defender for Cloud and apply its recommendations, it integrates with your operational processes and continuously monitors your environment based on your security policies.

The Defender for Cloud [overview page](overview-page.md) offers a centralized view of security across your Azure and connected non-Azure resources. 
<!--
Since this is a concept-focused article, mentioning specific portal screens feels out of scope.
-->

Here’s an example of what you’ll see:

:::image type="content" source="./media/overview-page/overview.png" alt-text="Screenshot of Defender for Cloud's overview page." lightbox="./media/overview-page/overview.png":::

> [!NOTE]
> Consider enabling threat intelligence features to detect and respond to active threats, such as botnet activity or other suspicious behaviors.
<!--
Not sure this note is too specific for this page. As an outsider, it seems like it is.
If kept, is there a link we can add so users can learn more?
-->

### Monitor for new or changed resources

Cloud environments constantly change. Defender for Cloud automatically discovers new resources and monitors them, including VMs, SQL databases, and PaaS workloads. If Data Collection is enabled in your [Security Policy](tutorial-security-policy.md), extra monitoring capabilities activate for your VMs.

In addition, Defender for Cloud helps you track configuration drift, baseline mismatches, and new security alerts across your existing resources to maintain a strong security posture.

### Restrict access and control applications

As part of your daily security operations, implement measures to limit access to your virtual machines and control which applications run on them. Reducing unnecessary inbound network traffic helps minimize exposure to attacks. Use the [just-in-time VM access](just-in-time-access-usage.yml) feature to grant secure, time-limited access only when needed.

## Incident response

Incident response is the process your organization uses to quickly detect, assess, and remediate security threats. Defender for Cloud supports your incident response efforts by providing [real-time alerts](alerts-overview.md) and tools to identify and investigate suspicious activity.

Each security alert includes context, impact, and suggested mitigations. Some alerts link to additional information or Azure resources. Your security team should actively monitor these alerts and respond promptly to minimize risk. 

Defender for Cloud alerts support these key incident response stages:

- **Detect**: Identify suspicious activity in your resources

- **Assess**: Gather more information about the activity

- **Diagnose**: Take recommended steps to fix the issue

Though Microsoft doesn’t create your incident response plan, Defender for Cloud aligns with the Azure Security Response in the Cloud lifecycle:

:::image type="content" source="./media/defender-for-cloud-planning-and-operations-guide/defender-for-cloud-planning-and-operations-guide-fig5-1.png" alt-text="Stages of incident response in the cloud lifecycle.":::

> [!NOTE]
> For guidance on building your own incident response plan, see the NIST [Computer Security Incident Handling Guide](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf).

### Example: Suspicious RDP activity

:::image type="content" source="./media/defender-for-cloud-planning-and-operations-guide/defender-for-cloud-planning-and-operations-guide-fig5-ga.png" alt-text="Screenshot of a suspicious activity report while it is taking place.":::

This report shows the time of the attack, source hostname, target VM, and mitigation steps. In some cases, source information may be missing. For more information, see [Missing Source Information in Defender for Cloud alerts](/archive/blogs/azuresecurity/missing-source-information-in-azure-security-center-alerts).

If you confirm a compromise, you can run a [workflow automation](workflow-automation.yml) that you set up beforehand to respond automatically.

## Related Content

- Learn more about [managing security alerts and daily operations](managing-and-responding-alerts.yml)
- Find answers to [common questions](faq-general.yml)