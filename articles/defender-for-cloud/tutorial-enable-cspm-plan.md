---
title: Protect your resources with Defender CSPM
author: Elazark
ms.author: elkrieger
description: Learn how to enable Defender CSPM on your Azure subscription for Microsoft Defender for Cloud and enhance your security posture.
ms.topic: install-set-up-deploy
ms.date: 09/05/2023
---

# Protect your resources with Defender CSPM

Defender Cloud Security Posture Management (CSPM) in Microsoft Defender for Cloud provides you with hardening guidance that helps you efficiently and effectively improve your security. CSPM also gives you visibility into your current security situation.

Defender for Cloud continually assesses your resources, subscriptions, and organization for security issues. Defender for Cloud shows you your security posture with the secure score. The secure score is an aggregated score of the security findings that tells you your current security situation. The higher the score, the lower the identified risk level.

When you enable Defender for Cloud, you automatically enable the **Foundational CSPM capabilities**. These capabilities are part of the free services offered by Defender for Cloud.

You have the ability to enable the **Defender CSPM** plan, which offers extra protections for your environments such as governance, regulatory compliance, cloud security explorer, attack path analysis and agentless scanning for machines.

> [!NOTE]
> Agentless scanning requires the **Subscription Owner** to enable the Defender CSPM plan. Anyone with a lower level of authorization can enable the Defender CSPM plan, but the agentless scanner won't be enabled by default due a lack of required permissions that are only available to the Subscription Owner. In addition, attack path analysis and security explorer won't populate with vulnerabilities because the agentless scanner is disabled.

For availability and to learn more about the features offered by each plan, see the [Defender CSPM plan options](concept-cloud-security-posture-management.md).

You can learn more about Defender CSPM's pricing on [the pricing page](https://azure.microsoft.com/pricing/details/defender-for-cloud/).

## Prerequisites

- You need a Microsoft Azure subscription. If you don't have an Azure subscription, you can [sign up for a free subscription](https://azure.microsoft.com/pricing/free-trial/).

- You must [enable Microsoft Defender for Cloud](get-started.md#enable-defender-for-cloud-on-your-azure-subscription) on your Azure subscription.

- Connect your [non-Azure machines](quickstart-onboard-machines.md), [AWS accounts](quickstart-onboard-aws.md) or [GCP projects](quickstart-onboard-gcp.md).

- In order to gain access to all of the features available from the CSPM plan, the plan must be enabled by the **Subscription Owner**.

## Enable the Defender CSPM plan

When you enable Defender for Cloud, you automatically receive the protections offered by the Foundational CSPM capabilities. In order to gain access to the other features provided by Defender CSPM, you need to enable the Defender CSPM plan on your subscription.

**To enable the Defender CSPM plan on your subscription**:

1. Sign in to the [Azure portal](https://portal.azure.com).

1. Search for and select **Microsoft Defender for Cloud**.

1. In the Defender for Cloud menu, select **Environment settings**.

1. Select the relevant Azure subscription, AWS account or GCP project.

1. On the Defender plans page, toggle the Defender CSPM plan to **On**.

1. Select **Save**.

## Enable the components of the Defender CSPM plan

Once the Defender CSPM plan is enabled on your subscription, you have the ability to enable the individual components of the Defender CSPM plan:

- **Agentless scanning for machines**: Scans your machines for installed software and vulnerabilities without relying on agents or impacting machine performance. You can disable the agentless scanner or add exclusion tags to your subscription.

- **Agentless discovery for Kubernetes**: API-based discovery of information about Kubernetes cluster architecture, workload objects, and setup. Required for Kubernetes inventory, identity and network exposure detection, risk hunting as part of the cloud security explorer. This extension is required for attack path analysis (Defender CSPM only).

- **Agentless container vulnerability assessments**: Provides vulnerability management for images stored in your container registries.

- **Sensitive data discovery**: Sensitive data discovery automatically discovers managed cloud data resources containing sensitive data at scale. This feature accesses your data, it is agentless, uses smart sampling scanning, and integrates with Microsoft Purview sensitive information types and labels.

- **Permissions management** - Insights into Cloud Infrastructure Entitlement Management (CIEM). CIEM ensures appropriate and secure identities and access rights in cloud environments. It helps understand access permissions to cloud resources and associated risks. Setup and data collection may take up to 24 hours.

**To enable the components of the Defender CSPM plan**:

1. On the Defender plans page, select **Settings**.

    :::image type="content" source="media/tutorial-enable-cspm-plan/cspm-settings.png" alt-text="Screenshot of the Defender plans page that shows where to select the settings option." lightbox="media/tutorial-enable-cspm-plan/cspm-settings.png":::

1. Select **On** for each component to enable it.

1. (Optional) For agentless scanning, select **Edit configuration**.

    :::image type="content" source="media/tutorial-enable-cspm-plan/cspm-configuration.png" alt-text="Screenshot that shows where to select edit configuration." lightbox="media/tutorial-enable-cspm-plan/cspm-configuration.png":::

    1. Enter a tag name and tag value for any machines to be excluded from scans.

    1. Select **Apply**.

1. Select **Continue**.

For code to cloud contextualization capabilities and automated developer remediation workflows that come with your Defender CSPM plan at no additional cost, [connect your DevOps environments](defender-for-devops-introduction.md) to Defender for Cloud.

## Next steps

[Cloud Security Posture Management (CSPM)](concept-cloud-security-posture-management.md)
