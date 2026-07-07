---
title: External key lifecycle in Azure Key Vault Managed HSM external key management (preview)
description: Learn how external keys are created, referenced, rotated, and deleted in Azure Key Vault Managed HSM external key management, and which key operations are supported
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: concept-article
ms.date: 07/06/2026
ms.author: mbaldwin
ai-usage: ai-assisted
#Customer intent: As a security or platform engineer using Managed HSM external key management, I need to understand the full lifecycle of an external key — creation, supported operations, rotation, deletion — and the constraints (such as external key identifier immutability) so I can plan operations correctly.
---

# External key lifecycle in Managed HSM external key management (preview)

> [!IMPORTANT]
> Managed HSM external key management is in **preview**. Preview features are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature might change before general availability.

## Overview

This article describes the full lifecycle of an external key in Managed HSM external key management: how you create a key reference, which operations are supported, how rotation works, and how to delete keys safely. External key lifecycle requires more planning than the Managed HSM key lifecycle because two independent systems — Managed HSM and your external HSM — must stay in sync. An operation that succeeds on one side but fails on the other can leave your environment in an inconsistent state that affects all wrap and unwrap operations.

## Key reference versus key material

In external key management, the key object in Managed HSM and the key material in your external HSM are separate entities.

| | **Managed HSM** | **External HSM** |
|---|---|---|
| **Stores** | Key reference (external key identifier + metadata + access control) | Key material (the actual KEK) |
| **Backed up by** | Managed HSM backup | Your HSM vendor tooling |
| **Deleted by** | Managed HSM key delete API | HSM vendor tooling |
| **Protected by** | Managed HSM RBAC + soft-delete | HSM vendor controls |

The **external key identifier** is the identifier that ties these two objects together. It's an opaque string that your EKM Proxy understands and uses to route operations to the correct key in your external HSM. Managed HSM stores the external key identifier in the key reference but never holds or processes the key material it points to.

This split has direct implications for backup and disaster recovery:

- **Managed HSM backup** preserves the key reference (the external key identifier, metadata, and access policy). It doesn't back up key material.
- **External HSM backup** is entirely your responsibility. If the external key is lost and you have no backup, wrap and unwrap operations against any Managed HSM key version that references that external key identifier fail permanently.

For the full responsibility breakdown, see [SLA and shared responsibility for Managed HSM external key management](external-key-management-shared-responsibility.md).

## Supported key types and sizes

| Key type | Sizes |
|---|---|
| `RSA` | 2048, 3072, 4096 bits |
| `oct` (AES) | 256 bits |

## Supported and unsupported operations

**Supported operations:**

| Operation | Supported algorithms |
|---|---|
| `wrapKey` | `RSA-OAEP`, `RSA-OAEP-256`, `A256KW`, `A256KWP` |
| `unwrapKey` | `RSA-OAEP`, `RSA-OAEP-256`, `A256KW`, `A256KWP` |

**Unsupported operations (preview):**

- `sign` and `verify`
- `encrypt` and `decrypt` (other than through wrap/unwrap)
- Secure Key Release (SKR)
- Confidential VM (CVM) disk encryption launch scenarios
- Server-side encryption for services that require `sign` or `encrypt` directly

> [!NOTE]
> Services that perform customer-managed key encryption using `wrapKey` and `unwrapKey` — for example, Azure Storage — work with external key management. Services that require direct `encrypt`, `sign`, or SKR don't.

## Create an external key reference

Before you create a key reference in Managed HSM, the key must already exist in your external HSM, and the EKM Proxy must be configured and reachable. The creation sequence is:

1. Create the key in your external HSM using your HSM vendor tooling.
1. Retrieve the external key identifier for that key from your EKM Proxy.
1. Create the key reference in Managed HSM:

   ```azurecli
   az keyvault key create \
     --hsm-name <Managed HSM-name> \
     --name <key-ref-name> \
     --external-key-id <external-key-identifier>
   ```

Managed HSM creates a key object that contains the external key identifier, the key type, and the access policy. It doesn't generate or import any key material.

For a complete walkthrough including external key management connection setup, see [Quickstart: Create your first external key using the Azure CLI](external-key-management-quickstart-cli.md).

> [!IMPORTANT]
> The external key identifier is **immutable** after the key reference is created. You can't change the external key identifier of an existing key version. If you need to point a key at a different external key, create a new key version with the new external key identifier — don't try to update the existing version.

## Authorization

All access control to external keys is enforced by Managed HSM local RBAC, exactly as it is for Managed HSM keys. The `Managed HSM EKM Administrator` role is required to create and manage external key management connections. All other key operations (wrap, unwrap, list, get) use the standard Managed HSM local RBAC roles. For details, see [Access control for Managed HSM](access-control.md).

Common roles for external key operations:

| Role | Purpose |
|---|---|
| `Managed HSM Crypto User` | Create and delete key references. Wrap and unwrap using existing keys. |
| `Managed HSM Crypto Auditor` | Read key metadata. |

The EKM Proxy and external HSM have no Azure identity context. They see only authenticated mTLS wrap and unwrap requests from Managed HSM. The proxy can't read Managed HSM role assignments or determine who originated a request.

For role definitions and assignment commands, see [Access control for Managed HSM](access-control.md).

## Rotation

External key rotation is manual and operator-driven. There's no service-managed auto-rotation for external keys. You must coordinate rotation between your external HSM and Managed HSM.

The rotation sequence is:

1. Generate the new key in your external HSM using your HSM vendor tooling.
1. Retrieve the new external key identifier from your EKM Proxy.
1. Create a new version of the existing Managed HSM key reference, pointing it at the new external key identifier:

   ```azurecli
   az keyvault key create \
     --hsm-name <Managed HSM-name> \
     --name <key-ref-name> \
     --external-key-id <new-external-key-identifier>
   ```

After you create the new version:

- **New wrap operations** use the latest key version automatically.
- **Existing ciphertext** that was wrapped under a previous version can still be unwrapped — the old versions remain active and reachable at their versioned URIs.
- **The key URI** (pool URI + key name, without a version segment) is stable across all versions. Services that reference the unversioned URI automatically pick up the new version for wrap operations.

> [!WARNING]
> Don't delete the old external key until you've confirmed that no ciphertext wrapped under the corresponding Managed HSM key version still needs to be decrypted. Deleting the external key while a Managed HSM key version still references its external key identifier causes all unwrap operations against that version to fail with `KeyNotFound`.

For command syntax and a worked example, see [Quickstart: Create your first external key using the Azure CLI](external-key-management-quickstart-cli.md).

## Backup and restore

**Managed HSM backup** captures the key reference — the external key identifier and key metadata. Use standard Managed HSM backup commands:

```azurecli
az keyvault key backup \
  --hsm-name <Managed HSM-name> \
  --name <key-ref-name> \
  --file <backup-file>
```

Restoring from a Managed HSM backup restores the key reference. If the external key that the external key identifier points to is still available, wrap and unwrap operations resume immediately after restore. If the external key is gone, the restored reference exists in Managed HSM but all operations against it fail.

**External HSM backup** is your responsibility. Back up key material using your HSM vendor tooling, following your vendor's documented procedures. Managed HSM has no visibility into or control over external key material backup.

> [!IMPORTANT]
> Plan your external HSM backup and restore procedures before you go to production. A Managed HSM key reference that points to a lost external key can't be recovered by Microsoft.

## Deletion

Deleting an external key involves two distinct operations on two different systems. Understand which one you're performing before you proceed.

### Delete the key reference (Managed HSM)

Deleting the Managed HSM key reference removes the routing object and access control surface in Managed HSM. It doesn't touch the external HSM or the key material.

```azurecli
az keyvault key delete \
  --hsm-name <Managed HSM-name> \
  --name <key-ref-name>
```

**Soft-delete** is enabled by default on all Managed HSM pools. A deleted key reference moves to the deleted state and is recoverable for the configured retention period (7–90 days).

**Purge protection**, when enabled, prevents permanent deletion until the retention period expires. Enable purge protection on any Managed HSM pool that stores production key references.

To permanently remove the reference after soft-delete:

```azurecli
az keyvault key purge \
  --hsm-name <Managed HSM-name> \
  --name <key-ref-name>
```

### Delete the key material (external HSM)

Deleting the key material in your external HSM is irreversible. Use your HSM vendor tooling and follow your vendor's procedures for key destruction.

> [!WARNING]
> If you delete the external key while a Managed HSM key reference (including soft-deleted references) still points to its external key identifier, all `wrapKey` and `unwrapKey` operations against that reference fail immediately with a `KeyNotFound` error from the proxy. This failure affects every Azure service that uses that key for encryption at rest.

**Recommended deletion sequence:**

1. Identify all Azure services using the key reference (check Azure Monitor logs for recent `wrapKey` and `unwrapKey` activity).
1. Rotate those services to a different key.
1. Confirm no wrap or unwrap activity remains against the key reference.
1. Delete (and optionally purge) the Managed HSM key reference.
1. Delete the key material in your external HSM using your HSM vendor tooling.

## Workload migration to a different key

You can't migrate or convert an external key into another key — Azure Key Vault and Managed HSM don't allow key material export between security boundaries. To switch a workload from an external key to a Managed HSM key (or to a different external key), you migrate the **workload** to a new key, not the key itself. The new key has a different key URI, and every Azure service consuming the source key must be reconfigured to point at the new URI.

For step-by-step instructions, see [Migrate workloads off external keys](external-key-management-migration.md). For the general Azure Key Vault pattern this follows, see [Migrate key workloads](../general/migrate-key-workloads.md).

## Related content

- [What is Managed HSM external key management?](external-key-management-overview.md)
- [Managed HSM external key management architecture](external-key-management-architecture.md)
- [SLA and shared responsibility for Managed HSM external key management](external-key-management-shared-responsibility.md)
- [Quickstart: Create your first external key using the Azure CLI](external-key-management-quickstart-cli.md)
- [Migrate workloads off external keys for Managed HSM external key management](external-key-management-migration.md)
- [Access control for Managed HSM](access-control.md)
- [Troubleshoot Managed HSM external key management](external-key-management-troubleshooting.md)
