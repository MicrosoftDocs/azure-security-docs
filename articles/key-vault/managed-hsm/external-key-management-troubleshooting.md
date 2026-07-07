---
title: Troubleshoot Azure Key Vault Managed HSM external key management (preview)
description: Diagnose and resolve common Azure Key Vault Managed HSM external key management failures, including connection, authentication, key, and operation errors
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: how-to
ms.date: 07/06/2026
ms.author: mbaldwin
ai-usage: ai-assisted
#Customer intent: As a Managed HSM external key management operator hitting an error, I need a structured troubleshooting guide that maps symptoms to causes and fixes across Managed HSM, the EKM Proxy, and the external HSM.
---

# Troubleshoot Managed HSM external key management (preview)

> [!IMPORTANT]
> Managed HSM external key management is in **preview**. Preview features are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature might change before general availability.

External key management failures can originate in any of three independently operated layers: Azure Key Vault Managed HSM, the customer-operated EKM Proxy, and the customer-operated external HSM. This guide is symptom-driven — find the symptom you're seeing, identify the layer it's coming from, and follow the resolution steps. Pay particular attention to the [external key identifier errors](#key-reference-and-external-key-identifier-errors) section: external key identifier mismatches are among the most common errors in new deployments.

Because the proxy and external HSM are outside Microsoft's operational boundary, you need to inspect logs from all three layers to diagnose most failures. For a full description of the shared-responsibility model, see [SLA and shared responsibility for Managed HSM external key management](external-key-management-shared-responsibility.md).

Use the following table to quickly identify which layer to investigate first based on the symptom you're seeing.

| Symptom | Most likely failing layer |
|---|---|
| `az keyvault ekm-connection check` fails | Managed HSM → Proxy network or TLS |
| `401`/`403` on all operations | Proxy authentication |
| `404 KeyNotFound` | Proxy key registration |
| `400 BadRequest` | Algorithm or payload encoding |
| `408`/`502`/`503` | Proxy → external HSM |
| `429 Throttled` | External HSM or proxy rate limit |
| Intermittent failures with no clear pattern | Proxy or HSM capacity or network stability |

## Error code reference

Use this table when you arrive with a specific HTTP status code or symbolic error name from `AzureDiagnostics`, a proxy log, or a customer report. Each row routes you to the symptom section with the resolution steps.

> [!IMPORTANT]
> Most codes originate at the customer-owned EKM Proxy or external HSM. Managed HSM passes them through unchanged into `AzureDiagnostics` (`ResultDescription` field). The code alone doesn't tell you whether the proxy reached the HSM — always cross-reference proxy logs with the external key identifier and a ±1 second timestamp window.

| Code | Symbolic name | Emitting layer | Fix owner | Most likely cause | Resolution section |
|---|---|---|---|---|---|
| *(no HTTP code)* | mTLS handshake failure | Network / TLS | Customer (proxy cert, DNS, NSG); Microsoft (Managed HSM client cert CA) | Server cert chain mismatch, SAN mismatch, expired cert, DNS failure, firewall block | [Connection and TLS errors](#connection-and-tls-errors) |
| `400` | `BadRequest` | Proxy | Customer | Unsupported algorithm, payload size mismatch, or wrong encoding (must be raw base64url, no padding) | [Algorithm and payload errors](#algorithm-and-payload-errors) |
| `401` | `Unauthorized` | Proxy | Customer (allow-list); Microsoft (Managed HSM client cert rotation) | Managed HSM client certificate isn't in the proxy allow-list — common after a Microsoft-driven cert rotation or a proxy restart | [Authentication errors at the proxy](#authentication-errors-at-the-proxy) |
| `403` | `Forbidden` | Proxy or external HSM | Customer | Proxy or HSM policy denies wrap or unwrap for the calling identity or for specific external key identifiers, or the key is disabled at the proxy | [Authentication errors at the proxy](#authentication-errors-at-the-proxy) |
| `404` | `KeyNotFound` | Proxy or external HSM | Customer | External key identifier mismatch — case, URL encoding, trailing slash — or the key isn't registered at the proxy | [Key reference and external key identifier errors](#key-reference-and-external-key-identifier-errors) |
| `408` | `RequestTimeout` | Proxy | Customer (HSM capacity) | External HSM is saturated and didn't respond within the proxy's internal budget | [Performance and timeout errors](#performance-and-timeout-errors) |
| `429` | `Throttled` | Proxy or external HSM | Customer | HSM or proxy is rate-limiting; respect the `Retry-After` header | [Performance and timeout errors](#performance-and-timeout-errors) |
| `500` | `InternalServerError` | Proxy | Customer | Unexpected proxy failure — check proxy process health and logs | [Performance and timeout errors](#performance-and-timeout-errors) |
| `502` | `BadGateway` | Proxy | Customer | External HSM is unreachable from the proxy, or the HSM returned an error | [Performance and timeout errors](#performance-and-timeout-errors) |
| `503` | `ServiceUnavailable` | Proxy | Customer | Proxy is temporarily unavailable (restart, failover, startup sequence) | [Performance and timeout errors](#performance-and-timeout-errors) |
| *(empty `ResultDescription`)* | Request never reached proxy | Network / TLS | Customer | Connection failed before the proxy received the request | [Connection and TLS errors](#connection-and-tls-errors) |
| *(Managed HSM-side, end-to-end > 250 ms)* | Per-request timeout | Managed HSM (enforcement) | Customer (proxy ↔ HSM placement) | Round-trip exceeded the 250 ms hard budget Managed HSM enforces | [Performance and timeout errors](#performance-and-timeout-errors) |

For the formal definition of each proxy error code and the JSON envelope they're returned in, see [EKM Proxy API reference for Managed HSM external key management](external-key-management-proxy-api-reference.md).

### Layer ownership

Use the **Diagnostic fields** column to know which fields to capture from each layer when you triage a code.

| Layer | Owner | Responsibilities | Surfaces as | Diagnostic fields to capture |
|---|---|---|---|---|
| **Managed HSM** | Microsoft | Forwards external key management calls; enforces the 250 ms per-request timeout; passes proxy errors through to `AzureDiagnostics` unchanged | mTLS failures from the Managed HSM side; per-request timeout | `AzureDiagnostics`: `TimeGenerated`, `OperationName`, `requestUri_s` (contains the external key identifier), `ResultType`, `ResultDescription` (contains HTTP code), `DurationMs`, `CallerIPAddress`, `correlationId_g` |
| **EKM Proxy** | Customer | mTLS termination, allow-list enforcement, request validation, HSM call orchestration, `Retry-After` shaping | `400`, `401`, `403`, `404`, `408`, `429`, `500`, `502`, `503` | Customer-shipped proxy logs: timestamp (UTC ISO 8601), external key identifier, algorithm, payload size, HTTP status or `code`, proxy → HSM round-trip latency, Managed HSM client certificate subject common name |
| **External HSM** | Customer (often via HSM vendor) | Key material, key policy, vendor-specific error semantics | `403`, `404`, `408`, `429`, or `502` *as relayed by the proxy* | Vendor-specific audit log: timestamp, key ID, operation (wrap, unwrap, disable, rotate), operator identity, result |

For the full Azure Monitor field reference and KQL queries, see [Logging and monitoring for Managed HSM external key management](external-key-management-logging-monitoring.md).

## Triage workflow

Start here before diving into a specific symptom section. This four-step sequence narrows the failing layer quickly.

> [!IMPORTANT]
> You need access to Managed HSM `AzureDiagnostics` logs, proxy logs, and external HSM audit logs to complete this triage. If diagnostic settings aren't configured on your Managed HSM, configure them before troubleshooting. See [Logging and monitoring for Managed HSM external key management](external-key-management-logging-monitoring.md).

Before you start, confirm the external key management connection exists and is configured as expected:

```azurecli
az keyvault ekm-connection list --hsm-name <Managed HSM Name>
```

If the connection is missing or the proxy host or port is wrong, recreate it. See [Quickstart: Create your first external key using the Azure CLI](external-key-management-quickstart-cli.md) for the full setup steps.

1. Run the connection check command to verify that Managed HSM can reach the proxy and complete the mTLS handshake:

   ```azurecli
   az keyvault ekm-connection check --hsm-name <Managed HSM Name>
   ```

   A clean output with no errors confirms that the mTLS handshake succeeded and the proxy's `/info` endpoint responded. If this command fails, start with [Connection and TLS errors](#connection-and-tls-errors).

1. Inspect the latest `Ekm`-prefixed entries in `AzureDiagnostics` for the Managed HSM side of the failure. For KQL queries and alert setup, see [Logging and monitoring for Managed HSM external key management](external-key-management-logging-monitoring.md).

   A non-`Success` `ResultType` on an external key management operation means Managed HSM forwarded the request to the proxy and received an error response or no response. The `ResultDescription` field often includes the HTTP status code from the proxy.

1. Inspect the corresponding requests in your proxy logs. Match by external key identifier, algorithm, and timestamp window (±1 second). A request that appears in Managed HSM logs but not in proxy logs points to a network or TLS failure before the proxy received it.

1. If the proxy logs show the request arrived but the operation failed, inspect your external HSM audit logs for the corresponding key operation. A failure at this layer typically surfaces as a `502 BadGateway` from the proxy.

## Connection and TLS errors

These errors appear when Managed HSM can't establish or complete the mTLS connection to the proxy. Run `az keyvault ekm-connection check --hsm-name <Managed HSM Name>` to trigger a live connectivity test and capture the exact error before consulting this table.

| Symptom | Likely cause | Resolution |
|---|---|---|
| `az keyvault ekm-connection check` returns an mTLS handshake failure | The proxy's server certificate doesn't chain to the CA registered on the external key management connection (`--server-ca-certificate`) | Re-run `az keyvault ekm-connection create` or update the connection with the correct root CA in PEM: `az keyvault ekm-connection update --hsm-name <Managed HSM Name> --server-ca-certificate correctca.pem` |
| `az keyvault ekm-connection check` returns `401` or `403` from the proxy | Managed HSM's client certificate isn't in the proxy's allow-list | Run `az keyvault ekm-connection certificate show --hsm-name <Managed HSM Name>` to get the current `subjectCommonName` and `caCertificates`, then add them to the proxy's allow-list. See [Configure networking and mTLS for Managed HSM external key management](external-key-management-networking.md) for allow-list options. |
| DNS resolution failure on connection check | The proxy hostname on the external key management connection isn't resolvable from Managed HSM | Verify the proxy hostname resolves via public DNS. Fix the DNS record or recreate the connection with the correct FQDN. |
| Connection timeout on connection check | A firewall or NSG is blocking traffic from Managed HSM to the proxy port | Open an inbound rule on the proxy's network for TCP 443 (required in preview) and verify the proxy is listening. |
| mTLS handshake fails with a SAN mismatch error | The proxy's server certificate Subject Alternative Name (SAN) doesn't match the hostname in the external key management connection | Confirm the proxy certificate has a SAN entry that matches the FQDN in the external key management connection. Reissue the certificate with the correct SAN, or update the external key management connection to use the hostname the certificate was issued for. |
| mTLS handshake fails after a proxy certificate renewal | The renewed proxy certificate uses a different root CA than the one registered on the external key management connection | Update the external key management connection with the new CA before deploying the new certificate: `az keyvault ekm-connection update --hsm-name <Managed HSM Name> --server-ca-certificate newca.pem`. Always update the connection first to avoid a window where the proxy presents a certificate Managed HSM can't validate. |
| mTLS handshake fails with a certificate-expired error | The proxy's server certificate passed its `notAfter` date | Renew the proxy server certificate. If the new certificate uses the same CA, deploy it to the proxy — no external key management connection update is required. If the CA changes, update the external key management connection first. |

## Authentication errors at the proxy

These errors indicate that the connection reached the proxy but the proxy rejected it during the authentication phase. Check both the proxy allow-list configuration and the Managed HSM client certificate validity.

> [!WARNING]
> Microsoft rotates the Managed HSM client certificate on a regular schedule. Configure your proxy to allow-list by subject common name and CA chain (using the `subjectCommonName` and `caCertificates` values from `az keyvault ekm-connection certificate show`) rather than by certificate fingerprint. Fingerprint-based allow-listing requires manual updates after every rotation and causes all wrap and unwrap operations to fail with `401 Unauthorized` during the gap. See [mTLS client certificate rotation](external-key-management-networking.md#mtls-client-certificate-rotation) for details.

| Symptom | Likely cause | Resolution |
|---|---|---|
| `401 Unauthorized` consistently on all proxy requests | The Managed HSM client certificate has expired or was recently rotated, and the proxy allow-list wasn't updated | Run `az keyvault ekm-connection certificate show --hsm-name <Managed HSM Name>` to get the current `subjectCommonName` and `caCertificates`, then update your proxy's allow-list. See [mTLS client certificate rotation](external-key-management-networking.md#mtls-client-certificate-rotation) for the recommended monitoring process. |
| `403 Forbidden` on wrap or unwrap for specific external key identifiers only | The external HSM policy denies the operation for those keys or for the calling identity | Check the HSM-side policy for the affected keys. Confirm the proxy's service account or partition has the wrap and unwrap permissions granted by your HSM administrator. |
| `401 Unauthorized` started after proxy restart or redeployment | The proxy's allow-list was reset or cleared during the restart | Run `az keyvault ekm-connection certificate show --hsm-name <Managed HSM Name>` to retrieve the current `subjectCommonName` and `caCertificates`, then re-add them to the proxy allow-list. |

## Key reference and external key identifier errors

These errors appear when Managed HSM or the proxy can't locate the external key. External key identifiers are immutable after assignment and case-sensitive; a single character difference causes a `404`.

| Symptom | Likely cause | Resolution |
|---|---|---|
| `KeyNotFound` (`404`) from the proxy | The external key identifier doesn't exist at the proxy or external HSM | Verify the external key identifier was registered at the proxy and that the spelling and URL-safe encoding match exactly what's bound on the Managed HSM key reference. External key identifiers are case-sensitive and must be URL-safe. |
| "Key disabled" error from the proxy | An HSM administrator disabled the external key at the HSM layer | Re-enable the key at the external HSM through your HSM vendor's management console or CLI. Confirm with your HSM administrator before re-enabling. |
| `az keyvault key create --external-key-id` fails | The external key identifier is malformed, already assigned to another key version, or contains characters that aren't URL-safe | Confirm the external key identifier uses only URL-safe characters, isn't already in use on another key version, and matches a key that exists at the proxy. |
| Key wrap or unwrap succeeds in the proxy logs but the calling Azure service reports a key error | The external key identifier stored on the Managed HSM key reference doesn't match the external key identifier the proxy knows about | Compare the external key identifier in the Managed HSM `requestUri_s` log field against the external key identifier registered at the proxy. Any mismatch — including trailing slashes, capitalization, or URL encoding differences — causes a `404`. |

## Algorithm and payload errors

These errors appear when the request reaches the proxy but the algorithm or payload doesn't conform to the EKM Proxy API contract. All `value` fields must use raw base64url encoding with no padding.

| Symptom | Likely cause | Resolution |
|---|---|---|
| `400 BadRequest` with "unsupported algorithm" | The requested algorithm isn't in the proxy's supported set for that key | Use one of the four supported algorithms: `RSA-OAEP`, `RSA-OAEP-256`, `A256KW`, or `A256KWP`. Confirm the key's `metadata` response lists the algorithm you're trying to use. |
| `400 BadRequest` with "payload size mismatch" | For RSA keys, the plaintext or wrapped payload length doesn't equal the key modulus size in bytes | Check that the payload is raw bytes base64url-encoded with no padding. For RSA-2048, the payload must be exactly 256 bytes; for RSA-3072, 384 bytes; for RSA-4096, 512 bytes. |
| `400 BadRequest` with an encoding error | The `value` field is PEM-encoded, hex-encoded, or padded base64 instead of raw base64url | All `value` fields in both request and response must be raw, unsigned, big-endian bytes encoded as base64url (RFC 4648 §5) with no padding. Don't use PEM, DER, JWK, hex, or padded base64. |
| `unwrapKey` returns different plaintext than expected | The caller is assuming `unwrapKey` is non-deterministic, but it's deterministic | `unwrapKey` is deterministic: the same ciphertext, key, and algorithm always produce the same plaintext. `wrapKey` with RSA-OAEP is non-deterministic because it uses random padding. If you're seeing unexpected plaintext, the input ciphertext or key selection is incorrect. |

## Performance and timeout errors

Managed HSM enforces a hard per-request timeout of 250 ms from the start of the mTLS-established request to the last byte of the response. Operations that exceed this budget fail from the perspective of the calling Azure service, even if the proxy and HSM eventually complete them.

| Symptom | Likely cause | Resolution |
|---|---|---|
| End-to-end latency consistently above 250 ms | High proxy → external HSM round-trip latency | Check the proxy → HSM latency field in proxy logs. If it's high, consider co-locating the proxy closer to the external HSM, or evaluating whether the HSM is under load. |
| `429 Throttled` from the proxy | The external HSM or proxy is rate-limiting requests | Respect the `Retry-After` header returned by the proxy and add exponential backoff in the calling service. Review your HSM capacity and request volume. |
| `408 RequestTimeout` from the proxy | The external HSM is saturated or slow and didn't respond within the proxy's internal budget | Investigate external HSM capacity. Review HSM vendor guidance for scaling or partition configuration. |
| `502 BadGateway` from the proxy | The external HSM is unreachable from the proxy | Check external HSM health using your HSM vendor's monitoring tools. Verify network connectivity between the proxy host and the HSM appliance. |
| Intermittent latency spikes during business hours | TLS session-cache misses causing full handshake overhead on each connection | Confirm the proxy is configured to cache TLS sessions and keep connections alive. A full TLS 1.3 handshake adds significant overhead when it occurs on every request. |
| `503 ServiceUnavailable` from the proxy | Proxy is temporarily unavailable, usually during a restart or failover | Retry after a short delay. If `503` persists, check the proxy process health and whether it completed its startup sequence. The proxy shouldn't return `503` for more than a few seconds during a normal restart. |

If you're consistently approaching or exceeding the 250 ms per-request budget, the most effective remediation is to reduce the network distance between the proxy and the external HSM. Consider deploying the proxy on the same local network segment as the HSM rather than routing through a WAN or inter-datacenter link.

## Audit-log diagnostics

Managed HSM logs every EKM Proxy call in `AzureDiagnostics` with an `OperationName` that starts with `Ekm`. These logs cover the Managed HSM layer only — the proxy and external HSM produce separate logs that you're responsible for capturing.

For full KQL query reference, alerting recommendations, and log field definitions, see [Logging and monitoring for Managed HSM external key management](external-key-management-logging-monitoring.md).

To quickly surface failures in the last hour during active investigation, run this query in your Log Analytics workspace:

```kusto
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.KEYVAULT"
| where OperationName contains "Ekm"
| where ResultType != "Success"
| where TimeGenerated > ago(1h)
| project TimeGenerated, OperationName, ResultType, ResultDescription, requestUri_s
| order by TimeGenerated desc
```

A non-`Success` `ResultType` on an external key management operation indicates that the failure originated at or beyond the proxy — Managed HSM itself processed the request successfully up to the point of forwarding it. Cross-reference the `requestUri_s` value (which contains the external key identifier) against proxy logs to pinpoint the failing layer.

After you identify failing entries, look at `ResultDescription` for the HTTP status code the proxy returned. Map that code to the relevant section in this article:

- `4xx` codes (client error) → check [Authentication errors at the proxy](#authentication-errors-at-the-proxy), [Key reference and external key identifier errors](#key-reference-and-external-key-identifier-errors), or [Algorithm and payload errors](#algorithm-and-payload-errors).
- `408` or `5xx` codes (server or gateway error) → check [Performance and timeout errors](#performance-and-timeout-errors).
- Empty `ResultDescription` or no proxy log entry → the request didn't reach the proxy; check [Connection and TLS errors](#connection-and-tls-errors).

> [!TIP]
> Correlate Managed HSM and proxy logs using the EKM Proxy Request Context fields (Request ID, Correlation ID, MHSM Pool Name) that Managed HSM passes to the proxy on every call. You can also match by external key identifier from `requestUri_s` and timestamp within a ±1 second window. See [Correlate logs across the three sources](external-key-management-logging-monitoring.md#correlate-logs-across-the-three-sources).

## Preview known issues

The following issues exist specifically in the preview.

- **Wrap and unwrap only.** Sign, verify, encrypt, decrypt, Secure Key Release (SKR), and Confidential VM launch scenarios fail with an operation-not-supported error. Only `wrapKey` and `unwrapKey` are supported in preview. Don't configure external key management keys for workloads that depend on operations outside this set.
- **Subscription-level enablement required.** External key management must be enabled on your subscription by your Microsoft account team before you can create an external key management connection. If it isn't enabled, connection creation fails with a "feature not enabled" error. Contact your Microsoft account team to request enablement. This is a one-time step per subscription.
- **Azure portal support is limited.** Not all external key management operations are available in the **Azure portal** during preview. The Azure CLI is the canonical path for all external key management configuration tasks. Use `az keyvault ekm-connection` subcommands for connection management.
- **Log correlation.** Cross-source log correlation between Managed HSM and the proxy uses the EKM Proxy Request Context fields (Request ID, Correlation ID, MHSM Pool Name) that Managed HSM passes on every call. See [Correlate logs across the three sources](external-key-management-logging-monitoring.md#correlate-logs-across-the-three-sources).

## Get help

Before filing a support ticket, review [SLA and shared responsibility for Managed HSM external key management](external-key-management-shared-responsibility.md) to confirm which layer owns the failure. The EKM Proxy and external HSM are outside Microsoft's support boundary — failures in those layers are your responsibility or your HSM vendor's responsibility.

For failures that are clearly on the Managed HSM side (for example, connection-creation failures unrelated to proxy configuration, or unexpected behavior in the Azure CLI external key management commands), file a support ticket through the [Azure portal](https://portal.azure.com) and include:

- Your Managed HSM name and resource ID.
- The exact Azure CLI command and output, with any sensitive values redacted.
- The relevant `AzureDiagnostics` log entries, including `correlationId_g`.
- The time range of the failure in UTC.
- The output of `az keyvault ekm-connection list --hsm-name <Managed HSM Name>` to show the connection configuration.

For proxy and external HSM issues, contact your HSM vendor directly. Microsoft doesn't have access to proxy logs, HSM audit logs, or HSM configuration, and can't diagnose failures in those components.

> [!NOTE]
> During preview, include "external key management preview" in the support ticket subject line. This routes your ticket to the team managing the preview.

## Related content

- [What is Managed HSM external key management?](external-key-management-overview.md)
- [Managed HSM external key management architecture](external-key-management-architecture.md)
- [SLA and shared responsibility for Managed HSM external key management](external-key-management-shared-responsibility.md)
- [Quickstart: Create your first external key using the Azure CLI](external-key-management-quickstart-cli.md)
- [Configure networking and mTLS for Managed HSM external key management](external-key-management-networking.md)
- [External key lifecycle in Managed HSM external key management](external-key-management-key-lifecycle.md)
- [Migrate workloads off external keys for Managed HSM external key management](external-key-management-migration.md)
- [Logging and monitoring for Managed HSM external key management](external-key-management-logging-monitoring.md)
- [Managed HSM logging](logging.md)
