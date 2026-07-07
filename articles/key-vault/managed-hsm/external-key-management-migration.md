---
title: Migrate workloads off external keys for Azure Key Vault Managed HSM external key management (preview)
description: How to migrate an Azure workload from a Managed HSM external key to a Managed HSM key, including re-encrypting data encryption keys under the new key
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: how-to
ms.date: 07/06/2026
ms.author: mbaldwin
ai-usage: ai-assisted
#Customer intent: As a Managed HSM operator using external key management, I need to know how to move a workload from an external key to a Managed HSM key (or to a different external key) when key material can't be exported, so I can plan the transition without putting protected data at risk.
---

# Migrate workloads off external keys for Managed HSM external key management (preview)

> [!IMPORTANT]
> Managed HSM external key management is in **preview**. Preview features are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature might change before general availability.

This article describes how to switch an Azure workload from an **external key** in Managed HSM external key management to a **Managed HSM key** (or to a different external key). It's intended for Managed HSM operators who are planning or executing this kind of transition.

## Key migration isn't supported — workload migration is

In external key management, a key's status as either an external key or a Managed HSM key is **immutable by design**: once a key reference is created in either mode, that mode is fixed for the life of that key. External key management blocks transitions in both directions at the API. As a result:

- You **can't convert** an existing external key into a Managed HSM key.
- You **can't convert** an existing Managed HSM key into an external key.
- You **can't transfer** key material between two different external HSMs through Managed HSM. Cross-HSM key-material transfer is an HSM-vendor operation that happens outside Azure.

This design decision exists to prevent accidental policy violations: if the same key URI could be backed by external key material at one point in time and by Managed HSM key material at another, customers and auditors could no longer reason about where a given key has lived. By making this distinction immutable per key, external key management keeps the policy and compliance story for any specific key URI unambiguous for its entire lifetime.

Instead, you migrate the **workload** to a new key. This is the same pattern Azure Key Vault uses for any cross-boundary key transition. For the general guidance, see [Migrate key workloads](../general/migrate-key-workloads.md). The rest of this article applies that pattern to external key management.

> [!IMPORTANT]
> The key URI of the source key is **not** preserved. The destination key has a new key URI, and every Azure service that consumed the old key must be reconfigured to use the new key URI. Plan accordingly.

## When to migrate a workload off an external key

Common reasons to switch a workload from an external key to a Managed HSM key:

- You're decommissioning your external HSM or ending an external key management partnership.
- A regulatory or sovereignty requirement that originally mandated physical control of the key outside Microsoft infrastructure no longer applies.
- You need lower latency or want your keys covered by the [Managed HSM SLA](https://azure.microsoft.com/support/legal/sla/key-vault-managed-hsm/v1_0/).
- You need cryptographic operations that external key management doesn't support — for example, `sign`, `verify`, `encrypt`, `decrypt`, Secure Key Release (SKR), or Confidential VM (CVM) launch scenarios.

You can also use the same workload-migration pattern to move from one external key to another (for example, when changing HSM vendors). The procedure is identical except that the destination key is another external key reference rather than a Managed HSM key.

## Plan the migration

Complete the following preparation steps before you start the cutover.

**Identify all downstream consumers.** Check Azure Monitor diagnostic logs for recent `wrapKey` and `unwrapKey` activity against the source key. Every Azure service that uses this key for encryption at rest is a migration dependency. Coordinate with the service owners — each service must be reconfigured to point at the new key URI, and the reconfiguration steps are service-specific.

**Confirm algorithm and key-size compatibility.** The destination key must use an algorithm that the consuming Azure service supports. If you're moving from an external RSA key to a Managed HSM RSA key, keep the bit length the same to avoid behavior differences in the consuming service. AES-256 (oct, 256) external keys map to AES-256 Managed HSM keys.

**Preserve the source key for backup access.** Don't delete or disable the source external key as part of the cutover. Older backups and snapshots of the consuming service's data are still encrypted under data encryption keys (DEKs) that were wrapped by the source key. Until those backups age out of your retention window or are re-encrypted under the new key, the source key must remain accessible for unwrap.

## Migration procedure

Follow this sequence to migrate a workload from a source key to a destination key. The same sequence applies whether the destination is a Managed HSM key or another external key.

1. **Create the destination key.** Create a new key in your Managed HSM. For key-creation details, see [External key lifecycle in Managed HSM external key management](external-key-management-key-lifecycle.md):
   - **Managed HSM destination:** Run `az keyvault key create` against your Managed HSM pool to create a Managed HSM key with the algorithm and size your workload requires.
   - **External destination:** Create a new external key reference using the [Quickstart: Create your first external key using the Azure CLI](external-key-management-quickstart-cli.md) procedure, pointing at a new external key identifier in your destination external HSM.

   Note the new key URI — configure your downstream Azure services to use this URI.

1. **Grant the workload access to the destination key.** Assign the consuming Azure service's managed identity to the appropriate Managed HSM local RBAC role (typically `Managed HSM Crypto User` or a service-specific equivalent). For role assignment steps, see [Managed HSM access control](access-control.md).

1. **Reconfigure each downstream Azure service to use the destination key.** This step is service-specific; follow the documentation for each service that consumes the source key. For example, see the [Azure Storage CMK reconfiguration steps](/azure/storage/common/customer-managed-keys-configure-existing-account). Most CMK-aware Azure services automatically rewrap their active DEKs under the new key shortly after the reconfiguration completes.

1. **Verify the new key is in use.** Confirm in the consuming service's portal blade or CLI that the new key URI is now the active CMK. Check Azure Monitor diagnostic logs for `wrapKey` and `unwrapKey` activity against the new key URI. Run a test write/read against the protected workload to confirm encryption is functioning end-to-end.

1. **Retain the source key for backup access.** Keep the source external key in place — and keep its EKM Proxy and external HSM operational — for as long as you need to restore from backups that were taken while the source key was active. The source key isn't being used for new wraps after the cutover, but it's still needed to unwrap historical DEKs during a restore.

1. **Decommission the source key when no protected data depends on it.** When all backups encrypted under the source key have aged out of your retention window (or have been re-encrypted under the destination key), you can decommission the source. See [Decommission the source external key](#decommission-the-source-external-key).

## Verify the destination key

Before you decommission the source, verify that the destination key is operating cleanly:

1. Confirm in the consuming Azure service that the new key URI is the active customer-managed key.
1. Confirm in Azure Monitor diagnostic logs that `wrapKey` and `unwrapKey` activity has shifted to the new key URI and that no errors are being recorded.
1. Run a test write/read or a service-specific encryption test (for example, write a new blob to a CMK-protected storage account, then read it back) and confirm it succeeds.
1. Confirm that historical reads — including reads from data written before the cutover — also succeed. Most services rewrap DEKs in the background after a CMK change, but the source key might still be invoked transiently while that completes.

If any of these checks fail, the source key is still intact. You can roll the consuming service back by reconfiguring it to point at the source key URI again.

## Decommission the source external key

> [!WARNING]
> Decommissioning the source external key is irreversible. Any data that's still encrypted under a DEK that was wrapped by the source key — and not yet re-encrypted under the destination key — becomes permanently unrecoverable once the source key is gone. Verify that no backups, snapshots, or replicas depend on the source key before you proceed.

Only decommission the source key after you've confirmed:

- The destination key is fully operational across every consumer.
- All backups, snapshots, and replicas have either aged out of your retention window or been re-encrypted under the destination key.
- No active or pending wrap or unwrap operations reference the source key (verify by reviewing Azure Monitor diagnostic logs for the source key URI over a representative window).

To decommission:

1. Delete (and optionally purge) the Managed HSM key reference for the source external key using `az keyvault key delete` and `az keyvault key purge`.
1. Delete the key material in your external HSM using your HSM vendor's tooling.

## What's not supported

- **Key type is immutable.** External key management blocks transitions between external and Managed HSM keys in both directions at the API. There's no in-place conversion in either direction. Both transitions require [workload migration to a new key](external-key-management-migration.md).
- **Key export.** Azure Key Vault and Managed HSM don't allow key material to be exported. There's no path to extract an external key (or a Managed HSM key) and import it elsewhere.
- **Cross-HSM key-material transfer through Managed HSM.** Moving key material between two different external HSMs is an HSM-vendor operation that happens outside Azure. Managed HSM isn't involved.
- **Cross-region migration of an external key management key.** Cross-region key migration follows standard Managed HSM cross-region practices and isn't external key management-specific. See [Migrate key workloads](../general/migrate-key-workloads.md).

## Related content

- [Migrate key workloads (general guidance)](../general/migrate-key-workloads.md)
- [External key lifecycle in Managed HSM external key management](external-key-management-key-lifecycle.md)
- [Quickstart: Create your first external key using the Azure CLI](external-key-management-quickstart-cli.md)
- [SLA and shared responsibility for Managed HSM external key management](external-key-management-shared-responsibility.md)
- [What is Managed HSM external key management?](external-key-management-overview.md)
