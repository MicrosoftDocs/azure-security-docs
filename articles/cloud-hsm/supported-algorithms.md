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

Azure Cloud HSM supports various cryptographic algorithms. Only the algorithms, modes/methods, and key lengths/curves/moduli supported by its hardware security modules are listed in this article.

For an overview of the service, see [Overview of Azure Cloud HSM](overview.md). For key management, see [Key management in Azure Cloud HSM](key-management.md). For authentication, see [Authentication in Azure Cloud HSM](authentication.md).

Algorithms Supported
Azure Cloud HSM supports various cryptographic algorithms. Only the algorithms, modes/methods, and key lengths/curves/moduli supported by its hardware security modules are listed in the following tables.

## Supported Non-FIPS Algorithms

Azure Cloud HSM and its hardware security modules support the following Non-FIPS approved algorithms.

| Algorithm | Use/Function |
|-----------|--------------|
| AES (noncompliant) | Key wrap (TR31/TR34/AES-CBC/AES-GCM, wrap/unwrap), DecimalTable/Data/PIN Encryption/Decryption. FF1/FF3-1 Data Encryption/Decryption<br>• AES GCM supports the IV length from 1 byte to 16 bytes |
| DES | Derive unique key per transaction (DUKPT), EMV key derivation, Derive PIN from Offset, Derive Offset from PIN, PIN Verification, PVV generation and Verification, CVV generation and verification, Export Symmetric key/Export Asymmetric key pair using TR31 wrapping, Import/Export using TR34, Import Decimal Table EMV script, EMV ARQC/ARPC, Data/PIN encryption/decryption |
| DES MAC | MAC generation and Verification |
| Double-DES | Derive unique key per transaction (DUKPT), EMV key derivation, Derive PIN from Offset, Derive Offset from PIN, PIN Verification, PVV generation and Verification, CVV generation and verification, Export Symmetric key/Export Asymmetric key pair using TR31 wrapping, Import/Export using TR34, Import Decimal Table, EMV script, EMV ARQC/ARPC, Data/PIN encryption/decryption |
| EC-AES | EC-AES wrap/unwrap (EC BYOK) |
| ECDH KDF | Key derivation using ECDH followed by HMAC/CMAC counter KDF |
| ECDSA (noncompliant) | Key generation, Sign, Verify<br>P192, Secp192k1, brainpoolP160r1, brainpool192r1, K-163 and B-163<br>(SHA-1, SHA-224, SHA-256, SHA-384, SHA-512) |
| EDDSA (noncompliant) | Key generation, Sign, Verify |
| KAS-ECC (noncompliant) | EC Key generation and ECDH<br>Curve25519 (128 bits), Curve448 (224 bits) |
| PBE | Key generation |
| RSA (noncompliant) | TR34 Import, TR34 Export, PIN block decryption, BYOK, Encrypt/Decrypt, Asymmetric key encapsulation and unencapsulation using PKCS#1-v1.5 padding with modulus size 2048 bits, 3072 bits and 4096 bits, Key generation, Sign, Verify (1024-bit) |
| Shamir's Key Share | Key share |
| Triple-DES (noncompliant) | Derive unique key per transaction (DUKPT), EMV key derivation, Derive PIN from Offset, Derive Offset from PIN, PIN Verification, PVV generation and verification, CVV generation and verification, Export Symmetric key/Export Asymmetric key pair using TR31 wrapping, Import/Export using TR34, Import Decimal Table, EMV script, EMV ARQC/ARPC, Data/PIN encryption/decryption |

### Non-FIPS Algorithms with Caveats

| Algorithm | Caveat | Use/Function |
|-----------|--------|--------------|
| AES | Cert A1947, Key unwrapping. Per IG D.G. | Legacy Key unwrap only<br>• ECB mode: Decrypt; 128, 192, and 256-bit<br>• CBC mode: Decrypt; 128, 192, and 256-bit |
| AES | Cert A1948, Key unwrapping. Per IG D.G | Legacy Key unwrap only<br>• ECB mode: Decrypt; 128, 192, and 256-bit<br>• CBC mode: Decrypt; 128, 192, and 256-bit |
| EC Diffie-Hellman with non-NIST recommended curves | Cert A1947, Provides between 112 bits and 256 bits of encryption strength. Per IGs D.F and C.A. | EC-DH<br>Secp224k1(112 bits), Secp256K1 (128 bits)<br>brainpoolP224r1(112 bits),<br>brainpoolP256r1(128 bits),<br>brainpoolP320r1(160 bits),<br>brainpoolP384r1(192 bits),<br>brainpoolP512r1(256 bits)<br>FRP256v1 (128 bits)<br>• Prime order curve, generated as per FIPS 186-4 Section 6.1.1 (SHA-1*, SHA2-224, SHA2-256, SHA2-384, SHA2-512) |
| ECDSA with non-NIST recommended curves | Cert A1947, Provides between 112 bits and 256 bits of encryption strength. Per IG C.A. | EC Key generation, sign<br>Secp256K1 (128 bits)<br>brainpoolP224r1(112 bits),<br>brainpoolP256r1(128 bits),<br>brainpoolP320r1(160 bits),<br>brainpoolP384r1(192 bits),<br>brainpoolP512r1(256 bits)<br>FRP256v1 (128 bits)<br>• Prime order curve, generated as per FIPS 186-4 Section 6.1.1 (SHA-1*, SHA2-224, SHA2-256, SHA2-384, SHA2-512) |
| SHA-1 | No security claimed per IG 2.4.A | Fingerprints |
| Triple-DES SP 800-38B | No security claimed per IG 2.4.A | Fingerprints<br>Key Sizes<br>• 192-bit (Generation, Verify) |

## Supported NIST FIPS Approved Algorithms

Azure Cloud HSM and its hardware security modules support the following NIST FIPS approved algorithms.

> **Important Note**: All symmetric key sizes represent the key strength.

| CAVP Cert | Algorithm and Standard | Mode/Method | Description/ Key Size(s) / Key Strength(s) | Use / Function |
|-----------|------------------------|-------------|-------------------------------------------|----------------|
| A1946 | RSA SigVer (FIPS186-4) | RSA SigVer (FIPS186-4) | RSA 2048 with SHA2-256<br>Uses SHA2-256 (FIPS 180-4) (#A1946) as the underlying digest algorithm | Firmware integrity verification by U-boot |
| A1946 | SHA2-256 (FIPS 180-4) | SHA2 | SHA 256 | Firmware integrity verification by U-boot |
| A1947 | AES-CBC (SP 800-38A) | AES | Encrypt/Decrypt: 128, 192, and 256-bit | Data encryption, decryption, key-wrap and key-unwrap |
| A1947 | AES-CCM (SP 800-38C) | AES | Encrypt/Decrypt: 128, 192, and 256-bit<br>Uses AES-ECB (SP 800-38A) (#A1947) as the underlying block cipher | Authenticated Data encryption, decryption |
| A1947 | AES-CMAC (SP 800-38B) | AES | CMAC generate and verify: 128, 192, and 256-bit<br>Uses AES-CBC (SP 800-38A) (#A1947) as the underlying block cipher | Message authentication code generation and verification |
| A1947 | AES-CTR (SP 800-38A) | AES | Encrypt/Decrypt: 128, 192, and 256-bit<br>Uses AES-ECB (SP 800-38A) (#A1947) as the underlying block cipher | Data encryption, decryption |
| A1947 | AES-ECB (SP 800-38A) | AES | Encrypt/Decrypt: 128, 192, and 256-bit | Data encryption, decryption, key-wrap and key-unwrap |
| A1947 | AES-GCM (SP 800-38D) | AES | Encrypt/Decrypt: 128, 192, and 256-bit<br>Uses AES-ECB (SP 800-38A) (#A1947) as the underlying block cipher | Authenticated Data encryption, decryption, key wrap, and key unwrap |
| A1947 | AES-GMAC (SP 800-38D) | AES | GMAC generation: 128, 192, and 256-bit<br>Uses AES-ECB (SP 800-38A) (#A1947) as the underlying block cipher | Message Authentication |
| A1947 | ECDSA SigGen (FIPS186-4) (CVL) | ECDSA SigGen (FIPS186-4) | SigGen Component:<br>P-224, P-256, P-384, P-521 (SHA-256, 384, and 512)<br>Uses Hash DRBG (SP 800-90Ar1) (#A1947) as underlying Random Generator<br>P-224, P-256, P-384, and P-521 curves providing 112, 128, 192, or 256 bits of encryption strength respectively | Signature generation |
| A1947 | Hash DRBG (SP 800-90Ar1) | Hash DRBG | SHA-512 based with security strength of 256-bit.<br>No prediction resistance<br>Uses SHA2-512 (#A1947) as the underlying digest algorithm | Random number generation for user, internal IVs, and salt |
| A1947 | HMAC-SHA2-256 (FIPS 198-1) | HMAC | HMAC-SHA2-256<br>Uses SHA2-256 (#A1947) as the underlying digest algorithm | MAC generation, verify, KAS and KDF |
| A1947 | HMAC-SHA2-384 (FIPS 198-1) | HMAC | HMAC-SHA2-384<br>Uses SHA2-384 (#A1947) as the underlying digest algorithm | MAC generation, verify, KAS and KDF |
| A1947 | HMAC-SHA2-512 (FIPS 198-1) | HMAC | HMAC-SHA2-512<br>Uses SHA2-512 (#A1947) as the underlying digest algorithm | MAC generation, verify, KAS and KDF |
| A1947 | KAS-ECC-SSC SP800-56Ar3 | KAS | ECC CDH staticUnified: P-224, P-256, P-384, P-521,<br>P-224, P-256, P384 and P-521 providing 112 bits,128 bits. 192 bits and 256 bits of encryption strength respectively<br>Uses Hash DRBG (SP800-90Ar1) (#A1947) as underlying Random Generator for the ECDSA KeyGen key pair<br>Uses ECDSA KeyVer (FIPS 186-4) (#A1948) and ECDSA KeyGen (FIPS 186-4) (#A2393) underlying verification for the key pair<br>Uses SHA2-256 (FIPS 180-4) (#A1947), SHA2-384 (FIPS 180-4) (#A1947) and SHA2-512 (FIPS 180-4) (#A1947) as underlying digest algorithm | Shared secret computation |
| A1947 | KDF TLS (CVL) (SP 800-135r1) | KDF TLS | TLS-KDF (v1.2)<br>v1.2: SHA2-256, SHA2-384, SHA2-512<br>Uses HMAC-SHA2-256 (FIPS 180-4) (#A1947), HMAC-SHA2-384 (FIPS 180-4) (#A1947) and HMAC-SHA2-512 (FIPS 180-4) (#A1947) as underlying MAC algorithm | TLS handshake |
| A1947 | RSA Decryption Primitive (SP 800-56Br2) (CVL) | RSA Decryption Primitive | 2048 bit, 3072 bit and 4096-bit<br>2048, 3072, and 4096-bit modulus providing 112, 128, and 150 bits of encryption strength respectively. | Decryption primitive |
| A1947 | RSA Signature Primitive (CVL) (FIPS 186-4) | RSA Signature Primitive | 2048 bit, 3072 bit and 4096-bit<br>2048, 3072, and 4096-bit modulus providing 112, 128, and 150 bits of encryption strength respectively. | Signature primitive |
| A1947 | SHA-1 (FIPS 180-4) | SHA-1 | SHA-1 | Message digest |
| A1947 | SHA2-256 (FIPS 180-4) | SHA2 | SHA2-256 | Message digest |
| A1947 | SHA2-384 (FIPS 180-4) | SHA2 | SHA2-384 | Message digest |
| A1947 | SHA2-512 (FIPS 180-4) | SHA2 | SHA2-512 | Message digest |
| A1947 | SHA3-224 (FIPS 202) | SHA3 | SHA3-224 | Message digest |
| A1947 | SHA3-256 (FIPS 202) | SHA3 | SHA3-256 | Message digest |
| A1947 | SHA3-384 (FIPS 202) | SHA3 | SHA3-384 | Message digest |
| A1947 | SHA3-512 (FIPS 202) | SHA3 | SHA3-512 | Message digest |
| A1947 | SHAKE-128 (FIPS 202) | SHAKE-128 | SHAKE-128 | Message digest |
| A1947 | SHAKE-256 (FIPS 202) | SHAKE-256 | SHAKE-256 | Message digest |
| A1947 | TDES-CBC (SP 800-38A) | TDES | 3-key Triple-DES decrypt (supports only 192-bit size)<br>*Legacy use only* | Data decryption |
| A1947 | TDES-ECB (SP 800-38A) | TDES | 3-key Triple-DES decrypt (supports only 192-bit size)<br>*Legacy use only* | Data decryption |
| A1948 | AES-CBC (SP 800-38A) | AES | Encrypt/Decrypt, 128 and 256-bit | Data encryption, decryption, and key unwrap |
| A1948 | AES-CMAC (SP 800-38B) | AES | CMAC generate and verify: 128, 192, and 256-bit<br>Uses AES-CBC (SP 800-38A) (#A1948) as the underlying block cipher | MAC generation and verification |
| A1948 | AES-GCM (KTS) (SP800-38D) | SP 800-38D and SP 800-38F. KTS (key wrapping and unwrapping) per IG D.G. | 128, 192, and 256-bit keys providing 128, 192, or 256 bits of encryption strength | Data encryption, decryption, key-wrap, and key-unwrap |
| A1948 | AES-GCM (SP 800-38D) | AES | GCM mode: Authenticated encrypt/decrypt; 128, 192, and 256-bit<br>Uses AES-CBC (SP 800-38A) (#A1948) as the underlying block cipher | Authenticated data encryption and decryption, key unwrap |
| A1948 | AES-KW (KTS) (SP 800-38F) | SP 800-38F. KTS (key wrapping and unwrapping) per IG D.G. | 128, 192 and 256-bit keys providing 128-bit, 192-bit, or 256 bits of encryption strength | Key wrapping/unwrapping |
| A1948 | AES-KW (SP 800-38F) | AES | KW, 128, 192, and 256-bit<br>Uses AES-CBC (#A1948) as underlying block cipher | Key wrapping/unwrapping |
| A1948 | AES-KWP (KTS) (SP 800-38F) | SP 800-38F. KTS (key wrapping and unwrapping) per IG D.G. | 128, 192 and 256-bit keys providing 128-bit, 192-bit, or 256 bits of encryption strength | Key wrapping/unwrapping |
| A1948 | AES-KWP (SP 800-38F) | AES | KWP, 128, 192, and 256-bit<br>Uses AES-CBC (#A1948) as underlying block cipher | Key wrapping/unwrapping |
| A1948 | Counter DRBG (SP 800-90Ar1) | Counter DRBG | AES 256 with df<br>No prediction resistance<br>Uses AES-CBC (SP 800-38A) (#A1948) as the underlying block cipher | Random number generation for user, internal IVs, and salt |
| A1948 | ECDSA KeyGen (FIPS186-4) | Key Gen | Key Gen: P-224, P-256, P-384, P-521<br>Uses Counter DRBG (SP800-90Ar1) (#A1948) as underlying Random Generator | Key generation |
| A1948 | ECDSA KeyVer (FIPS186-4) | ECDSA KeyVer (FIPS186-4) | Key Ver; P-224, P-256, P-384 and P-521 | Key Verification |
| A1948 | ECDSA SigGen (FIPS186-4) | ECDSA SigGen (FIPS186-4) | Signature generation: P-224, P-256, P-384, P-521 (SHA2-256, SHA2-384, SHA2-512)<br>Uses Hash DRBG (SP800-90Ar1) (#A1947) as underlying Random generator<br>Uses SHA2-256 (FIPS 180-4) (#A1948), SHA2-384 (FIPS 180-4) (#A1948) and SHA2-512 (FIPS 180-4) (#A1948) as underlying digest algorithm | Signature Generation |
| A1948 | ECDSA SigVer (FIPS186-4) | ECDSA SigVer(FIPS186-4) | Signature verify: P-224, P-256, P-384, P-521 (SHA-1, SHA2-256, SHA2-384, and SHA2-512)<br>Uses SHA-1 (FIPS 180-4) (#A1948), SHA2-256 (FIPS 180-4) (#A1948) and SHA2-384 (FIPS 180-4) (#A1948) and SHA2-512 (FIPS 180-4) (#A1948) as underlying digest algorithm | Signature Verification |
| A1948 | HMAC-SHA-1 (FIPS 198-1) | HMAC | HMAC-SHA-1<br>Uses SHA-1 (FIPS 180-4) (#A1948) as underlying digest algorithm | MAC generation, Verify and KDF |
| A1948 | HMAC-SHA2-256 (FIPS 198-1) | HMAC | HMAC-SHA2-256<br>Uses SHA2-256 (FIPS 180-4) (#A1948) as underlying digest algorithm | MAC generation, verify, KAS, and KDF |
| A1948 | HMAC-SHA2-384 (FIPS 198-1) | HMAC | HMAC-SHA2-384<br>Uses SHA2-384 (FIPS 180-4) (#A1948) as underlying digest algorithm | MAC generation, verify, KAS, and KDF |
| A1948 | HMAC-SHA2-512 (FIPS 198-1) | HMAC | HMAC-SHA2-512<br>Uses SHA2-512((FIPS 180-4) (#A1948) as underlying digest algorithm | MAC generation, verify, KAS, and KDF |
| A1948 | KAS-ECC (KAS) (SP 800-56Ar3) | SP 800-56Arev3. KAS-ECC per IG D.F Scenario 2 path (2). | P-521 curve providing 256 bits of encryption strength | Cloning protocol |
| A1948 | KAS-ECC SP800-56Ar3 | KAS | ECC CDH Ephemeral Unified P-521 with OneStep KDF (HMAC-SHA2-512)<br>Uses Counter DRBG (#A1948) as underlying Random number<br>Uses ECDSA KeyGen (FIPS186-4) (#A1948) as underlying ECC KeyGen and ECDSA KeyVer (#A1948) as underlying ECDSA Key Verification algorithm.<br>Uses SHA2-512) (FIPS 180-4) as underlying digest algorithm | Cloning protocol |
| A1948 | KAS-ECC-SSC SP800-56Ar3 | KAS | ECC CDH Static Unified: P-224, P-256, P384, P-521, B-233, B-283, B-409, B-571, K-233, K-283, K-409, K-571<br>P-224, P-256, P384 and P-521 providing 112 bits,128 bits. 192 bits and 256 bits of encryption strength respectively<br>Uses Counter DRBG (SP800-90Ar1) (#A1948) as underlying Random Generator for the ECDSA KeyGen key pair<br>Uses ECDSA KeyVer (FIPS 186-4) (#A1948) and ECDSA KeyGen (FIPS 186-4) (#A1948) underlying Verification and generation for the key pair<br>Uses SHA2-256 (FIPS 180-4) (#A1948), SHA2-384 (FIPS 180-4) (#A1948), SHA2-512(FIPS 180-4) (#A1948) as the underlying digest algorithm | Shared secret computation |
| A1948 | KAS-IFC-SSC (SP 800-56Br2) | KAS | RSA-based key exchange scheme KAS1 and KAS2<br>RSA 2048, 3072, and 4096-bit<br>2048, 3072, and 4096-bit modulus providing 112, 128, and 150 bits of encryption strength respectively<br>Uses Counter DRBG (SP800-90Ar1) (#A1948) as underlying Random Generator for the RSA Key Pair Generation<br>Uses HMAC-SHA2-256 (#A1948), HMAC-SHA2-384 (#A1948) and HMAC-SHA2-512 (#A1948) as underlying MAC algorithm<br>Uses RSA KeyGen (FIPS 186-4) [#A1948] as underlying RSA key generation algorithm | PEK and KLK generation and certificate authentication |
| A1948 | KDA HKDF SP800-56Cr1 | KDA HKDF SP800-56Cr1 | HKDF<br>Uses Counter DRBG (SP800-90Ar1) (#A1948) as underlying Random Generator<br>Uses HMAC-SHA2-256 (#A1948), HMAC-SHA2-384 (#A1948) and HMAC-SHA2-512 (#A1948) as underlying MAC algorithm | ECDH key derivation and ECDH key wrap |
| A1948 | KDA OneStep SP800-56Cr1 | KDA OneStep SP800-56Cr1 | Hash-based KDF<br>Uses Counter DRBG (SP800-90Ar1) (#A1948) as underlying Random Generator<br>Uses HMAC-SHA2-256 (#A1948), HMAC-SHA2-384 (#A1948) and HMAC-SHA2-512 (#A1948) as underlying MAC algorithm | ECDH key derivation and ECDH key wrap |
| A1948 | KDA TwoStep SP800-56Cr1 | KDA TwoStep SP800-56Cr1 | HMAC/CMAC-based extract-expand KDFs<br>Uses Counter DRBG (SP800-90Ar1) (#A1948) as underlying Random Generator<br>Uses HMAC-SHA2-256 (#A1948), HMAC-SHA2-384 (#A1948) and HMAC-SHA2-512 (#A1948) as underlying MAC algorithm<br>Uses AES-CBC (#A1948) as underlying Block cipher for AES-CMAC | ECDH key derivation and ECDH key wrap |
| A1948 | KDF ANS 9.63 (CVL) (SP 800-135r1) | KDF ANS 9.63 | SHA-2 224, 256, 384, and 512<br>Uses SHA2-256 (#A1948), SHA2-384 (#A1948) and SHA2-512 (#A1948) as underlying digest algorithm | Key derivation and key agreement schemes |
| A1948 | KDF SP800-108 | KDF SP800-108 | HMAC-SHA-1, SHA2-224, SHA2-256, 384, and 512 KDF<br>Uses HMAC-SHA-1 (#A1948), HMAC-SHA2-256 (#A1948), HMAC-SHA2-384 (#A1948) and HMAC-SHA2-512 (#A1948) as underlying MAC algorithm | Key derivation |
| A1948 | KTS-IFC (KTS) (SP 800-56Br2) | SP 800-56Brev2. KTS-IFC (key encapsulation and unencapsulation) per IG D.G. | 2048, 3072, or 4096-bit modulus providing 112, 128, or 150 bits of encryption strength | Asymmetric key encapsulation and unencapsulation |
| A1948 | KTS-IFC (SP 800-56Br2) | KTS | KTS-OAEP-basic RSA 2048, 3072, and 4096-bit<br>Uses counter DRBG (SP800-90Ar1) (#A1948) as underlying Random Generator<br>Uses HMAC-SHA2-256 (FIPS 180-4) (#A1948), HMAC-SHA2-384 (FIPS 180-4) (#A1948), HMAC-SHA2-512 (FIPS 180-4) (#A1948) as underlying digest algorithm<br>Uses RSA KeyGen (FIPS 186-4) (#A1948) as underlying key generation algorithm | Asymmetric key encapsulation and unencapsulation |
| A1948 | PBKDF (SP 800-132) | PBKDF | HMAC with SHA-1, SHA-2 256, 384, and 512<br>Uses HMAC-SHA-1 (#A1948), HMAC-SHA2-256 (#A1948), HMAC-SHA2-384 (#A1948) and HMAC-SHA2-512 (#A1948) as underlying MAC algorithm | User credentials storage. Master key derived from PBKDF is not used for deriving keys for data encryption/protection, but instead used only for storing passwords as hashes. |
| A1948 | RSA Decryption Primitive (SP 800-56Br2) (CVL) | RSA Decryption Primitive | Modulus sizes: 2048, 3072, and 4096-bit | RSA key transport |
| A1948 | RSA KeyGen (FIPS186-4) | RSA KeyGen (FIPS186-4) | Key generation: 2048, 3072, and 4096-bit<br>Uses Counter DRBG (SP800-90Ar1) (#A1948) as underlying Random Generator | RSA key generation |
| A1948 | RSA SigGen (FIPS186-4) | RSA SigGen (FIPS186-4) | FIPS 186-4 PKCS #1 1.5 and PSS Sig Gen: 2048, 3096, and 4096-bit (SHA-2 256, 384, and 512)<br>Uses SHA2-256 (FIPS 180-4) (#A1948), SHA2-384 (FIPS 180-4) (#A1948), SHA2-512 (#A1948) as underlying digest algorithm<br>2048, 3072, and 4096-bit modulus providing 112, 128, and 150 bits of encryption strength respectively. | Signature generation |
| A1948 | RSA SigVer (FIPS186-4) | RSA SigVer (FIPS186-4) | FIPS 186-4 PKCS #1 1.5 and PSS SigVer: 1024, 2048, 3072, and 4096-bit (SHA-1, SHA-2 256, 384, and 512)<br>Uses SHA-1 (FIPS 180-4)(#A1947) SHA2-256 (FIPS 180-4) (#A1948), SHA2-384 (FIPS 180-4) (#A1948), SHA2-512 (#A1948) as underlying digest algorithm<br>2048, 3072, and 4096-bit modulus providing 112, 128, and 150 bits of encryption strength respectively. | Signature verification<br>User authentication<br>Firmware update verification |
| A1948 | SHA-1 (FIPS 180-4) | SHA-1 | SHA-1 | Digests, HMAC, and KDFs |
| A1948 | SHA2-256 (FIPS 180-4) | SHA2 | SHA2-256 | Digests, HMAC, signature generation, and KDFs |
| A1948 | SHA2-384 (FIPS 180-4) | SHA2 | SHA2-384 | Digests, HMAC, signature generation, and KDFs |
| A1948 | SHA2-512 (FIPS 180-4) | SHA2 | SHA2-512 | Digests, HMAC, signature generation, and KDFs |
| A1948 | TDES-ECB | TDES | Triple-DES 3-key decrypt.<br>*Legacy use only* | Prerequisite for TDES-KW. |
| A1948 | TDES-KW (KTS) (SP 800-38F) | SP 800-38F. KTS (key unwrapping) per IG D.G | 192-bit key providing 112-bit encryption strength<br>*Legacy use only* | Key unwrapping |
| A1948 | TDES-KW (SP 800-38F) | TDES | TKW 3-key DES decrypt<br>Uses TDES-ECB (#A1948) as underlying block cipher<br>*Legacy use only* | Key unwrapping |
| A2393 | ECDSA KeyGen (FIPS186-4) | ECDSA KeyGen (FIPS186-4) | Key Gen: P-224, P-256, P-384, P-521<br>Uses Hash DRBG (SP800-90Ar1) (#A1947) as underlying Random Generator for the ECDSA KeyGen key pair | Key Generation |
| A2393 | KTS-IFC (KTS) (SP 800-56Br2) | SP 800-56Brev2. KTS-IFC (key encapsulation and unencapsulation) per IG D.G. | 2048, 3072, or 4096-bit modulus providing 112, 128, or 150 bits of encryption strength | Asymmetric key encapsulation and unencapsulation in hybrid environment |
| A2393 | KTS-IFC (SP 800-56Br2) | KTS | RSA key wrap and unwrap of symmetric keys in KTS-OAEP-Basic padding. 2048, 3072, 4096-bit modulus<br>Uses Counter DRBG (SP800-90Ar1) (#A1948) as underlying Random Generator<br>Uses HMAC-SHA2-256 (FIPS 180-4) (#A1947), HMAC-SHA2-384 (FIPS 180-4) (#A1947), HMAC-SHA2-512 (FIPS 180-4) (#A1947) as underlying digest algorithm<br>Uses RSA KeyGen (FIPS 186-4) (#A2393) as underlying key generation algorithm | Asymmetric key encapsulation and unencapsulation in hybrid environment |
| A2393 | RSA KeyGen (FIPS186-4) | RSA KeyGen (FIPS186-4) | Key generation: 2048, 3072, and 4096-bit<br>Uses Hash DRBG (SP800-90Ar1) (#A1947) as underlying Random Generator for the ECDSA KeyGen key pair | RSA key generation in hybrid environment |
| KAS-ECC-SSC SP800-56Ar3 (#A1947) and KDF TLS (CVL) (#A1947) | KAS TLS (SP 800-56Ar3) | SP 800-56Arev3. KAS-ECC per IG D.F Scenario 2 path (2) | P-224, P-256, P-384, P-521 curves providing 112, 128, 192, or 256 bits of encryption strength | TLS |
| KAS-ECC-SSC SP800-56Ar3 (#A1948)<br>KDF ANS 9.63 (CVL) (SP 800-135r1) (#A1948) KAS-ECC-SSC | KAS ANS 9.63 (SP 800-56Ar3) | SP 800-56Arev3. KAS-ECC per IG D.F Scenario 2 path (2). | P-224, P-256, P-384, and P-521 curves providing 112, 128, 192, or 256 bits of encryption strength | ECDH key derivation and ECDH-AES key wrap |
| KAS-ECC-SSC SP800-56Ar3 (#A1948), KDA HKDF SP800-56Cr1 (#A1948) | KAS KDA HKDF (SP 800-56Ar3) | SP 800-56Arev3. KAS-ECC per IG D.F Scenario 2 path (2). | P-224, P-256, P-384, and P-521 curves providing 112, 128, 192, or 256 bits of encryption strength | ECDH key derivation and ECDH-AES key wrap |
| KAS-ECC-SSC SP800-56Ar3 (#A1948), KDA OneStep SP800-56Cr1 (#A1948) | KAS KDA ONESTEP (SP 800-56Ar3) | SP 800-56Arev3. KAS-ECC per IG D.F Scenario 2 path (2). | P-224, P-256, P-384, and P-521 curves providing 112, 128, 192, or 256 bits of encryption strength | ECDH key derivation and ECDH-AES key wrap |
| KAS-ECC-SSC SP800-56Ar3 (#A1948), KDA TwoStep SP800-56Cr1 (#A1948) | KAS KDA TWOSTEP (SP 800-56Ar3) | SP 800-56Arev3. KAS-ECC per IG D.F Scenario 2 path (2). | P-224, P-256, P-384, and P-521 curves providing 112, 128, 192, or 256 bits of encryption strength | ECDH key derivation and ECDH-AES key wrap |
| KAS-IFC-SSC (SP 800-56Br2) (#A1948)<br>KDA HKDF SP800-56Cr1 (#A1948) | KAS-IFC HKDF (SP800-56Br2) | SP 800-56Brev2. KAS-IFC per IG D.F Scenario 1 path (2). | 2048-bit, 3072-bit and 4096-bit modulus providing between 112 and 150 bits of encryption strength | PEK and KLK generation and certificate authentication |
| KAS-IFC-SSC (SP 800-56Br2) (#A1948)<br>KDA OneStep SP800-56Cr1 (#A1948) | KAS-IFC OneStep (SP800-56Br2) | SP 800-56Brev2. KAS-IFC per IG D.F Scenario 1 path (2). | 2048-bit, 3072-bit and 4096-bit modulus providing between 112 and 150 bits of encryption strength | PEK and KLK generation and certificate authentication |
| KAS-IFC-SSC (SP 800-56Br2) (#A1948)<br>KDA TwoStep SP800-56Cr1 (#A1948) | KAS-IFC TwoStep (SP800-56Br2) | SP 800-56Brev2. KAS-IFC per IG D.F Scenario 1 path (2). | 2048-bit, 3072-bit and 4096-bit modulus providing between 112 and 150 bits of encryption strength | PEK and KLK generation and certificate authentication |
| N/A | ENT (P) (SP 800-90B) | N/A | SP 800-90B entropy source | Entropy Source |
| Vendor Affirmed | CKG SP 800-133Rev2 | CKG | Please refer to section 2.3.2 Algorithm Specific Information | Cryptographic Key Generation; SP 800-133Rev2 and IG D.H |

## What's next?

- [Overview of Azure Cloud HSM](overview.md)
- [Key management in Azure Cloud HSM](key-management.md)
- [Authentication in Azure Cloud HSM](authentication.md)
- [Back up and restore Azure Cloud HSM resources](backup-restore.md)
- [Network security for Azure Cloud HSM](network-security.md)
- [User management in Azure Cloud HSM](user-management.md)
- [Service limits for Azure Cloud HSM](service-limits.md)
- [Troubleshoot Azure Cloud HSM](troubleshoot.md)
