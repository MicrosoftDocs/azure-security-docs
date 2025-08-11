---
title: Identify and remediate attack paths
author: Elazark
ms.author: elkrieger
description: Learn how to identify and remediate attack paths in Microsoft Defender for Cloud and enhance the security of your environment.
ms.topic: how-to
ms.date: 05/18/2025
ms.custom: sfi-image-nochange
#customer intent: As a security analyst, I want to learn how to identify and remediate attack paths in Microsoft Defender for Cloud so that I can enhance the security of my environment.
---

# Identify and remediate attack paths

Defender for Cloud uses a [proprietary algorithm to locate potential attack paths](concept-attack-path.md#what-is-an-attack-path) specific to your multicloud environment. With the refined attack path experience, Defender for Cloud focuses on real, externally-driven and exploitable threats rather than broad scenarios. The algorithm detects attack paths that begin outside your organization and progress to business-critical targets, helping you cut through the noise and act faster.

You can use attack path analysis to address security issues that pose immediate threats and have the greatest potential for exploitation in your environment. Defender for Cloud analyzes which security issues are part of externally-initiated attack paths that attackers could use to breach your environment. It also highlights the security recommendations you need to resolve to mitigate these issues.

By default attack paths are organized by risk level. The risk level is determined by a context-aware risk-prioritization engine that considers the risk factors of each resource. Learn more about how Defender for Cloud [prioritizes security recommendations](risk-prioritization.md).

## Prerequisites

- You must [enable Defender Cloud Security Posture Management (CSPM)](enable-enhanced-security.md) and have [agentless scanning](enable-vulnerability-assessment-agentless.md) enabled.

- **Required roles and permissions**: Security Reader, Security Admin, Reader, Contributor, or Owner.

> [!NOTE]
> With the refined attack path experience, you may see an empty Attack Path page, as attack paths now focus on real, externally-driven and exploitable threats rather than broad scenarios. This helps reduce noise and prioritize imminent risks.

**To view attack paths that are related to containers**:

- You must [enable agentless container posture extension](tutorial-enable-cspm-plan.md) in Defender CSPM
    or
- You can [enable Defender for Containers](defender-for-containers-enable.md), and install the relevant agents in order to view attack paths that are related to containers. This also gives you the ability to [query](how-to-manage-cloud-security-explorer.md#build-a-query) containers data plane workloads in security explorer.

- **Required roles and permissions**: Security Reader, Security Admin, Reader, Contributor, or Owner.

## Identify attack paths

The attack path page shows you an overview of all of your attack paths. You can also see your affected resources and a list of active attack paths.

:::image type="content" source="media/concept-cloud-map/attack-path-homepage.png" alt-text="Screenshot of a sample attack path homepage." lightbox="media/concept-cloud-map/attack-path-homepage.png":::

You can use Attack path analysis  to locate the biggest risks to your environment and to remediate them.

**To identify attack paths**:

1. Sign in to the [Azure portal](https://portal.azure.com).

1. Navigate to **Microsoft Defender for Cloud** > **Attack path analysis**.

    :::image type="content" source="media/how-to-manage-attack-path/attack-path-blade.png" alt-text="Screenshot that shows the attack path analysis page on the main screen." lightbox="media/how-to-manage-attack-path/attack-path-blade.png":::

1. Select an attack path.

1. Select a node.

    :::image type="content" source="media/how-to-manage-attack-path/node-select.png" alt-text="Screenshot of the attack path screen that shows you where the nodes are located for selection." lightbox="media/how-to-manage-attack-path/node-select.png":::
   > [!NOTE]
   > If you have limited permissions—especially across subscriptions—you might not see full attack path details. This is expected behavior designed to protect sensitive data.
   > To view all details, make sure you have the necessary permissions. 

1. Select **Insight** to view the associated insights for that node.

    :::image type="content" source="media/how-to-manage-attack-path/insights.png" alt-text="Screenshot of the insights tab for a specific node." lightbox="media/how-to-manage-attack-path/insights.png":::

1. Select **Recommendations**.

    :::image type="content" source="media/how-to-manage-attack-path/attack-path-recommendations.png" alt-text="Screenshot that shows you where to select recommendations on the screen." lightbox="media/how-to-manage-attack-path/attack-path-recommendations.png":::

1. Select a recommendation.

1. [Remediate the recommendation](implement-security-recommendations.md).

## Remediate attack paths

Once you're done with your investigation of an attack path and you review all of the associated findings and recommendations, you can start to remediate the attack path.

**To remediate an attack path**:

1. Navigate to **Microsoft Defender for Cloud** > **Attack path analysis**.

1. Select an attack path.

1. Select **Remediation**.

    :::image type="content" source="media/how-to-manage-attack-path/recommendations-tab.png" alt-text="Screenshot of the attack path that shows you where to select remediation." lightbox="media/how-to-manage-attack-path/recommendations-tab.png":::

1. Select a recommendation.

1. [Remediate the recommendation](implement-security-recommendations.md).

Once an attack path is resolved, it can take up to 24 hours for an attack path to be removed from the list.

## Remediate all recommendations within an attack path

Attack path analysis grants you the ability to see all recommendations by attack path without having to check each node individually. You can resolve all recommendations without having to view each node individually.

The remediation path contains two types of recommendation:

- **Recommendations** - Recommendations that mitigate the attack path.
- **Additional recommendations** - Recommendations that reduce the exploitation risks, but don’t mitigate the attack path.

**To resolve all recommendations**:

1. Sign in to the [Azure portal](https://portal.azure.com).

1. Navigate to **Microsoft Defender for Cloud** > **Attack path analysis**.

1. Select an attack path.

1. Select **Remediation**.

    :::image type="content" source="media/how-to-manage-attack-path/bulk-recommendations.png" alt-text="Screenshot that shows where to select on the screen to see the attack paths full list of recommendations." lightbox="media/how-to-manage-attack-path/bulk-recommendations.png":::

1. Expand **Additional recommendations**.

1. Select a recommendation.

1. [Remediate the recommendation](implement-security-recommendations.md).

Once an attack path is resolved, it can take up to 24 hours for an attack path to be removed from the list.

## Next Step

> [!div class="nextstepaction"]
> [Build queries with cloud security explorer](how-to-manage-cloud-security-explorer.md).
