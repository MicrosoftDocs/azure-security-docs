---
author: msmbaldwin
ms.service: azure-key-vault
ms.topic: include
ms.date: 06/12/2024
ms.author: msmbaldwin

# Used by Key Vault CLI quickstarts

---

Use the Azure CLI [az keyvault create](/cli/azure/keyvault#az-keyvault-create) command to create a Key Vault in the resource group from the previous step. You will need to provide some information:

- Key vault name: A string of 3 to 24 characters that can contain only numbers (0-9), letters (a-z, A-Z), and hyphens (-)

  > [!Important]
  > Each key vault must have a unique name. Replace \<your-unique-keyvault-name\> with the name of your key vault in the following examples.

- Resource group name: **myResourceGroup**.
- The location: **EastUS**.

```azurecli
az keyvault create --name "<your-unique-keyvault-name>" --resource-group "myResourceGroup"
```

The output of this command shows properties of the newly created key vault. Take note of these two properties:

- **Vault Name**: The name you provided to the `--name` parameter.
- **Vault URI**: In this example, the Vault URI is https://&lt;your-unique-keyvault-name&gt;.vault.azure.net/. Applications that use your vault through its REST API must use this URI.
