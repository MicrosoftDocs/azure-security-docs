---
title: Create and retrieve attributes of a managed key in Azure Key Vault – Azure PowerShell
description: Quickstart showing how to set and retrieve a managed key from Azure Key Vault using Azure PowerShell
author: msmbaldwin
ms.author: mbaldwin
ms.date: 03/26/2026
ms.topic: quickstart
ms.service: azure-key-vault
ms.subservice: keys
ms.custom: devx-track-azurepowershell, mode-api
#Customer intent: As a security admin who is new to Azure, I want to use Key Vault to securely store keys and passwords in Azure
---
# Quickstart: Provision and activate a Managed HSM by using PowerShell

In this quickstart, you create and activate an Azure Key Vault Managed HSM (Hardware Security Module) by using PowerShell. [!INCLUDE [Managed HSM description](../includes/managed-hsm/intro.md)]

## Prerequisites

If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.

[!INCLUDE [azure-powershell-requirements-no-header.md](~/reusable-content/azure-powershell/azure-powershell-requirements-no-header.md)]

## Create a resource group

A resource group is a logical container into which you deploy and manage Azure resources. Use the Azure PowerShell [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) cmdlet to create a resource group named `<resource-group>` in the `<location>` location. 

```azurepowershell-interactive
New-AzResourceGroup -Name "<resource-group>" -Location "<location>"
```

## Get your principal ID

To create a Managed HSM, you need your Microsoft Entra principal ID.  To get your ID, use the Azure PowerShell [Get-AzADUser](/powershell/module/az.resources/get-azaduser) cmdlet, and pass your email address to the `UserPrincipalName` parameter:

```azurepowershell-interactive
Get-AzADUser -UserPrincipalName "<user-principal-name>"
```

Your principal ID is returned in the format, "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx".

## Create a Managed HSM

Creating a Managed HSM is a two-step process:
1. Provision a Managed HSM resource.
1. Activate your Managed HSM by downloading an artifact called the *security domain*.

### Provision a Managed HSM

Use the Azure PowerShell [New-AzKeyVaultManagedHsm](/powershell/module/az.keyvault/new-azkeyvaultmanagedhsm) cmdlet to create a new Managed HSM. Provide the following information:

- Managed HSM name: A string of 3 to 24 characters that can contain only numbers (0-9), letters (a-z, A-Z), and hyphens (-).

  > [!Important]
  > Each Managed HSM must have a unique name. Replace `<hsm-name>` with your own unique Managed HSM name in the following examples.

- Resource group name: `<resource-group>`.
- Location: Your selected location.
- Your principal ID: Pass the Microsoft Entra principal ID that you obtained in the last section to the "Administrator" parameter. 

```azurepowershell-interactive
New-AzKeyVaultManagedHsm -Name "<hsm-name>" -ResourceGroupName "<resource-group>" -Location "<location>" -Administrator "<principal-id>" -SoftDeleteRetentionInDays "<retention-days>"
```
> [!NOTE]
> The create command can take a few minutes. When it returns successfully, you're ready to activate your HSM.

[!INCLUDE [Managed HSM billing warning](../includes/managed-hsm/billing-warning.md)]

The output of this cmdlet shows properties of the newly created Managed HSM. Take note of these two properties:

- **Name**: The name you provided for the Managed HSM.
- **HsmUri**: The URI for your HSM (for example, `https://<hsm-name>.managedhsm.azure.net/`). Applications that use your vault through its REST API must use this URI.

At this point, your Azure account is the only one authorized to perform any operations on this new HSM.

### Activate your Managed HSM

All data plane commands are disabled until you activate the HSM. You can't create keys or assign roles. Only the designated administrators that you assign during the create command can activate the HSM. To activate the HSM, you must download the [Security Domain](security-domain.md).

[!INCLUDE [Security domain prerequisites](../includes/managed-hsm/security-domain-prerequisites.md)]

> [!TIP]
> OpenSSL for Windows is available from [the OpenSSL website](https://slproweb.com/products/Win32OpenSSL.html).

Use the Azure PowerShell [Export-AzKeyVaultSecurityDomain](/powershell/module/az.keyvault/export-azkeyvaultsecuritydomain) cmdlet to download the security domain and activate your Managed HSM. The following example uses three RSA key pairs (only public keys are needed for this command) and sets the quorum to two.

```azurepowershell-interactive
Export-AzKeyVaultSecurityDomain -Name "<hsm-name>" -Certificates "cert_0.cer", "cert_1.cer", "cert_2.cer" -OutputPath "<hsm-name>-SD.json" -Quorum 2
```

Store the security domain file and the RSA key pairs securely. You need them for disaster recovery or for creating another Managed HSM that shares the same security domain so the two can share keys.

After you successfully download the security domain, your HSM is in an active state and ready for you to use.

## Clean up resources

[!INCLUDE [Create a key vault](~/reusable-content/ce-skilling/azure/includes/delete-resource-group-powershell.md)]

[!INCLUDE [Managed HSM cleanup warning](../includes/managed-hsm/cleanup-warning.md)]
## Next steps

In this quickstart, you created and activated a Managed HSM. To learn more about Managed HSM and how to integrate it with your applications, continue on to these articles:

- Review [Secure your Azure Managed HSM deployment](secure-managed-hsm.md).
- Read an [Overview of Azure Managed HSM](overview.md).
- See the reference for the [Azure PowerShell Key Vault cmdlets](/powershell/module/az.keyvault/).
