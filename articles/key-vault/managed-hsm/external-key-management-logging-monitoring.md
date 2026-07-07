---
title: Logging and monitoring for Azure Key Vault Managed HSM external key management (preview)
description: How to capture, query, and alert on Azure Key Vault Managed HSM external key management operations across Managed HSM, the EKM Proxy, and the external HSM
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: how-to
ms.date: 07/06/2026
ms.author: mbaldwin
ai-usage: ai-assisted
#Customer intent: As a security or platform operator running a Managed HSM external key management deployment, I need to know which logs are emitted by which party, how to query them, and what to alert on so I can run the system reliably and prove compliance.
---

# Logging and monitoring for Managed HSM external key management (preview)

> [!IMPORTANT]
> Managed HSM external key management is in **preview**. Preview features are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature might change before general availability.

## Overview

Managed HSM external key management spans three independently operated components: Azure Key Vault Managed HSM, the customer-operated EKM Proxy, and the customer-operated external HSM. Each component produces its own logs. No single log stream gives you the complete picture of a wrap or unwrap operation.

Microsoft emits logs for everything inside the Managed HSM service boundary. You're responsible for capturing and retaining logs from the EKM Proxy and the external HSM. This split reflects the [shared-responsibility model](external-key-management-shared-responsibility.md) that applies to all external key management deployments.

This article describes what each party logs, how to configure and query Managed HSM logs, how to capture proxy and HSM logs, how to correlate across all three sources, and what to alert on to detect problems early.

## Where each operation is logged

The following table summarizes which component logs each category of event, who owns that log stream, and which fields you can expect.

| Log source | Owner | Events captured | Example fields |
|---|---|---|---|
| **Managed HSM** (`AzureDiagnostics`) | Microsoft | Every EKM Proxy call; authentication; authorization; management-plane changes | `TimeGenerated`, `Resource`, `OperationName`, `requestUri_s`, `ResultType`, `ResultDescription`, `DurationMs`, `CallerIPAddress`, `CorrelationId` |
| **EKM Proxy** | Customer or HSM vendor | Wrap/unwrap calls; authentication failures; proxy → HSM call status; operational health metrics | Timestamp, external key identifier, algorithm, payload size, result, latency, Managed HSM client cert subject common name, EKM Proxy Request Context fields (Request ID, Correlation ID, MHSM Pool Name) |
| **External HSM** | Customer or HSM vendor | Key operations; admin login events; key lifecycle events (create, delete, disable, rotate) | Vendor-specific; typically timestamp, key ID, operation, operator identity, result |

## Configure Azure Monitor diagnostic settings for Managed HSM

Managed HSM external key management operations flow through the same Azure Monitor diagnostic pipeline as other Managed HSM operations. If you've already configured diagnostic settings for your Managed HSM instance, external key management log entries appear there automatically, distinguished by their `OperationName`.

If you haven't configured diagnostic settings yet, follow these steps:

1. In the [Azure portal](https://portal.azure.com), go to your Managed HSM resource.
1. Select **Diagnostic settings** under **Monitoring**.
1. Select **Add diagnostic setting**.
1. Enter a name for the setting.
1. Under **Logs**, select **AuditEvent**.
1. Under **Destination details**, choose one of the following destinations:
   - **Log Analytics workspace** — recommended for querying with Kusto Query Language (KQL) and setting up alerts.
   - **Storage account** — recommended for long-term, low-cost retention.
   - **Event Hub** — recommended for streaming to a SIEM.
1. Select **Save**.

For detailed configuration steps and a description of every log field, see [Managed HSM logging](logging.md).

> [!TIP]
> Send logs to both a Log Analytics workspace (for alerting and querying) and a storage account (for compliance retention) using two separate diagnostic settings on the same resource.

## Query Managed HSM external key management operations with KQL

All EKM Proxy calls produce log entries whose `OperationName` contains `Ekm`. Use the following KQL queries in your Log Analytics workspace to monitor external key management activity.

### View recent external key management activity

Use this query to get a quick view of recent external key management operations across your Managed HSM instance.

```kusto
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.KEYVAULT"
| where OperationName contains "Ekm"
| project TimeGenerated, Resource, OperationName, requestUri_s, ResultType, ResultDescription
| order by TimeGenerated desc
```

### Find failures in the last 24 hours

Use this query to identify error patterns and group them by hour. A non-`Success` `ResultType` indicates a failure that originated in the proxy or external HSM.

```kusto
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.KEYVAULT"
| where OperationName contains "Ekm"
| where ResultType != "Success"
| where TimeGenerated > ago(24h)
| summarize Failures = count() by ResultType, ResultDescription, bin(TimeGenerated, 1h)
| order by TimeGenerated desc
```

### Measure latency distribution

Use this query to track p50, p95, and p99 latency over five-minute windows. If `DurationMs` isn't populated for your workspace, correlate with proxy logs instead — the proxy captures end-to-end latency including the HSM leg that `DurationMs` doesn't include.

```kusto
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.KEYVAULT"
| where OperationName contains "Ekm"
| where TimeGenerated > ago(1h)
| summarize p50 = percentile(DurationMs, 50), p95 = percentile(DurationMs, 95), p99 = percentile(DurationMs, 99) by bin(TimeGenerated, 5m)
```

## Capture and store proxy logs

The EKM Proxy is customer-operated. Microsoft doesn't have access to proxy logs. Your logging strategy depends on how the proxy is deployed.

At minimum, configure your proxy to emit a structured log record for each inbound request. Each record should include:

- Timestamp (UTC, ISO 8601).
- External key identifier of the key being wrapped or unwrapped.
- Algorithm (`A256KW`, `A256KWP`, `RSA-OAEP`, or `RSA-OAEP-256`).
- Payload size in bytes.
- Operation result (success or error code).
- Round-trip latency to the external HSM (milliseconds).
- Client certificate subject or thumbprint.
- EKM Proxy Request Context fields (Request ID, Correlation ID, MHSM Pool Name).

Also log authentication failures (rejected or unrecognized client certificates) and proxy → HSM call outcomes separately so you can distinguish proxy-side errors from HSM-side errors.

Ship proxy logs to your preferred SIEM or log aggregation platform. If your proxy runs in Azure (for example, in Azure Container Apps or Azure Kubernetes Service), you can route logs to the same Log Analytics workspace as your Managed HSM logs for unified querying.

For operational health, expose metrics — latency p50/p95/p99, error rate, and active connection count — through your proxy's health endpoint or a sidecar metrics exporter.

## Capture and store external HSM logs

External HSM logging is vendor-specific. Consult your HSM vendor's documentation for the precise mechanism.

Regardless of vendor, capture the following categories of events:

- **Key operations** — every wrap and unwrap call, including the key ID, algorithm, and result.
- **Admin and login events** — any privileged session that modifies HSM configuration.
- **Key lifecycle events** — key creation, deletion, disabling, and rotation.

Retain HSM audit logs according to your compliance requirements. Many regulatory frameworks require 1–7 years of cryptographic audit log retention. Microsoft can't satisfy those requirements from Managed HSM logs alone because Microsoft doesn't have visibility into HSM-side operations.

## Correlate logs across the three sources

Managed HSM and the customer-operated proxy logs can be correlated using the following fields:

- EKM Proxy Request Context fields (Request ID, Correlation ID, MHSM Pool Name).
- External key identifier — present in the Managed HSM `requestUri_s` field and in the proxy's operation log.
- Timestamp window — match within a ±1 second window, accounting for clock skew between Azure and your proxy host.

## Alerting recommendations

Configure the following alerts on your Log Analytics workspace to catch problems before they affect your workload. All queries apply to the `AzureDiagnostics` table.

| Alert | Condition | Rationale |
|---|---|---|
| **Sustained external key management error rate** | More than 1% of external key management operations fail over a 5-minute window | Indicates a systemic problem with the proxy or HSM rather than isolated transient errors. |
| **Proxy authentication failures** | Any non-zero count of authentication failures in a 5-minute window | A rejected or unrecognized Managed HSM client certificate is always suspicious and warrants immediate investigation. |
| **p95 latency above threshold** | `DurationMs` p95 exceeds 250 ms over a 5-minute window | 250 ms is the Managed HSM timeout budget. Operations approaching this threshold are at risk of failing. |
| **Managed HSM client certificate expiry** | Alert 30 days before the certificate in the external key management connection expires | Expired certificates cause mTLS handshake failures and stop all external-key operations. For certificate rotation steps, see [Configure networking and mTLS for Managed HSM external key management](external-key-management-networking.md). |
| **Zero external key management operations** | No `EkmProxyOperation` entries for more than 15 minutes when operations are expected | Might indicate the proxy is unreachable or the connection is misconfigured. |
| **Consecutive timeout errors** | Three or more `504`-class proxy errors within 1 minute | Timeouts suggest the proxy or HSM is under load or unreachable rather than returning a structured error. |

## Compliance and retention

Managed HSM logs cover Microsoft's side of the operation boundary. They don't capture what happens inside the EKM Proxy or the external HSM. If your compliance framework requires an unbroken audit trail from the key management service through to the physical HSM, you must maintain proxy and HSM logs yourself.

Specifically:

- **Proxy logs** demonstrate that only authorized Managed HSM instances called the proxy and that every operation was recorded.
- **External HSM logs** demonstrate that key material was accessed only through authenticated proxy connections.

Neither log stream is accessible to Microsoft. Retaining and protecting them is your responsibility, as described in [SLA and shared responsibility for Managed HSM external key management](external-key-management-shared-responsibility.md).

Set your retention period to meet the longest applicable compliance requirement across all frameworks you're subject to. Common frameworks require 1–7 years.

## Troubleshooting from logs

This article focuses on observability — what to capture and how to query it. For guidance on diagnosing specific error conditions, mapping error codes to root causes, and remediating failures, see [Troubleshoot Managed HSM external key management](external-key-management-troubleshooting.md).

## Related content

- [Managed HSM logging](logging.md)
- [What is Managed HSM external key management?](external-key-management-overview.md)
- [Managed HSM external key management architecture](external-key-management-architecture.md)
- [SLA and shared responsibility for Managed HSM external key management](external-key-management-shared-responsibility.md)
- [Configure networking and mTLS for Managed HSM external key management](external-key-management-networking.md)
- [Troubleshoot Managed HSM external key management](external-key-management-troubleshooting.md)
