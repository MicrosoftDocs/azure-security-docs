---
title: Review security recommendations
description: Learn how to review security recommendations in Microsoft Defender for Cloud and improve the security posture of your environments.
ms.topic: how-to
author: ElazarK
ms.author: elkrieger
ms.date: 04/21/2025
#customer intent: As a security analyst, I want to learn how to review security recommendations in Microsoft Defender for Cloud so that I can improve the security posture of my environments.
---

# Review security recommendations

In Microsoft Defender for Cloud, resources and workloads are assessed against built-in and custom security standards enabled in your Azure subscriptions, Amazon Web Services (AWS) accounts, and Google Cloud Platform (GCP) projects. Based on those assessments, security recommendations provide practical steps to remediate security issues and improve security posture.

Defender for Cloud proactively uses a dynamic engine that assesses the risks in your environment while considering the potential for exploitation and the potential business affect on your organization. The engine prioritizes security recommendations based on the risk factors of each resource. The context of the environment determines these risk factors. This context includes the resource's configuration, network connections, and security posture.

## Prerequisites

- You must [enable Defender Cloud Security Posture Management (CSPM)](enable-enhanced-security.md) on your environment.

> [!NOTE]
> Recommendations are included by default with Defender for Cloud, but you can't see [risk prioritization](risk-prioritization.md) without Defender CSPM enabled on your environment.

## Review the recommendations page

Review all of the details related to a recommendation before trying to understand the process needed to resolve the recommendation. Ensure that all of the recommendation details are correct before resolving the recommendation.

**To review a recommendation's details**:

1. Sign in to the [Azure portal](https://portal.azure.com/).

1. Navigate to **Defender for Cloud** > **Recommendations**.

1. Select a recommendation.

1. On the recommendation page, review the following details:
    - **Risk level** - The vulnerability and business effect of the underlying security issue, considering environmental resource context such as internet exposure, sensitive data, lateral movement, and more.
    - **Risk factors** - Environmental factors of the resource affected by the recommendation, which influences the vulnerability and business effect of the underlying security issue. Examples of risk factors include internet exposure, sensitive data, and lateral movement potential.
    - **Resource** - The name of the affected resource.
    - **Status** - The status of the recommendation, such as unassigned, on time, or overdue.
    - **Description** - A brief description of the security issue.
    - **Attack Paths** - The number of attack paths.
    - **Scope** - The affected subscription or resource.
    - **Freshness** - The freshness interval of the recommendation.
    - **Last change date** - The date this recommendation was last changed.
    - **Severity** - The severity of the recommendation (High, Medium, or Low). More details are provided.
    - **Owner** - The person assigned to the recommendation.
    - **Due date** - The assigned due date for resolving the recommendation.
    - **Tactics & techniques** - The tactics and techniques mapped to MITRE ATT&CK.

## Explore a recommendation

You can perform various actions to interact with recommendations. If an option isn't available, it's not relevant to the recommendation.

**To explore a recommendation**:

1. Sign in to the [Azure portal](https://portal.azure.com/).

1. Navigate to **Defender for Cloud** > **Recommendations**.

1. Select a recommendation.

1. In the recommendation, you can perform these actions:

    - Select **Open query** to view detailed information about the affected resources with an Azure Resource Graph Explorer query.

    - Select **View policy definition** to view the Azure Policy entry for the underlying recommendation, if relevant.

    - Select **View recommendation for all resources** to view all resources affected by the recommendation.

1. In **Take action**:

    - **Remediate**: A description of the manual steps required to resolve the security issue on the affected resources. For recommendations with the **Fix** option, you can select **View remediation logic** before applying the suggested fix to your resources.

    - **Recommendation owner and set due date**: If you have a [governance rule](governance-rules.md) enabled for the recommendation, you can assign an owner and due date.

    - **Exempt**: You can exempt resources from the recommendation or disable specific findings using disable rules.

    - **Workflow automation**: Set a logic app to trigger with the recommendation.

    :::image type="content" source="media/review-security-recommendations/recommendation-take-action.png" alt-text="Screenshot that shows what you can see in the recommendation when you select the take action tab." lightbox="media/review-security-recommendations/recommendation-take-action.png":::

1. In **Findings**, you can review affiliated findings by severity.

     :::image type="content" source="media/review-security-recommendations/recommendation-findings.png" alt-text="Screenshot of the findings tab in a recommendation that shows all of the attack paths for that recommendation." lightbox="media/review-security-recommendations/recommendation-findings.png":::

1. In **Graph**, you can view and investigate all context that is used for risk prioritization, including [attack paths](how-to-manage-attack-path.md). You can select a node in an attack path to view the details of the selected node.

     :::image type="content" source="media/review-security-recommendations/recommendation-graph.png" alt-text="Screenshot of the graph tab in a recommendation that shows all of the attack paths for that recommendation." lightbox="media/review-security-recommendations/recommendation-graph.png":::

1. View more details by selecting a node.

    :::image type="content" source="media/review-security-recommendations/select-node.png" alt-text="Screenshot of a node located in the graph tab that is selected and showing the additional details." lightbox="media/review-security-recommendations/select-node.png":::

1. Select **Insights**.

1. In the vulnerability dropdown menu, select a vulnerability to view the details.

    :::image type="content" source="media/review-security-recommendations/insights.png" alt-text="Screenshot of the insights tab for a specific node." lightbox="media/review-security-recommendations/insights.png":::

1. (Optional) Select **Open the vulnerability page** to view the associated recommendation page.

1. [Remediate the recommendation](implement-security-recommendations.md).

## Group recommendations by title

Defender for Cloud's recommendation page allows you to group recommendations by title. This feature is useful when you want to remediate a recommendation that affects multiple resources due to a specific security issue.

**To group recommendations by title**:

1. Sign in to the [Azure portal](https://portal.azure.com/).

1. Navigate to **Defender for Cloud** > **Recommendations**.

1. Select **Group by title**.

    :::image type="content" source="media/review-security-recommendations/group-by-title.png" alt-text="Screenshot of the recommendations page that shows where the group by title toggle is located on the screen." lightbox="media/review-security-recommendations/group-by-title.png":::

## Manage recommendations assigned to you

Defender for Cloud supports governance rules for recommendations, to assign a recommendation owner or due date for action. Governance rules help ensure accountability and an SLA for recommendations.

- Recommendations are listed as **On time** until their due date passes, when they're changed to **Overdue**.
- Before the recommendation is overdue, it doesn't affect the secure score.
- You can also apply a grace period during which overdue recommendations don't affect the secure score.

[Learn more](governance-rules.md) about configuring governance rules.

**To manage recommendations assigned to you**:

1. Sign in to the [Azure portal](https://portal.azure.com/).

1. Navigate to **Defender for Cloud** > **Recommendations**.

1. Select **Add filter** > **Owner**.

1. Select your user entry.

1. Select **Apply**.

1. In the recommendation results, review the recommendations, including affected resources, risk factors, attack paths, due dates, and status.

1. Select a recommendation to review it further.


1. In **Take action** > **Change owner & due date**, select **Edit assignment** to change the recommendation owner and due date if necessary.
    - By default, the owner of the resource receives a weekly email listing the recommendations assigned to them.
    - If you select a new remediation date, specify reasons for remediation by that date in **Justification**.
    - In **Set email notifications**, you can:
        - Override the default weekly email to the owner.
        - Notify owners weekly with a list of open/overdue tasks.
        - Notify the owner's direct manager with an open task list.

1. Select **Save**.

> [!NOTE]
> Changing the expected completion date doesn't change the due date for the recommendation, but security partners can see that you plan to update the resources by the specified date.

## Review recommendations in Azure Resource Graph

You can use [Azure Resource Graph](/azure/governance/resource-graph/) to write a [Kusto Query Language (KQL)](/azure/data-explorer/kusto/query/) to query Defender for Cloud security posture data across multiple subscriptions. Azure Resource Graph provides an efficient way to query at scale across cloud environments by viewing, filtering, grouping, and sorting data.

**To review recommendations in Azure Resource Graph**:

1. Sign in to the [Azure portal](https://portal.azure.com/).

1. Navigate to **Defender for Cloud** > **Recommendations**.

1. Select a recommendation.

1. Select **Open query**.

1. You can open the query in one of two ways:

   - **Query returning affected resource** - Returns a list of all of the resources affected by this recommendation.
   - **Query returning security findings** - Returns a list of all security issues found by the recommendation.

1. Select **run query**.

    :::image type="content" source="./media/review-security-recommendations/run-query.png" alt-text="Screenshot of Azure Resource Graph Explorer showing the results for the recommendation shown in the previous screenshot." lightbox="media/review-security-recommendations/run-query.png":::

1. Review the results.

## How are recommendations classified?

Every security recommendation from Defender for Cloud is given one of three severity ratings:

- **High severity**: Address these recommendations immediately, as they indicate a critical security vulnerability that an attacker could exploit to gain unauthorized access to your systems or data. Examples of high severity recommendations include unprotected secrets on a machine, overly permissive inbound NSG rules, clusters allowing images to be deployed from untrusted registries, and unrestricted public access to storage accounts or databases.

- **Medium severity**: These recommendations indicate a potential security risk that should be addressed in a timely manner, but might not require immediate attention. Examples of medium severity recommendations include containers that share sensitive host namespaces, web apps that don't use managed identities, Linux machines that don't require SSH keys during authentication, and unused credentials left in the system after 90 days of inactivity.

- **Low severity**: These recommendations indicate a relatively minor security issue that can be addressed at your convenience. Examples of low severity recommendations include the need to disable local authentication in favor of Microsoft Entra ID, health issues with your endpoint protection solution, best practices not being followed with network security groups, or misconfigured logging settings that could make it harder to detect and respond to security incidents.

The internal views of an organization might differ from Microsoft’s classification of a specific recommendation. So, it's always a good idea to review each recommendation carefully and consider its potential effect on your security posture before deciding how to address it.

> [!NOTE]
> Defender CSPM customers have access to a richer classification system where recommendations are shown a more dynamic **Risk level** that utilizes the *context* of the resource and all related resources. Learn more about [risk prioritization](risk-prioritization.md).

### Example

In this example, this recommendation details page shows 15 affected resources:

:::image type="content" source="./media/review-security-recommendations/open-query.png" alt-text="Screenshot of the Open Query button on the recommendation details page." lightbox="media/review-security-recommendations/open-query.png":::

When you open the underlying query, and run it, Azure Resource Graph Explorer returns the same affected resources for this recommendation.

## Next step

> [!div class="nextstepaction"]
> [Remediate security recommendations](implement-security-recommendations.md)
