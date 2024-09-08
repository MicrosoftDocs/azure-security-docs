---
title: Alerts for AI workloads
description: This article lists the security alerts for AI workloads visible in Microsoft Defender for Cloud.
ms.topic: reference
ms.custom: linux-related-content
ms.date: 06/03/2024
ai-usage: ai-assisted
---

# Alerts for AI workloads

This article lists the security alerts you might get for AI workloads from Microsoft Defender for Cloud and any Microsoft Defender plans you enabled. The alerts shown in your environment depend on the resources and services you're protecting, and your customized configuration.  

> [!NOTE]
> Some of the recently added alerts powered by Microsoft Defender Threat Intelligence and Microsoft Defender for Endpoint might be undocumented.

[Learn how to respond to these alerts](managing-and-responding-alerts.yml).

[Learn how to export alerts](continuous-export.md).

> [!NOTE]
> Alerts from different sources might take different amounts of time to appear. For example, alerts that require analysis of network traffic might take longer to appear than alerts related to suspicious processes running on virtual machines.

## AI workload alerts

### Detected credential theft attempts on an Azure OpenAI model deployment

(AI.Azure_CredentialTheftAttempt)

**Description**: The credential theft alert is designed to notify the SOC when credentials are detected within GenAI model responses to a user prompt, indicating a potential breach. This alert is crucial for detecting cases of credential leak or theft, which are unique to generative AI and can have severe consequences if successful.

**[MITRE tactics](alerts-reference.md#mitre-attck-tactics)**: Credential Access, Lateral Movement, Exfiltration

**Severity**: Medium

### A Jailbreak attempt on an Azure OpenAI model deployment was blocked by Azure AI Content Safety Prompt Shields

(AI.Azure_Jailbreak.ContentFiltering.BlockedAttempt)

**Description**: The Jailbreak alert, carried out using a direct prompt injection technique, is designed to notify the SOC there was an attempt to manipulate the system prompt to bypass the generative AI’s safeguards, potentially accessing sensitive data or privileged functions. It indicated that such attempts were blocked by Azure Responsible AI Content Safety (AKA Prompt Shields), ensuring the integrity of the AI resources and the data security.

**[MITRE tactics](alerts-reference.md#mitre-attck-tactics)**: Privilege Escalation, Defense Evasion

**Severity**: Medium

### A Jailbreak attempt on an Azure OpenAI model deployment was detected by Azure AI Content Safety Prompt Shields

(AI.Azure_Jailbreak.ContentFiltering.DetectedAttempt)

**Description**: The Jailbreak alert, carried out using a direct prompt injection technique, is designed to notify the SOC there was an attempt to manipulate the system prompt to bypass the generative AI’s safeguards, potentially accessing sensitive data or privileged functions. It indicated that such attempts were detected by Azure Responsible AI Content Safety (AKA Prompt Shields), but were not blocked due to content filtering settings or due to low confidence.

**[MITRE tactics](alerts-reference.md#mitre-attck-tactics)**: Privilege Escalation, Defense Evasion

**Severity**: Medium

### Sensitive Data Exposure Detected in Azure OpenAI Model Deployment

(AI.Azure_DataLeakInModelResponse.Sensitive)

**Description**: The sensitive data leakage alert is designed to notify the SOC that a GenAI model responded to a user prompt with sensitive information, potentially due to a malicious user attempting to bypass the generative AI’s safeguards to access unauthorized sensitive data.

**[MITRE tactics](alerts-reference.md#mitre-attck-tactics)**: Collection

**Severity**: Medium

### Corrupted AI application\model\data directed a phishing attempt at a user

(AI.Azure_PhishingContentInModelResponse)

**Description**: This alert indicates a corruption of an AI application developed by the organization, as it has actively shared a known malicious URL used for phishing with a user. The URL originated within the application itself, the AI model or the data the application can access.

**[MITRE tactics](alerts-reference.md#mitre-attck-tactics)**: Impact (Defacement)

**Severity**: High

### Phishing URL shared in an AI application

(AI.Azure_PhishingContentInAIApplication)

**Description**: This alert indicates a potential corruption of an AI application, or a phishing attempt by one of the end users. The alert determines that a malicious URL used for phishing was passed during a conversation through the AI application, however the origin of the URL (user or application) is unclear. 

**[MITRE tactics](alerts-reference.md#mitre-attck-tactics)**: Impact (Defacement), Collection

**Severity**: High

### Phishing attempt detected in an AI application

(AI.Azure_PhishingContentInUserPrompt)

**Description**: This alert indicates a URL used for phishing attack was sent by a user to an AI application. The content typically lures visitors into entering their corporate credentials or financial information into a legitimate looking website. Sending this to an AI application may be for the purpose of corrupting it, poisoning the data sources it has access to, or gaining access to employees or other customers via the application's tools.

**[MITRE tactics](alerts-reference.md#mitre-attck-tactics)**: Collection

**Severity**: High

> [!NOTE]
> For alerts that are in preview: [!INCLUDE [Legalese](./includes/defender-for-cloud-preview-legal-text.md)]

## Next steps

- [Security alerts in Microsoft Defender for Cloud](alerts-overview.md)
- [Manage and respond to security alerts in Microsoft Defender for Cloud](managing-and-responding-alerts.yml)
- [Continuously export Defender for Cloud data](continuous-export.md)
