---
title: Tutorial - Use Azure Key Vault with a virtual machine in Python
description: In this tutorial, you configure a virtual machine a Python application to read a secret from your key vault.
ms.service: azure-key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 07/07/2026
ms.devlang: python
ms.custom: devx-track-python, devx-track-azurecli, devx-track-azurepowershell
# Customer intent: As a developer I want to use Azure Key vault to store secrets for my app, so that they are kept secure.
---

# Tutorial: Use Azure Key Vault with a virtual machine in Python

Azure Key Vault helps you protect keys, secrets, and certificates, such as API keys and database connection strings.

In this tutorial, you set up a Python application to read information from Azure Key Vault by using a managed identity for Azure resources. You learn how to:

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

[!INCLUDE [Create a resource group and key vault](~/reusable-content/ce-skilling/azure/includes/key-vault/create-resource-group-key-vault-pivot.md)]

## Populate your key vault with a secret

[!INCLUDE [Create a secret](~/reusable-content/ce-skilling/azure/includes/key-vault/create-secret.md)]

## Create a virtual machine

Create a VM named **myVM** by using one of the following methods:

| Linux | Windows |
|--|--|
| [Azure CLI](/azure/virtual-machines/linux/quick-create-cli) | [Azure CLI](/azure/virtual-machines/windows/quick-create-cli) |
| [PowerShell](/azure/virtual-machines/linux/quick-create-powershell) | [PowerShell](/azure/virtual-machines/windows/quick-create-powershell) |
| [Azure portal](/azure/virtual-machines/linux/quick-create-portal) | [Azure portal](/azure/virtual-machines/windows/quick-create-portal) |

To create a Linux VM using the Azure CLI, use the [az vm create](/cli/azure/vm) command.  The following example adds a user account named *azureuser*. The `--generate-ssh-keys` parameter is used to automatically generate an SSH key, and put it in the default key location (*~/.ssh*). 

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

[!INCLUDE [Using RBAC to provide access to a key vault](~/reusable-content/ce-skilling/azure/includes/key-vault/rbac/upn-secrets-user-pivot.md)]

## Sign in to the VM

To sign in to the VM, follow the instructions in [Connect and sign in to an Azure virtual machine running Linux](/azure/virtual-machines/linux-vm-connect) or [Connect and sign in to an Azure virtual machine running Windows](/azure/virtual-machines/windows/connect-logon).

To sign in to a Linux VM, use `ssh` with the `<public-ip-address>` from the [Create a virtual machine](#create-a-virtual-machine) step:

```terminal
ssh azureuser@<public-ip-address>
```

## Install Python libraries on the VM

On the VM, install the two Python libraries the sample script uses: `azure-keyvault-secrets` and `azure-identity`.

On a Linux VM, install them with `pip3`:

```bash
pip3 install azure-keyvault-secrets azure-identity
```

## Create and edit the sample Python script

On the VM, create a Python file named `sample.py`. Paste the following code into the file, replacing `<vault-name>` with your key vault name:

```python
from azure.keyvault.secrets import SecretClient
from azure.identity import DefaultAzureCredential

key_vault_name = "<vault-name>"
key_vault_uri = f"https://{key_vault_name}.vault.azure.net"
secret_name = "mySecret"

credential = DefaultAzureCredential()
client = SecretClient(vault_url=key_vault_uri, credential=credential)
retrieved_secret = client.get_secret(secret_name)

print(f"The value of secret '{secret_name}' in '{key_vault_name}' is: '{retrieved_secret.value}'")
```

## Run the sample Python app

Run `sample.py`. If everything is configured correctly, the app prints the secret value:

```bash
python3 sample.py

The value of secret 'mySecret' in '<vault-name>' is: 'Success!'
```

## Clean up resources

When you no longer need them, delete the VM and the key vault. The fastest way is to delete the resource group they belong to:

```azurecli
az group delete -g "myResourceGroup"
```

## Next steps

[Azure Key Vault REST API](/rest/api/keyvault/)
