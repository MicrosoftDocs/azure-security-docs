---
title: Understand Defender for Servers data collection in Microsoft Defender for Cloud
description: Understand how the Defender for Servers plan collects data. 
ms.topic: concept-article
ms.author: dacurwin
author: dcurwin
ms.date: 02/19/2025
#customer intent: As a reader, I want to understand how to plan the deployment of Defender for Servers agents and extensions.
---

# Understand data collection in Defender for Servers

This article helps you to understand how [Defender for Servers](defender-for-servers-overview.md) in Microsoft Defender for Cloud collects data for assessment.

## Before you begin

This article is the *fourth* article in the Defender for Servers planning guide. Before you begin, review the earlier articles:

1. Start [planning your deployment](plan-defender-for-servers.md).
1. Review [Defender for Servers access roles](plan-defender-for-servers-roles.md).
1. Select a [Defender for Servers plan](plan-defender-for-servers-select-plan.md)

## Data collection

Defender for Servers uses a number of methods to collect machine information, including [agentless machine scanning](concept-agentless-data-collection.md) and the [Defender for Endpoint agent](integration-defender-for-endpoint.md).

**Feature** | **Data collection method**
--- | ---
[Assess machines for an EDR solution](detect-endpoint-detection-response-solutions.md) | Agentless scanning
[Assess Defender for Endpoint as an EDR solution](endpoint-detection-response.md) | Agentless scanning.
[Scan software inventory](/defender-vulnerability-management/tvm-software-inventory) | Agentless scanning.<br/><br/> Software inventory is provided by the integration with Defender Vulnerability Management.
[Scan for vulnerabilities](auto-deploy-vulnerability-assessment.md) | Agentless scanning<br/>Agent-based scanning with the Defender for Endpoint agent.<br/>Bring your own license (BYOL) scanning with a [supported third-party solution](deploy-vulnerability-assessment-byol-vm.md).<br/><br/>[Learn about](auto-deploy-vulnerability-assessment.md#hybrid-scanning-behavior) hybrid scanning using both agentless and agent-based scanning.
[Scan machines for secrets](secrets-scanning-servers.md) | Agentless scanning.
[Scan machines for malware](agentless-malware-scanning.md) | Agentless scanning.<br/><br/> [Next-generation antimalware protection](/defender-endpoint/next-generation-protection) is also provided by Defender for Endpoint integration, using the Defender for Endpoint agent.
[Scan for OS misconfigurations](operating-system-misconfiguration.md) | Assess OS configuration against compute security baselines in the Microsoft Cloud Security Benchmark using the [Azure Machine Configuration extension](security-baseline-guest-configuration.md).
[Scan for file and registry changes with file integrity monitoring](file-integrity-monitoring-overview.md) | Defender for Endpoint agent.
[Scan for system and patch updates](enable-periodic-system-updates.md) | Relies on [Azure Update Manager VM extension](/azure/update-manager/workflow-update-manager).
[Use free data ingestion benefit](data-ingestion-benefit.md) | Azure Monitor agent (AMA).

## Next steps

[Understand how data is stored and when you need a Log Analytics workspace](plan-defender-for-servers-data-workspace.md)
