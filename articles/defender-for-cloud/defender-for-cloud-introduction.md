---
title: What is Microsoft Defender for Cloud?
description: Use Microsoft Defender for Cloud to protect your Azure, hybrid, and multicloud resources and workloads.
ms.topic: overview
ms.date: 02/23/2025
#customer intent: As a security professional, I want to understand Microsoft Defender for Cloud so that I can protect my cloud resources.
---

# What is Microsoft Defender for Cloud?

Microsoft Defender for Cloud is a cloud-native application protection platform (CNAPP) that includes security measures and practices designed to protect cloud-based applications from various cyber threats and vulnerabilities. Defender for Cloud includes:

- A development security operations (DevSecOps) solution that unifies security management at the code level across multicloud and multi-pipeline environments
- A cloud security posture management (CSPM) solution that identifies actions to prevent breaches
- A cloud workload protection platform (CWPP) with protections for servers, containers, storage, databases, and other workloads

:::image type="content" source="media/defender-for-cloud-introduction/defender-for-cloud-pillars.png" alt-text="Diagram that shows the core functionality of Microsoft Defender for Cloud.":::

> [!NOTE]
> For Defender for Cloud pricing information, see the [pricing page](https://azure.microsoft.com/pricing/details/defender-for-cloud/).

The Microsoft 365 Defender portal helps security teams investigate attacks on cloud resources, devices, and identities. Microsoft 365 Defender provides an overview of attacks, including suspicious and malicious events in cloud environments. Microsoft 365 Defender achieves this by correlating all alerts and incidents, including cloud alerts and incidents.

Learn more about the [integration between Microsoft Defender for Cloud and Microsoft Defender XDR](concept-integration-365.md).

## Secure cloud applications

Defender for Cloud helps you incorporate good security practices early in the software development process, or DevSecOps. You can protect your code management environments and code pipelines, and get insights into your development environment security posture from a single location. Defender for Cloud enables security teams to manage DevOps security across multi-pipeline environments.

Today's applications require security awareness at the code, infrastructure, and runtime levels to ensure that deployed applications are hardened against attacks.

| Capability                                                   | What problem does it solve?                                  | Get started                                                  | Defender plan       |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------- |
| [Code pipeline insights](defender-for-devops-introduction.md) | Empowers security teams with the ability to protect applications and resources from code to cloud across multi-pipeline environments, including GitHub, Azure DevOps, and GitLab. DevOps security findings, such as Infrastructure as Code (IaC) misconfigurations and exposed secrets, can then be correlated with other contextual cloud security insights to prioritize remediation in code. | Connect [Azure DevOps](quickstart-onboard-devops.md), [GitHub](quickstart-onboard-github.md), and [GitLab](quickstart-onboard-gitlab.md) repositories to Defender for Cloud | Foundational CSPM (Free) and Defender CSPM |

## Improve your security posture

The security of your cloud and on-premises resources relies on proper configuration and deployment. Defenders for Cloud recommendations identify steps to secure your environment.

Defender for Cloud includes free Foundational CSPM capabilities. Enable advanced CSPM capabilities with the Defender CSPM plan.

| Capability | What problem does it solve? | Get started | Defender plan |
|--|--|--|--|
| [Centralized policy management](security-policy-concept.md) | Define the security conditions that you want to maintain across your environment. The policy translates to recommendations that identify resource configurations that violate your security policy. The [Microsoft cloud security benchmark](concept-regulatory-compliance.md) is a built-in standard that applies security principles with detailed technical implementation guidance for Azure and other cloud providers (such as Amazon Web Services (AWS) and Google Cloud Platform (GCP). | [Customize a security policy](create-custom-recommendations.md) | Foundational CSPM (Free) |
| [Secure score](secure-score-security-controls.md) | Summarize your security posture based on the security recommendations. As you remediate recommendations, your secure score improves. | [Track your secure score](secure-score-access-and-track.md) | Foundational CSPM (Free) |
| [Multicloud coverage](plan-multicloud-security-get-started.md) | Connect to your multicloud environments with agentless methods for CSPM insight and CWP protection. | Connect your [Amazon AWS](quickstart-onboard-aws.md) and [Google GCP](quickstart-onboard-gcp.md) cloud resources to Defender for Cloud | Foundational CSPM (Free) |
| [Cloud Security Posture Management (CSPM)](concept-cloud-security-posture-management.md) | Use the dashboard to see weaknesses in your security posture. | [Enable CSPM tools](enable-enhanced-security.md) | Foundational CSPM (Free) |
| [Advanced Cloud Security Posture Management](concept-cloud-security-posture-management.md) | Get advanced tools to identify weaknesses in your security posture, including:</br>- Governance to drive actions to improve your security posture</br>- Regulatory compliance to verify compliance with security standards</br>- Cloud security explorer to build a comprehensive view of your environment | [Enable CSPM tools](enable-enhanced-security.md) | Defender CSPM |
| [Data Security Posture Management](concept-data-security-posture.md) | Data security posture management automatically discovers datastores containing sensitive data, and helps reduce risk of data breaches. | [Enable data security posture management](data-security-posture-enable.md) | Defender CSPM or Defender for Storage |
| [Attack path analysis](concept-attack-path.md#what-is-an-attack-path) | Model traffic on your network to identify potential risks before you implement changes to your environment. | [Build queries to analyze paths](how-to-manage-attack-path.md) | Defender CSPM |
| [Cloud Security Explorer](concept-attack-path.md#what-is-cloud-security-explorer) | A map of your cloud environment that lets you build queries to find security risks. | [Build queries to find security risks](how-to-manage-cloud-security-explorer.md) | Defender CSPM |
| [Security governance](governance-rules.md) | Drive security improvements through your organization by assigning tasks to resource owners and tracking progress in aligning your security state with your security policy. | [Define governance rules](governance-rules.md) | Defender CSPM |
| [Microsoft Entra Permissions Management](/azure/active-directory/cloud-infrastructure-entitlement-management/) | Provide comprehensive visibility and control over permissions for any identity and any resource in Azure, AWS, and GCP. | [Review your Permission Creep Index (CPI)](other-threat-protections.md#entra-permission-management-formerly-cloudknox) | Defender CSPM |

## Protect cloud workloads

Proactive security principles require implementing security practices to protect your workloads from threats. Cloud workload protections (CWP) provide workload-specific recommendations to guide you to the right security controls to protect your workloads.

When your environment is threatened, security alerts immediately indicate the nature and severity of the threat so you can plan your response. After identifying a threat in your environment, respond quickly to limit the risk to your resources.

| Capability | What problem does it solve? | Get started | Defender plan |
| ---------- | --------------------------- | ----------- | ------------------------- |
| Protect cloud servers | Provide server protections through Microsoft Defender for Endpoint or extended protection with just-in-time network access, file integrity monitoring, vulnerability assessment, and more. | [Secure your multicloud and on-premises servers](defender-for-servers-introduction.md) | Defender for Servers |
| Identify threats to your storage resources | Detect unusual and potentially harmful attempts to access or exploit your storage accounts using advanced threat detection capabilities and Microsoft Threat Intelligence data to provide contextual security alerts. | [Protect your cloud storage resources](defender-for-storage-introduction.md) | Defender for Storage |
| Protect cloud databases | Protect your entire database estate with attack detection and threat response for the most popular database types in Azure to protect the database engines and data types, according to their attack surface and security risks. | [Deploy specialized protections for cloud and on-premises databases](quickstart-enable-database-protections.md) | - Defender for Azure SQL Databases</br>- Defender for SQL servers on machines</br>- Defender for Open-source relational databases</br>- Defender for Azure Cosmos DB |
| Protect containers | Secure your containers so you can improve, monitor, and maintain the security of your clusters, containers, and their applications with environment hardening, vulnerability assessments, and run-time protection. | [Find security risks in your containers](defender-for-containers-introduction.md) | Defender for Containers |
| [Infrastructure service insights](asset-inventory.md) | Diagnose weaknesses in your application infrastructure that can leave your environment susceptible to attack. | - [Identify attacks targeting applications running over App Service](defender-for-app-service-introduction.md)</br>- [Detect attempts to exploit Key Vault accounts](defender-for-key-vault-introduction.md)</br>- [Get alerted on suspicious Resource Manager operations](defender-for-resource-manager-introduction.md)</br>- [Expose anomalous Domain Name System (DNS) activities](defender-for-dns-introduction.md) | - Defender for App Service</br>- Defender for Key Vault</br>- Defender for Resource Manager</br>- Defender for DNS |
| [Security alerts](alerts-overview.md) | Get informed of real-time events that threaten the security of your environment. Alerts are categorized and assigned severity levels to indicate proper responses. | [Manage security alerts]( managing-and-responding-alerts.yml) | Any workload protection Defender plan |
| [Security incidents](alerts-overview.md#what-are-security-incidents) | Identify attack patterns by correlating alerts and integrate with Security Information and Event Management (SIEM), Security Orchestration, Automation, and Response (SOAR), and IT classic deployment model solutions to respond to threats and reduce risk to your resources. | [Export alerts to SIEM, SOAR, or ITSM systems](export-to-siem.md) | Any workload protection Defender plan |

[!INCLUDE [Defender for DNS note](./includes/defender-for-dns-note.md)]

## Learn More

For more information about Defender for Cloud and how it works, see:

- A [step-by-step walkthrough](https://mslearn.cloudguides.com/guides/Protect%20your%20multi-cloud%20environment%20with%20Microsoft%20Defender%20for%20Cloud) of Defender for Cloud
- An interview about Defender for Cloud with an expert in cybersecurity in [Lessons Learned from the Field](episode-six.md)
- [Microsoft Defender for Cloud - Use cases](https://techcommunity.microsoft.com/t5/microsoft-defender-for-cloud/microsoft-defender-for-cloud-use-cases/ba-p/2953619)
- [Microsoft Defender for Cloud PoC Series - Microsoft Defender for Containers](https://techcommunity.microsoft.com/t5/microsoft-defender-for-cloud/microsoft-defender-for-cloud-poc-series-microsoft-defender-for/ba-p/3064644)
- Learn how [Microsoft Defender for Cloud provides data security](data-security.md)

## Next steps

> [!div class="nextstepaction"]
> [Enable Microsoft Defender plans](enable-enhanced-security.md)
