---
title: Remediate Defender for Endpoint misconfiguration issues in Microsoft Defender for Cloud
description: Identify and remediate Defender for Endpoint misconfigurations in Defender for Cloud
author: dcurwin
ms.author: dacurwin
ms.topic: how-to
ms.date: 02/19/2025
ai-usage: ai-assisted
#customer intent: As a user, I want to learn how to review and remediate endpoint detection and response recommendations in order to ensure the security of my virtual machine.
---

# Remediate Defender for Endpoint misconfigurations (agentless)

Microsoft Defender for Cloud integrates natively with [Defender for Endpoint](/defender-endpoint/microsoft-defender-endpoint) to provide endpoint detection and response (EDR) capabilities for machines connected to Defender for Cloud.

As part of these [integrated EDR capabilities](integration-defender-for-endpoint.md), Defender for Cloud agentlessly scans machines to assess whether they're running an EDR solution. In addition, for machines using Defender for Endpoint as an EDR, Defender for Servers agentlessly scans the machines with security checks that assess whether Defender for Endpoint is configured correctly. Checks include:

- `Both full and quick scans are out of 7 days`
- `Signature out of date`
- `Anti-virus is off or partially configured`

When misconfigurations are found, Defender for Cloud makes recommendations to fix them. This article describes how to remediate those recommendations.

> [!NOTE]
>
> - Defender for Cloud uses agentless scanning to assess EDR settings.
> - Agentless scanning replaces the Log Analytics agent (also known as the Microsoft Monitoring Agent (MMA)), which was previously used to collect machine data.
> - The use of MMA is retired. Scanning using the MMA was deprecated in November 2024.

## Prerequisites

**Requirement** | **Details**
--- | ---
**Plan** | [Defender for Cloud](connect-azure-subscription.md) must be available in the Azure subscription and one of these plans must be enabled:<br/><br/>- [Defender for Servers Plan 2](tutorial-enable-servers-plan.md)<br/>- [Defender Cloud Security Posture Management (CSPM)](tutorial-enable-cspm-plan.md)
**Agentless scanning** | [Agentless scanning for machines](concept-agentless-data-collection.md) must be turned on. It's enabled by default in the plans, but if you need to turn it on manually, [follow these instructions](enable-agentless-scanning-vms.md).
**Machines** | Defender for Endpoint must be running as the EDR solution on VMs.

## Investigate misconfiguration recommendations

1. In **Defender for Cloud** > **Recommendations**.

1. Search for and select one of the following recommendations:

    - `EDR configuration issues should be resolved on virtual machines`
    - `EDR configuration issues should be resolved on EC2s`
    - `EDR configuration issues should be resolved on GCP virtual machines`
    -
    -

    :::image type="content" source="media/endpoint-detection-response/configurable-solutions.png" alt-text="Screenshot that shows the recommendations that configure your endpoint detection and solution and remediate misconfigurations." lightbox="media/endpoint-detection-response/configurable-solutions.png":::

1. Select each of the security checks to review the affected resources.

    :::image type="content" source="media/endpoint-detection-response/affected-resources.png" alt-text="Screenshot that shows a selected security check and the affected resources." lightbox="media/endpoint-detection-response/affected-resources.png":::

1. Expand the **Affected resources** section

     :::image type="content" source="media/endpoint-detection-response/affected-resources-section.png" alt-text="Screenshot that shows you where you need to select on screen to expand the affected resources section.":::

1. Review the resource findings.
    :::image type="content" source="media/endpoint-detection-response/resources-findings.png" alt-text="Screenshot that shows the findings of an affected unhealthy resource." lightbox="media/endpoint-detection-response/resources-findings.png":::

1. Drill into the security check, and follow the remediation steps.

    :::image type="content" source="media/endpoint-detection-response/security-check-remediation.png" alt-text="Screenshot that shows the additional details section.":::

After the process is completed, it can take up to 24 hours until your machine appears in the **Healthy resources** tab.

## Related content

[Verify](endpoint-detection-response.md) that machines have an EDR solution configured.
