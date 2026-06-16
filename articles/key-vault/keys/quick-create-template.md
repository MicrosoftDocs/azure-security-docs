---
title: Azure Quickstart - Create an Azure key vault and a key by using Azure Resource Manager template | Microsoft Docs
description: Quickstart showing how to create Azure key vaults, and add key to the vaults by using Azure Resource Manager template (ARM template).
services: key-vault
ms.service: azure-key-vault
ms.subservice: keys
ms.topic: quickstart
ms.custom: mvc, subject-armqs, mode-arm, devx-track-arm-template
ms.date: 05/14/2026
#Customer intent: As a security admin who is new to Azure, I want to use Key Vault to securely store keys and passwords in Azure.
---

# Quickstart: Create an Azure key vault and a key by using ARM template

[Azure Key Vault](../general/overview.md) is a cloud service that provides a secure store for secrets, such as keys, passwords, and certificate. This quickstart focuses on the process of deploying an Azure Resource Manager template (ARM template) to create a key vault and a key.

[!INCLUDE [About Azure Resource Manager](~/reusable-content/ce-skilling/azure/includes/resource-manager-quickstart-introduction.md)]

If your environment meets the prerequisites and you're familiar with using ARM templates, select the **Deploy to Azure** button. The template will open in the Azure portal.

:::image type="content" source="~/reusable-content/ce-skilling/azure/media/template-deployments/deploy-to-azure-button.svg" alt-text="Button to deploy the Resource Manager template to Azure." border="false" link="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.keyvault%2Fkey-vault-key-create%2Fazuredeploy.json":::

## Prerequisites

To complete this article:

- If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.
- User would need to have an Azure built-in role assigned, recommended role **contributor**. [Learn more here](/azure/role-based-access-control/role-assignments-portal)

## Review the template

The template used in this quickstart is from [Azure Quickstart Templates](/samples/azure/azure-quickstart-templates/key-vault-key-create/).

:::code language="json" source="~/quickstart-templates/quickstarts/microsoft.keyvault/key-vault-key-create/azuredeploy.json":::

Two resources are defined in the template:

- [Microsoft.KeyVault/vaults](/azure/templates/microsoft.keyvault/vaults?tabs=json): create an Azure key vault.
- [Microsoft.KeyVault/vaults/keys](/azure/templates/microsoft.keyvault/vaults/keys?tabs=json): create a key vault key.

The template creates the key vault with Azure RBAC authorization enabled. This means the vault uses Azure role-based access control (Azure RBAC) for data plane authorization, rather than access policies.

More Azure Key Vault template samples can be found in [Azure Quickstart Templates](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Keyvault&pageNumber=1&sort=Popular).

## Template parameters

|Parameter  |Description  |
|---------|---------|
|**vaultName** | Name of the new key vault. Must be globally unique within the `vault.azure.net` namespace. |
|**keyName** | Name of the key to create in the vault. |
|**location** | Azure region for the resources. Defaults to the resource group location. |
|**skuName** | Vault SKU. `standard` (default) or `premium`. |
|**keyType** | The type of key to create. One of `RSA`, `RSA-HSM`, `EC`, `EC-HSM`. Default is `RSA`. See [JsonWebKeyType](/rest/api/keyvault/keys/create-key/create-key#jsonwebkeytype). |
|**keySize** | Size in bits of the key (RSA only). Default is `2048`. |
|**curveName** | Elliptic-curve name (EC keys only). One of `P-256`, `P-256K`, `P-384`, `P-521`. See [JsonWebKeyCurveName](/rest/api/keyvault/keys/create-key/create-key#jsonwebkeycurvename). |
|**keyOps** | Permitted [JSON Web Key operations](https://tools.ietf.org/html/draft-ietf-jose-json-web-key-41) for the key (for example, `["sign","verify","encrypt","decrypt","wrapKey","unwrapKey"]`). Empty array (default) allows all operations. |

## Deploy the template

1. Select the following image to sign in to Azure and open a template. The template creates a key vault and a key.

    :::image type="content" source="~/reusable-content/ce-skilling/azure/media/template-deployments/deploy-to-azure-button.svg" alt-text="Button to deploy the Resource Manager template to Azure." border="false" link="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.keyvault%2Fkey-vault-key-create%2Fazuredeploy.json":::

2. Select or enter the following values. Unless it's specified, use the default value.

    * **Subscription**: select an Azure subscription.
    * **Resource group**: select **Create new**, enter a unique name for the resource group, and then select **OK**.
    * **Region**: select a location. For example, **Central US**.
    * **Vault Name**: enter a name for the key vault, which must be globally unique within the `vault.azure.net` namespace.
    * **Key Name**: enter a name for the key that you store in the key vault.
    * **Sku Name**: select **standard** or **premium**. The default is **standard**.
    * **Key Type**: select a key type. The default is **RSA**.
    * **Key Size**: enter a key size (RSA keys). The default is **2048**.
    * **Curve Name**: for EC keys, select a curve. Leave blank for RSA keys.

3. Select **Review + create**, then select **Create**. After the key vault and key have been deployed successfully, you get a notification.

You can also use Azure PowerShell, the Azure CLI, or the REST API to deploy the template. To learn other deployment methods, see [Deploy templates](/azure/azure-resource-manager/templates/deploy-powershell).

## Assign a Key Vault RBAC role

The key vault created by this template uses Azure RBAC for authorization. To access keys through the data plane (for example, using the Azure CLI or Azure PowerShell), you need to assign yourself an appropriate role.

1. Get your Microsoft Entra user object ID:

    # [CLI](#tab/CLI)
    ```azurecli-interactive
    az ad signed-in-user show --query id -o tsv
    ```

    # [PowerShell](#tab/PowerShell)
    ```azurepowershell-interactive
    (Get-AzADUser -SignedIn).Id
    ```

    ---

2. Assign the **Key Vault Crypto Officer** role to yourself on the key vault:

    # [CLI](#tab/CLI)
    ```azurecli-interactive
    echo "Enter your key vault name:" &&
    read keyVaultName &&
    az role assignment create --role "Key Vault Crypto Officer" \
        --assignee-object-id $(az ad signed-in-user show --query id -o tsv) \
        --scope $(az keyvault show --name $keyVaultName --query id -o tsv)
    ```

    # [PowerShell](#tab/PowerShell)
    ```azurepowershell-interactive
    $keyVaultName = Read-Host -Prompt "Enter your key vault name"
    $kvId = (Get-AzKeyVault -VaultName $keyVaultName).ResourceId
    $userId = (Get-AzADUser -SignedIn).Id
    New-AzRoleAssignment -ObjectId $userId -RoleDefinitionName "Key Vault Crypto Officer" -Scope $kvId
    ```

    ---

    > [!NOTE]
    > Role assignments might take a minute or two to propagate.

## Review deployed resources

You can use the Azure portal to check the key vault and the key. Alternatively, use the following Azure CLI or Azure PowerShell script to list the key created.

# [CLI](#tab/CLI)

```azurecli-interactive
echo "Enter your key vault name:" &&
read keyVaultName &&
az keyvault key list --vault-name $keyVaultName &&
echo "Press [ENTER] to continue ..."
```

# [PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$keyVaultName = Read-Host -Prompt "Enter your key vault name"
Get-AzKeyVaultKey -vaultName $keyVaultName
Write-Host "Press [ENTER] to continue..."
```

---

## Creating key using ARM template is different from creating key via data plane

### Creating a key via ARM

- It's only possible to create *new* keys. It isn't possible to update existing keys, nor create new versions of existing keys. If the key already exists, then the existing key is retrieved from storage and used (no write operations will occur).
- To be authorized to use this API, the caller needs to have the **"Microsoft.KeyVault/vaults/keys/write"** role-based access control (Azure RBAC) Action. The built-in "Key Vault Contributor" role is sufficient, since it authorizes all Azure RBAC Actions that match the pattern "Microsoft.KeyVault/*".

  :::image type="content" source="../media/keys-quick-template-1.png" alt-text="Create a key via ARM 1":::
  :::image type="content" source="../media/keys-quick-template-2.png" alt-text="Create a key via ARM 2":::

### Existing API (creating key via data plane)

- It's possible to create new keys, update existing keys, and create new versions of existing keys.
- The caller must be authorized to use this API. If the vault is enabled for Azure RBAC, the caller must have "Microsoft.KeyVault/vaults/keys/create/action" Azure RBAC DataAction.

## Clean up resources

Other Key Vault quickstarts and tutorials build upon this quickstart. If you plan to continue on to work with subsequent quickstarts and tutorials, you may wish to leave these resources in place.
When no longer needed, delete the resource group, which deletes the Key Vault and related resources. To delete the resource group by using Azure CLI or Azure PowerShell:

# [CLI](#tab/CLI)

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName &&
echo "Press [ENTER] to continue ..."
```

# [PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
Remove-AzResourceGroup -Name $resourceGroupName
Write-Host "Press [ENTER] to continue..."
```

---

[!INCLUDE [Soft-delete note](~/reusable-content/ce-skilling/azure/includes/key-vault/key-vault-cleanup-soft-delete-note.md)]

## Next steps

In this quickstart, you created a key vault and a key using an ARM template, and validated the deployment. To learn more about Key Vault and Azure Resource Manager, see these articles.

- Read an [Overview of Azure Key Vault](../general/overview.md)
- Learn more about [Azure Resource Manager](/azure/azure-resource-manager/management/overview)
- Review the [Key Vault security overview](../general/secure-key-vault.md)
