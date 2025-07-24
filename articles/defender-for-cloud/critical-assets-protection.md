---
title: Critical assets protection (Preview)
description: Learn how to identify and protect your critical assets in Microsoft Defender for Cloud with Microsoft Security Exposure Management.
ms.topic: conceptual
ms.date: 05/18/2025
ms.custom: sfi-image-nochange
---

# Critical assets protection in Microsoft Defender for Cloud

Critical assets protection enables security administrators to automatically tag the "crown jewel" resources that are most critical to their organizations, allowing Defender for Cloud to provide them with the highest level of protection and prioritize security issues on these assets above anything else.

Defender for Cloud suggests pre-defined classification rules that were developed by our research team to discover critical assets automatically, and allows you to create custom classification rules based on your business and organizational conventions.

Critical asset rules are bi-directionally synced with Microsoft Security Exposure Management - rules that were created in Microsoft Security Exposure Management are synced to Defender for Cloud, and vice versa. [Learn more about critical assets protection in Microsoft Security Exposure Management](/security-exposure-management/critical-asset-management).

## Availability

| Aspect | Details |
|--|--|
| Release state | General Availability|
| Prerequisites | Defender Cloud Security Posture Management (CSPM) enabled  |
| Required Microsoft Entra ID built-in roles: | To create/edit/read classification rules: Security Operator or higher <br> To read classification rules: Global Reader, Security Reader  |
| Clouds: | All commercial clouds |

## Set up critical asset rules

1. Sign in to the [Azure portal](https://portal.azure.com).

1. Navigate to **Microsoft Defender for Cloud** > **Environment Settings**.
1. Select the **Resource criticality** tile.

    :::image type="content" source="media/critical-assets-protection/resource-criticality-tile.png" alt-text="Screenshot of the resource criticality tile." lightbox="media/critical-assets-protection/resource-criticality-tile.png":::

1. The **Critical asset management** pane opens. Select **Open Microsoft Defender portal.**"

    :::image type="content" source="media/critical-assets-protection/critical-asset-management-pane.png" alt-text="Screenshot of the critical asset management pane." lightbox="media/critical-assets-protection/critical-asset-management-pane.png":::

1. You then arrive at the **Critical asset management** page in the **Microsoft Defender XDR** portal.

    :::image type="content" source="media/critical-assets-protection/critical-asset-management-page.png" alt-text="Screenshot of critical asset management page." lightbox="media/critical-assets-protection/critical-asset-management-page.png":::

1. To create custom critical asset rules to tag your resources as **Critical resources** in Defender for Cloud, select the **Create a new classification** button.

    :::image type="content" source="media/critical-assets-protection/create-new-classification.png" alt-text="Screenshot of Create a new classification button." lightbox="media/critical-assets-protection/create-new-classification.png":::

1. Add a name and description for your new classification, and use under **Query builder**, select **Cloud resource** to build your critical assets rule. Then select **Next**.

    :::image type="content" source="media/critical-assets-protection/create-critical-asset-classification.png" alt-text="Screenshot of how to create critical asset classification." lightbox="media/critical-assets-protection/create-critical-asset-classification.png":::

1. On the **Preview assets** page, you can see a list of assets that match the rule you created. After reviewing the page, select **Next**.

    :::image type="content" source="media/critical-assets-protection/preview-assets.png" alt-text="Screenshot of Preview assets page, showing a list of all assets that match the rule." lightbox="media/critical-assets-protection/preview-assets.png":::

1. On the **Assign criticality** page, assign the criticality level to all assets matching the rule. Then select **Next**.

    :::image type="content" source="media/critical-assets-protection/assign-criticality.png" alt-text="A screenshot of the Assign criticality page." lightbox="media/critical-assets-protection/assign-criticality.png":::

1. You can then see the **Review and finish** page. Review the results, and once you approve, select **Submit**.

    :::image type="content" source="media/critical-assets-protection/review-finish.png" alt-text="Screenshot of the Review and finish page." lightbox="media/critical-assets-protection/review-finish.png":::

1. After you select **Submit**, you can close the **Microsoft Defender XDR** portal. You should wait for up to two hours until all assets matching your rule are tagged as **Critical**.

> [!NOTE]
> Your critical asset rules apply to all the resources in the tenant that match the rule's condition.

## View and protect your critical assets in Defender for Cloud

1. Once your assets are updated, go to the [Attack path analysis](how-to-manage-attack-path.md) page in Defender for Cloud. You can see all the attack paths to your critical assets.

    :::image type="content" source="media/critical-assets-protection/attack-path-analysis.png" alt-text="Screenshot of attack path analysis page." lightbox="media/critical-assets-protection/attack-path-analysis.png":::

1. If you select an attack path title, you can see its details. Select the target, and under **Insights - Critical resource**, you can see the critical asset tagging information.

    :::image type="content" source="media/critical-assets-protection/critical-resource-insights.png" alt-text="Screenshot of critical resource insights." lightbox="media/critical-assets-protection/critical-resource-insights.png":::

1. In the **Recommendations** page of Defender for Cloud, select the **Preview available** banner to see all the recommendations, which are now prioritized based on asset criticality.

    :::image type="content" source="media/critical-assets-protection/recommendations-page.png" alt-text="Screenshot of the recommendations page, showing critical resources." lightbox="media/critical-assets-protection/recommendations-page.png":::

1. Select a recommendation, and then choose the **Graph** tab. Then choose the target, and select the **Insights** tab. You can see the critical asset tagging information.

    :::image type="content" source="media/critical-assets-protection/recommendation-insights.png" alt-text="Screenshot of critical asset insights for recommendations." lightbox="media/critical-assets-protection/recommendation-insights.png":::

1. In the **Inventory** page of Defender for Cloud, you can see the critical assets in your organization.

    :::image type="content" source="media/critical-assets-protection/inventory-page.png" alt-text="Screenshot of inventory page with critical assets tagged." lightbox="media/critical-assets-protection/inventory-page.png":::

1. To run custom queries on your critical assets, go to the **Cloud Security Explorer** page in Defender for Cloud.

    :::image type="content" source="media/critical-assets-protection/cloud-security-explorer-page.png" alt-text="Screenshot of Cloud Security Explorer page with query for critical assets." lightbox="media/critical-assets-protection/cloud-security-explorer-page.png":::

## Related content

For more information about improving your cloud security posture, see [Cloud security posture management (CSPM)](concept-cloud-security-posture-management.md).
