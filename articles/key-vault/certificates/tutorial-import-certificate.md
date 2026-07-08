---
title: Tutorial - Import a certificate in Key Vault using Azure portal
description: Tutorial showing how to import a certificate in Azure Key Vault
services: key-vault

ms.service: azure-key-vault
ms.subservice: certificates
ms.topic: tutorial
ms.custom: mvc, sfi-image-nochange
ms.date: 07/07/2026

ms.devlang: azurecli
---
# Tutorial: Import a certificate in Azure Key Vault

Azure Key Vault is a cloud service that provides a secure store for secrets. You can securely store keys, passwords, certificates, and other secrets. In this tutorial, you create a key vault and use it to import a certificate. For more information about Key Vault, see the [overview](../general/overview.md).

The tutorial shows you how to:

> [!div class="checklist"]
> * Create a key vault.
> * Import a certificate in Key Vault using the portal.
> * Import a certificate in Key Vault using the CLI.
> * Import a certificate in Key Vault using PowerShell.


Before you begin, read [Key Vault basic concepts](../general/basic-concepts.md). 

If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.

## Sign in to Azure

Sign in to the [Azure portal](https://portal.azure.com).

## Create a key vault

Create a key vault using one of these three methods:

- [Create a key vault using the Azure portal](../general/quick-create-portal.md)
- [Create a key vault using the Azure CLI](../general/quick-create-cli.md)
- [Create a key vault using Azure PowerShell](../general/quick-create-powershell.md)

## Import a certificate to your key vault
> [!NOTE]
> By default, imported certificates have exportable private keys. You can use the SDK, Azure CLI, or PowerShell to define policies that prevent the private key from being exported.

To import a certificate to the vault, you need a PEM or PFX certificate file on disk. If the certificate is in PEM format, the PEM file must contain the key and the x509 certificates. This operation requires the `certificates/import` permission (or the **Key Vault Certificates Officer** role when the vault uses Azure RBAC).

> [!IMPORTANT]
> Key Vault supports the PFX and PEM certificate formats.
> - A `.pem` file contains one or more X509 certificate files.
> - A `.pfx` file is an archive format that stores several cryptographic objects in a single file, such as a server certificate, its matching private key, and (optionally) an intermediate CA.

In this example, you import a certificate named **ExampleCertificate** from the path `/path/to/cert.pem`. You can import a certificate from the Azure portal, the Azure CLI, or Azure PowerShell.

# [Azure portal](#tab/azure-portal)

1. On the page for your key vault, select **Certificates**.
2. Select **Generate/Import**.
3. On the **Create a certificate** screen, choose the following values:
    - **Method of Certificate Creation**: **Import**.
    - **Certificate Name**: `ExampleCertificate`.
    - **Upload Certificate File**: Select the certificate file from disk.
    - **Password**: If the certificate file is password protected, enter the password. Otherwise, leave it blank. After the certificate is imported, Key Vault removes the password.
4. Select **Create**.

:::image type="content" source="../media/certificates/tutorial-import-cert/cert-import.png" alt-text="Importing a certificate through the Azure portal":::

When importing a .pem file, check if the format is the following:

-----BEGIN CERTIFICATE-----<br>
MIID2TCCAsGg...<br>
-----END CERTIFICATE-----<br>
-----BEGIN PRIVATE KEY-----<br>
MIIEvQIBADAN...<br>
-----END PRIVATE KEY-----<br>

When you import a certificate, Key Vault automatically populates certificate parameters such as validity period, issuer name, and activation date.

After you see the message that the certificate imported successfully, select it in the list to view its properties.

:::image type="content" source="../media/certificates/tutorial-import-cert/cert-properties.png" alt-text="Properties of a newly imported certificate in the Azure portal":::

# [Azure CLI](#tab/azure-cli)

Import a certificate into your key vault using the Azure CLI [az keyvault certificate import](/cli/azure/keyvault/certificate#az-keyvault-certificate-import) command:

```azurecli
az keyvault certificate import --vault-name "<vault-name>" -n "ExampleCertificate" -f "/path/to/ExampleCertificate.pem"
```

After importing the certificate, you can view the certificate using the Azure CLI [az keyvault certificate show](/cli/azure/keyvault/certificate#az-keyvault-certificate-show) command.

```azurecli
az keyvault certificate show --vault-name "<vault-name>" --name "ExampleCertificate"
```

# [Azure PowerShell](#tab/azure-powershell)

Import a certificate into Key Vault by using the Azure PowerShell [Import-AzKeyVaultCertificate](/powershell/module/az.keyvault/import-azkeyvaultcertificate) cmdlet:

```azurepowershell
Import-AzKeyVaultCertificate -VaultName "<vault-name>" -Name "ExampleCertificate" -FilePath "C:\path\to\ExampleCertificate.pem"
```

If the file is password protected, pass the password as a `SecureString` with the `-Password` parameter.

After you import the certificate, view it by using the [Get-AzKeyVaultCertificate](/powershell/module/az.keyvault/get-azkeyvaultcertificate) cmdlet:

```azurepowershell
Get-AzKeyVaultCertificate -VaultName "<vault-name>" -Name "ExampleCertificate"
```

---

You created a key vault, imported a certificate, and viewed the certificate's properties.

## Clean up resources

Other Key Vault quickstarts and tutorials build on this tutorial. If you plan to work with them, leave these resources in place. When you no longer need the resources, delete the resource group. Deleting the resource group deletes the key vault and any related resources. To delete the resource group in the portal:

1. Enter the resource group name in the search box at the top of the portal, and select it from the results.
1. Select **Delete resource group**.
1. In the **TYPE THE RESOURCE GROUP NAME:** box, enter the resource group name and select **Delete**.

## Next steps

In this tutorial, you created a key vault and imported a certificate. To learn more about Key Vault and integrate it with your applications, see:

- [Manage certificate creation in Azure Key Vault](./create-certificate-scenarios.md)
- [Import certificates by using the REST API](/rest/api/keyvault/certificates/import-certificate/import-certificate)
- [Key Vault security overview](../general/secure-key-vault.md)
