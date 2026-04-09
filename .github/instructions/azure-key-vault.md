---
applyTo: "articles/key-vault/general/**,articles/key-vault/keys/**,articles/key-vault/secrets/**,articles/key-vault/certificates/**,articles/key-vault/index.yml,articles/key-vault/includes/**"
---

# Azure Key Vault (AKV) docset context

## Scope

The Azure Key Vault docset covers the **vault** resource type and its four object types:

| Subdirectory | Object type | Notes |
|---|---|---|
| `general/` | N/A (service-level) | Authentication, networking, RBAC, monitoring, recovery, developer guides |
| `keys/` | Cryptographic keys | Software and HSM-protected keys, BYOK, key rotation, policy grammar |
| `secrets/` | Secrets | Passwords, connection strings, rotation tutorials, dev guides |
| `certificates/` | Certificates | Lifecycle, CA integration, import/export, renewal |

Root-level files (`index.yml`, `policy-reference.md`, `resource-graph-samples.md`, `security-controls-policy.md`) are shared hub/reference pages.

## Relationship to Managed HSM

The `managed-hsm/` subdirectory is **physically** inside `articles/key-vault/` but is **logically a separate, peer service** with a completely distinct engineering and PM team. Treat it as a separate docset:

- **Do not** add cross-references from AKV articles to MHSM unless there's a strong, specific reason.
- **Do not** make bulk changes to AKV that sweep into `managed-hsm/`.
- When scoping operations to "Key Vault docs," **always exclude `managed-hsm/`** unless explicitly told otherwise.
- Long-term goal: fully divorce the two docsets so MHSM has no dependencies on `general/`, `secrets/`, `keys/`, or `certificates/`.

There are currently ~26 cross-references from MHSM into AKV (mostly into `general/` for monitoring and service-limits content). These represent the coupling to be unwound over time.

## History and known debt

The original Key Vault documentation was written primarily around the secrets scenario. When the docset was split into four subdirectories (`general/`, `keys/`, `secrets/`, `certificates/`), **legacy content was pushed into `secrets/`**. As a result, `secrets/` is likely to contain articles that are:

- Out of date or no longer accurate
- Duplicative of content that now lives in `general/`
- Covering deprecated features (e.g., managed storage account keys, retired April 2026)

When auditing or updating `secrets/`, assume that older articles may need retirement rather than refreshing.

## Deprecated features

- **Managed storage account keys**: Fully deprecated. All articles were retired in April 2026 (PR #2707) and redirect to `/azure/storage/common/authorize-data-access`. The modern approach is Microsoft Entra ID + managed identities + Azure RBAC for storage access.
- **Access policies (legacy)**: Vault access policies still work but are legacy. Azure RBAC is the recommended model. Articles covering access policies are marked "(legacy)" in the TOC.
- **nCipher BYOK method**: Deprecated since June 30, 2021. Article retired in April 2026 (PR #2714) and redirects to `hsm-protected-keys-byok`. The current BYOK method supports all vendors listed in `hsm-protected-keys.md`.
- **AppAuthentication .NET library**: Deprecated. Referenced in `general/developers-guide.md`. Modern approach is `Azure.Identity` with `DefaultAzureCredential`.
- **Control Plane API versions before 2026-02-01**: Retiring February 27, 2027. See `general/access-control-default.md`.

## Authorization model

AKV supports two authorization models:
1. **Azure RBAC** (recommended) — control-plane and data-plane access via Azure role assignments
1. **Vault access policies** (legacy) — per-vault policies granting key/secret/certificate permissions

New content should always demonstrate Azure RBAC. Access policy content should be clearly marked as legacy.

## FIPS compliance (source of truth)

The canonical FIPS compliance table is in `keys/about-keys.md` (lines 69-74):

| Key type | Compliance |
|---|---|
| Software-protected (HSM Platform 0) | FIPS 140-2 Level 1 |
| HSM Platform 1 in vaults (Premium SKU) | FIPS 140-2 Level 2 |
| HSM Platform 2 in vaults (Premium SKU) | FIPS 140-3 Level 3 |
| Keys in Managed HSM | FIPS 140-3 Level 3 |

All **new** keys and key versions are created on HSM Platform 2. Any other article referencing FIPS levels should match this table.

## Current SDK versions (as of April 2026)

When auditing quickstarts, verify versions against these current stable releases:

| Language | Keys package | Secrets package | Certs package | Identity package |
|---|---|---|---|---|
| Java (Maven) | azure-security-keyvault-keys **4.10.6** | azure-security-keyvault-secrets **4.10.6** | azure-security-keyvault-certificates **4.10.6** | azure-identity **1.18.2** |
| .NET (NuGet) | Azure.Security.KeyVault.Keys **4.9.0** | Azure.Security.KeyVault.Secrets | Azure.Security.KeyVault.Certificates | Azure.Identity |
| Python (pip) | azure-keyvault-keys **4.11.0** | azure-keyvault-secrets | azure-keyvault-certificates | azure-identity |
| JavaScript (npm) | @azure/keyvault-keys **4.10.0** | @azure/keyvault-secrets | @azure/keyvault-certificates | @azure/identity |
| Go | azkeys **v1.4.0** | azsecrets | azcertificates | azidentity |

## Current API versions

| Context | Version | Notes |
|---|---|---|
| Bicep/ARM (`Microsoft.KeyVault/vaults`) | **2024-11-01** | Replace any 2021-*-preview or 2022-* |
| REST data plane | **7.6** | Replace 7.0–7.3 in URL examples |

## Docset structure and key articles

### `general/` (52 articles)

Core topics by theme:

- **Authentication & RBAC**: `authentication.md`, `rbac-guide.md`, `rbac-access-policy.md`, `rbac-migration.md`, `assign-access-policy.md`, `access-control-default.md`
- **Network security**: `network-security.md`, `access-behind-firewall.md`, `private-link-service.md`, `private-link-diagnostics.md`, `overview-vnet-service-endpoints.md`
- **Monitoring & logging**: `logging.md`, `howto-logging.md`, `monitor-key-vault.md`, `monitor-key-vault-reference.md`, `alert.md`, `event-grid-overview.md`
- **Recovery**: `soft-delete-overview.md`, `key-vault-recovery.md`, `backup.md`
- **Governance**: `azure-policy.md`, `secure-key-vault.md`
- **Developer**: `developers-guide.md`, `client-libraries.md`, `apps-api-keys-secrets.md`
- **Operations**: `move-resourcegroup.md`, `move-subscription.md`, `service-limits.md`, `overview-throttling.md`
- **Reference**: `whats-new.md`, `common-error-codes.md`, `rest-error-codes.md`, `overview-security-worlds.md`

Notable: Many files in general/ have ms.date from April 2025 (~12 months old), especially monitoring, alert, Event Grid, and quickstart articles.

### `keys/` (28 articles, audited April 2026 in PR #2714)

- **Conceptual core**: `about-keys.md` (overview) → `about-keys-details.md` (types/algorithms) → `secure-keys.md` (best practices)
- **BYOK cluster**: `byok-specification.md` → `hsm-protected-keys.md` (overview + vendor table) → `hsm-protected-keys-byok.md` (implementation)
- **Key rotation**: `how-to-configure-key-rotation.md`
- **Reference**: `policy-grammar.md` (secure key release policy grammar)
- **JS developer guides**: 9 articles (get-started + 8 operation-specific). The 04/14/2025 cluster hasn't been refreshed.
- **Quickstarts**: 11 files (portal, CLI, PowerShell, Bicep, ARM, Terraform, Go, Java, .NET, Node, Python)

### `secrets/` (22 articles, partially audited in PR #2712)

- **Conceptual core**: `about-secrets.md` (overview), `secure-secrets.md` (best practices, ai-assisted)
- **Rotation tutorials**: `tutorial-rotation.md` (single-credential), `tutorial-rotation-dual.md` (dual-credential)
- **JS developer guides**: 7 articles (get-started + 6 operation-specific). The 04/14/2025 cluster hasn't been refreshed.
- **Quickstarts**: 11 files (same language coverage as keys/)
- **Special**: `multiline-secrets.md` (handling JSON, certs, RSA keys as multi-line secrets)

Known: This docset inherited legacy content from the original monolithic KV docs. Managed storage articles were retired in PR #2707.

### `certificates/` (21 articles, not yet audited)

- **Conceptual core**: `about-certificates.md` (overview), `certificate-access-control.md` (permissions), `create-certificate.md` (creation methods), `certificate-scenarios.md` (getting started)
- **Lifecycle**: `overview-renew-certificate.md`, `tutorial-rotate-certificates.md`
- **CA integration**: `how-to-integrate-certificate-authority.md` (DigiCert/GlobalSign)
- **Import/Export**: `tutorial-import-certificate.md`, `how-to-export-certificate.md`, `create-certificate-signing-request.md`
- **Security**: `secure-certificates.md` (best practices, ai-assisted)
- **Quickstarts**: 8 files (portal, CLI, PowerShell, .NET, Node, Python, Go, Java)
- **FAQ**: `faq.yml` — **ms.date 05/25/2022, nearly 4 years old**. High priority for refresh or retirement.

## Audit history

| PR | Scope | Date | Summary |
|---|---|---|---|
| #2707 | secrets/, general/ | April 2026 | Retired 4 managed-storage-account articles, added Cloud HSM Event Hub tutorial |
| #2712 | secrets/, general/ | April 2026 | Fixed 13 broken access-policy links, updated Java SDK versions, fixed deprecated CLI property, standardized placeholders, renamed images |
| #2714 | keys/ | April 2026 | Retired nCipher article, fixed FIPS levels, updated Java SDK/API versions, standardized placeholders, added Utimaco link, labeled access policy section legacy |

## Common issues to check during audits

1. **Stale SDK versions** in quickstart files (especially Java — historically lags behind)
1. **Old Bicep/ARM API versions** (2021-*-preview) in template quickstarts
1. **Deprecated CLI properties** (`objectId` → `id`)
1. **Links to consolidated access-policy files** (assign-access-policy-cli/powershell/portal.md → assign-access-policy.md)
1. **FIPS version errors** (140-2 vs 140-3 confusion, especially for Managed HSM)
1. **YOUR-VALUE placeholder style** (should be angle-bracket `<your-value>`)
1. **Past-tense deprecation warnings** ("will not be supported after [past date]")
1. **Missing `updated-for-az` include** on articles using PowerShell
1. **Image case mismatches** (never fix with case-only rename — rename the entire file)
1. **JavaScript developer guide staleness** — the 04/14/2025 cluster across both keys/ and secrets/ hasn't been refreshed

## Redirect file

Key Vault redirects go in `.openpublishing.redirection.key-vault.json` (not the generic `.openpublishing.redirection.json`).

## Article counts (as of April 2026)

- `general/`: 52 articles
- `keys/`: 28 articles (was 29; nCipher retired)
- `secrets/`: 22 articles (was 26; 4 managed-storage retired)
- `certificates/`: 21 articles
- Root: 5 files
