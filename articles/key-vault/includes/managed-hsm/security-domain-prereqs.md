---
author: msmbaldwin
ms.author: mbaldwin
ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: include
ms.date: 03/13/2026
# Include: Managed HSM security domain prerequisites
---

To activate your HSM, you need:
- To provide a minimum of three RSA key-pairs (up to a maximum of 10)
- To specify the minimum number of keys required to decrypt the security domain (called a *quorum*)

To activate the HSM, you send at least three (maximum 10) RSA public keys to the HSM. The HSM encrypts the security domain with these keys and sends it back. Once this security domain download is successfully completed, your HSM is ready to use. You also need to specify quorum, which is the minimum number of private keys required to decrypt the security domain.

The following example shows how to use `openssl` to generate three self-signed certificates.

```console
openssl req -newkey rsa:2048 -nodes -keyout cert_0.key -x509 -days 365 -out cert_0.cer
openssl req -newkey rsa:2048 -nodes -keyout cert_1.key -x509 -days 365 -out cert_1.cer
openssl req -newkey rsa:2048 -nodes -keyout cert_2.key -x509 -days 365 -out cert_2.cer
```

> [!NOTE]
> Even if the certificate has "expired," it can still be used to restore the security domain.

> [!IMPORTANT]
> Create and store the RSA key pairs and security domain file generated in this step securely.
