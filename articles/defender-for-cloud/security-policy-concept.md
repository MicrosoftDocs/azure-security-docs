---
title: Security policies in Microsoft Defender for Cloud 
description: Learn about security policies, standards, and recommendations to improve your cloud security posture in Microsoft Defender for Cloud.
ms.topic: conceptual
ms.date: 11/27/2023
---

# Security policies in Defender for Cloud

Security policies in Microsoft Defender for Cloud consist of security standards and recommendations that help to improve your cloud security posture.

Security standards define rules, compliance conditions for those rules, and actions (effects) to be taken if conditions aren't met. Defender for Cloud assesses resources and workloads against the security standards enabled in your Azure subscriptions, Amazon Web Services (AWS) accounts, and Google Cloud Platform (GCP) projects. Based on those assessments, security recommendations provide practical steps to help you remediate security issues.

## Security standards

Security standards in Defender for Cloud come from these sources:

- **Microsoft cloud security benchmark (MCSB)**: The MCSB standard is applied by default when you onboard cloud accounts to Defender. Your [secure score](secure-score-security-controls.md) is based on assessment against some MCSB recommendations.

- **Regulatory compliance standards**: When you enable one or more [Defender for Cloud plans](defender-for-cloud-introduction.md), you can add standards from a wide range of predefined regulatory compliance programs.

- **Custom standards**: You can create custom security standards in Defender for Cloud, and then add built-in and custom recommendations to those custom standards as needed.

Security standards in Defender for Cloud are based on [Azure Policy](/azure/governance/policy/overview) [initiatives](/azure/governance/policy/concepts/initiative-definition-structure) or on the Defender for Cloud native platform. Currently, Azure standards are based on Azure Policy. AWS and GCP standards are based on Defender for Cloud.

### Work with security standards

Here's what you can do with security standards in Defender for Cloud:

- **Modify the built-in MCSB for the subscription**: When you enable Defender for Cloud, the MCSB is automatically assigned to all Defender for Cloud registered subscriptions. [Learn more about managing the MCSB standard](manage-mcsb.md).

- **Add regulatory compliance standards**: If you have one or more paid plans enabled, you can assign built-in compliance standards against which to assess your Azure, AWS, and GCP resources. [Learn more about assigning regulatory standards](update-regulatory-compliance-packages.yml).

- **Add custom standards**: If you have at least one paid Defender plan enabled, you can define new [custom standards](custom-security-policies.md) and [custom recommendations](create-custom-recommendations.md) in the Defender for Cloud portal. You can then add recommendations to those standards.

### Custom standards

Custom standards appear alongside built-in standards in the **Regulatory compliance** dashboard.

Recommendations derived from assessments against custom standards appear together with recommendations from built-in standards. Custom standards can contain built-in and custom recommendations.

### Custom recommendations

Using custom recommendations based on Kusto Query Language (KQL) is the recommended approach and is supported for all clouds, but requires enabling the [Defender CSPM plan](concept-cloud-security-posture-management.md). With these recommendations, you specify a unique name, a description, remediation steps, severity, and relevant standards. You add recommendation logic with KQL. A query editor provides a built-in query template that you can tweak, or you can write your KQL query.

Alternatively, all Azure customers can onboard their Azure Policy custom initiatives as custom recommendations (legacy approach).

For more information, see [Create custom security standards and recommendations in Microsoft Defender for Cloud](create-custom-recommendations.md).

## Security recommendations

Defender for Cloud periodically and continuously analyzes and assesses the security state of protected resources against  defined security standards, to identify potential security misconfigurations and weaknesses. Defender for Cloud then provides recommendations based on assessment findings.

Each recommendation provides the following information:

- A short description of the issue
- Remediation steps for implementing the recommendation
- Affected resources
- Risk level
- Risk factors
- Attack paths

Every recommendation in Defender for Cloud has an associated risk level that represents how exploitable and impactful the security issue is in your environment. The risk assessment engine takes into account factors such as internet exposure, sensitivity of data, lateral movement possibilities, and attack path remediation. You can prioritize recommendations based on their risk levels.

> [!IMPORTANT]
> [Risk prioritization](risk-prioritization.md) doesn't affect the secure score.

### Example

The MCSB standard is an Azure Policy initiative that includes multiple compliance controls. One of these controls is "Storage accounts should restrict network access using virtual network rules."

Defender for Cloud continuously assesses resources. If it finds any that don’t satisfy this control, it marks them as noncompliant and triggers a recommendation. In this case, guidance is to harden Azure Storage accounts that aren't protected with virtual network rules.

## Next steps

- Learn more about [regulatory compliance standards](concept-regulatory-compliance-standards.md), [MCSB](concept-regulatory-compliance.md), and [improving regulatory compliance](regulatory-compliance-dashboard.md).
- Learn more about [security recommendations](review-security-recommendations.md).
