---
title: Quickstart - Azure Key Vault Managed HSM client library for Python
description: Learn how to access keys in Azure Managed HSM using the Python client library
author: msmbaldwin
ms.author: mbaldwin
ms.date: 03/13/2026
ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: quickstart
ms.devlang: python
ms.custom: devx-track-python, mode-api
#Customer intent: As a developer, I want to programmatically access keys in Managed HSM using the Python SDK
---

# Quickstart: Azure Key Vault Managed HSM client library for Python

Get started with the Azure Key Vault Managed HSM client library for Python. [!INCLUDE [Managed HSM description](../includes/managed-hsm/intro.md)]

In this quickstart, you learn how to access and perform cryptographic operations on keys in a Managed HSM using the Python client library.

Managed HSM client library resources:

[API reference documentation](/python/api/overview/azure/keyvault-keys-readme) | [Library source code](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/keyvault/azure-keyvault-keys) | [Package (PyPI)](https://pypi.org/project/azure-keyvault-keys/)

## Prerequisites

[!INCLUDE [Managed HSM SDK prerequisites](../includes/managed-hsm/sdk-prereqs.md)]
- [Python 3.8+](https://www.python.org/downloads/)

## Set up your local environment

This quickstart uses the Azure Identity library with Azure CLI to authenticate to Azure services. Developers can also use Visual Studio Code to authenticate their calls. For more information, see [Authenticate the client with Azure Identity client library](/python/api/overview/azure/identity-readme).

### Sign in to Azure

Run the `az login` command to sign in:

```azurecli
az login
```

### Create a project folder and virtual environment

1. Create a project folder and navigate to it:

    ```bash
    mkdir mhsm-python-app && cd mhsm-python-app
    ```

1. Create and activate a virtual environment:

    ```bash
    python -m venv .venv
    source .venv/bin/activate  # On Windows: .venv\Scripts\activate
    ```

### Install the packages

Install the Azure Identity and Key Vault Keys client libraries:

```bash
pip install azure-identity azure-keyvault-keys
```

### Create the sample code

Create a file named `mhsm_keys.py` with the following code. Replace `<your-managed-hsm-name>` with your Managed HSM name, and `<your-key-name>` with an existing key name.

```python
from azure.identity import DefaultAzureCredential
from azure.keyvault.keys import KeyClient
from azure.keyvault.keys.crypto import CryptographyClient, EncryptionAlgorithm

# Use DefaultAzureCredential for automatic credential selection
credential = DefaultAzureCredential()

# Connect to Managed HSM - replace with your HSM URI
hsm_uri = "https://<your-managed-hsm-name>.managedhsm.azure.net"
key_client = KeyClient(vault_url=hsm_uri, credential=credential)

# Get a key reference
key_name = "<your-key-name>"
print(f"Retrieving key '{key_name}' from Managed HSM...")
key = key_client.get_key(key_name)
print(f"Key retrieved. Key type: {key.key_type}")

# Perform cryptographic operations
crypto_client = CryptographyClient(key, credential)

# Encrypt data
plaintext = b"Hello, Managed HSM!"
print(f"\nOriginal text: {plaintext.decode()}")

encrypt_result = crypto_client.encrypt(EncryptionAlgorithm.rsa_oaep_256, plaintext)
print(f"Encrypted (hex): {encrypt_result.ciphertext.hex()[:64]}...")

# Decrypt data
decrypt_result = crypto_client.decrypt(EncryptionAlgorithm.rsa_oaep_256, encrypt_result.ciphertext)
print(f"Decrypted text: {decrypt_result.plaintext.decode()}")

print("\nDone!")
```

### Run the application

Run the application:

```bash
python mhsm_keys.py
```

You should see output similar to:

```output
Retrieving key 'myrsakey' from Managed HSM...
Key retrieved. Key type: RSA-HSM

Original text: Hello, Managed HSM!
Encrypted (hex): 5a8f3b2c1d4e5f6a7b8c9d0e1f2a3b4c...
Decrypted text: Hello, Managed HSM!

Done!
```

## Understanding the code

### Authentication with DefaultAzureCredential

[!INCLUDE [DefaultAzureCredential explanation](../includes/managed-hsm/sdk-default-credential.md)]

### Key operations

The `KeyClient` class provides methods to:
- Create, get, update, and delete keys
- List keys and key versions
- Backup and restore keys

The `CryptographyClient` class provides cryptographic operations:
- Encrypt and decrypt data
- Sign and verify signatures
- Wrap and unwrap keys

[!INCLUDE [Assign Managed HSM roles](../includes/managed-hsm/sdk-role-assignment.md)]

## Clean up resources

When no longer needed, delete the resource group and all related resources:

```azurecli
az group delete --name ContosoResourceGroup
```

[!INCLUDE [Managed HSM cleanup warning](../includes/managed-hsm/cleanup-warning.md)]

## Next steps

[!INCLUDE [SDK next steps](../includes/managed-hsm/sdk-next-steps.md)]
