---
applyTo: "articles/key-vault/managed-hsm/**"
---

# Azure Key Vault Managed HSM (MHSM) docset context

## Scope

The Managed HSM docset covers **Azure Key Vault Managed HSM**, a fully managed, single-tenant, FIPS 140-3 Level 3 validated HSM service. It supports only cryptographic key operations (no secrets or certificates).

Key content areas:
- Provisioning and quickstarts (CLI, PowerShell, ARM template, SDK)
- Key management, rotation, and BYOK import
- Access control (local RBAC, built-in roles, security domain)
- Networking (private endpoints, IP firewall, network security)
- Monitoring and logging (Azure Monitor, Sentinel, alerts)
- Disaster recovery, backup/restore, multi-region replication
- Compliance (key attestation, firmware updates, control-your-data)

## Relationship to Azure Key Vault

Managed HSM is **physically** a subdirectory of `articles/key-vault/` but is **logically a peer service**, not a subservice:

- The MHSM engineering and PM team is **completely separate** from the AKV vault team.
- MHSM shares `ms.service: azure-key-vault` with `ms.subservice: managed-hsm` only for historical/organizational reasons.
- **Long-term goal**: Fully divorce the MHSM docset from AKV so it has zero dependencies on `general/`, `secrets/`, `keys/`, or `certificates/`.

### Current cross-references into AKV (coupling to unwind)

There are currently ~19 references from MHSM articles into AKV content:

| AKV target | MHSM files referencing it | Count | Notes |
|---|---|---|---|
| `general/monitor-key-vault-reference.md` | `logging-azure-monitor.md`, `configure-alerts.md`, `sentinel.md` | ~11 | Heaviest dependency — monitoring data reference |
| `general/` (other) | `index.yml`, `key-management.md`, `role-management.md`, `access-control.md` | ~5 | RBAC guide, developers guide, quickstart comparisons |
| `keys/` | `overview.md`, `mhsm-control-data.md`, `key-management.md` | ~3 | Key types, BYOK, policy grammar |
| `secrets/`, `certificates/` | (none) | 0 | No coupling |

When editing MHSM content, prefer self-contained content or links to Azure-wide docs (e.g., Azure Monitor, Azure RBAC) over links into the AKV docset. Each cross-reference removed moves toward the divorce goal.

### Guidelines

- **Do not** assume MHSM and AKV vault share the same features, limits, or behavior. MHSM is keys-only, uses local RBAC (not vault access policies), and has different service limits.
- **Do not** sweep MHSM into bulk AKV changes unless explicitly requested.
- When told to work on "Key Vault docs," **assume MHSM is excluded** unless specifically mentioned.
- MHSM has its own security baseline, its own soft-delete behavior, and its own network security model.

## Authorization model

MHSM uses **local RBAC only** (not Azure RBAC for data plane, not vault access policies):
- Built-in roles: Managed HSM Administrator, Managed HSM Crypto Officer, Managed HSM Crypto User, and others
- Role assignments are scoped to the HSM instance or individual keys
- Azure Resource Manager authorization can be optionally enabled for management operations (see `authorize-azure-resource-manager.md`)

This is fundamentally different from AKV vault authorization. Do not conflate the two.

## Docset structure and key articles

### Core conceptual articles

- **overview.md** — Product overview: FIPS 140-3, BYOK, single-tenant isolation
- **managed-hsm-technical-details.md** — Deep technical: key sovereignty, TEE architecture, Intel SGX, confidential computing
- **mhsm-control-data.md** — Customer control narrative, encryption models, portfolio positioning vs Standard/Premium vaults
- **access-control.md** — Dual-plane model (control vs data plane), local RBAC fundamentals
- **security-domain.md** — Cryptographic ownership, Shamir Secret Sharing, DR foundation. **Most-linked article** (referenced by 11 files)

### Content by theme

| Theme | Articles |
|---|---|
| **Provisioning** | `quick-create-cli.md`, `quick-create-powershell.md`, `quick-create-template.md` |
| **SDK quickstarts** | `quickstart-dotnet.md` (.NET 6+), `quickstart-python.md` (3.8+), `quickstart-javascript.md` (Node 18+) |
| **Key management** | `key-management.md`, `key-rotation.md`, `key-attestation.md`, `hsm-protected-keys-byok.md` |
| **Access control** | `access-control.md`, `built-in-roles.md`, `role-management.md`, `how-to-secure-access.md`, `authorize-azure-resource-manager.md` |
| **Networking** | `network-security.md`, `configure-network-security.md` (IP firewall, preview), `private-link.md` |
| **Monitoring** | `logging.md` (storage-based), `logging-azure-monitor.md`, `configure-alerts.md`, `sentinel.md`, `azure-policy.md` (preview) |
| **DR & recovery** | `backup-restore.md`, `disaster-recovery-guide.md`, `multi-region-replication.md`, `recovery.md`, `soft-delete-overview.md` |
| **Security** | `secure-managed-hsm.md` (best practices, ai-assisted), `security-domain.md` |
| **Compliance** | `firmware-update.md`, `mhsm-control-data.md`, `key-attestation.md` |
| **Advanced** | `tls-offload-library.md`, `third-party-solutions.md`, `scaling-guidance.md`, `policy-grammar.md` |

### Reusable includes

MHSM uses shared includes from `~/reusable-content/` and local includes:
- `managed-hsm/intro.md` (5 quickstarts)
- `managed-hsm/sdk-prerequisites.md` (3 SDK quickstarts)
- `managed-hsm/cleanup-warning.md` (6 files)
- `managed-hsm/security-domain-storage.md` (2 files)
- `managed-hsm/cli-sign-in.md` (4 files)

## Known limitations and gotchas

These are important constraints documented across the docset:

- **Key version hard limit: 100 per key** — rotation counts toward this; no recovery if limit reached
- **Replication latency: up to 6 minutes** for cross-region writes (multi-region-replication.md)
- **RSA 1024-bit import NOT supported** — 2048+ required
- **Luna HSM RSA 4K** requires firmware 7.4.0+
- **Event Grid notifications NOT supported** for Managed HSM (unlike AKV vaults)
- **IP Network Firewall** is in preview, max 10 CIDR ranges
- **Azure Policy integration** is preview with 30-minute enforcement delay
- **Immutability policies NOT supported** on backup storage accounts

## Known documentation debt

- **Monitoring fragmentation**: Logging split across `logging.md` (storage-based) and `logging-azure-monitor.md` with unclear delineation
- **RBAC scattered across 4 articles**: Users must read access-control + built-in-roles + role-management + how-to-secure-access for full picture
- **BYOK guidance spread** across 4+ files with no central decision flowchart
- **Recovery/soft-delete overlap**: `recovery.md` and `soft-delete-overview.md` cover similar ground
- **REST API versions undocumented**: No explicit api-version strings in articles

## Compliance and hardware

- **FIPS 140-3 Level 3** validated (Marvell LiquidSecurity HSM Adapter)
- **NIST SP 800-57 Part 1** — 2-year key rotation recommendation
- ISO 270x, SOC 1/2/3, FedRAMP-High, PCI
- SLA: 99.99% for multi-region configurations

## Audit history

| PR | Date | Summary |
|---|---|---|
| #2707 | April 2026 | Added Cloud HSM Event Hub tutorial (related service, not MHSM directly) |

## Article count (as of April 2026)

39 markdown articles plus `index.yml` and `toc.yml`
