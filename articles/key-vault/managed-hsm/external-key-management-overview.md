---
title: What is Azure Key Vault Managed HSM external key management? (preview)
description: Learn what Managed HSM external key management is, why and when to use it, which HSM vendors support it, and the limits of the preview
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: overview
ms.date: 07/07/2026
ms.author: mbaldwin
ai-usage: ai-assisted
#Customer intent: As a security architect at an organization with strict sovereignty or regulatory requirements, I want to understand whether Azure Managed HSM external key management is the right key-management option for my workload.
---

# What is Managed HSM external key management? (preview)

> [!IMPORTANT]
> Managed HSM external key management is in **preview**. Preview features are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature might change before general availability.

Managed HSM external key management lets you keep your Key Encryption Key (KEK) in a customer-owned, customer-operated HSM that runs entirely outside Microsoft infrastructure. When an Azure service needs to wrap or unwrap a data encryption key, Managed HSM delegates that operation to your external HSM through a customer-run **EKM Proxy** that you or your HSM vendor operate.

External key management is designed for organizations with strict regulatory or digital-sovereignty requirements that legally or contractually mandate physical control of key material outside Microsoft datacenters. It's a last-resort, gated option — not a general-purpose upgrade to Managed HSM keys.

## Why use external key management?

External key management is the right choice when you have a hard requirement that your key material never physically resides on Microsoft infrastructure. Specific drivers include:

- **Maximum key control.** The KEK never resides in or transits Microsoft infrastructure. Only your hardware performs wrap and unwrap operations.
- **Digital sovereignty and compliance.** External key management meets the most stringent regulatory and sovereignty requirements. Because you physically control the HSM, you can disconnect it at any time to stop all cryptographic operations.
- **Vendor agnosticism.** Microsoft publishes a standard EKM Proxy API. Any HSM vendor can implement a compliant proxy, and you can implement one yourself.
- **Clear responsibility boundary.** Microsoft is responsible for Managed HSM up to and including the service boundary. You and your HSM vendor own everything beyond it: the proxy, the external HSM, networking, availability, and support.
- **No Microsoft surcharge.** External key management doesn't add a per-operation fee on top of standard Managed HSM pricing. You bear the cost of your proxy infrastructure and your HSM vendor licensing.

## When not to use external key management

External key management trades availability, performance, and operational simplicity for physical key control outside Microsoft infrastructure. If you don't have a legal or contractual requirement that mandates that control, use Managed HSM keys instead.

Specific reasons to choose Managed HSM keys over external key management:

- **No service-level agreement (SLA) on external keys.** The [Managed HSM SLA](https://azure.microsoft.com/support/legal/sla/key-vault-managed-hsm/v1_0/) covers Managed HSM keys only. Availability of wrap and unwrap operations for external keys depends entirely on your proxy and HSM.
- **Lower availability and performance.** Every external key operation adds network round-trips to your proxy and HSM. Managed HSM keys are faster and more available by design.
- **Higher operational cost.** You're responsible for provisioning, patching, scaling, and monitoring the proxy and the external HSM.
- **Microsoft provides no proxy or HSM support.** Microsoft doesn't ship, run, or support the EKM Proxy or the external HSM. That responsibility belongs to you or your HSM vendor.
- **Wrap and unwrap only.** External key management supports `wrapKey` and `unwrapKey` operations. Sign, verify, encrypt, decrypt, Secure Key Release (SKR), and Confidential VM launch scenarios are out of scope.

## Comparison: External keys versus Managed HSM keys

| | **External key** | **Managed HSM key** |
|---|---|---|
| **Key material location** | Customer-owned HSM outside Microsoft infrastructure | FIPS 140-3 Level 3 HSM inside Azure |
| **Key ownership and sovereignty** | Customer (or your HSM vendor) — key material resides in your own HSM | Customer — sole control of key material through partition ownership; Microsoft has no access to your keys |
| **SLA** | None (depends on your proxy and HSM) | Covered by [Managed HSM SLA](https://azure.microsoft.com/support/legal/sla/key-vault-managed-hsm/v1_0/) |
| **Latency** | Higher (proxy + HSM round-trip added to each operation) | Lower (no external round-trip) |
| **Supported operations** | `wrapKey`, `unwrapKey` only | Full key operation set |
| **Networking** | You manage: public endpoint to proxy, secured by mutual TLS (mTLS) | Managed by Azure |
| **Logging** | Azure Monitor for Managed HSM; proxy and HSM logs are your responsibility | Azure Monitor, fully integrated |
| **Pricing** | [Standard Managed HSM pricing](https://azure.microsoft.com/pricing/details/key-vault/) plus your proxy and HSM costs | [Standard Managed HSM pricing](https://azure.microsoft.com/pricing/details/key-vault/) |
| **Recommended use case** | Regulatory or sovereignty mandate requiring physical key control outside Microsoft | All other encryption-at-rest workloads |

## How external key management works (high level)

When an Azure service performs an encryption-at-rest operation, Managed HSM identifies whether the key has an external key identifier. If it does, Managed HSM forwards the operation to your EKM Proxy rather than processing it locally. The proxy authenticates with the external HSM over mutual TLS and returns the result to Managed HSM, which then returns it to the caller.

The request flow looks like this:

1. An Azure service (for example, Azure Storage) calls Managed HSM for a `wrapKey` or `unwrapKey` operation.
1. Managed HSM identifies the key as external through its external key identifier and forwards the operation to the customer's EKM Proxy.
1. The EKM Proxy authenticates with the external HSM over mutual TLS and submits the operation.
1. The external HSM performs the operation and returns the result to the proxy.
1. The proxy returns the result to Managed HSM, which returns it to the calling Azure service.

For the full architecture and network topology options, see [Managed HSM external key management architecture](external-key-management-architecture.md).

## HSM vendors supporting external key management (preview)

The following HSM vendors support the Managed HSM external key management API specification.


| Vendor name | Supported products | More information |
|---|---|---|
| Entrust | Entrust Cryptographic Security Platform Key Manager, backed by Entrust nShield HSMs | [Entrust key management solutions](https://www.entrust.com/products/key-management) |
| Eviden | Eviden Proteccio HSM and Eviden KMS | [Eviden Proteccio HSM](https://eviden.com/solutions/cybersecurity/data-encryption/trustway-proteccio-nethsm/) and [Eviden KMS](https://eviden.com/solutions/cybersecurity/data-encryption/trustway-dataprotect/) |
| Fortanix | Data Security Manager | [Fortanix solutions for Microsoft Azure](https://www.fortanix.com/resources/solution-briefs/fortanix-solutions-for-microsoft-azure) |
| Futurex | CryptoHub | [Futurex](https://www.futurex.com/solutions/microsoft-managed-hsm-external-key-management) |
| Securosys SA | Primus HSM family and Securosys Cloud HSM | [Securosys External Key Management Proxy](https://docs.securosys.com/azure-ekm/overview) |
| Thales | Thales Luna HSM | [Advanced encryption with comprehensive Azure key management](https://cpl.thalesgroup.com/encryption/microsoft-azure) |
| Utimaco | Enterprise Secure Key Manager (ESKM) and Enterprise Key Manager as a Service (EKMaaS) | [Utimaco Enterprise Secure Key Manager](https://utimaco.com/key-management/enterprise-secure-key-manager) |

This list is informational. Microsoft makes no representations about the compatibility, availability, or quality of any particular vendor's proxy implementation. Each HSM vendor is responsible for the functionality of its proxy and for supporting its own customers. Evaluate vendors based on the deployment guidance, conformance results, and support commitments that each vendor publishes for its own product.

Microsoft doesn't ship a proxy. Vendors implement the proxy using a public EKM Proxy API. You can also implement your own proxy against the same public API.

## Preview scope and limitations

Keep the following constraints in mind during the preview:

- **Gated access.** External key management must be enabled on your subscription by your Microsoft account team. Contact them to request enablement. To qualify for onboarding and use of external key management, you must have an assigned Microsoft account manager and meet a monetary requirement of USD 10 million ($10M) or greater in overall committed Azure revenue annually.
- **Region.** Available in **all Azure public regions** at preview launch. Sovereign clouds (Azure Government, Azure China) are out of scope for the preview.
- **Supported operations.** Wrap and unwrap only (`wrapKey`, `unwrapKey`).
- **Supported scenarios.** Encryption-at-rest for Azure services that support customer-managed keys (CMK) with Managed HSM.
- **No SLA.** There's no SLA for external keys, the EKM Proxy, or the external HSM.
- **Error transparency.** Failures attributable to the proxy or external HSM are surfaced in Managed HSM logs. These failures don't count against the Managed HSM SLA.
- **External key identifier immutability.** An external key identifier is immutable after assignment. Key rotation is achieved by creating new key versions, not by modifying an existing external key identifier.

## Get started

Start with the overview articles and quickstarts that match your role and tool preference.

- [Managed HSM external key management architecture](external-key-management-architecture.md)
- [SLA and shared responsibility for Managed HSM external key management](external-key-management-shared-responsibility.md)
- [Quickstart: Create your first external key using the Azure CLI](external-key-management-quickstart-cli.md)
- [Quickstart: Create your first external key using the Azure portal](external-key-management-quickstart-portal.md)
- [Configure networking and mTLS for Managed HSM external key management](external-key-management-networking.md)
- [External key lifecycle in Managed HSM external key management](external-key-management-key-lifecycle.md)
- [Logging and monitoring for Managed HSM external key management](external-key-management-logging-monitoring.md)
- [Troubleshoot Managed HSM external key management](external-key-management-troubleshooting.md)
- [Migrate workloads off external keys for Managed HSM external key management](external-key-management-migration.md)
- [Managed HSM external key management frequently asked questions](external-key-management-faq.yml)
- [EKM Proxy API reference for Managed HSM external key management](external-key-management-proxy-api-reference.md)

## Related content

- [Managed HSM overview](overview.md)
- [Access control for Managed HSM](access-control.md)
- [How to choose the right key management solution](/azure/security/fundamentals/key-management-choose)
