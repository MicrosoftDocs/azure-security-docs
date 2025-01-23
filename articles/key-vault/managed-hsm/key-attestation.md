---
title: Validate Azure Managed HSM keys with key attestation
description: Validate cryptographic key integrity in Azure Managed HSM with key attestation, ensuring compliance and protection against unauthorized access.
services: key-vault
author: msmbaldwin

ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: tutorial
ms.date: 10/10/2024
ms.author: mbaldwin

---

# Validate Azure Managed HSM keys with key attestation

Key attestation is a functionality of Azure Managed HSM. It enables a way to validate the integrity and authenticity of cryptographic keys stored within the hardware security module (HSM). It allows organizations to verify that keys were generated and stored within a trusted, FIPS 140-3 Level 3 certified HSM without ever leaving the FIPS boundary. By providing cryptographic proof that the keys are securely handled, key attestation enhances trust in key management processes, enabling compliance with stringent security standards and regulations. This feature is especially valuable in scenarios where customers need assurance that their keys are protected from unauthorized access, even from cloud providers.

## Prerequisites

- AZ CLI version: azure-cli 2.62.0 or higher.
- API version:   7.6-preview.1 or higher.
- Python version: Python 3.8.10 or higher.
- Permissions: Crypto user of the Managed HSM or a custom role with `getkey` permission.

## Key attestation process

The key attestation process has six steps:

- Downloading the Python scripts from Marvell's website and our GitHub repo
- Retrieving key attestation data from the HSM via Azure CLI
- Extracting attestation blob and certificates
- Verifying the key's authenticity with a certificate chain
- Verifying the attestation data file
- Parsing the attributes of the attestation binary

### Download python scripts

Download the following Python scripts required for key attestation
- verify_attest.py, parse_v2.py from our [HSM vendor Marvell's website](https://www.marvell.com/products/security-solutions/nitrox-hs-adapters/software-key-attestation.html#VerifyingAttestations)
- extract_files.py and validate_certifiate_chains.py from [GitHub](https://github.com/Azure/azure-managed-hsm-key-attestation)

### Get attestation data

Get attestation data for a specific key from the HSM using the Azure CLI [az rest](/cli/azure/reference-index#az-rest) command. The JSON file contains key properties, attestation blob and all certificates required for key attestation.

Usage:

```azurecli
az rest --method get --uri  https://<poolname>.managedhsm.azure.net/keys/<poolname>/<keyname>/<keyversion>?api-version=7.6-preview.1 --resource https://managedhsm.azure.net > attestation.json
```

Example:  

```azurecli
az rest --method get --uri  https://ContosoHSM.managedhsm.azure.net/keys/ContosoHSM/RSA2048key?api-version=7.6-preview.1 --resource https://managedhsm.azure.net? > ContosoHSMattestation.json
```

### Extract attestation blob and certificates

Extract attestation blob and certificates from the JSON file from above step. Asymmetric keys will have two binary files as an output with -pri and -pub suffix. Symmetric keys will just have one binary file -pri suffix.

Usage: `python3 extract_files.py --attestation_json_file <attestation.json> --cert_bundle <cert_file.pem>  --attestation_binary_file <attest_data.dat>`

Example: `python3 extract_files.py --attestation_json_file attestation.json --cert_bundle cert_file.pem  --attestation_binary_file attest_data.dat`

### Verify the key's authenticity with a certificate chain

The python script validate_certificate_chains.py constructs a certificate chain to confirm that the key is signed by Marvell, the HSM vendor's root. Additionally, the script verifies that the key is signed with a Microsoft-signed certificate

Usage: `python3 validate_certificate_chains.py --cert_bundle_file <cert_bundle cert_file.pem>  --marvel_parittion_certificate <marvell_issued_partition_certificate.pem> --microsoft_parittion_certificate <microsoft_issued_partition_certificate.pem>`

Example:

```python
python3 validate_certificate_chains.py --cert_bundle_file cert_bundle cert_file.pem  --marvel_parittion_certificate marvell_certificate.pem --microsoft_parittion_certificate microsoft_certificate.pem
```

### Verify the attestation data file

Python script verify_attest.py enables you to verify the attestation data file (.dat). This needs to be done twice once for Marvell cert and the other for Microsoft Cert.

Usage: `python3 verify_attest.py <partition.cert> <attestation.dat>`

Example:

```python
python3 verify_attest.py marvell_certificate.pem attest_data.dat-pri
python3 verify_attest.py microsoft_certificate.pem attest_data.dat-pri

python3 verify_attest.py marvell_certificate.pem attest_data.dat-pub
python3 verify_attest.py microsoft_certificate.pem attest_data.dat-pub
```

### Parse attributes of the attestation binary

Usage: `python3 parse_v2.py <attest_data.dat>`

Example: `python3 parse_v2.py ContosoHSM_attest_data.dat`

## Next steps

For a getting-started tutorial for an administrator, see [What is Managed HSM?](overview.md).

For more information about usage logging for Managed HSM logging, see [Managed HSM logging](logging.md).
