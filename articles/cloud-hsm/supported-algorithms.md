---
title: Supported cryptographic algorithms in Azure Cloud HSM
description: Review the cryptographic algorithms, modes, and key sizes supported by Azure Cloud HSM hardware security modules.
author: msmbaldwin
ms.service: azure-cloud-hsm
ms.topic: conceptual
ms.date: 08/14/2025
ms.author: mbaldwin
#customer intent: As a security architect or developer, I want to know which cryptographic algorithms are supported by Azure Cloud HSM so I can plan my application integration and compliance.
---

# Supported cryptographic algorithms in Azure Cloud HSM

Azure Cloud HSM supports a variety of cryptographic algorithms. Only the algorithms, modes/methods, and key lengths/curves/moduli supported by its hardware security modules are listed below.

For an overview of the service, see [Overview of Azure Cloud HSM](overview.md). For key management, see [Key management in Azure Cloud HSM](key-management.md). For authentication, see [Authentication in Azure Cloud HSM](authentication.md).

## Supported non-FIPS algorithms

Azure Cloud HSM and its hardware security modules support the following non-FIPS-approved algorithms:

# Algorithms supported

Azure Cloud HSM supports a variety of cryptographic algorithms. Only the algorithms, modes/methods, and key lengths/curves/moduli supported by its hardware security modules are listed below.

## Supported Non-FIPS Algorithms

Azure Cloud HSM and its hardware security modules support the following non-FIPS-approved algorithms.


| Algorithm                      | Use / Function |
| ------------------------------ | -------------- |
| **AES (non-compliant)**        | Key wrap (TR31/TR34/AES-CBC/AES-GCM, wrap/unwrap), DecimalTable/Data/PIN Encryption/Decryption. AES-GCM supports IV length from 1–16 bytes |
| **DES**                        | DUKPT, EMV key derivation, PIN/Offset derivation, PIN verification, PVV/CVV generation & verification, TR31/TR34 import/export, EMV ARQC/ARPC, data/PIN encryption/decryption |
| **DES MAC**                    | MAC generation & verification |
| **Double-DES**                 | Same functions as DES, with DUKPT and EMV key derivation |
| **EC-AES**                     | EC-AES wrap/unwrap (EC BYOK) |
| **ECDH KDF**                   | Key derivation using ECDH + HMAC/CMAC counter KDF |
| **ECDSA (non-compliant)**      | Key generation, Sign, Verify. Curves: P192, Secp192k1, brainpoolP160r1, brainpool192r1, K-163, B-163 |
| **EDDSA (non-compliant)**      | Key generation, Sign, Verify |
| **KAS-ECC (non-compliant)**    | EC Key generation, ECDH. Curves: Curve25519 (128-bit), Curve448 (224-bit) |
| **PBE**                        | Key generation |
| **RSA (non-compliant)**        | TR34 import/export, PIN block decryption, BYOK, encrypt/decrypt, PKCS#1-v1.5 encapsulation/decapsulation (2048/3072/4096-bit), keygen/sign/verify (1024-bit) |
| **Shamir’s Key Share**         | Key share |
| **Triple-DES (non-compliant)** | Same as DES, with DUKPT and EMV key derivation |

### Algorithm caveats


Some algorithms are non-FIPS due to legacy or curve restrictions:

- **AES** Cert A1947/A1948: Legacy key unwrap only (ECB/CBC decrypt; 128/192/256-bit)
- **EC Diffie-Hellman** (non-NIST curves) – e.g., Secp256k1, brainpoolP-series, FRP256v1
- **ECDSA** (non-NIST curves) – same curves as above
- **SHA-1** – no security claimed (fingerprints only)
- **Triple-DES SP 800-38B** – no security claimed (fingerprints only)

---


## Supported NIST FIPS-approved algorithms

Azure Cloud HSM and its hardware security modules support the following NIST FIPS-approved algorithms:

| Cert      | Algorithm / Standard                                               | Mode / Method                     | Key Size(s) / Strength | Use / Function                        |
| --------- | ------------------------------------------------------------------ | --------------------------------- | ---------------------- | ------------------------------------- |
| **A1946** | RSA SigVer (FIPS 186-4)                                            | RSA 2048 + SHA2-256               | 112-bit strength       | Firmware integrity verification       |
| **A1946** | SHA2-256 (FIPS 180-4)                                              | —                                 | —                      | Firmware integrity verification       |
| **A1947** | AES-CBC, AES-CCM, AES-CMAC, AES-CTR, AES-ECB, AES-GCM, AES-GMAC    | Encrypt/Decrypt, MAC, wrap/unwrap | 128/192/256-bit        | Data protection, key wrap/unwrap, MAC |
| **A1947** | ECDSA SigGen (FIPS 186-4)                                          | P-224/P-256/P-384/P-521           | 112–256-bit            | Signature generation                  |
| **A1947** | Hash DRBG (SP 800-90A)                                             | SHA-512                           | 256-bit                | RNG for keys, IVs, salt               |
| **A1947** | HMAC-SHA2-256/384/512                                              | —                                 | —                      | MAC, KAS, KDF                         |
| **A1947** | KAS-ECC-SSC (SP 800-56A)                                           | P-224/P-256/P-384/P-521           | 112–256-bit            | Shared secret computation             |
| **A1947** | KDF TLS (SP 800-135r1)                                             | TLS v1.2                          | —                      | TLS handshake                         |
| **A1947** | RSA Decryption / Signature Primitives                              | 2048/3072/4096-bit                | 112–150-bit            | Decrypt / sign                        |
| **A1947** | SHA-1, SHA-2 (256/384/512), SHA-3 (224/256/384/512), SHAKE-128/256 | —                                 | —                      | Message digests                       |
| **A1947** | TDES-CBC/ECB (3-key)                                               | 192-bit                           | —                      | Data decryption (legacy)              |

Additional **A1948** certs cover similar AES, ECDSA, HMAC, KAS, KDA, KDF, and KTS modes, including:

- AES-GCM (KTS) / AES-KW / AES-KWP
- Counter DRBG (AES-256)
- ECDSA KeyGen/KeyVer/SigGen/SigVer
- HMAC (SHA-1, SHA-2 variants)
- KAS-ECC (SP 800-56A/B), KAS-IFC (RSA key agreement)
- KDA (HKDF, OneStep, TwoStep)
- KDF (ANS 9.63, SP 800-108)
- KTS-IFC (SP 800-56B)

---

## What's next?

- [Overview of Azure Cloud HSM](overview.md)
- [Key management in Azure Cloud HSM](key-management.md)
- [Authentication in Azure Cloud HSM](authentication.md)
- [Back up and restore Azure Cloud HSM resources](backup-restore.md)
- [Network security for Azure Cloud HSM](network-security.md)
- [User management in Azure Cloud HSM](user-management.md)
- [Service limits for Azure Cloud HSM](service-limits.md)
- [Troubleshoot Azure Cloud HSM](troubleshoot.md)
