---
title: Azure Key Vault Managed HSM external key management architecture (preview)
description: Learn how Azure Key Vault Managed HSM external key management works end to end, including the control plane, data plane, EKM Proxy, request flow, and connectivity model
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: concept-article
ms.date: 07/06/2026
ms.author: mbaldwin
ai-usage: ai-assisted
#Customer intent: As a security architect, I need to understand how Managed HSM external key management is wired end-to-end so that I can plan deployment, networking, and operations correctly.
---

# Managed HSM external key management architecture (preview)

> [!IMPORTANT]
> Managed HSM external key management is in **preview**. Preview features are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature might change before general availability.

Managed HSM external key management lets you keep cryptographic key material in a hardware security module that your organization operates, while still using Azure Key Vault Managed HSM to enforce access control and serve as the key management surface that Azure services consume. This article describes how every component fits together, from the Azure service that triggers a wrap or unwrap call, through Managed HSM, through the EKM Proxy, and into your external HSM.

If you're a security architect evaluating external key management for a regulated workload, a platform engineer planning the deployment, or an SRE who will own steady-state operations, this article gives you the end-to-end picture you need before you start configuration.

## Architecture at a glance

The following diagram shows the complete request path for an external key management wrap or unwrap operation.

:::image type="content" source="media/external-key-management/external-key-management-architecture.svg" alt-text="High-level diagram showing an Azure service calling Managed HSM, which crosses the Microsoft and customer trust boundary over the EKM Proxy API protected by mTLS to reach an EKM Proxy hosted in the customer's external data center; the EKM Proxy then calls the Key Management Service (external HSM), which performs the wrap or unwrap operation. The key encryption key (KEK) never leaves the external HSM." lightbox="media/external-key-management/external-key-management-architecture.svg":::

The chain of calls is:

`Azure service (CMK) → Managed HSM Service → EKM Proxy (regional, customer-operated) → External HSM (customer-operated)`

Managed HSM owns authorization. The EKM Proxy and external HSM own key material. Microsoft never sees the key encryption key (KEK) that lives in your external HSM.

## Components

The external key management deployment consists of three components, each with distinct roles and responsibilities.

### Managed HSM

Managed HSM is a Microsoft-operated, FIPS 140-3 Level 3 validated service. In an external key management deployment, Managed HSM:

- Stores key references called external key identifiers rather than key material.
- Enforces authentication and authorization using Microsoft Entra ID and Managed HSM local RBAC, identical to the way it handles Managed HSM keys.
- Looks up the external key management connection bound to a pool, then opens an mTLS connection to the EKM Proxy to perform wrap and unwrap operations.
- Emits `EkmProxyOperation` audit log entries for every proxy call.

### EKM Proxy

The EKM Proxy is a customer-operated HTTP+JSON service that implements the EKM Proxy API specification. It mediates between Managed HSM and your external HSM.

Key characteristics:

- **Not trusted by Azure.** The proxy sits outside Azure's trust boundary. Managed HSM treats it as a customer-controlled endpoint. Authorization decisions are never delegated to the proxy.
- **Stateless by design.** The proxy translates requests; it doesn't store key material or session state.
- **Regional deployment recommended.** Deploy one proxy instance per region (Americas, Europe, APAC, Middle East and Africa) and co-locate it with, or close to, the Managed HSM pool it serves. This keeps round-trip latency within budget.
- **One-to-many mapping.** One Managed HSM pool maps to exactly one proxy at a time. A single proxy can serve multiple pools. Dedicating a proxy per customer is a recommended operational practice, not a requirement.

The proxy must enforce its own access controls. At minimum, it should validate the Managed HSM client certificate on every inbound mTLS connection and reject requests from unknown issuers.

### External HSM

The external HSM is customer-owned and customer-operated. It holds the actual KEK material. Microsoft never has access to it. The EKM Proxy communicates with the external HSM using a vendor-specific protocol; that protocol is outside the scope of this architecture.

### External key management connection

An external key management connection is a Managed HSM data-plane object that binds a pool to a single proxy. It contains:

- The proxy host and optional path prefix.
- The server CA trust anchor that Managed HSM uses to validate the proxy's TLS certificate.
- The Managed HSM client certificate used for mTLS toward the proxy.

One pool maps to exactly one external key management connection at a time. You can update the connection to point to a different proxy, but the mapping is always one-to-one.

### External key

An external key is a Managed HSM key object whose material lives outside Managed HSM. It's identified by an external key identifier that the proxy understands. Two important constraints apply:

- The external key identifier is **immutable**. You can't change the external key identifier of an existing key version.
- Key rotation requires creating a new key version with a new external key identifier. The key URI (pool URI + key name) remains stable across versions; only the version segment changes.

## Data plane APIs

External key management operations all use the Managed HSM **data plane**. There's no separate Azure Resource Manager (control plane) surface for external key management.

**Data plane admin API** operations require a Managed HSM pool with external key management enabled for your subscription by your Microsoft account team:

- Create, update, and delete external key management connection objects.
- Create external keys (key objects that reference an external key identifier).

**Data plane keys API** operations happen at runtime inside the Managed HSM service:

- Wrap and unwrap calls that Managed HSM proxies to your external HSM.
- Role assignment reads used to authorize every incoming request.

There's no new data-plane API surface for callers. Azure services that use customer-managed keys call the same Managed HSM key URI they use today; external key management is transparent to them.

## Request flow for wrap and unwrap operations

1. An Azure service that uses customer-managed keys (for example, Azure Storage) initiates a wrap or unwrap call against the Managed HSM key URI.
1. Managed HSM authenticates and authorizes the caller using its existing RBAC model. This step is identical to Managed HSM key behavior.
1. Managed HSM identifies the key as **external** (the key object has an external key identifier) and looks up the bound external key management connection.
1. Managed HSM opens an mTLS connection to the EKM Proxy and issues `POST /{ekid}/wrapkey` or `POST /{ekid}/unwrapkey`. Supported cipher suites and the minimum TLS version are defined in the EKM Proxy API specification.
1. The EKM Proxy validates the inbound Managed HSM client certificate against its configured root CA, then translates and forwards the operation to the external HSM using a vendor-specific protocol.
1. The external HSM performs the cryptographic operation and returns the result to the proxy.
1. The proxy returns the result to Managed HSM, which returns it to the calling Azure service.
1. Managed HSM writes an `EkmProxyOperation` entry to its diagnostic logs. Customers and HSM vendors are responsible for proxy-side and HSM-side logs; Microsoft doesn't have access to them.

> [!NOTE]
> Only `wrapKey` and `unwrapKey` reach the EKM Proxy. Read operations such as `GetKey` are answered from key metadata that Managed HSM captures when the external key is created and stores in Managed HSM. These calls don't reach the proxy or the external HSM.

## Latency budget

Every proxy round-trip must complete in **250 ms or less**. Managed HSM times out and returns an error if the proxy doesn't respond within that window.

Plan your network path and HSM capacity to stay within budget:

- Co-locate the proxy with the Managed HSM region.
- Size your HSM cluster to handle peak wrap/unwrap throughput without queuing.

For network topology guidance, see [Configure networking and mTLS for Managed HSM external key management](external-key-management-networking.md).

## Connectivity model

In the initial preview release, Managed HSM reaches the EKM Proxy over the **public internet** on a publicly resolvable FQDN. This is the only supported connectivity model. The connection is secured by mutual TLS (mTLS) rather than network-layer restrictions.

For step-by-step setup instructions, see [Configure networking and mTLS for Managed HSM external key management](external-key-management-networking.md).

## Authentication model

**Caller to Managed HSM:** Standard Microsoft Entra ID token-based authentication, followed by Managed HSM local RBAC authorization. This is unchanged from Managed HSM key behavior.

**Managed HSM to EKM Proxy:** Mutual TLS (mTLS). Managed HSM presents an X.509 client certificate specific to the Managed HSM pool. The proxy presents a server certificate chained to the CA trust anchor configured in the same connection. Both sides must successfully authenticate before any payload is exchanged.

**EKM Proxy to external HSM:** Vendor-defined. The protocol and credentials used between the proxy and the HSM are entirely under customer and vendor control. This leg is outside the scope of the external key management architecture.

## Trust boundary

The following properties define what Azure trusts and what it doesn't:

- The EKM Proxy is **not trusted** by Azure. Azure treats the proxy as a customer-controlled component outside Azure's trust boundary. A compromised proxy can't elevate privileges within Azure or modify Managed HSM authorization decisions.
- All authorization decisions happen **inside Managed HSM**. The proxy has no Azure identity and no access to Managed HSM RBAC. It can't grant or deny access to any Azure resource.
- The proxy must enforce its own perimeter. At minimum, configure the proxy to validate the Managed HSM client certificate's common name and root CA on every inbound connection and reject requests that don't match.
- External key material never leaves your external HSM. Managed HSM passes ciphertext blobs to the proxy, and the proxy passes them to the HSM. The HSM performs the unwrap or wrap inside its tamper-resistant boundary.

## Key lifecycle

External keys have a lifecycle that differs from Managed HSM keys in a few important ways. For full details, see [External key lifecycle in Managed HSM external key management](external-key-management-key-lifecycle.md).

- **Creation by reference.** External key objects are created in Managed HSM by providing an external key identifier. Managed HSM doesn't generate the key material; that's your external HSM's responsibility.
- **Rotation.** Rotate by creating a new key version with a new external key identifier. The external key identifier of an existing version can't be changed.
- **Key type is immutable.** A key's status as either an external key or a Managed HSM key is fixed at creation. External key management blocks transitions between the two types in both directions at the API. To switch a workload between key types, [migrate the workload to a new key](external-key-management-migration.md), which gives the workload a new key URI.

## Observability

**Azure-side logs:** Every proxy call produces an `EkmProxyOperation` record in the Managed HSM diagnostic logs. The record includes the operation type (wrap or unwrap), the external key identifier, the HTTP status code returned by the proxy, and the latency.

**Customer-side logs:** Microsoft doesn't have visibility into the EKM Proxy or the external HSM. Customers and HSM vendors are responsible for logging and retaining proxy and HSM audit trails to meet their compliance requirements.

**Surfaced failures:** Timeouts, authentication failures, certificate expiry, and rate limiting all surface as proxy errors in Managed HSM logs, tagged with the originating HTTP status code from the proxy. For a full list of error codes and remediation steps, see [Troubleshoot Managed HSM external key management](external-key-management-troubleshooting.md).

## Failure modes and resilience

| Failure | Effect on external key management operations | Effect on Managed HSM key operations |
|---|---|---|
| Proxy unreachable | Wrap and unwrap against external keys fail. | None. Managed HSM keys are unaffected. |
| Proxy TLS certificate expired | mTLS handshake fails; wrap and unwrap fail. | None. |
| Managed HSM client certificate expired | mTLS handshake fails; wrap and unwrap fail. | None. |
| External HSM unavailable | Proxy returns an error; wrap and unwrap fail. | None. |
| Rate limiting (429 from proxy) | Wrap and unwrap fail with a throttling error. | None. |

**External key availability is a shared responsibility.** Microsoft maintains the Managed HSM SLA for the service itself. Availability of external keys depends on the proxy and external HSM that you or your HSM vendor operate. Before you go to production, define your target availability for the proxy and HSM, and design accordingly (redundant proxy instances, HSM clusters, and network path diversity).

For the full shared-responsibility matrix, see [SLA and shared responsibility for Managed HSM external key management](external-key-management-shared-responsibility.md).

## Related content

- [What is Managed HSM external key management?](external-key-management-overview.md)
- [SLA and shared responsibility for Managed HSM external key management](external-key-management-shared-responsibility.md)
- [Quickstart: Create your first external key using the Azure CLI](external-key-management-quickstart-cli.md)
- [Configure networking and mTLS for Managed HSM external key management](external-key-management-networking.md)
- [External key lifecycle in Managed HSM external key management](external-key-management-key-lifecycle.md)
- [Troubleshoot Managed HSM external key management](external-key-management-troubleshooting.md)
- [EKM Proxy API reference for Managed HSM external key management](external-key-management-proxy-api-reference.md)
