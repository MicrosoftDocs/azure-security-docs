---
title: Configure Microsoft Defender for Containers components
description: Configure the container protections of Microsoft Defender for Containers
ms.topic: how-to
author: dcurwin
ms.author: dacurwin
ms.custom: devx-track-azurecli, sfi-image-nochange
zone_pivot_groups: k8s-host
ms.date: 03/13/2025
---

# Configure Microsoft Defender for Containers components

[Microsoft Defender for Containers](defender-for-containers-introduction.md) is the cloud-native solution for securing your containers. It helps protect your clusters whether they're running in:

- **Azure Kubernetes Service (AKS)**: Microsoft's managed service for developing, deploying, and managing containerized applications.

- **Amazon Elastic Kubernetes Service (EKS) in a connected Amazon Web Services (AWS) account**: Amazon's managed service for running Kubernetes on AWS without needing to install, operate, and maintain your own Kubernetes control plane or nodes.

- **Google Kubernetes Engine (GKE) in a connected Google Cloud Platform (GCP) project**: Google's managed environment for deploying, managing, and scaling applications by using GCP infrastructure.

- **Other Kubernetes distributions** (using Azure Arc-enabled Kubernetes): Cloud Native Computing Foundation (CNCF) certified Kubernetes clusters hosted on-premises or on infrastructure as a service (IaaS). For more information, see [Containers support matrix in Defender for Cloud](support-matrix-defender-for-containers.md).

You can first learn how to connect and help protect your containers in these articles:

- [Protect your Azure containers with Defender for Containers](tutorial-enable-containers-azure.md)
- [Protect your on-premises Kubernetes clusters with Defender for Containers](tutorial-enable-containers-arc.md)
- [Protect your Amazon Web Services (AWS) containers with Defender for Containers](tutorial-enable-container-aws.md)
- [Protect your Google Cloud Platform (GCP) containers with Defender for Containers](tutorial-enable-container-gcp.md)

You can also learn more by watching these videos from the *Defender for Cloud in the field* video series:

- [Microsoft Defender for Containers in a multicloud environment](episode-nine.md)
- [Protecting containers in GCP with Defender for Containers](episode-ten.md)

::: zone pivot="defender-for-container-arc"
> [!NOTE]
> Defender for Containers support for Azure Arc-enabled Kubernetes clusters is a preview feature. The preview feature is available on a self-service, opt-in basis.
>
> Previews are provided *as is* and *as available*. They're excluded from the service-level agreements and the limited warranty.
>
> To learn more about the supported operating systems, feature availability, outbound proxy, and more, see [Containers support matrix in Defender for Cloud](support-matrix-defender-for-containers.md).
::: zone-end

::: zone pivot="defender-for-container-aks"
[!INCLUDE [Prerequisites](./includes/defender-for-container-prerequisites-aks.md)]
::: zone-end

::: zone pivot="defender-for-container-arc,defender-for-container-eks,defender-for-container-gke"
[!INCLUDE [Prerequisites](./includes/defender-for-container-prerequisites-arc-eks-gke.md)]
::: zone-end

::: zone pivot="defender-for-container-aks"
[!INCLUDE [Enable plan for AKS](./includes/defender-for-containers-enable-plan-aks.md)]
::: zone-end

::: zone pivot="defender-for-container-arc"
[!INCLUDE [Enable plan for Arc](./includes/defender-for-containers-enable-plan-arc.md)]
::: zone-end

::: zone pivot="defender-for-container-eks"
[!INCLUDE [Enable plan for EKS](./includes/defender-for-containers-enable-plan-eks.md)]
::: zone-end

::: zone pivot="defender-for-container-gke"
[!INCLUDE [Enable plan for GKE](./includes/defender-for-containers-enable-plan-gke.md)]
::: zone-end


::: zone pivot="defender-for-container-arc,defender-for-container-eks,defender-for-container-gke"
[!INCLUDE [Remove the sensor](./includes/defender-for-containers-remove-extension.md)]
::: zone-end

::: zone pivot="defender-for-container-aks"
[!INCLUDE [Assign a custom workspace](./includes/defender-for-containers-assign-workspace-aks.md)]
::: zone-end

::: zone pivot="defender-for-container-arc"
[!INCLUDE [Assign a custom workspace](./includes/defender-for-containers-assign-workspace-arc.md)]
::: zone-end

::: zone pivot="defender-for-container-aks"
[!INCLUDE [Remove the sensor](./includes/defender-for-containers-remove-profile.md)]
::: zone-end

## Next steps

Now that you enabled Defender for Containers, you can:

- [Scan your supported environment images for vulnerabilities with Microsoft Defender Vulnerability Management](agentless-vulnerability-assessment-azure.md)
- Check out [common questions](faq-defender-for-containers.yml) about Defender for Containers.
