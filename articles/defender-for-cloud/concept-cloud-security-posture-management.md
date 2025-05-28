---
title: Cloud Security Posture Management (CSPM)
description: Learn more about Cloud Security Posture Management (CSPM) in Microsoft Defender for Cloud and how it helps improve your security posture.
ms.topic: concept-article
ms.date: 04/21/2025
#customer intent: As a reader, I want to understand the concept of Cloud Security Posture Management (CSPM) in Microsoft Defender for Cloud.
---

# Cloud security posture management (CSPM)

One of the main features of Microsoft Defender for Cloud is cloud security posture management (CSPM). CSPM provides detailed visibility into the security state of your assets and workloads and offers hardening guidance to help you improve your security posture.

Defender for Cloud continually assesses your resources against security standards defined for your Azure subscriptions, Amazon Web Service (AWS) accounts, and Google Cloud Platform (GCP) projects. Defender for Cloud issues security recommendations based on these assessments.

By default, when you enable Defender for Cloud on an Azure subscription, the [Microsoft Cloud Security Benchmark (MCSB)](concept-regulatory-compliance.md) compliance standard is enabled and provides recommendations to secure your multicloud environments. Defender for Cloud provides an aggregated [secure score](secure-score-security-controls.md) based on some of the MCSB recommendations. A higher score indicates a lower identified risk level.

## CSPM plans

Defender for Cloud offers two CSPM plan options:

- **Foundational CSPM** - A free plan enabled by default for subscriptions and accounts that onboard to Defender for Cloud.

- **Defender CSPM** - A paid plan that provides extra capabilities beyond the foundational CSPM plan. This version of the plan offers more advanced security posture features such as AI security posture, attack path analysis, risk prioritization, and more.

## Plan availability

Learn more about [Defender CSPM pricing](https://azure.microsoft.com/pricing/details/defender-for-cloud/).

The following table summarizes each plan and their cloud availability.

| Feature | Foundational CSPM | Defender CSPM | Cloud availability |
|--|--|--|--|
| [Security recommendations](review-security-recommendations.md) | :::image type="icon" source="./media/icons/yes-icon.png"::: | :::image type="icon" source="./media/icons/yes-icon.png":::| Azure, AWS, GCP, on-premises , Docker Hub, JFrog Artifactory|
| [Asset inventory](asset-inventory.md) | :::image type="icon" source="./media/icons/yes-icon.png"::: | :::image type="icon" source="./media/icons/yes-icon.png"::: | Azure, AWS, GCP, on-premises, Docker Hub, JFrog Artifactory |
| [Secure score](secure-score-security-controls.md) | :::image type="icon" source="./media/icons/yes-icon.png"::: | :::image type="icon" source="./media/icons/yes-icon.png"::: | Azure, AWS, GCP, on-premises , Docker Hub, JFrog Artifactory |
| Data visualization and reporting with Azure Workbooks | :::image type="icon" source="./media/icons/yes-icon.png"::: | :::image type="icon" source="./media/icons/yes-icon.png"::: | Azure, AWS, GCP, on-premises |
| [Data exporting](export-to-siem.md) | :::image type="icon" source="./media/icons/yes-icon.png"::: | :::image type="icon" source="./media/icons/yes-icon.png"::: | Azure, AWS, GCP, on-premises |
| [Workflow automation](workflow-automation.yml) | :::image type="icon" source="./media/icons/yes-icon.png"::: | :::image type="icon" source="./media/icons/yes-icon.png"::: | Azure, AWS, GCP, on-premises |
| Tools for remediation | :::image type="icon" source="./media/icons/yes-icon.png"::: | :::image type="icon" source="./media/icons/yes-icon.png"::: | Azure, AWS, GCP, on-premises, Docker Hub, JFrog Artifactory |
| [Microsoft Cloud Security Benchmark](concept-regulatory-compliance.md) | :::image type="icon" source="./media/icons/yes-icon.png"::: | :::image type="icon" source="./media/icons/yes-icon.png"::: | Azure, AWS, GCP |
| [AI security posture management](ai-security-posture.md) | - | :::image type="icon" source="./media/icons/yes-icon.png"::: | Azure, AWS |
| [Agentless VM vulnerability scanning](enable-agentless-scanning-vms.md) | - | :::image type="icon" source="./media/icons/yes-icon.png"::: | Azure, AWS, GCP |
| [Agentless VM secrets scanning](secrets-scanning-servers.md) | - | :::image type="icon" source="./media/icons/yes-icon.png"::: | Azure, AWS, GCP |
| [Attack path analysis](how-to-manage-attack-path.md) | - | :::image type="icon" source="./media/icons/yes-icon.png"::: | Azure, AWS, GCP, Docker Hub, JFrog Artifactory |
| [Risk prioritization](risk-prioritization.md) | - | :::image type="icon" source="./media/icons/yes-icon.png"::: | Azure, AWS, GCP , Docker Hub, JFrog Artifactory |
| [Risk hunting with security explorer](how-to-manage-cloud-security-explorer.md) | - | :::image type="icon" source="./media/icons/yes-icon.png"::: | Azure, AWS, GCP , Docker Hub, JFrog Artifactory |
| [Code-to-cloud mapping for containers](container-image-mapping.md) | - | :::image type="icon" source="./media/icons/yes-icon.png"::: | GitHub, Azure DevOps<sup>[2](#footnote1)</sup> , Docker Hub, JFrog Artifactory|
| [Code-to-cloud mapping for IaC](iac-template-mapping.md) | - | :::image type="icon" source="./media/icons/yes-icon.png"::: | Azure DevOps<sup>[2](#footnote1)</sup>, , Docker Hub, JFrog Artifactory |
| [PR annotations](review-pull-request-annotations.md) | - | :::image type="icon" source="./media/icons/yes-icon.png"::: | GitHub, Azure DevOps<sup>[2](#footnote1)</sup> |
| [Internet exposure analysis](internet-exposure-analysis.md) | - | :::image type="icon" source="./media/icons/yes-icon.png"::: | Azure, AWS, GCP, Docker Hub, JFrog Artifactory |
| [External attack surface management](concept-easm.md) | - | :::image type="icon" source="./media/icons/yes-icon.png"::: | Azure, AWS, GCP, Docker Hub, JFrog Artifactory |
| [Permissions Management (CIEM)](permissions-management.md) | - | :::image type="icon" source="./media/icons/yes-icon.png"::: | Azure, AWS, GCP, Docker Hub, JFrog Artifactory |
| [Regulatory compliance assessments](concept-regulatory-compliance-standards.md) | - | :::image type="icon" source="./media/icons/yes-icon.png"::: | Azure, AWS, GCP, , Docker Hub, JFrog Artifactory |
| [ServiceNow Integration](integration-servicenow.md) | - | :::image type="icon" source="./media/icons/yes-icon.png"::: | Azure, AWS, GCP |
| [Critical assets protection](critical-assets-protection.md) | - | :::image type="icon" source="./media/icons/yes-icon.png"::: | Azure, AWS, GCP |
| [Governance to drive remediation at-scale](governance-rules.md) | - | :::image type="icon" source="./media/icons/yes-icon.png"::: | Azure, AWS, GCP , Docker Hub, JFrog Artifactory|
| [Data security posture management (DSPM), Sensitive data scanning](concept-data-security-posture.md) | - | :::image type="icon" source="./media/icons/yes-icon.png"::: | Azure, AWS, GCP<sup>[1](#footnote1)</sup> |
| [Agentless discovery for Kubernetes](concept-agentless-containers.md) | - | :::image type="icon" source="./media/icons/yes-icon.png"::: | Azure, AWS, GCP |
|[Custom Recommendations](create-custom-recommendations.md)|-|:::image type="icon" source="./media/icons/yes-icon.png":::|Azure, AWS, GCP, Docker Hub, JFrog Artifactory|
| [Agentless code-to-cloud containers vulnerability assessment](agentless-vulnerability-assessment-azure.md) | - | :::image type="icon" source="./media/icons/yes-icon.png"::: | Azure, AWS, GCP, Docker Hub, JFrog Artifactory |
| [API security posture management (Preview)](api-security-posture-overview.md) | - | :::image type="icon" source="./media/icons/yes-icon.png"::: | Azure |
| [Azure Kubernetes Service security dashboard (Preview)](cluster-security-dashboard.md) | - | :::image type="icon" source="./media/icons/yes-icon.png"::: | Azure |


<sup><a name="footnote1"></a>1</sup>: GCP sensitive data discovery [only supports Cloud Storage](concept-data-security-posture-prepare.md#whats-supported). 
<sup><a name="footnote1"></a>2</sup>: DevOps security capabilities, such as code-to-cloud contextualization powering security explorer, attack paths, and pull request annotations for Infrastructure-as-Code security findings, are only available when you enable the paid Defender CSPM plan. Learn more about DevOps security [support and prerequisites](devops-support.md).

## Integrations

Microsoft Defender for Cloud now has built-in integrations to help you use partner systems to seamlessly manage and track tickets, events, and customer interactions. You can push recommendations to a partner ticketing tool and assign responsibility to a team for remediation.

Integration streamlines your incident response process and improves your ability to manage security incidents. You can track, prioritize, and resolve security incidents more effectively.

You can choose which ticketing system to integrate. For preview, only ServiceNow integration is supported. For more information about configuring ServiceNow integration, see [Integrate ServiceNow with Microsoft Defender for Cloud (preview)](integration-servicenow.md).

## Plan pricing

- Review the [Defender for Cloud pricing page](https://azure.microsoft.com/pricing/details/defender-for-cloud/) to learn about Defender CSPM pricing. You can also [estimate costs with the Defender for Cloud cost calculator](cost-calculator.md).

- DevOps security posture capabilities such as Pull request annotations, code to cloud mapping, attack path analysis, and cloud security explorer are only available through the paid Defender CSPM plan. The free foundational security posture management plan provides Azure DevOps recommendations. Learn more about the features provided by [Azure DevOps security features](devops-support.md#azure-devops).

- For subscriptions that use both Defender CSPM and Defender for Containers plans, free vulnerability assessment is calculated based on free image scans provided via the Defender for Containers plan, as summarized [in the Microsoft Defender for Cloud pricing page](https://azure.microsoft.com/pricing/details/defender-for-cloud/). You can also [estimate costs with the Defender for Cloud cost calculator](cost-calculator.md).

- Defender CSPM protects all multicloud workloads, but billing is applied only on specific resources. The following tables list the billable resources when Defender CSPM is enabled on Azure subscriptions, AWS accounts, or GCP projects.

    | Azure Service | Resource types | Exclusions |
    |---|---|---|
    | Compute | Microsoft.Compute/virtualMachines<br/>Microsoft.Compute/virtualMachineScaleSets/virtualMachines<br/>Microsoft.ClassicCompute/virtualMachines | - Deallocated VMs<br/>- Databricks VMs |
    | Storage | Microsoft.Storage/storageAccounts | Storage accounts without blob containers or file shares |
  | DBs | Microsoft.Sql/servers<br/>Microsoft.DBforPostgreSQL/flexibleServers*<br/>Microsoft.DBforMySQL/flexibleServers*<br/>Microsoft.DBforPostgreSQL/servers<br/>Microsoft.DBforMySQL/servers<br/>Microsoft.Sql/managedInstances<br/>Microsoft.DBforMariaDB/servers<br/>Microsoft.Synapse/workspaces | --- |
    *Starting June 1st, 2025<br/>


    | AWS Service | Resource types | Exclusions |
    |---|---|---|
    | Compute | EC2 instances |  Deallocated VMs |
    | Storage | S3 Buckets | --- |
    | DBs | RDS instances | --- |

    | GCP Service | Resource types | Exclusions |
    |---|---|---|
    | Compute |  1. Google Compute instances<br/> 2. Google Instance Group | Instances with nonrunning states |
    | Storage | Storage buckets | - Buckets from classes: 'nearline', 'coldline', 'archive'<br/>- Buckets from regions other than: europe-west1, us-east1, us-west1, us-central1, us-east4, asia-south1, northamerica-northeast1 |
    | DBs | Cloud SQL Instances | --- |

## Azure cloud support

For commercial and national cloud coverage, review the [features supported in Azure cloud environments](support-matrix-cloud-environment.md).

## Next steps

- Watch [Predict future security incidents! Cloud Security Posture Management with Microsoft Defender](https://www.youtube.com/watch?v=jF3NSR_OepI).
- Learn about [security standards and recommendations](security-policy-concept.md).
- Learn about [secure score](secure-score-security-controls.md).
