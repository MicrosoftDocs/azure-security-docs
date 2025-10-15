---
title: Quickstart -  Azure Key Vault secret client library for JavaScript (version 4)
description: Learn how to create, retrieve, and delete secrets from an Azure key vault using the JavaScript client library with either JavaScript or TypeScript
author: msmbaldwin
ms.author: mbaldwin
ms.date: 05/01/2025
ms.service: azure-key-vault
ms.subservice: secrets
ms.topic: quickstart
ms.devlang: javascript
ms.custom: devx-track-js, mode-api, passwordless-js, devx-track-ts
zone_pivot_groups: programming-languages-set-functions-nodejs
---

# Quickstart: Azure Key Vault secret client library for JavaScript

Get started with the Azure Key Vault secret client library for JavaScript. [Azure Key Vault](../general/overview.md) is a cloud service that provides a secure store for secrets. You can securely store keys, passwords, certificates, and other secrets. Azure key vaults may be created and managed through the Azure portal. In this quickstart, you learn how to create, retrieve, and delete secrets from an Azure key vault using the JavaScript client library.

Key Vault client library resources:

[API reference documentation](/javascript/api/overview/azure/key-vault) | [Library source code](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/keyvault) | [Package (npm)](https://www.npmjs.com/package/@azure/keyvault-secrets)

For more information about Key Vault and secrets, see:
- [Key Vault Overview](../general/overview.md)
- [Secrets Overview](about-secrets.md)

::: zone pivot="programming-language-javascript"

## Prerequisites

- An Azure subscription - [create one for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- Current [Node.js LTS](https://nodejs.org).
- [Azure CLI](/cli/azure/install-azure-cli)

::: zone-end

::: zone pivot="programming-language-typescript"

## Prerequisites

- An Azure subscription - [create one for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- Current [Node.js LTS](https://nodejs.org).
- [TypeScript 5+](https://www.typescriptlang.org/download/)
- [Azure CLI](/cli/azure/install-azure-cli).

::: zone-end

This quickstart assumes you are running [Azure CLI](/cli/azure/install-azure-cli).

## Sign in to Azure

1. Run the `login` command.

    ```azurecli
    az login
    ```

    If the CLI can open your default browser, it will do so and load an Azure sign-in page.

    Otherwise, open a browser page at [https://aka.ms/devicelogin](https://aka.ms/devicelogin) and enter the authorization code displayed in your terminal.

2. Sign in with your account credentials in the browser.

### Create a resource group and key vault

[!INCLUDE [Create a resource group and key vault](../includes/key-vault-python-qs-rg-kv-creation.md)]

### Grant access to your key vault

[!INCLUDE [Using RBAC to provide access to a key vault](~/reusable-content/ce-skilling/azure/includes/key-vault/rbac/upn-secrets-officer-cli.md)]

## Create new Node.js application

Create a Node.js application that uses your key vault.

1. In a terminal, create a folder named `key-vault-node-app` and change into that folder:

    ```terminal
    mkdir key-vault-node-app && cd key-vault-node-app
    ```

1. Initialize the Node.js project:

    ```terminal
    npm init -y
    ```

## Install Key Vault packages

1. Using the terminal, install the Azure Key Vault secrets client library, [@azure/keyvault-secrets](https://www.npmjs.com/package/@azure/keyvault-secrets) for Node.js.

    ```terminal
    npm install @azure/keyvault-secrets
    ```

1. Install the Azure Identity client library, [@azure/identity](https://www.npmjs.com/package/@azure/identity) package to authenticate to a Key Vault.

    ```terminal
    npm install @azure/identity
    ```

## Grant access to your key vault

[!INCLUDE [Using RBAC to provide access to a key vault](~/reusable-content/ce-skilling/azure/includes/key-vault/rbac/upn-secrets-officer-cli.md)]

## Set environment variables

This application is using key vault endpoint as an environment variable called `KEY_VAULT_URL`.

### [Windows](#tab/windows)

```cmd
set KEY_VAULT_URL=<your-key-vault-endpoint>
````

### [PowerShell](#tab/powershell)

Windows PowerShell
```powershell
$Env:KEY_VAULT_URL="<your-key-vault-endpoint>"
```

### [macOS or Linux](#tab/linux)

```cmd
export KEY_VAULT_URL=<your-key-vault-endpoint>
```
---


## Authenticate and create a client

Application requests to most Azure services must be authorized. Using the [DefaultAzureCredential](/javascript/api/@azure/identity/#@azure-identity-getdefaultazurecredential) method provided by the [Azure Identity client library](/javascript/api/@azure/identity) is the recommended approach for implementing passwordless connections to Azure services in your code. `DefaultAzureCredential` supports multiple authentication methods and determines which method should be used at runtime. This approach enables your app to use different authentication methods in different environments (local vs. production) without implementing environment-specific code. 

In this quickstart, `DefaultAzureCredential` authenticates to key vault using the credentials of the local development user logged into the Azure CLI. When the application is deployed to Azure, the same `DefaultAzureCredential` code can automatically discover and use a managed identity that is assigned to an App Service, Virtual Machine, or other services. For more information, see [Managed Identity Overview](/azure/active-directory/managed-identities-azure-resources/overview).

In this code, the endpoint of your key vault is used to create the key vault client. The endpoint format looks like `https://<your-key-vault-name>.vault.azure.net` but may change for sovereign clouds. For more information about authenticating to key vault, see [Developer's Guide](/azure/key-vault/general/developers-guide#authenticate-to-key-vault-in-code).

## Code example

The code samples below will show you how to create a client, set a secret, retrieve a secret, and delete a secret. 

This code uses the following [Key Vault Secret classes and methods](/javascript/api/overview/azure/keyvault-secrets-readme):
    
* [DefaultAzureCredential](/javascript/api/@azure/identity/#@azure-identity-getdefaultazurecredential)
* [SecretClient class](/javascript/api/@azure/keyvault-secrets/secretclient)
    * [setSecret](/javascript/api/@azure/keyvault-secrets/secretclient#@azure-keyvault-secrets-secretclient-setsecret)
    * [getSecret](/javascript/api/@azure/keyvault-secrets/secretclient#@azure-keyvault-secrets-secretclient-getsecret)
    * [updateSecretProperties](/javascript/api/@azure/keyvault-secrets/secretclient#@azure-keyvault-secrets-secretclient-updatesecretproperties)
    * [beginDeleteSecret](/javascript/api/@azure/keyvault-secrets/secretclient#@azure-keyvault-secrets-secretclient-begindeletesecret)

### Set up the app framework

::: zone pivot="programming-language-javascript"

* Create new text file and paste the following code into the **index.js** file. 

    ```javascript
    const { SecretClient } = require("@azure/keyvault-secrets");
    const { DefaultAzureCredential } = require("@azure/identity");
    
    async function main() {
      // If you're using MSI, DefaultAzureCredential should "just work".
      // Otherwise, DefaultAzureCredential expects the following three environment variables:
      // - AZURE_TENANT_ID: The tenant ID in Azure Active Directory
      // - AZURE_CLIENT_ID: The application (client) ID registered in the AAD tenant
      // - AZURE_CLIENT_SECRET: The client secret for the registered application
      const credential = new DefaultAzureCredential();
    
      const keyVaultUrl = process.env["KEY_VAULT_URL"];
      if(!keyVaultUrl) throw new Error("KEY_VAULT_URL is empty");
    
      const client = new SecretClient(keyVaultUrl, credential);
    
      // Create a secret
      // The secret can be a string of any kind. For example,
      // a multiline text block such as an RSA private key with newline characters,
      // or a stringified JSON object, like `JSON.stringify({ mySecret: 'MySecretValue'})`.
      const uniqueString = new Date().getTime();
      const secretName = `secret${uniqueString}`;
      const result = await client.setSecret(secretName, "MySecretValue");
      console.log("result: ", result);
    
      // Read the secret we created
      const secret = await client.getSecret(secretName);
      console.log("secret: ", secret);
    
      // Update the secret with different attributes
      const updatedSecret = await client.updateSecretProperties(secretName, result.properties.version, {
        enabled: false
      });
      console.log("updated secret: ", updatedSecret);
    
      // Delete the secret immediately without ability to restore or purge.
      await client.beginDeleteSecret(secretName);
    }
    
    main().catch((error) => {
      console.error("An error occurred:", error);
      process.exit(1);
    });
    ```

## Run the sample application

1. Run the app:

    ```terminal
    node index.js
    ```
1. The create and get methods return a full JSON object for the secret:

    ```JSON
    {
        "value": "MySecretValue",
        "name": "secret1637692472606",
        "properties": {
            "createdOn": "2021-11-23T18:34:33.000Z",
            "updatedOn": "2021-11-23T18:34:33.000Z",
            "enabled": true,
            "recoverableDays": 90,
            "recoveryLevel": "Recoverable+Purgeable",
            "id": "https: //YOUR-KEYVAULT-ENDPOINT.vault.azure.net/secrets/secret1637692472606/YOUR-VERSION",
            "vaultUrl": "https: //YOUR-KEYVAULT-ENDPOINT.vault.azure.net",
            "version": "YOUR-VERSION",
            "name": "secret1637692472606"
        }
    }
    ```

    The update method returns the **properties** name/values pairs:

    ```JSON
    "createdOn": "2021-11-23T18:34:33.000Z",
    "updatedOn": "2021-11-23T18:34:33.000Z",
    "enabled": true,
    "recoverableDays": 90,
    "recoveryLevel": "Recoverable+Purgeable",
    "id": "https: //YOUR-KEYVAULT-ENDPOINT/secrets/secret1637692472606/YOUR-VERSION",
    "vaultUrl": "https: //YOUR-KEYVAULT-ENDPOINT",
    "version": "YOUR-VERSION",
    "name": "secret1637692472606"
    ```
::: zone-end
::: zone pivot="programming-language-typescript"
* Create new text file and paste the following code into the **index.ts** file. 

    :::code language="typescript" source="~/azure-typescript-e2e-apps/quickstarts/key-vault/src/secrets.ts" :::

## Run the sample application

1. Build the TypeScript app:

    ```terminal
    tsc
    ```

1. Run the app:

    ```terminal
    node index.js
    ```

1. The create and get methods return a full JSON object for the secret:

    ```JSON
    {
        "value": "MySecretValue",
        "name": "secret1637692472606",
        "properties": {
            "createdOn": "2021-11-23T18:34:33.000Z",
            "updatedOn": "2021-11-23T18:34:33.000Z",
            "enabled": true,
            "recoverableDays": 90,
            "recoveryLevel": "Recoverable+Purgeable",
            "id": "https: //YOUR-KEYVAULT-ENDPOINT.vault.azure.net/secrets/secret1637692472606/YOUR-VERSION",
            "vaultUrl": "https: //YOUR-KEYVAULT-ENDPOINT.vault.azure.net",
            "version": "YOUR-VERSION",
            "name": "secret1637692472606"
        }
    }
    ```

    The update method returns the **properties** name/values pairs:

    ```JSON
    "createdOn": "2021-11-23T18:34:33.000Z",
    "updatedOn": "2021-11-23T18:34:33.000Z",
    "enabled": true,
    "recoverableDays": 90,
    "recoveryLevel": "Recoverable+Purgeable",
    "id": "https: //YOUR-KEYVAULT-ENDPOINT/secrets/secret1637692472606/YOUR-VERSION",
    "vaultUrl": "https: //YOUR-KEYVAULT-ENDPOINT",
    "version": "YOUR-VERSION",
    "name": "secret1637692472606"
    ```
::: zone-end





## Integrating with App Configuration

The Azure SDK provides a helper method, [parseKeyVaultSecretIdentifier](/javascript/api/@azure/keyvault-secrets/#parseKeyVaultSecretIdentifier_string_), to parse the given Key Vault Secret ID. This is necessary if you use [App Configuration](/azure/azure-app-configuration/) references to Key Vault. App Config stores the Key Vault Secret ID. You need the _parseKeyVaultSecretIdentifier_ method to parse that ID to get the secret name. Once you have the secret name, you can get the current secret value using code from this quickstart.  

## Next steps

In this quickstart, you created a key vault, stored a secret, and retrieved that secret. To learn more about Key Vault and how to integrate it with your applications, continue on to the articles below.

- Read an [Overview of Azure Key Vault](../general/overview.md)
- Read an [Overview of Azure Key Vault Secrets](about-secrets.md)
- How to [Secure access to a key vault](../general/security-features.md)
- See the [Azure Key Vault developer's guide](../general/developers-guide.md)
- Review the [Key Vault security overview](../general/security-features.md)
