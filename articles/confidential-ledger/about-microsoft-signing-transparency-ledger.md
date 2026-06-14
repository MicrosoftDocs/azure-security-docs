---
title: About Microsoft's Signing Transparency Ledger
titleSuffix: Azure Confidential Ledger
description: Learn about Microsoft's Signing Transparency Ledger, a workload of Confidential Computing Ledger that enhances software supply chain security through transparent, auditable signing records.
services: confidential-ledger
author: msmbaldwin
ms.service: azure-confidential-ledger
ms.topic: overview
ai-usage: ai-assisted
ms.date: 06/10/2026
ms.author: mbaldwin
---

# About Microsoft's Signing Transparency Ledger

Microsoft's Signing Transparency is a workload of [Confidential Computing Ledger](index.yml). It's a cloud-managed service that enhances trust and security in software supply chains. The service is generally available; verification through Microsoft's Signing Transparency is currently scoped to specific Microsoft services. The service addresses software supply chain threats that traditional code signing alone can't fully prevent. It builds on the Zero Trust principle of "never trust, always verify." It uses an append-only log to verifiably record each signature, with keys protected in a secure confidential computing enclave. This approach allows organizations and auditors to independently verify cryptographic proof of service releases, improving security and accountability.

The service is built on the same confidential computing platform as [Azure confidential ledger](overview.md), using the [Confidential Consortium Framework](https://microsoft.github.io/CCF/main/) to provide tamper-evident, cryptographically verifiable records. For details about the underlying platform, see [Architecture](architecture.md).

For technical details about the underlying architecture, standards, and cryptographic mechanisms, see [Microsoft's Signing Transparency concepts](microsoft-signing-transparency-concepts.md). To inspect the open-source SCITT application that runs the service, see the [scitt-ccf-ledger repository](https://github.com/microsoft/scitt-ccf-ledger).

## What is Microsoft's Signing Transparency?

Microsoft's Signing Transparency maintains a public, open-source, append-only ledger of software signing events. The service acts as a verifiable notary, creating a permanent record of who signed what and when, with cryptographic proof that prevents secret modifications.

The service uses [COSE envelopes](microsoft-signing-transparency-concepts.md#scitt-standards-and-cose-envelope) compliant with the [SCITT Architecture](https://datatracker.ietf.org/doc/draft-ietf-scitt-architecture/) and [SCITT COSE Receipts](https://datatracker.ietf.org/doc/draft-ietf-cose-merkle-tree-proofs/) standards for supply chain security. All ledger operations use cryptographic keys created in and never leaving secure confidential computing enclaves. The service verifies signer identity against trust policies before recording events and issues tamper-evident cryptographic receipts for independent verification.

## Supported services

Microsoft's Signing Transparency Ledger currently provides transparency for:

- [Microsoft Azure Attestation (MAA)](../attestation/overview.md)
- [Azure Key Vault Managed HSM](../key-vault/managed-hsm/overview.md)
- [Azure confidential ledger](overview.md)
- Microsoft's Signing Transparency

To search the ledger for a specific service's entries, use the service identifier listed in [Service identifiers for supported services](microsoft-signing-transparency-concepts.md#service-identifiers-for-supported-services).

## Independent audit of Microsoft's Signing Transparency

[IOActive](https://ioactive.com/) conducted an independent security assessment of Microsoft's Signing Transparency on Azure (built on CCF) from April through June 2025. The evaluation validated strong implementation security and secure integration, and confirmed the service met its security commitments. For details, see:

- [Code Review & Dynamic Fuzzing of Microsoft's Signing Transparency](https://www.ioactive.com/code-review-dynamic-fuzzing-of-microsofts-signing-transparency/) (IOActive blog post)
- [Microsoft Signing Transparency Service Security Assessment](https://www.ioactive.com/wp-content/uploads/2025/10/Microsoft-Signing-Transparency-Service-Security-Assessment-IOActive-Public-Facing-Report.pdf) (IOActive public-facing report, PDF)

## Key capabilities

| Capability | Description |
|------------|-------------|
| **Countersigning COSE envelopes** | Adds a layer of policy enforcement to signed artifacts by using [COSE (CBOR Object Signing and Encryption)](microsoft-signing-transparency-concepts.md#scitt-standards-and-cose-envelope) envelopes. Any modification to the artifact or signature breaks the countersignature, making tampering detectable. |
| **Immutable Merkle tree ledger** | Records all signing events in an append-only [Merkle tree](microsoft-signing-transparency-concepts.md#merkle-tree-and-ledger-structure) ledger. Each entry is cryptographically linked, preventing alteration or removal. |
| **Receipts for auditing and compliance** | Issues cryptographic [receipts](microsoft-signing-transparency-concepts.md#receipts-and-inclusion-proofs) with inclusion proofs that anyone can verify, supporting compliance audits and independent verification. |

## How it works

Microsoft's Signing Transparency Ledger follows this workflow:

1. A Microsoft developer or build system signs code and submits the COSE_Sign1 envelope to the service.
1. The service verifies the signature and signer identity against its trust policy to ensure the owning service is the one who can publish a new signing event like a Software Bill of Materials (SBOM).
1. The service countersigns the envelope and records the event in the [immutable ledger](microsoft-signing-transparency-concepts.md#merkle-tree-and-ledger-structure).
1. A cryptographic receipt with inclusion proof is generated and returned.
1. Anyone can independently verify the signing event (for example, SBOM) by using the receipt.

For detailed technical information about COSE envelopes, Merkle tree structures, and the cryptographic verification process, see [Microsoft's Signing Transparency Ledger concepts](microsoft-signing-transparency-concepts.md). To learn how to perform verification, see [Verify ledger integrity and inspect entries](microsoft-signing-transparency-verify-signatures.md).

## Why transparency matters in software supply chains

Traditional code signing verifies software comes from a trusted source but doesn't prevent bad actors from using compromised or stolen signing keys to distribute malicious updates. Modern software supply chain attacks exploit this limitation.

Signing Transparency addresses this issue by recording every signature in a tamper-evident, publicly accessible ledger. This approach creates accountability: bad actors with stolen keys can still create valid signatures, but they can't hide their signing activity from detection. Any unexpected signature becomes visible to auditors.

Key security improvements:

- **Detection over prevention**: Even if signing keys are compromised, unauthorized use or missing entries is detectable.
- **Audit capability**: All signing events are permanently recorded and verifiable by any party.
- **Policy enforcement**: Services can verify that all releases went through the specific signing identity channels initiated by the owning service (for example, specific identities are used to sign Microsoft service-specific builds).

## Security and trust benefits

Signing Transparency provides these security capabilities:

- **[Tamper detection](microsoft-signing-transparency-concepts.md#tamper-detection-and-auditability)**: The append-only ledger and Merkle tree structure make unauthorized modifications immediately detectable. Any change to a recorded entry breaks the cryptographic chain.

- **[Independent verification](microsoft-signing-transparency-concepts.md#receipts-and-inclusion-proofs)**: Receipts contain cryptographic proofs that anyone can verify without contacting the service. Verifiers can confirm an artifact's signature was logged at a specific time without trusting the distribution channel.

- **Audit trail**: All signing events are permanently recorded with metadata including signer identity, timestamp, and policy validation results. These records support compliance audits and incident investigations.

- **Key compromise detection**: If an attacker uses a compromised signing key, the unexpected signing event appears in the public ledger. Monitoring tools can alert on anomalous signing patterns.

- **Rollback protection**: Merkle tree receipts include freshness proofs that prevent rollback attacks. Verifiers can confirm they're seeing the latest version, not an older vulnerable release.

- **Hardware and firmware support**: The same COSE envelope and ledger mechanisms work for firmware and hardware components, supporting end-to-end supply chain verification.

## Use cases

Microsoft's Signing Transparency Ledger supports these scenarios:

- **Release verification**: Confirm that production releases went through the authorized signing workflow by checking receipts against the public ledger.
- **Compliance auditing**: Generate audit reports of all signing events for a service or time period, with cryptographic proof of each event.
- **Anomaly detection**: Monitor the ledger for unexpected signing patterns, such as signatures from unknown identities or off-hours activity.
- **Incident response**: When investigating potential supply chain compromises, verify which artifacts were signed and when.
- **Firmware and hardware**: Verify signatures on firmware updates, hardware security modules, and other infrastructure components.

To learn how to verify signatures for supported services, see [Verify ledger integrity and inspect entries](microsoft-signing-transparency-verify-signatures.md).

## Next steps

- [Microsoft's Signing Transparency Ledger concepts](microsoft-signing-transparency-concepts.md)
- [Usage: payloads, claims, and auditing](microsoft-signing-transparency-usage.md)
- [Verify ledger integrity and inspect entries](microsoft-signing-transparency-verify-signatures.md)
