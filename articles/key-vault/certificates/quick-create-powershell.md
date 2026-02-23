---
title: Quickstart - Set & view Azure Key Vault certificates with Azure PowerShell
description: Quickstart showing how to set and retrieve a certificate from Azure Key Vault using Azure PowerShell
services: key-vault
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: certificates
ms.topic: quickstart
ms.custom: mvc, devx-track-azurepowershell, mode-api
ms.date: 01/30/2026
ms.author: mbaldwin
#Customer intent: As a security admin who is new to Azure, I want to use Key Vault to securely store keys and passwords in Azure
---
# Quickstart: Set and retrieve a certificate from Azure Key Vault using Azure PowerShell

In this quickstart, you create a key vault in Azure Key Vault with Azure PowerShell. Azure Key Vault is a cloud service that works as a secure secrets store. You can securely store keys, passwords, certificates, and other secrets. For more information on Key Vault, review the [Overview](../general/overview.md). Azure PowerShell is used to create and manage Azure resources using commands or scripts. Afterwards, you store a certificate.

## Prerequisites

If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.

[!INCLUDE [azure-powershell-requirements-no-header.md](~/reusable-content/azure-powershell/azure-powershell-requirements-no-header.md)]

## Create a resource group

[!INCLUDE [Create a resource group](~/reusable-content/ce-skilling/azure/includes/create-resource-group-powershell.md)]

## Create a key vault

[!INCLUDE [Create a key vault](~/reusable-content/ce-skilling/azure/includes/key-vault/create-key-vault-powershell.md)]

## Give your user account permissions to manage certificates in Key Vault

[!INCLUDE [Using RBAC to provide access to a key vault](~/reusable-content/ce-skilling/azure/includes/key-vault/rbac/upn-certificate-officer-powershell.md)]

## Add a certificate to Key Vault

You can now add a certificate to the vault. This certificate could be used by an application.

Use these commands to create a self-signed certificate with policy called **ExampleCertificate** :

```azurepowershell-interactive
$Policy = New-AzKeyVaultCertificatePolicy -SecretContentType "application/x-pkcs12" -SubjectName "CN=contoso.com" -IssuerName "Self" -ValidityInMonths 6 -ReuseKeyOnRenewal

Add-AzKeyVaultCertificate -VaultName "<your-unique-keyvault-name>" -Name "ExampleCertificate" -CertificatePolicy $Policy
```

You can now reference this certificate that you added to Azure Key Vault by using its URI. Use **`https://<your-unique-keyvault-name>.vault.azure.net/certificates/ExampleCertificate`** to get the current version. 

To view previously stored certificate:

```azurepowershell-interactive
Get-AzKeyVaultCertificate -VaultName "<your-unique-keyvault-name>" -Name "ExampleCertificate"
```

**Troubleshooting**:

Operation returned an invalid status code 'Forbidden'

If you receive this error, the account accessing the Azure Key Vault does not have the proper permissions to create certificates.

Run the following Azure PowerShell command to assign the proper RBAC role:

```azurepowershell-interactive
New-AzRoleAssignment -SignInName <your-email-address> -RoleDefinitionName "Key Vault Certificates Officer" -Scope "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.KeyVault/vaults/<KeyVaultName>"
```

## Clean up resources

[!INCLUDE [Clean up resources](~/reusable-content/ce-skilling/azure/includes/delete-resource-group-powershell.md)]

## Next steps

In this quickstart, you created a Key Vault and stored a certificate in it. To learn more about Key Vault and how to integrate it with your applications, continue on to the articles below.

- Read an [Overview of Azure Key Vault](../general/overview.md)
- See the reference for the [Azure PowerShell Key Vault cmdlets](/powershell/module/az.keyvault/)
- Review the [Key Vault security overview](../general/secure-key-vault.md)
- Review [certificates-specific security best practices](secure-certificates.md)
