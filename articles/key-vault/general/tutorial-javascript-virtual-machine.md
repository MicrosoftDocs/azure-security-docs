---
title: Tutorial - Use Azure Key Vault with a virtual machine in JavaScript
description: In this tutorial, you configure a virtual machine a JavaScript application to read a secret from your key vault.
ms.service: azure-key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 07/07/2026
ms.devlang: javascript
ms.custom: devx-track-js, devx-track-azurecli, devx-track-azurepowershell
# Customer intent: As a developer I want to use Azure Key vault to store secrets for my app, so that they are kept secure.
---

# Tutorial: Use Azure Key Vault with a virtual machine in JavaScript

Azure Key Vault helps you protect keys, secrets, and certificates, such as API keys and database connection strings.

In this tutorial, you set up a Node.js application to read information from Azure Key Vault by using a managed identity for Azure resources. You learn how to:

> [!div class="checklist"]
> * Create a key vault
> * Store a secret in Key Vault
> * Create an Azure Linux virtual machine
> * Enable a [managed identity](/entra/identity/managed-identities-azure-resources/overview) for the virtual machine
> * Grant the required permissions for the console application to read data from Key Vault
> * Retrieve a secret from Key Vault

Before you begin, read [Key Vault basic concepts](basic-concepts.md).

If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).

## Prerequisites

For Windows, Mac, and Linux:
  * [Git](https://git-scm.com/downloads)
  * A current version of the [Azure CLI](/cli/azure/install-azure-cli). Run `az --version` to check the installed version.

## Sign in to Azure

Sign in to Azure with the Azure CLI:

```azurecli
az login
```

## Create a resource group and key vault

[!INCLUDE [Create a resource group and key vault](~/reusable-content/ce-skilling/azure/includes/key-vault/create-resource-group-key-vault-cli.md)]

## Populate your key vault with a secret

[!INCLUDE [Create a secret](~/reusable-content/ce-skilling/azure/includes/key-vault/create-secret.md)]

## Create a virtual machine

Create a VM named **myVM** by using one of the following methods:

| Linux | Windows |
|--|--|
| [Azure CLI](/azure/virtual-machines/linux/quick-create-cli) | [Azure CLI](/azure/virtual-machines/windows/quick-create-cli) |
| [PowerShell](/azure/virtual-machines/linux/quick-create-powershell) | [PowerShell](/azure/virtual-machines/windows/quick-create-powershell) |
| [Azure portal](/azure/virtual-machines/linux/quick-create-portal) | [Azure portal](/azure/virtual-machines/windows/quick-create-portal) |

To create a Linux VM by using the Azure CLI, use the [az vm create](/cli/azure/vm) command. The following example adds a user account named `azureuser`. The `--generate-ssh-keys` parameter generates an SSH key and stores it in the default key location (`~/.ssh`).

```azurecli-interactive
az vm create \
  --resource-group <resource-group> \
  --name myVM \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys
```

Note the value of `publicIpAddress` in the output.

## Assign an identity to the VM

Create a system-assigned managed identity for the VM by using the [az vm identity assign](/cli/azure/vm/identity#az-vm-identity-assign) command:

```azurecli
az vm identity assign --name "myVM" --resource-group "<resource-group>"
```

Note the system-assigned identity in the output:

```output
{
  "systemAssignedIdentity": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "userAssignedIdentities": {}
}
```

## Assign permissions to the VM identity

Grant the VM's managed identity the **Key Vault Secrets User** role on the key vault:

```azurecli
az role assignment create --role "Key Vault Secrets User" --assignee "<system-assigned-identity>" --scope /subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.KeyVault/vaults/<vault-name>
```

## Sign in to the VM

To sign in to the VM, follow the instructions in [Connect and sign in to an Azure virtual machine running Linux](/azure/virtual-machines/linux-vm-connect) or [Connect and sign in to an Azure virtual machine running Windows](/azure/virtual-machines/windows/connect-logon).

To sign in to a Linux VM, use `ssh` with the `<public-ip-address>` from the [Create a virtual machine](#create-a-virtual-machine) step:

```terminal
ssh azureuser@<public-ip-address>
```

## Install Node.js and npm libraries on the VM

On the VM, install the two npm libraries the sample script uses: [@azure/keyvault-secrets](https://www.npmjs.com/package/@azure/keyvault-secrets) and [@azure/identity](https://www.npmjs.com/package/@azure/identity).

1. In the SSH terminal, install Node.js and npm:

    ```bash
    curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash - && \
        sudo apt-get install -y nodejs
    ```

1. Create an app directory and initialize the Node.js package:

    ```bash
    mkdir app && cd app && npm init -y
    ```

1. Install the Azure packages with `npm`:

    ```bash
    npm install @azure/keyvault-secrets @azure/identity
    ```

## Create and edit the sample JavaScript file

1. On the VM in the `app` directory, create a JavaScript file named `index.js`:

    ```bash
    touch index.js
    ```

1. Open the file with the [Nano](https://www.nano-editor.org/dist/latest/cheatsheet.html) text editor:

    ```bash
    nano index.js
    ```

1. Paste the following code into the editor, replacing `<vault-name>` with your key vault name:

    ```javascript
    // index.js
    
    const { SecretClient } = require("@azure/keyvault-secrets");
    const { DefaultAzureCredential } = require("@azure/identity");
    
    // Your Azure Key Vault name and secret name
    const keyVaultName = "<vault-name>";
    const keyVaultUri = `https://${keyVaultName}.vault.azure.net`;
    const secretName = "mySecret";
    
    // Authenticate to Azure
    const credential = new DefaultAzureCredential();
    const client = new SecretClient(keyVaultUri, credential);
    
    // Get Secret with Azure SDK for JS
    const getSecret = async (secretName) => {
    
        return (await client.getSecret(secretName)).value;
    }
    
    getSecret(secretName).then(secretValue => {
        console.log(`The value of secret '${secretName}' in '${keyVaultName}' is: '${secretValue}'`);
    }).catch(err => {
        console.log(err);
    })
    ```

1. Save the file with <kbd>Ctrl</kbd>+<kbd>X</kbd>.
1. At the `Save modified buffer?` prompt, enter <kbd>y</kbd>.
1. At the `File Name to Write: index.js` prompt, press <kbd>Enter</kbd>.

## Run the sample Node.js app

Run `index.js`. If everything is configured correctly, the app prints the secret value:

```bash
node index.js

The value of secret 'mySecret' in '<vault-name>' is: 'Success!'
```

## Clean up resources

When you no longer need them, delete the VM and the key vault. The fastest way is to delete the resource group they belong to:

```azurecli
az group delete -g "myResourceGroup"
```

## Next steps

[Azure Key Vault REST API](/rest/api/keyvault/)
