---
title: Connect Endor Labs to Defender for Cloud
description: Learn how to connect Endor Labs with Microsoft Defender for Cloud to enhance vulnerability analysis and gain visibility of critical vulnerabilities.
author: dcurwin
ms.author: dacurwin
ms.topic: how-to
ms.date: 11/20/2024
ai-usage: ai-assisted
ms.custom: sfi-image-nochange
#customer intent: As a user, I want to learn how to connect my ServiceNow account with Microsoft Defender for Cloud so that I can enhance the existing vulnerability analysis security capabilities that are provided by Defender for Cloud for comprehensive code to runtime visibility of critical vulnerabilities.
---

# Connect Endor Labs to Defender for Cloud

Microsoft Defender for Cloud's integration with Endor Labs enhances the security of software applications by identifying and mitigating vulnerabilities in partner dependencies. This integration streamlines the discovery and remediation processes, improving overall security.

This article provides a detailed explanation of the benefits and procedures needed to connect Endor Labs to Defender for Cloud. By following the steps outlined, security teams are able to gain enhanced visibility and control over potential threats, from the initial stages of code development all the way through to runtime.

## Prerequisites

- You need a Microsoft Azure subscription. If you don't have an Azure subscription, you can [sign up for a free subscription](https://azure.microsoft.com/pricing/free-trial/).

- You must [enable Microsoft Defender for Cloud](get-started.md#enable-defender-for-cloud-on-your-azure-subscription) on your Azure subscription.

- You must [enable Defender Cloud Security Posture Management (CSPM)](tutorial-enable-cspm-plan.md) on your Azure subscription.

- Connect your DevOps environments to Defender for Cloud:
  - [Azure DevOps organizations](quickstart-onboard-devops.md)
  - [GitHub organizations](quickstart-onboard-github.md)
  - [GitLab groups](quickstart-onboard-devops.md)

- Have an account with [Endor Labs](https://www.endorlabs.com/).

- Have an [API key from Endor Labs](https://docs.endorlabs.com/administration/api-keys/) with read-only permissions. We recommend setting the expiration date to be 180 days.

- You must have the appropriate role to:
  - **Create DevOps connectors**: Security Admin or Contributor assigned at the **subscription level** through Azure role-based-access control (RBAC).
  - **Create the Endor Labs connector**: Security Administrator (or higher) assigned at the **tenant level** through Microsoft Entra. Permissions can be granted through [Privileged Identity Management](/entra/id-governance/privileged-identity-management/pim-configure).
  - **View reachability analysis findings**: Security Admin or Security Reader assigned at the **subscription level** through Azure role-based-access control (RBAC) on the subscription that hosts the DevOps connector.

- You can only have one connector to Endor Labs per tenant.

- Findings from Endor Labs are only shown if the corresponding repository is also connected to Defender for Cloud.

## Connect Endor Labs

To connect your Endor Labs account to Defender for Cloud:

1. Sign in to the [Azure portal](https://portal.azure.com/).

1. Navigate to **Microsoft Defender for Cloud** > **Environment settings**.

1. Select **Integrations**.

   :::image type="content" border="true" source="./media/connect-endor-labs/integrations.png" alt-text="Screenshot of environment settings page that shows where to select the ServiceNow option.":::

1. Select **Add integration** > **Endor Labs**.

   :::image type="content" border="true" source="./media/connect-endor-labs/add-endor-labs.png" alt-text="Screenshot that shows where the Add Integration button is and the ServiceNow option." lightbox="media/connect-endor-labs/add-endor-labs.png":::

    > [!NOTE]
    > The option to add the Endor labs integration isn't available if you don't have the appropriate permissions, or if you already have an existing connector to Endor Labs.

1. Enter an Endor Labs namespace, API key ID, API secret.

    :::image type="content" source="media/connect-endor-labs/enter-information.png" alt-text="Screenshot that shows where the button is located to add your Endor Labs connection.":::

1. Select **Create**.

A notice appears after the integration is successfully created. Defender for Cloud scans repositories that are connected to Endor Labs, and populates with results after six hours.

## Related content

- [FAQ about the Endor Labs integration](faq-endor-labs.yml)
