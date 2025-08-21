---
title: Internet exposure analysis
description: Learn how Defender for Cloud detects, assesses, and mitigates internet exposure for your multicloud resources to enhance security.
author: dcurwin
ms.author: dacurwin
ms.service: defender-for-cloud
ms.topic: concept-article
ms.date: 01/14/2025
ms.custom: template-concept
#customer intent: As a user, I want to understand how Defender for Cloud detects and assesses internet exposure for my multicloud resources. This knowledge will help me identify and mitigate potential security risks effectively.
---

# Internet exposure analysis

Internet Exposure Analysis is a key capability that helps organizations identify which cloud resources are exposed to the public internet, both intentionally or unintentionally, and prioritize remediation based on the risk and scope of that exposure.

Defender for Cloud uses internet exposure to determine the risk level of your misconfigurations enabling high quality posture insights across attack paths, risk-based posture assessments, and signal prioritization across Defender for Cloud posture. 

## How Defender for Cloud detects internet exposure

Defender for Cloud determines if a resource is exposed to the internet by analyzing both:
1.	Control-plane configuration (e.g., public IPs, load balancers)
2.	Network-path reachability (analyzing routing, security and firewall rules) 

Detecting internet exposure can be as simple as checking if a virtual machine (VM) has a public Internet Protocol (IP) address. However, the process can be more complex. Defender for Cloud attempts to locate internet-exposed resources in complex multi-cloud architectures. For example, a VM might not be directly exposed to the internet but could be behind a load balancer, which distributes network traffic across multiple servers to ensure no single server becomes overwhelmed.

The following table lists the resources that Defender for Cloud assesses for internet exposure: 

| Category | Services/Resources |
|--|--|
| Virtual machines | Azure VM <br> Amazon Web Service (AWS) EC2 <br> Google Cloud Platform (GCP) compute instance |
| Virtual machine clusters | Azure Virtual Machine Scale Set <br> GCP instance groups |
| Databases (DB) | Azure SQL <br> Azure PostgreSQL <br> Azure MySQL <br> Azure SQL Managed Instance <br> Azure MariaDB <br> Azure Cosmos DB <br> Azure Synapse <br> AWS Relational Database Service (RDS) DB <br> GCP SQL admin instance |
| Storage | Azure Storage <br> AWS S3 buckets <br> GCP storage buckets |
| AI | Azure OpenAI Service <br> Azure AI Services <br> Azure Cognitive Search |
| Containers | Azure Kubernetes Service (AKS) <br> AWS EKS <br> GCP GKE |
| API | Azure API Management Operations |

The following table lists the network components that Defender for Cloud assesses for internet exposure:

| Category | Services/Resources |
|----------|--------------------|
| Azure    | Application gateway <br> Load Balancer <br> Azure Firewall <br> Network Security Groups <br> vNet/Subnets |
| AWS      | Elastic load balancer |
| GCP      | Load balancer |

## Internet Exposure Width

> [!NOTE]
> currently supports multicloud compute instances only, including Azure VMs/VMSS, AWS EC2, and GCP VMs and exposure width insights and risk factors are applied only to cloud compute resources. Other Defender for Cloud resource types (e.g., Databases, Storage, Containers) will be supported in future phases.

Internet Exposure Width represents the risks based on how broadly a resource (e.g. virtual machine) is exposed to the public internet. It plays a critical role in helping security teams understand not just whether a resource is internet-exposed, but how wide or narrow that exposure is, and how that affects the severity and prioritization of security insights presented in Attack paths and security recommendations.

### How It Works

Defender for cloud automatically analyzes your internet-facing resources and categorizes them based on their exposure level
- **Wide exposure:** Resources accessible from any IP address on the internet
- **Narrow exposure:** Resources accessible only from specific IP ranges, ports, or protocols
- Attack paths that involve widely exposed resources now clearly indicate this in the title, such as "Widely internet exposed VM has high permissions to storage account"
- The exposure width calculated is then used to determine the Attack path generation and Risk based recommendation that helps you to rightly priortize the severity of the findings by adding specific labels to the following experiences
- A new "Exposure width" insight is available on the Cloud Security Explorer, allowing users to query all supported resources that are widely exposed

## How to view internet exposed resources

Defender for Cloud offers a few different ways to view internet-exposed resources.

- [Cloud Security Explorer](how-to-manage-cloud-security-explorer.md) - The Cloud Security Explorer lets you run graph-based queries on the Cloud Security Graph. On the Cloud Security Explorer page, you can run a query to find resources exposed to the internet. This query returns all your attached resources exposed to the internet and lets you view any associated details.

- [Attack Path Analysis](how-to-manage-attack-path.md) -  The Attack Path Analysis page lets you view attack paths that an attacker could take to reach a specific resource. With Attack Path Analysis, you can view a visual representation of the attack path and see which resources are exposed to the internet. Internet exposure often serves as an entry point for attack paths, especially when the resource has vulnerabilities. Internet-exposed resources often lead to targets with sensitive data.

- [Recommendations](review-security-recommendations.md) - Defender for Cloud prioritizes recommendations based on their exposure to the internet including the width of the exposure.

## Defender External Attack Surface Management integration

Defender for Cloud also integrates with Defender External Attack Surface Management to assess resources for internet exposure by attempting to contact them from an external source and seeing if they respond.

Learn more about the [Defender External Attack Surface Management integration](concept-easm.md).

## Related content

- [Investigating risk with security explorer/attack paths](concept-attack-path.md)
- [Identify and remediate attack paths](how-to-manage-attack-path.md)
- [Remediate recommendations](implement-security-recommendations.md)
