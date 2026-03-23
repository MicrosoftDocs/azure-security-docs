---
author: msmbaldwin
ms.author: mbaldwin
ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: include
ms.date: 03/13/2026
# Include: Security domain RSA key pair generation prerequisites
---

To activate your HSM, you need:

- A minimum of three RSA key pairs (maximum 10)
- The minimum number of keys required to decrypt the security domain (called a *quorum*)

You send at least three (maximum 10) RSA public keys to the HSM. The HSM encrypts the security domain with these keys and sends it back. Once the security domain download completes successfully, your HSM is ready to use. You also need to specify the quorum, which is the minimum number of private keys required to decrypt the security domain.

The following example shows how to use `openssl` to generate three self-signed certificates:

```console
openssl req -newkey rsa:2048 -nodes -keyout cert_0.key -x509 -days 365 -out cert_0.cer
openssl req -newkey rsa:2048 -nodes -keyout cert_1.key -x509 -days 365 -out cert_1.cer
openssl req -newkey rsa:2048 -nodes -keyout cert_2.key -x509 -days 365 -out cert_2.cer
```

The certificate expiration date doesn't affect security domain operations—even an "expired" certificate can still be used to restore the security domain.

> [!IMPORTANT]
> These RSA private keys are the root of trust for your Managed HSM. For production environments, generate these keys using an air-gapped system or on-premises HSM, and store them securely. See [Security domain best practices](/azure/key-vault/managed-hsm/security-domain#generating-the-rsa-key-pairs-securely) for detailed guidance.
