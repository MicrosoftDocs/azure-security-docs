---
title: Microsoft Security Copilot in Defender for Cloud
description: Learn about the benefits of copilot in Microsoft Defender for Cloud and how it applies to analyzing your security posture.
ms.date: 09/25/2025
author: ElazarK
ms.author: elkrieger
ms.topic: concept-article
ai-usage: ai-assisted
# customer intent: As a security professional, I want to understand the benefits of Copilot in Microsoft Defender for Cloud and how it can help me analyze my security posture.
---

# Microsoft Security Copilot in Defender for Cloud

Microsoft Defender for Cloud integrates Microsoft Security Copilot and Microsoft Copilot for Azure into its experience. These integrations let you ask security-related questions, receive responses, and automatically trigger the necessary skills to analyze, summarize, remediate, and delegate recommendations by using natural language prompts.

Both [Security Copilot](/copilot/security/microsoft-security-copilot) and [Copilot for Azure](/azure/copilot/overview?wt.mc_id=copilot_1a_webpage_gdc) are cloud-based AI platforms that provide a natural language copilot experience. These platforms help security professionals understand the context and effect of recommendations, remediate or delegate tasks, and address misconfigurations in code.

Defender for Cloud's integration with Security Copilot and Copilot for Azure on the recommendations page lets you enhance your security posture and mitigate risks in your environments. It streamlines the process of understanding and implementing recommendations, making your security management more efficient and effective.

## Learn more about Security Copilot

If you're new to Security Copilot, you should familiarize yourself with it by reading these articles:
- [What is Microsoft Security Copilot?](/security-copilot/microsoft-security-copilot)
- [Microsoft Security Copilot experiences](/security-copilot/experiences-security-copilot)
- [Get started with Microsoft Security Copilot](/security-copilot/get-started-security-copilot)
- [Understand authentication in Microsoft Security Copilot](/security-copilot/authentication)
- [Prompting in Microsoft Security Copilot](/security-copilot/prompting-security-copilot)

## Security Copilot integration in Defender for Cloud

Defender for Cloud integrates Copilot directly into the Defender for Cloud experience. With this integration, you can analyze, summarize, remediate, and delegate your recommendations by using natural language prompts.

:::image type="content" source="media/copilot-security-in-defender-for-cloud/analyze-copilot.png" alt-text="Screenshot showing the location of the 'Analyze with Copilot' button on the recommendations page." lightbox="media/copilot-security-in-defender-for-cloud/analyze-copilot.png":::

When you open Copilot, you can use natural language prompts to ask questions about the recommendations. Copilot provides responses in natural language, helping you understand the context of the recommendation. It explains the effect of implementing the recommendation and provides steps for implementation.

Some sample prompts include:

- Show critical risks for publicly exposed resources
- Show critical risks to sensitive data
- Show resources with high severity vulnerabilities

Copilot can assist with refining recommendations, providing summaries, remediation steps, and delegation. It enhances your ability to analyze and act on recommendations.

:::image type="content" source="media/copilot-security-in-defender-for-cloud/summarize-copilot.png" alt-text="Screenshot showing the location of the 'Summarize with Copilot' button on a recommendation." lightbox="media/copilot-security-in-defender-for-cloud/summarize-copilot.png":::

## Key features

The following section provides information about the available features in Security Copilot in Defender for Cloud.

### Data processing workflow

When you use Security Copilot in Defender for Cloud, the following data processing workflow takes place:

1. A user enters a prompt in the Copilot interface.
1. Copilot for Azure receives the prompt.
1. Copilot for Azure evaluates the prompt and the active page to determine the skills needed to resolve the prompt.
1. If the prompt is security related and the skill is available, Security Copilot executes the skills and sends back a response to Copilot in Azure for presentation.
1. If a security-related prompt is received but the skill is unavailable, Azure Copilot searches all of its available skills to find the most relevant skills to resolve the prompt. It then sends a response to the user.

    :::image type="content" source="media/copilot-security-in-defender-for-cloud/data-process-workflow.png" alt-text="Diagram showing the data processing workflow of the Copilot experience in Defender for Cloud." lightbox="media/copilot-security-in-defender-for-cloud/data-process-workflow.png":::

Check out the [Security Copilot FAQs](faq-copilot.yml).

## Enable the Security Copilot integration in Defender for Cloud

Security Copilot in Defender for Cloud doesn't rely on any of the available plans in Defender for Cloud. Security Copilot is available for all users when you:

1. [Enable Defender for Cloud on your environment](connect-azure-subscription.md).
1. [Have access to Azure Copilot](/azure/copilot/overview).
1. [Have Security Compute Units assigned for Security Copilot](/copilot/security/get-started-security-copilot).

To enjoy the full range of Security Copilot's capabilities in Defender for Cloud, enable the [Defender for Cloud Security Posture Management (DCSPM) plan](concept-cloud-security-posture-management.md#plan-availability) on your environments. The DCSPM plan includes many extra security features such as [Attack path analysis](how-to-manage-attack-path.md), [Risk prioritization](risk-prioritization.md), and more, all of which you can navigate and manage by using Security Copilot. Without the DCSPM plan, you can still use Security Copilot in Defender for Cloud, but with limited capacity.

## Monitor your usage

Security Copilot has a usage limit. When the usage in your organization nears its limit, you're notified when you submit a prompt. To avoid a disruption of service, contact the Azure capacity owner or contributor to increase the Security Compute Units (SCU) or limit the number of prompts.

Learn more about [usage limits](/copilot/security/manage-usage). 

## Provide feedback

Your feedback on the Defender for Cloud integration with Security Copilot helps with development. To provide feedback, in Copilot, select **How's this response?** at the bottom of each completed prompt and select any of the following options:

- **Looks right** - Select this option if the results are accurate, based on your assessment.
- **Needs improvement** - Select this option if any detail in the results is incorrect or incomplete, based on your assessment.
- **Inappropriate** - Select this option if the results contain questionable, ambiguous, or potentially harmful information.

For each feedback option, you can provide more information in the next dialog box that appears. Whenever possible, and when the result is **Needs improvement**, write a few words explaining what can be done to improve the outcome. If you entered prompts specific to Defender for Cloud and the results aren't related, include that information.

## Privacy and data security in Security Copilot

When you interact with Security Copilot to get Defender TI data, Copilot pulls that data from Defender for Cloud. The prompts, the data retrieved, and the output shown in the prompt results are processed and stored within the Copilot service. 

[Learn more about privacy and data security in Microsoft Security Copilot](/security-copilot/privacy-data-security).

## Related content

- [Analyze recommendations with Microsoft Security Copilot](analyze-with-copilot.md)
- [Summarize recommendations with Microsoft Security Copilot](summarize-with-copilot.md)
- [Remediate recommendations with Microsoft Security Copilot](remediate-with-copilot.md)
- [Delegate recommendations with Microsoft Security Copilot](delegate-with-copilot.md)
- [Remediate code with Microsoft Security Copilot](remediate-code-with-copilot.md)
