---
title: SLA and shared responsibility for Azure Key Vault Managed HSM external key management (preview)
description: Understand the SLA tradeoffs and the Microsoft, customer, and HSM vendor responsibility split when you use Managed HSM external key management
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: concept-article
ms.date: 07/06/2026
ms.author: mbaldwin
ai-usage: ai-assisted
#Customer intent: As a security or compliance lead evaluating Managed HSM external key management, I need to understand exactly which SLAs apply and which party is responsible for each part of the system before I commit to using external key management.
---

# SLA and shared responsibility for Managed HSM external key management (preview)

> [!IMPORTANT]
> Managed HSM external key management is in **preview**. Preview features are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature might change before general availability.

Managed HSM external key management extends Managed HSM by keeping the key encryption key (KEK) in customer-controlled infrastructure outside of Azure. That arrangement shifts a meaningful portion of operational responsibility from Microsoft to you or to your HSM vendor.

This article makes those boundaries explicit so that you can decide whether external key management fits your workload before you commit to using it.

## Headline tradeoff

> [!IMPORTANT]
> External keys, the EKM Proxy, and the external HSM are **not covered** by the [Managed HSM SLA](https://azure.microsoft.com/support/legal/sla/key-vault-managed-hsm/v1_0/). Microsoft commits to availability of Managed HSM up to the point at which it calls your EKM Proxy. Everything beyond that point is your responsibility (or your HSM vendor's). For workloads that require Microsoft's SLA, use Managed HSM keys instead.

## What Microsoft is responsible for

Microsoft operates the service infrastructure inside the Managed HSM service boundary. This covers the service itself, not control of your key material:

- **Managed HSM service availability**, covered by the published Managed HSM SLA.
- **Enforcing the authentication and authorization** that you define through your Microsoft Entra tenant and Managed HSM local RBAC.
- **Emitting audit events** for Managed HSM-side external key management operations (`EkmProxyOperation`).
- **Outbound mTLS connection initiation** from Managed HSM to the EKM Proxy.
- **Error transparency**: surfacing proxy and external-HSM errors back to you and to support teams so that root cause is unambiguous.
- **Documentation, samples, and the public EKM Proxy API specification.**

## What you (the customer) are responsible for

You control access to your keys, and you own everything outside the Managed HSM service boundary:

- **Caller authentication and authorization** — the Microsoft Entra identities and Managed HSM local RBAC role assignments that govern who can use your keys, all controlled within your tenant.
- **Managed HSM diagnostic logging** — enabling Managed HSM logging and routing it to Azure Monitor or another destination you choose, including retention.
- Operating and securing the **external HSM** — yourself, or through an operator you designate (such as your HSM vendor or another third party you choose).
- Operating and securing the **EKM Proxy** — yourself, or through an operator you designate (such as your HSM vendor or another third party you choose).
- **Network availability** between Managed HSM and the proxy over its public endpoint.
- **mTLS certificate provisioning, distribution, and rotation** on the proxy side.
- **Performance and capacity planning** for the proxy and external HSM under your workload.
- **Logging and audit at the proxy and HSM**. Azure can't see these logs.
- **Backup, restore, and disaster recovery** for external key material.
- Choosing whether your KEK is **single-region or multi-region**, including geographic redundancy of the proxy and HSM.
- **Compliance attestations** for the customer-operated portions of the system.

## What the proxy and HSM operator are responsible for

You can operate the EKM Proxy and external HSM yourself, or designate an operator (such as your HSM vendor or another third party). Whoever operates these components is responsible for:

- Implementing the EKM Proxy correctly per the EKM Proxy API specification.
- Meeting the **≤ 250 ms** per-request latency budget.
- Supporting the required cryptographic algorithms (`A256KW`, `A256KWP`, `RSA-OAEP`, `RSA-OAEP-256`) and key types (AES-256, RSA 2048 / 3072 / 4096).
- Strict adherence to encoding rules: raw unsigned big-endian bytes, base64url — never PEM, JWK, hex, or decimal strings.
- Providing support and incident response for the proxy and external HSM they operate.

## Shared-responsibility summary

| Area | Microsoft | Customer | Designated operator |
|---|---|---|---|
| Managed HSM service availability | ✅ Owns | | |
| Caller authentication and authorization (Entra identities, local RBAC) | Enforces | ✅ Owns | |
| External KEK material custody | | ✅ Owns | |
| EKM Proxy implementation | | ✅ Owns | When delegated |
| EKM Proxy operation (deploy, scale, patch) | | ✅ Owns | When delegated |
| mTLS certificate lifecycle (Managed HSM client cert) | ✅ Owns | | |
| mTLS certificate lifecycle (proxy server cert and trusted root CA) | | ✅ Owns | When delegated |
| Network path between Managed HSM and proxy | | ✅ Owns | |
| External HSM availability | | ✅ Owns | When delegated |
| Wrap and unwrap latency | | ✅ Owns | When delegated |
| Managed HSM-side logging | Emits audit events | ✅ Configures and retains | |
| Logs (proxy and HSM side) | | ✅ Owns | When delegated |
| Backup and recovery of external keys | | ✅ Owns | |
| Geographic redundancy of external keys | | ✅ Owns | |
| Compliance attestations | ✅ Azure scope | ✅ Customer scope | ✅ Operator scope |
| Incident response | ✅ Managed HSM scope | ✅ Proxy and HSM scope | When delegated |

## SLA implications

The **Managed HSM SLA** continues to apply to Managed HSM key operations and to Managed HSM's role in external key management operations.

There's **no SLA for external keys, the EKM Proxy, or the external HSM.** This is by design and reflects that those components are operated outside of Microsoft's control.

Failures attributable to the proxy or external HSM — timeouts, authentication failures, throttling — are reported with proxy-originated error codes and **don't count against the Managed HSM SLA**.

If your workload requires SLA-backed key availability, use Managed HSM keys.

## Performance expectations

The per-request latency budget through the proxy is **≤ 250 ms**. Managed HSM times out if the proxy doesn't respond within that window.

Microsoft doesn't mask, smooth, or compensate for proxy or HSM latency. Your end-to-end performance reflects the slowest link in the chain.

There are **no dedicated retry strategies** designed to hide proxy or HSM unavailability. Errors propagate to callers.

## Error transparency

Errors that originate in the proxy or external HSM are surfaced with their HTTP status code and message back to Managed HSM and into Azure diagnostic logs.

This makes triage unambiguous: support engineers on both sides of the boundary can tell at a glance whether an issue is on Microsoft's side or yours. In preview, there's no shared correlation ID between Managed HSM and proxy logs — correlate manually by matching the external key identifier, algorithm, and timestamp within a ±1 second window across both sources.

## Pricing

[Standard Managed HSM pricing](https://azure.microsoft.com/pricing/details/key-vault/) applies as usual.

There's **no additional Microsoft surcharge** for using external key management.

You separately bear the cost of the EKM Proxy compute and the external HSM, typically charged by your HSM vendor.

## Support boundaries

Microsoft supports Managed HSM and the published EKM Proxy API specification.

Microsoft doesn't provide a reference proxy implementation as a supported product. Sample code in this documentation is illustrative only.

Raise issues isolated to the proxy or external HSM with your HSM vendor or your operations team.

Microsoft support can help correlate symptoms across the boundary using audit logs, the external key identifier, and timestamp alignment.

## When to choose Managed HSM keys instead

External key management isn't the right choice for every workload. Consider Managed HSM keys when:

- You don't have a regulatory or sovereignty requirement that mandates physical control of key material outside of Microsoft infrastructure.
- You want the Managed HSM SLA to apply to your keys.
- You don't have the operational capacity to run an external HSM and proxy with high availability.
- You need cryptographic operations beyond wrap and unwrap — for example, sign, verify, encrypt, or decrypt.
- You want to use Secure Key Release (SKR).

## Related content

- [What is Managed HSM external key management?](external-key-management-overview.md)
- [Managed HSM external key management architecture](external-key-management-architecture.md)
- [Configure networking and mTLS for Managed HSM external key management](external-key-management-networking.md)
- [Logging and monitoring for Managed HSM external key management](external-key-management-logging-monitoring.md)
- [Troubleshoot Managed HSM external key management](external-key-management-troubleshooting.md)
- [Managed HSM SLA](https://azure.microsoft.com/support/legal/sla/key-vault-managed-hsm/v1_0/)
- [How to choose the right key-management solution](/azure/security/fundamentals/key-management-choose)
