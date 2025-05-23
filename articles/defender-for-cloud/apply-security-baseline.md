---
title: Review OS misconfiguration recommendations in Microsoft Defender for Cloud
description: Learn how Microsoft Defender for Cloud uses the guest configuration to compare machine OS settings with baselines in Microsoft Cloud Security Benchmark.
ms.topic: how-to
ms.author: dacurwin
author: dcurwin
ms.date: 02/19/2025
---

# Remediate OS misconfigurations

Microsoft Defender for Cloud provides security recommendations to improve organizational security posture and reduce risk. An important element in risk reduction is machine hardening.

Defender for Cloud assesses operating system settings against compute security baselines provided by the [Microsoft Cloud Security Benchmark (MCSB)](/security/benchmark/azure/introduction). Machine information is gathered for assessment using the Azure Policy machine configuration extension (formerly known as the guest configuration) on the machine. [Learn more](operating-system-misconfiguration.md)

This article describes how to review recommendations made by the assessment.

## Prerequisites

**Requirements** | **Details**
--- | ---
**Plan** | [Defender for Servers Plan 2 must be enabled](tutorial-enable-servers-plan.md)
**Extension** | The [Azure Policy machine configuration must be installed on machines](security-baseline-guest-configuration.md).

This feature previously used the Log Analytics agent (also known as the Microsoft Monitoring agent (MMA)) to collect information. If you still have the MMA in use, you might receive duplicate recommendations for the same machine. To avoid this, you can [disable the MMA on the machine](prepare-deprecation-log-analytics-mma-agent.md#duplicate-recommendations).

## Review and remediate OS baseline recommendations

1. In Defender for Cloud, open the **Recommendations** page.
1. Select the relevant recommendation.
    - For **Windows** machines, [Vulnerabilities in security configuration on your Windows machines should be remediated (powered by Guest Configuration)](https://portal.azure.com/#blade/Microsoft_Azure_Security/RecommendationsBlade/assessmentKey/8c3d9ad0-3639-4686-9cd2-2b2ab2609bda).
    - For **Linux** machines, [Vulnerabilities in security configuration on your Linux machines should be remediated (powered by Guest Configuration)](https://portal.azure.com/#blade/Microsoft_Azure_Security/RecommendationsBlade/assessmentKey/1f655fb7-63ca-4980-91a3-56dbc2b715c6)  

        :::image type="content" source="media/apply-security-baseline/recommendations-baseline.png" alt-text="The two recommendations for comparing the OS configuration of machines with the relevant Azure security baseline." lightbox="media/apply-security-baseline/recommendations-baseline.png":::

1. On the recommendation details page you can see the affected resources, and specific security findings. [Remediate](implement-security-recommendations.md) the recommendation.

## Query recommendations

Defender for Cloud uses [Azure Resource Graph](/azure/governance/resource-graph/overview?branch=main) for API, and portal queries, to query recommendation information. You can utilize these resources to create your own queries to retrieve information.

You can learn how to [review recommendations in Azure Resource Graph](review-security-recommendations.md#review-recommendations-in-azure-resource-graph).

Here are two sample queries you can use:

- **Query all unhealthy rules for a specific resource**

    ```rest
    Securityresources 
    | where type == "microsoft.security/assessments/subassessments" 
    | extend assessmentKey=extract(@"(?i)providers/Microsoft.Security/assessments/([^/]*)", 1, id) 
    | where assessmentKey == '1f655fb7-63ca-4980-91a3-56dbc2b715c6' or assessmentKey ==  '8c3d9ad0-3639-4686-9cd2-2b2ab2609bda' 
    | parse-where id with machineId:string '/providers/Microsoft.Security/' * 
    | where machineId  == '{machineId}'
    ```

- **All Unhealthy Rules and the amount if Unhealthy machines for each**

    ```rest
    securityresources 
    | where type == "microsoft.security/assessments/subassessments" 
    | extend assessmentKey=extract(@"(?i)providers/Microsoft.Security/assessments/([^/]*)", 1, id) 
    | where assessmentKey == '1f655fb7-63ca-4980-91a3-56dbc2b715c6' or assessmentKey ==  '8c3d9ad0-3639-4686-9cd2-2b2ab2609bda' 
    | parse-where id with * '/subassessments/' subAssessmentId:string 
    | parse-where id with machineId:string '/providers/Microsoft.Security/' * 
    | extend status = tostring(properties.status.code) 
    | summarize count() by subAssessmentId, status
    ```

## Next steps

Learn more about [Azure Resource Graph's query language](/azure/governance/resource-graph/concepts/query-language?branch=main).
