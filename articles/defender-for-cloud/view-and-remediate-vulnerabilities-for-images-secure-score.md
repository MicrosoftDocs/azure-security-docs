---
title: Assess vulnerabilities for images running on your Kubernetes clusters (Secure Score)
description: Learn how to view and remediate vulnerabilities for images running on your Kubernetes clusters (Secure Score).
ms.service: defender-for-cloud
ms.custom: build-2023
ms.topic: how-to
ms.date: 07/15/2025
---

# View and remediate vulnerabilities for images running on Kubernetes clusters (secure score)

> [!NOTE]
> This page describes the classic secure score approach to vulnerability management in Defender for Cloud. Customers using Defender CSPM should use the new risk-based approach: [View and remediate vulnerabilities for images running on your Kubernetes clusters (Risk based)](view-and-remediate-vulnerabilities-for-images.md).

Defender for Cloud gives you the ability to prioritize the remediation of vulnerabilities in images that are currently being used within your environment using the [Running container images should have vulnerability findings resolved](https://portal.azure.com/#view/Microsoft_Azure_Security_CloudNativeCompute/KubernetesRuntimeVisibilityRecommendationDetailsBlade/assessmentKey/41503391-efa5-47ee-9282-4eff6131462ce) recommendation.

To provide findings for the recommendation, Defender for Cloud uses [agentless discovery for Kubernetes](defender-for-containers-introduction.md) or the [Defender sensor](tutorial-enable-containers-azure.md#deploy-the-defender-sensor-in-azure) to create a full inventory of your Kubernetes clusters and their workloads and correlates that inventory with the vulnerability reports created for your registry images. The recommendation shows your running containers with the vulnerabilities associated with the images that are used by each container and remediation steps.

Defender for Cloud presents the findings and related information as recommendations, including related information such as remediation steps and relevant CVEs. You can view the identified vulnerabilities for one or more subscriptions, or for a specific resource.

Within each recommendation, resources are grouped into tabs:  

- **Healthy resources** – relevant resources, which either aren't impacted or on which you've already remediated the issue.  
- **Unhealthy resources** – resources that are still impacted by the identified issue.  
- **Not applicable resources** – resources for which the recommendation can't give a definitive answer. The not applicable tab also includes reasons for each resource.

## View vulnerabilities on a specific cluster

**To view vulnerabilities for a specific cluster, do the following:**  

1. Open the **Recommendations** page. If you are on the new risk-based page, select **Switch to classic view** in the menu item on the top of the page. Use the **>** arrow to open the sub-levels.  If issues were found, you'll see the recommendation [Running container images should have vulnerability findings resolved (powered by Microsoft Defender Vulnerability Management)](https://portal.azure.com/#blade/Microsoft_Azure_Security/RecommendationsBlade/assessmentKey/c609cf0f-71ab-41e9-a3c6-9a1f7fe1b8d5). Select the recommendation.

    :::image type="content" source="media/view-and-remediate-vulnerabilities-for-images-secure-score/running-image-recommendation-line.png" alt-text="Screenshot showing the recommendation line for running container images should have vulnerability findings resolved." lightbox="media/view-and-remediate-vulnerabilities-for-images-secure-score/running-image-recommendation-line.png":::

1. The recommendation details page opens showing the list of Kubernetes clusters ("affected resources") and categorizes them as healthy, unhealthy, and not applicable, based on the images used by your workloads. Select the relevant cluster for which you want to remediate vulnerabilities.

    :::image type="content" source="media/view-and-remediate-vulnerabilities-for-images-secure-score/running-select-cluster.png" alt-text="Screenshot showing the affected clusters for the recommendation." lightbox="media/view-and-remediate-vulnerabilities-for-images-secure-score/running-select-cluster.png":::

1. The cluster details page opens. It lists all currently running containers categorized into three tabs based on the vulnerability assessments of the images used by those containers. Select the specific container you want to explore.

    :::image type="content" source="media/view-and-remediate-vulnerabilities-for-images-secure-score/running-select-container.png" alt-text="Screenshot showing where to select a specific container." lightbox="media/view-and-remediate-vulnerabilities-for-images-secure-score/running-select-container.png":::

1. This pane includes a list of the container vulnerabilities. Select each vulnerability to [resolve the vulnerability](#remediate-vulnerabilities).

    :::image type="content" source="media/view-and-remediate-vulnerabilities-for-images-secure-score/running-list-vulnerabilities.png" alt-text="Screenshot showing the list of container vulnerabilities." lightbox="media/view-and-remediate-vulnerabilities-for-images-secure-score/running-list-vulnerabilities.png":::

## View container images affected by a specific vulnerability

**To view findings for a specific vulnerability, do the following:**

1. Open the **Recommendations** page, using the **>** arrow to open the sub-levels. If issues were found, you'll see the recommendation [Running container images should have vulnerability findings resolved (powered by Microsoft Defender Vulnerability Management)](https://portal.azure.com/#blade/Microsoft_Azure_Security/RecommendationsBlade/assessmentKey/c609cf0f-71ab-41e9-a3c6-9a1f7fe1b8d5). Select the recommendation.

    :::image type="content" source="media/view-and-remediate-vulnerabilities-for-images-secure-score/running-image-recommendation-line.png" alt-text="Screenshot showing the recommendation line for running container images should have vulnerability findings resolved." lightbox="media/view-and-remediate-vulnerabilities-for-images-secure-score/running-image-recommendation-line.png":::

1. The recommendation details page opens with additional information. This information includes the list of vulnerabilities impacting the clusters. Select the specific vulnerability.

    :::image type="content" source="media/view-and-remediate-vulnerabilities-for-images-secure-score/running-select-vulnerability.png" alt-text="Screenshot showing the list of vulnerabilities impacting the container clusters." lightbox="media/view-and-remediate-vulnerabilities-for-images-secure-score/running-select-vulnerability.png":::

1. The vulnerability details pane opens. This pane includes a detailed description of the vulnerability, images affected by that vulnerability, and links to external resources to help mitigate the threats, affected resources, and information on the software version that contributes to [resolving the vulnerability](#remediate-vulnerabilities).

    :::image type="content" source="media/view-and-remediate-vulnerabilities-for-images-secure-score/running-containers-affected.png" alt-text="Screenshot showing the list of container images impacted by the vulnerability." lightbox="media/view-and-remediate-vulnerabilities-for-images-secure-score/running-containers-affected.png":::

## Remediate vulnerabilities

Use these steps to remediate each of the affected images found either in a specific cluster or for a specific vulnerability:

1. Follow the steps in the remediation section of the recommendation pane.
1. When you've completed the steps required to remediate the security issue, replace each affected image in your cluster, or replace each affected image for a specific vulnerability:
    1. Build a new image (including updates for each of the packages) that resolves the vulnerability according to the remediation details.
    1. Push the updated image and delete the old image. It might take up to 24 hours for the previous image to be removed from the results, and for the new image to be included in the results.
    1. Use the new image across all vulnerable workloads.
1. Check the recommendations page for the recommendation [Running container images should have vulnerability findings resolved](https://portal.azure.com/#view/Microsoft_Azure_Security_CloudNativeCompute/KubernetesRuntimeVisibilityRecommendationDetailsBlade/assessmentKey/41503391-efa5-47ee-9282-4eff6131462c).
1. If the recommendation still appears and the image you've handled still appears in the list of vulnerable images, check the remediation steps again.

## Next steps

- Learn how to [view and remediate vulnerabilities for registry images](view-and-remediate-vulnerability-assessment-findings.md).
- Learn more about the Defender for Cloud [Defender plans](defender-for-cloud-introduction.md#cloud-workload-protection-platform-cwpp)
