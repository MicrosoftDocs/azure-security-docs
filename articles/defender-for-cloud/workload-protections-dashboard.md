---
title: Review workload protection in Microsoft Defender for Cloud
description: Review workload protection in the Workload protections dashboard in Microsoft Defender for Cloud
ms.topic: how-to
ms.date: 07/15/2025
---

# Review workload protection

Microsoft Defender for Cloud provides unified view into threat detection and protection for protected resources with the interactive **Workload protections** dashboard.

:::image type="content" source="~/reusable-content/ce-skilling/azure/media/defender-for-cloud/sample-defender-dashboard-numbered.png" alt-text="An example of Defender for Cloud's workload protections dashboard." lightbox="~/reusable-content/ce-skilling/azure/media/defender-for-cloud/sample-defender-dashboard-numbered.png":::

## Defender for Cloud coverage

In the **Defender for Cloud coverage** section of the dashboard, you can see the resources types in your subscription that are eligible for protection by Defender for Cloud. Wherever relevant, you can upgrade here as well. If you want to upgrade all possible eligible resources, select **Upgrade all**.

## Security alerts

The **Security alerts** section shows alerts. When Defender for Cloud detects a threat in any area of your environment, it generates an alert. These alerts describe details of the affected resources, suggested remediation steps, and in some cases an option to trigger a logic app in response. Selecting anywhere in this graph opens the **Security alerts page**.

## Advanced protection

Defender for Cloud includes many advanced threat protection capabilities for virtual machines, SQL databases, containers, web applications, your network, and more. In this advanced protection section, you can see the status of the resources in your selected subscriptions for each of these protections. Select any of them to go directly to the configuration area for that protection type.

## Insights

Insights provide you with news, suggested reading, and high priority alerts that are relevant in your environment.

## Prerequisite

The plan must be enabled at the subscription level to ensure proper functionality. Resources onboarded to Defender for Cloud at the resource level won’t be eligible for the capabilities available through the Workload protection blade.

## Next steps

[Learn about](defender-for-cloud-introduction.md) workloads you can protect in Defender for Cloud
