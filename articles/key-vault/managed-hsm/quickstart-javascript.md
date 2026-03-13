---
title: Quickstart - Azure Key Vault Managed HSM client library for JavaScript
description: Learn how to access keys in Azure Managed HSM using the JavaScript client library
author: msmbaldwin
ms.author: mbaldwin
ms.date: 03/13/2026
ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: quickstart
ms.devlang: javascript
ms.custom: devx-track-js, mode-api
#Customer intent: As a developer, I want to programmatically access keys in Managed HSM using the JavaScript SDK
---

# Quickstart: Azure Key Vault Managed HSM client library for JavaScript

Get started with the Azure Key Vault Managed HSM client library for JavaScript. [!INCLUDE [Managed HSM description](../includes/managed-hsm/intro.md)]

In this quickstart, you learn how to access and perform cryptographic operations on keys in a Managed HSM using the JavaScript client library.

Managed HSM client library resources:

[API reference documentation](/javascript/api/overview/azure/keyvault-keys-readme) | [Library source code](https://github.com/Azure/azure-sdk-for-js/tree/main/sdk/keyvault/keyvault-keys) | [Package (npm)](https://www.npmjs.com/package/@azure/keyvault-keys)

## Prerequisites

[!INCLUDE [Managed HSM SDK prerequisites](../includes/managed-hsm/sdk-prereqs.md)]
- [Node.js 18 LTS or later](https://nodejs.org/)

## Set up your local environment

This quickstart uses the Azure Identity library with Azure CLI to authenticate to Azure services. Developers can also use Visual Studio Code to authenticate their calls. For more information, see [Authenticate the client with Azure Identity client library](/javascript/api/overview/azure/identity-readme).

### Sign in to Azure

Run the `az login` command to sign in:

```azurecli
az login
```

### Create a project folder and initialize

1. Create a project folder and navigate to it:

    ```bash
    mkdir mhsm-js-app && cd mhsm-js-app
    ```

1. Initialize the project:

    ```bash
    npm init -y
    ```

### Install the packages

Install the Azure Identity and Key Vault Keys client libraries:

```bash
npm install @azure/identity @azure/keyvault-keys
```

### Create the sample code

Create a file named `index.js` with the following code. Replace `<your-managed-hsm-name>` with your Managed HSM name, and `<your-key-name>` with an existing key name.

```javascript
const { DefaultAzureCredential } = require("@azure/identity");
const { KeyClient, CryptographyClient } = require("@azure/keyvault-keys");

async function main() {
    // Use DefaultAzureCredential for automatic credential selection
    const credential = new DefaultAzureCredential();

    // Connect to Managed HSM - replace with your HSM URI
    const hsmUri = "https://<your-managed-hsm-name>.managedhsm.azure.net";
    const keyClient = new KeyClient(hsmUri, credential);

    // Get a key reference
    const keyName = "<your-key-name>";
    console.log(`Retrieving key '${keyName}' from Managed HSM...`);
    const key = await keyClient.getKey(keyName);
    console.log(`Key retrieved. Key type: ${key.keyType}`);

    // Perform cryptographic operations
    const cryptoClient = new CryptographyClient(key, credential);

    // Encrypt data
    const plaintext = Buffer.from("Hello, Managed HSM!");
    console.log(`\nOriginal text: ${plaintext.toString()}`);

    const encryptResult = await cryptoClient.encrypt("RSA-OAEP-256", plaintext);
    console.log(`Encrypted (base64): ${encryptResult.result.toString("base64").substring(0, 64)}...`);

    // Decrypt data
    const decryptResult = await cryptoClient.decrypt("RSA-OAEP-256", encryptResult.result);
    console.log(`Decrypted text: ${decryptResult.result.toString()}`);

    console.log("\nDone!");
}

main().catch((error) => {
    console.error("An error occurred:", error);
    process.exit(1);
});
```

### Run the application

Run the application:

```bash
node index.js
```

You should see output similar to:

```output
Retrieving key 'myrsakey' from Managed HSM...
Key retrieved. Key type: RSA-HSM

Original text: Hello, Managed HSM!
Encrypted (base64): NWE4ZjNiMmMxZDRlNWY2YTdiOGM5ZDBlMWYyYTNiNGM...
Decrypted text: Hello, Managed HSM!

Done!
```

## Understanding the code

### Authentication with DefaultAzureCredential

[!INCLUDE [DefaultAzureCredential explanation](../includes/managed-hsm/sdk-defaultcredential.md)]

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

- Learn about [Secure access to your managed HSMs](how-to-secure-access.md)
- Configure [automated key rotation](key-rotation.md)
- Review [Managed HSM best practices](secure-managed-hsm.md)
- Learn about [Managed HSM local RBAC built-in roles](built-in-roles.md)
