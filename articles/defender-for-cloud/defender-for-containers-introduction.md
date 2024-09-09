---
title: Overview of Container security in Microsoft Defender for Containers
description: Learn about Microsoft Defender for Containers
ms.topic: overview
author: dcurwin
ms.author: dacurwin
ms.date: 01/09/2024
---

# Overview of container protection in Defender for Cloud

Microsoft Defender for Containers is a cloud-native solution to improve, monitor, and maintain the security of your containerized assets (Kubernetes clusters, Kubernetes nodes, Kubernetes workloads, container registries, container images and more), and their applications, across multicloud and on-premises environments.

Defender for Containers assists you with four core domains of container security:

- [**Security posture management**](#security-posture-management) - performs continuous monitoring of cloud APIs,  Kubernetes APIs, and Kubernetes workloads to discover cloud resources, provide comprehensive inventory capabilities, detect misconfigurations and provide guidelines to  mitigate them,  provide contextual risk assessment, and empowers users to perform enhanced risk hunting capabilities through the Defender for Cloud security explorer.

- [**Vulnerability assessment**](#vulnerability-assessment) -  provides agentless vulnerability assessment for Azure, AWS, and GCP with remediation guidelines, zero configuration, daily rescans, coverage for OS and language packages, and exploitability insights.

- [**Run-time threat protection**](#run-time-protection-for-kubernetes-nodes-and-clusters) - a rich threat detection suite for Kubernetes clusters, nodes, and workloads, powered by Microsoft leading threat intelligence, provides mapping to MITRE ATT&CK framework for easy understanding of risk and relevant context, automated response, and [SIEM/XDR integration](kubeaudit-events-advanced-hunting.md).

- **Deployment & monitoring**- Monitors your Kubernetes clusters for missing sensors and provides frictionless at-scale deployment for sensor-based capabilities, support for standard Kubernetes monitoring tools, and management of unmonitored resources.

You can learn more by watching this video from the Defender for Cloud in the Field video series: [Microsoft Defender for Containers](episode-three.md).

## Microsoft Defender for Containers plan availability

| Aspect | Details |
|--|--|
| Release state: | General availability (GA)<br> Certain features are in preview. For a full list, see the [Containers support matrix in Defender for Cloud](support-matrix-defender-for-containers.md)|
| Feature availability | Refer to the [Containers support matrix in Defender for Cloud](support-matrix-defender-for-containers.md) for additional information on feature release state and availability.|
| Pricing: | **Microsoft Defender for Containers** is billed as shown on the [pricing page](https://azure.microsoft.com/pricing/details/defender-for-cloud/) |
| Required roles and permissions: | • To deploy the required components, see the [permissions for each of the components](monitoring-components.md#defender-for-containers-extensions)<br> • **Security admin** can dismiss alerts<br> • **Security reader** can view vulnerability assessment findings<br> See also [Roles for remediation](permissions.md#roles-used-to-automatically-provision-agents-and-extensions) and [Azure Container Registry roles and permissions](/azure/container-registry/container-registry-roles) |
| Clouds: | View the [Containers support matrix in Defender for Cloud](support-matrix-defender-for-containers.md) to see cloud availability. |

## Security posture management

### Agentless capabilities

- **Agentless discovery for Kubernetes** - provides zero footprint, API-based discovery of your Kubernetes clusters, their configurations and deployments.

- **[Agentless vulnerability assessment](agentless-vulnerability-assessment-azure.md)** - provides vulnerability assessment for all container images in supported registries, including recommendations for the registry itself, quick scans of new images, daily refresh of results, exploitability insights, and more. Running containers are also scanned for vulnerability assessment, agnostic to the container's image registry. Vulnerability information is added to the security graph for contextual risk assessment and calculation of attack paths, and hunting capabilities.

- **Comprehensive inventory capabilities**  - enables you to explore resources, pods, services, repositories, images and configurations through [security explorer](how-to-manage-cloud-security-explorer.md#build-a-query-with-the-cloud-security-explorer) to easily monitor and manage your assets.

- **[Enhanced risk-hunting](how-to-manage-cloud-security-explorer.md)** - enables security admins to actively hunt for posture issues in their containerized assets through queries (built-in and custom) and [security insights](attack-path-reference.md#insights) in the [security explorer](how-to-manage-cloud-security-explorer.md)
- **Control plane hardening** - continuously assesses the configurations of your clusters and compares them with the initiatives applied to your subscriptions. When it finds misconfigurations, Defender for Cloud generates security recommendations that are available on Defender for Cloud's Recommendations page. The recommendations let you investigate and remediate issues.

  You can use the resource filter to review the outstanding recommendations for your container-related resources, whether in asset inventory or the recommendations page:

  :::image type="content" source="media/defender-for-containers/resource-filter.png" alt-text="Screenshot showing you where the resource filter is located." lightbox="media/defender-for-containers/resource-filter.png":::

  For details included with this capability, review [container recommendations](recommendations-reference-container.md ), and look for recommendations with type "Control plane"

### Sensor-based capabilities

**Binary drift detection** - Defender for Containers provides a sensor-based capability that alerts you about potential security threats by detecting unauthorized external processes within containers. You can define drift policies to specify conditions under which alerts should be generated, helping you distinguish between legitimate activities and potential threats. For more information, see [Binary drift protection (preview)](binary-drift-detection.md).

**Kubernetes data plane hardening** - To protect the workloads of your Kubernetes containers with best practice recommendations, you can install the [Azure Policy for Kubernetes](/azure/governance/policy/concepts/policy-for-kubernetes). Learn more about [monitoring components](monitoring-components.md) for Defender for Cloud.

With the add-on on your Kubernetes cluster, every request to the Kubernetes API server is monitored against the predefined set of best practices before being persisted to the cluster. You can then configure it to enforce the best practices and mandate them for future workloads.

For example, you can mandate that privileged containers shouldn't be created, and any future requests to do so are blocked.

You can learn more about [Kubernetes data plane hardening](kubernetes-workload-protections.md).

## Vulnerability assessment

Defender for Containers scans the container images in Azure Container Registry (ACR), Amazon AWS Elastic Container Registry (ECR), Google Artifact Registry (GAR), and Google Container Registry (GCR) to provide agentless vulnerability assessment for your container images, including registry and runtime recommendations, remediation guidance, quick scans of new images, real-world exploit insights, exploitability insights, and more.

Now for [public preview in the AKS environment](agentless-vulnerability-assessment-azure#how-vulnerability-assessment-for-images-and-containers-works), Defender for Containers also performs a daily scan of all running containers to provide updated vulnerability assessment, agnostic to the container's image registry.

Vulnerability information powered by Microsoft Defender Vulnerability Management is added to the [cloud security graph](concept-attack-path.md#what-is-cloud-security-graph) for contextual risk, calculation of attack paths, and hunting capabilities.

Learn more about [vulnerability assessments for Defender for Containers supported environments with Microsoft Defender Vulnerability Management](agentless-vulnerability-assessment-azure.md)

## Run-time protection for Kubernetes nodes and clusters

Defender for Containers provides real-time threat protection for [supported containerized environments](support-matrix-defender-for-containers.md) and generates alerts for suspicious activities. You can use this information to quickly remediate security issues and improve the security of your containers.

Threat protection is provided for Kubernetes at cluster level, node level, and workload level and includes both sensor based coverage that requires the [Defender sensor](defender-for-cloud-glossary.md#defender-sensor) and agentless coverage that is based on analysis of the Kubernetes audit logs. Security alerts are only triggered for actions and deployments that occur after you enabled Defender for Containers on your subscription.

Examples of security events that Microsoft Defenders for Containers monitors include:

- Exposed Kubernetes dashboards
- Creation of high privileged roles
- Creation of sensitive mounts

You can view security alerts by selecting the Security alerts tile at the top of the Defender for Cloud's overview page, or the link from the sidebar.

  :::image type="content" source="media/managing-and-responding-alerts/overview-page-alerts-links.png" alt-text="Screenshot showing how to get to the security alerts page from Microsoft Defender for Cloud's overview page." lightbox="media/managing-and-responding-alerts/overview-page-alerts-links.png":::

The security alerts page opens:

   :::image type="content" source="media/defender-for-containers/view-containers-alerts.png" alt-text="Screenshot showing you where to view the list of alerts." lightbox="media/defender-for-containers/view-containers-alerts.png":::

Security alerts for runtime workload in the clusters can be recognized by the `K8S.NODE_` prefix of the alert type.  For a full list of the cluster level alerts, see the [reference table of alerts](alerts-containers.md).

Defender for Containers also includes host-level threat detection with over 60 Kubernetes-aware analytics, AI, and anomaly detections based on your runtime workload.

Defender for Cloud monitors the attack surface of multicloud Kubernetes deployments based on the [MITRE ATT&CK® matrix for Containers](https://www.microsoft.com/security/blog/2021/04/29/center-for-threat-informed-defense-teams-up-with-microsoft-partners-to-build-the-attck-for-containers-matrix/), a framework developed by the [Center for Threat-Informed Defense](https://mitre-engenuity.org/cybersecurity/center-for-threat-informed-defense/) in close partnership with Microsoft.

## Learn more

Learn more about Defender for Containers in the following blogs:

- [Introducing Microsoft Defender for Containers](https://techcommunity.microsoft.com/t5/microsoft-defender-for-cloud/introducing-microsoft-defender-for-containers/ba-p/2952317)
- [Demonstrating Microsoft Defender for Cloud](https://techcommunity.microsoft.com/t5/microsoft-defender-for-cloud/how-to-demonstrate-the-new-containers-features-in-microsoft/ba-p/3281172)

## Next steps

In this overview, you learned about the core elements of container security in Microsoft Defender for Cloud. To enable the plan, see:

- [Enable Defender for Containers](defender-for-containers-enable.md)
- Check out [common questions](faq-defender-for-containers.yml) about Defender for Containers.
