---
title: Remediate code with Microsoft Security Copilot
author: Elazark
ms.author: elkrieger
description: Learn how to remediate code with Copilot in Microsoft Defender for Cloud and improve your security posture.
ms.topic: how-to
ms.date: 09/25/2025
#customer intent: As a security professional, I want to understand how to use Copilot to remediate code in Defender for Cloud so that I can improve my security posture.
---

# Remediate code with Microsoft Security Copilot

Microsoft Defender for Cloud's integration with Microsoft Security Copilot lets you remediate Infrastructure as Code (IaC) misconfigurations in your code repositories. Remediating IaC findings with Copilot lets you address security misconfigurations and vulnerabilities early in the development cycle by automatically generating pull requests (PRs) that correct the identified weaknesses. This remediation ensures that security issues in code are addressed accurately and promptly.

## Prerequisites

- [Enable Defender for Cloud on your environment](connect-azure-subscription.md).
- [Connect your Azure DevOps environment to Defender for Cloud](quickstart-onboard-devops.md).
- [Configure the Microsoft Security DevOps Azure DevOps extension](azure-devops-extension.yml).
- [Review and ensure you meet the DevOps security support and prerequisites requirements](devops-support.md).
- [Have access to Azure Copilot](/azure/copilot/overview).
- [Have Security Compute Units assigned for Microsoft Security Copilot](/copilot/security/get-started-security-copilot).

## Remediate an Infrastructure as Code scanning finding

With Microsoft Security Copilot in Defender for Cloud, you can remediate recommendations.

1. Sign in to the [Azure portal](https://portal.azure.com).

1. Search for and select **Microsoft Defender for Cloud**.

1. Go to **Recommendations**.

1. Search for and select the **Azure DevOps repositories should have infrastructure as code scanning findings resolved** recommendation.

    :::image type="content" source="media/remediate-code-with-copilot/search-recommendation.png" alt-text="Screenshot that shows the recommendation that you searched for." lightbox="media/remediate-code-with-copilot/search-recommendation.png":::

1. Select **Reduce risk with Copilot**.

    :::image type="content" source="media/remediate-code-with-copilot/copilot-summarize.png" alt-text="Screenshot that shows where the Summarize with copilot button is located." lightbox="media/remediate-code-with-copilot/copilot-summarize.png":::

1. Select **Help me remediate this recommendation**.

1. Select **security check**.

1. Select the appropriate description.
 
1. Select **Select**.

    :::image type="content" source="media/remediate-code-with-copilot/select-select.png" alt-text="Screenshot that shows where the select button is located." lightbox="media/remediate-code-with-copilot/select-select.png":::

1. Review the summary of the code fix.

1. Select **Submit**.

1. Select the provided link.

1. Review the pull request.

Once the pull request is generated in your code repository, a developer should review and approve the pull request for merging into the code base.

## Next step

> [!div class="nextstepaction"]
> [Scan for misconfigurations in Infrastructure as Code](iac-vulnerabilities.md#view-details-and-remediation-information-for-applied-iac-rules)
