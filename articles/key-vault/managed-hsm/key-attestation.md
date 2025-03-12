---
title: Validate Azure Managed HSM keys with key attestation
description: Validate cryptographic key integrity in Azure Managed HSM with key attestation, ensuring compliance and protection against unauthorized access.
services: key-vault
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: tutorial
ms.date: 03/12/2025
ms.author: mbaldwin

---

# Validate Azure Managed HSM keys with key attestation

Key Attestation is a functionality of Azure Managed HSM. It enables a way to validate the integrity and authenticity of cryptographic keys stored within the hardware security module (HSM). It allows organizations to verify that keys have been generated and stored within a trusted, FIPS 140-3 Level 3 certified HSM without ever leaving the FIPS boundary. By providing cryptographic proof that the keys are securely handled, key attestation enhances trust in key management processes, enabling compliance with stringent security standards and regulations. This feature is especially valuable in scenarios where customers need assurance that their keys are protected from unauthorized access, even from cloud providers.

## Pre-requisites

- **AZ CLI version (2.69.0 or higher)**: Run `az --version` to find the version. If you need to install or upgrade, see [Install the Azure CLI](/cli/azure/install-azure-cli).
- **Python version: (3.13.2 or higher)**: Run `python3 --version` to find the version.
- **Pip3 version: (24.3.1 or higher)**: Run `pip3 --version` to find the version.
- **Permissions:** Crypto user of the Managed HSM or a custom role with getkey permissions

## Download or clone the GitHub repository

Download or clone the GitHub repository with all the files required for key attestation.   [Insert repository name here and link it] from GitHub.

## Set up a virtual environment and install the required Python packages

Set up a virtual environment and install the required Python packages from `requirements.txt`. In this example, we are naming the virtual environment “attestation”. Make sure you are in the repository you downloaded or cloned in step 1.

# [Windows](#tab/windows)

```sh
python3 -m venv attestation
attestation\Scripts\activate.bat
pip3 install -r requirements.txt
```

# [Linux](#tab/linux)

```sh
python3 -m venv attestation
attestation\Scripts\activate
pip3 install -r requirements.txt
```
---

## Get attestation data for a specific key from the HSM

Get attestation data for a specific key from the HSM using the AZ CLI command below. Including key version in the URI is optional. The JSON file contains key properties, attestation blob, and all certificates required for key attestation. In this example, the JSON file is named `attestation.json`.

**Usage:**

```sh
az rest --method get --uri https://<poolname>.managedhsm.azure.net/keys/<keyname>/<keyversion>/attestation?api-version=7.6-preview.1 --resource https://managedhsm.azure.net > <filename>.json
```

**Example:**

```sh
az rest --method get --uri https://contoso.managedhsm.azure.net/keys/contosokey/attestation?api-version=7.6-preview.1 --resource https://managedhsm.azure.net > attestation.json
```

## Validate the attestation data

The Python script `validate_attestation.py` extracts the attestation blob and certificates from the JSON file in the above step. It constructs a certificate chain to confirm that the key is signed by Marvell, the HSM vendor’s root, and additionally verifies that the key is signed with a Microsoft-signed certificate. It will also parse the attributes of the attestation binary and print the results. Symmetric keys will receive both public and private key attestation, whereas asymmetric keys will receive only private key attestation. There is an optional parameter of `--v` or `--verbose` which can be included to view the properties of the certificate chain and additional information on the attributes of the key.

**Usage:**

```sh
python3 validate_attestation.py -af <attestation.json> --v <true/false>
```

**Example:**

```sh
python3 validate_attestation.py -af attestation.json --v true
```

When running in verbose mode, you will see how we establish a certificate chain validation and which certificates are used to verify the integrity of the attestation blob for both Marvell and Microsoft. This includes a Marvell root certificate, which you can download and review is the same we used in our script here: [Insert Marvell link].
