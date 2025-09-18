---
title: Protect your Amazon Web Service (AWS) accounts containers with Defender for Containers
description: Learn how to enable the Defender for Containers plan on your Amazon Web Service (AWS) accounts for Microsoft Defender for Cloud.
ms.topic: install-set-up-deploy
ms.date: 05/13/2025
ms.custom: sfi-image-blocked
---

# Protect your Amazon Web Service (AWS) containers with Defender for Containers

Defender for Containers in Microsoft Defender for Cloud is the cloud-native solution that is used to secure your containers so you can improve, monitor, and maintain the security of your clusters, containers, and their applications.

Learn more about [Overview of Microsoft Defender for Containers](defender-for-containers-introduction.md).

You can learn more about Defender for Container's pricing on the [pricing page](https://azure.microsoft.com/pricing/details/defender-for-cloud/). You can also [estimate costs with the Defender for Cloud cost calculator](cost-calculator.md).

## Prerequisites

- You need a Microsoft Azure subscription. If you don't have an Azure subscription, you can [sign up for a free subscription](https://azure.microsoft.com/pricing/free-trial/).

- You must [enable Microsoft Defender for Cloud](get-started.md#enable-defender-for-cloud-on-your-azure-subscription) on your Azure subscription.

- [Connect your AWS account to Microsoft Defender for Cloud](quickstart-onboard-aws.md#connect-your-aws-account)

- Verify your Kubernetes nodes can access source repositories of your package manager. For information about the requirements, see [Network requirements](defender-for-containers-enable.md?tabs=aks-deploy-portal%2Ck8s-deploy-asc%2Ck8s-verify-asc%2Ck8s-remove-arc%2Caks-removeprofile-api&pivots=defender-for-container-eks&preserve-view=true#network-requirements).

- Ensure the following [Azure Arc-enabled Kubernetes network requirements](/azure/azure-arc/kubernetes/quickstart-connect-cluster) are validated.

## Enable the Defender for Containers plan on your AWS account

To protect your EKS clusters, you need to enable the Containers plan on the relevant AWS account connector.

**To enable the Defender for Containers plan on your AWS account**:

1. Sign in to the [Azure portal](https://portal.azure.com).

1. Search for and select **Microsoft Defender for Cloud**.

1. In the Defender for Cloud menu, select **Environment settings**.

1. Select the relevant AWS account.

    :::image type="content" source="media/tutorial-enable-containers-aws/aws-account.png" alt-text="Screenshot of Defender for Cloud's environment settings page showing an AWS connector." lightbox="media/tutorial-enable-containers-aws/aws-account.png":::

1. Set the toggle for the **Containers** plan to **On**.

    :::image type="content" source="media/tutorial-enable-containers-aws/aws-containers-enabled.png" alt-text="Screenshot of enabling Defender for Containers for an AWS connector." lightbox="media/tutorial-enable-containers-aws/aws-containers-enabled.png":::

1. To change optional configurations for the plan, select **Settings**.

    :::image type="content" source="media/defender-for-containers-enable-plan-eks/containers-threat-protection.png" alt-text="Screenshot of the settings for the Containers plan in the Defender for Cloud environment settings with Agentless threat protection highlighted." lightbox="media/defender-for-containers-enable-plan-eks/containers-threat-protection.png":::

    - The [**Agentless threat protection**](defender-for-containers-introduction.md#run-time-protection-for-kubernetes-nodes-and-clusters) feature provides runtime protection to your cluster containers. The feature sends Kubernetes audit logs to Microsoft Defender. Set the **Agentless threat protection** toggle to **On** and set the retention period of your audit logs.

        > [!NOTE]
        > If you disable this configuration, then the `Threat detection (control plane)` feature will be disabled. Learn more about [features availability](supported-machines-endpoint-solutions-clouds-containers.md).

   - **[K8S API access](defender-for-containers-architecture.md#how-does-agentless-discovery-for-kubernetes-in-aws-work)** sets permissions to allow API-based discovery of your Kubernetes clusters. To enable, set the **K8S API access** toggle to **On**.
     > [!NOTE]
     > In case your EKS cluster public endpoint is restricted, the cluster's settings will be automatically updated to include Microsoft Defender for Cloud's CIDR block.
     > Defender for Cloud requires allowing access to the Kubernetes API server from the following IP ranges: 172.212.245.192/28, 48.209.1.192/28.
          
     
    - [**Registry access**](agentless-vulnerability-assessment-aws.md) sets permissions to allow vulnerability assessment of images stored in ECR. To enable, set the **Registry access** toggle to **On**.

2. Select **Next: Review and generate**.

3. Select **Update**.

> [!NOTE]
> To enable or disable individual Defender for Containers capabilities, either globally or for specific resources, see [How to enable Microsoft Defender for Containers components](defender-for-containers-enable.md).

## Deploy the Defender sensor in EKS clusters

[!INCLUDE [Install with Helm note](./includes/helm-install-note.md)]

Azure Arc-enabled Kubernetes, the Defender sensor, and Azure Policy for Kubernetes should be installed and running on your EKS clusters. There's a dedicated Defender for Cloud recommendation that can be used to install these extensions (and Azure Arc if necessary):

- `EKS clusters should have Microsoft Defender's extension for Azure Arc installed`

**To deploy the required extensions**:

1. From Defender for Cloud's **Recommendations** page, search for one of the recommendations by name.

1. Select an unhealthy cluster.

   > [!IMPORTANT]
   > You must select the clusters one at a time.
> > Don't select the clusters by their hyperlinked names: select anywhere else in the relevant row.
   
1. Select **Fix**.

1. Defender for Cloud generates a script in the language of your choice:
    - For Linux, select **Bash**.
    - For Windows, select **PowerShell**.

1. Select **Download remediation logic**.

1. Run the generated script on your cluster.

## Next steps

- For advanced enablement features for Defender for Containers, see the [Enable Microsoft Defender for Containers](defender-for-containers-enable.md) page.

- [Overview of Microsoft Defender for Containers](defender-for-containers-introduction.md).
