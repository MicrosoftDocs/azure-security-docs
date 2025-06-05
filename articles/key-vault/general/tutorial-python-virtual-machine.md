---
title: Tutorial - Use Azure Key Vault with a virtual machine in Python | Microsoft Docs
description: In this tutorial, you configure a virtual machine a Python application to read a secret from your key vault.
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 04/16/2025
ms.author: mbaldwin
ms.devlang: python
ms.custom: devx-track-python, devx-track-azurecli, devx-track-azurepowershell
# Customer intent: As a developer I want to use Azure Key vault to store secrets for my app, so that they are kept secure.
---

# Tutorial: Use Azure Key Vault with a virtual machine in Python

Azure Key Vault helps you to protect keys, secrets, and certificates, such as API keys and database connection strings.

In this tutorial, you set up a Python application to read information from Azure Key Vault by using managed identities for Azure resources. You learn how to:

> [!div class="checklist"]
> * Create a key vault
> * Store a secret in Key Vault
> * Create an Azure Linux virtual machine
> * Enable a [managed identity](/azure/active-directory/managed-identities-azure-resources/overview) for the virtual machine
> * Grant the required permissions for the console application to read data from Key Vault
> * Retrieve a secret from Key Vault

Before you begin, read [Key Vault basic concepts](basic-concepts.md). 

If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## Prerequisites

For Windows, Mac, and Linux:
  * [Git](https://git-scm.com/downloads)
  * This tutorial requires that you run the Azure CLI locally. You must have the Azure CLI version 2.0.4 or later installed. Run `az --version` to find the version. If you need to install or upgrade the CLI, see [Install Azure CLI 2.0](/cli/azure/install-azure-cli).

## Log in to Azure

To log in to Azure by using the Azure CLI, enter:

```azurecli
az login
```

## Create a resource group and key vault

[!INCLUDE [Create a resource group and key vault](~/reusable-content/ce-skilling/azure/includes/key-vault/create-resource-group-key-vault-pivot.md)]

## Populate your key vault with a secret

[!INCLUDE [Create a secret](../includes/key-vault-create-secret.md)]

## Create a virtual machine

Create a VM called **myVM** using one of the following methods:

| Linux | Windows |
|--|--|
| [Azure CLI](/azure/virtual-machines/linux/quick-create-cli) | [Azure CLI](/azure/virtual-machines/windows/quick-create-cli) |
| [PowerShell](/azure/virtual-machines/linux/quick-create-powershell) | [PowerShell](/azure/virtual-machines/windows/quick-create-powershell) |
| [Azure portal](/azure/virtual-machines/linux/quick-create-portal) | [The Azure portal](/azure/virtual-machines/windows/quick-create-portal) |

To create a Linux VM using the Azure CLI, use the [az vm create](/cli/azure/vm) command.  The following example adds a user account named *azureuser*. The `--generate-ssh-keys` parameter is used to automatically generate an SSH key, and put it in the default key location (*~/.ssh*). 

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys
```

Note the value of `publicIpAddress` in the output.

## Assign an identity to the VM

Create a system-assigned identity for the virtual machine by using the Azure CLI [az vm identity assign](/cli/azure/vm/identity#az-vm-identity-assign) command:

```azurecli
az vm identity assign --name "myVM" --resource-group "myResourceGroup"
```

Note the system-assigned identity that's displayed in the following code. The output of the preceding command would be: 

```output
{
  "systemAssignedIdentity": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "userAssignedIdentities": {}
}
```

## Assign permissions to the VM identity

[!INCLUDE [Using RBAC to provide access to a key vault](~/reusable-content/ce-skilling/azure/includes/key-vault/rbac/upn-secrets-user-pivot.md)]

## Log in to the VM

To sign in to the virtual machine, follow the instructions in [Connect and sign in to an Azure virtual machine running Linux](/azure/virtual-machines/linux-vm-connect) or [Connect and sign in to an Azure virtual machine running Windows](/azure/virtual-machines/windows/connect-logon).


To log into a Linux VM, you can use the ssh command with the \<publicIpAddress\> given in the [Create a virtual machine](#create-a-virtual-machine) step:

```terminal
ssh azureuser@<PublicIpAddress>
```

## Install Python libraries on the VM

On the virtual machine, install the two Python libraries we'll be using in our Python script: `azure-keyvault-secrets` and `azure.identity`.  

On a Linux VM, for instance, you can install these using `pip3`:

```bash
pip3 install azure-keyvault-secrets

pip3 install azure.identity
```

## Create and edit the sample Python script

On the virtual machine, create a Python file called **sample.py**. Edit the file to contain the following code, replacing \<your-unique-keyvault-name\> with the name of your key vault:

```python
from azure.keyvault.secrets import SecretClient
from azure.identity import DefaultAzureCredential

key_vault_name = "<your-unique-keyvault-name>"
key_vault_uri = f"https://{key_vault_name}.vault.azure.net"
secret_name = "mySecret"

credential = DefaultAzureCredential()
client = SecretClient(vault_url=key_vault_uri, credential=credential)
retrieved_secret = client.get_secret(secret_name)

print(f"The value of secret '{secret_name}' in '{key_vault_name}' is: '{retrieved_secret.value}'")
```

## Run the sample Python app

Lastly, run **sample.py**. If all has gone well, it should return the value of your secret:

```bash
python3 sample.py

The value of secret 'mySecret' in '<your-unique-keyvault-name>' is: 'Success!'
```

## Clean up resources

When they're no longer needed, delete the virtual machine and your key vault. You can be done quickly by deleting the resource group to which they belong:

```azurecli
az group delete -g myResourceGroup
```

## Next steps

[Azure Key Vault REST API](/rest/api/keyvault/)
