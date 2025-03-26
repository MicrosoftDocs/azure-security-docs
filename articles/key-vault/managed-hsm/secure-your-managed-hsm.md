---
title: Secure access to a managed HSM - Azure Key Vault Managed HSM
description: Learn how to secure access to Managed HSM using Azure RBAC and Managed HSM local RBAC
services: key-vault
author: msmbaldwin
ms.custom: devx-track-azurecli

ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: tutorial
ms.date: 01/30/2024
ms.author: mbaldwin
# Customer intent: As a managed HSM administrator, I want to set access control and configure the Managed HSM, so that I can ensure it's secure and auditors can properly monitor all activities for this Managed HSM.
---

# Tutorial: Secure access to your managed HSMs

Azure Key Vault Managed HSM is a cloud service that safeguards encryption keys. Because this data is sensitive and business critical, you need to secure access to your managed HSMs by allowing only authorized applications and users to access it.

This tutorial walks you through a simple example that shows how to achieve separation of duties and access control using Azure RBAC and Managed HSM local RBAC. For an overview of the Managed HSM access control model, see [Access control for Managed HSM](access-control.md).

## Prerequisites

To complete the steps in this article, you must have the following items:

* A subscription to Microsoft Azure. If you don't have one, you can sign up for a [free trial](https://azure.microsoft.com/pricing/free-trial).
* The Azure CLI version 2.25.0 or later. Run `az --version` to find the version. If you need to install or upgrade, see [Install the Azure CLI]( /cli/azure/install-azure-cli).
* A managed HSM in your subscription. See [Quickstart: Provision and activate a managed HSM using Azure CLI](quick-create-cli.md) to provision and activate a managed HSM.

[!INCLUDE [cloud-shell-try-it.md](~/reusable-content/ce-skilling/azure/includes/cloud-shell-try-it.md)]

## Sign in to Azure

# [Azure CLI](#tab/azure-cli)

To sign in to Azure using the CLI you can type:

```azurecli
az login
```

For more information on login options via the CLI, see [sign in with Azure CLI](/cli/azure/authenticate-azure-cli)

# [Azure PowerShell](#tab/azure-powershell)

To sign in to Azure using PowerShell, you can type:

```powershell
Connect-AzAccount
```

For more information on login options via PowerShell, see [sign in with Azure PowerShell](/powershell/azure/authenticate-azure-powershell).

---

## Securing access

In this example, we're developing an application that uses an RSA 2,048-bit key for sign operations. Our application runs in an Azure virtual machine (VM) with a [managed identity](/azure/active-directory/managed-identities-azure-resources/overview). Both the RSA key used for signing is stored in our managed HSM.

For details about roles and permissions, see [Access control for Managed HSM](access-control.md#role-based-access-control-for-managed-hsm).

# [Azure CLI](#tab/azure-cli)

The following Azure CLI snippets demonstrate how to assign roles for the security team, auditors, and the application:

```azurecli-interactive
# Assign the "Managed HSM Contributor" role to the security team
az role assignment create --assignee-object-id $(az ad group show -g 'Contoso Security Team' --query 'id' -o tsv) --assignee-principal-type Group --role "Managed HSM Contributor"

# Assign the "Managed HSM Administrator" role to the security team
az keyvault role assignment create --hsm-name ContosoMHSM --assignee $(az ad group show -g 'Contoso Security Team' --query 'id' -o tsv) --scope / --role "Managed HSM Administrator"

# Enable logging
hsmresource=$(az keyvault show --hsm-name ContosoMHSM --query id -o tsv)
storageresource=$(az storage account show --name contosologstorage --query id -o tsv)
az monitor diagnostic-settings create --name MHSM-Diagnostics --resource $hsmresource --logs '[{"category": "AuditEvent","enabled": true}]' --storage-account $storageresource

# Assign the "Crypto Auditor" role to auditors
az keyvault role assignment create --hsm-name ContosoMHSM --assignee $(az ad group show -g 'Contoso App Auditors' --query 'objectId' -o tsv) --scope / --role "Managed HSM Crypto Auditor"

# Assign the "Crypto User" role to the VM's managed identity
az keyvault role assignment create --hsm-name ContosoMHSM --assignee $(az vm identity show --name "vmname" --resource-group "ContosoAppRG" --query objectId -o tsv) --scope / --role "Managed HSM Crypto User"
```

# [Azure PowerShell](#tab/azure-powershell)

The following PowerShell snippets demonstrate how to assign roles for the security team, auditors, and the application:

```powershell
# Assign the "Managed HSM Contributor" role to the security team
$securityTeamId = (Get-AzADGroup -DisplayName 'Contoso Security Team').Id
New-AzRoleAssignment -ObjectId $securityTeamId -RoleDefinitionName "Managed HSM Contributor" -PrincipalType Group

# Assign the "Managed HSM Administrator" role to the security team
$hsmName = "ContosoMHSM"
New-AzKeyVaultRoleAssignment -HsmName $hsmName -ObjectId $securityTeamId -RoleDefinitionName "Managed HSM Administrator" -Scope "/"

# Enable logging
$hsmResourceId = (Get-AzKeyVault -HsmName $hsmName).ResourceId
$storageResourceId = (Get-AzStorageAccount -Name "contosologstorage").ResourceId
Set-AzDiagnosticSetting -Name "MHSM-Diagnostics" -ResourceId $hsmResourceId -Category "AuditEvent" -Enabled $true -StorageAccountId $storageResourceId

# Assign the "Crypto Auditor" role to auditors
$auditorsGroupId = (Get-AzADGroup -DisplayName 'Contoso App Auditors').Id
New-AzKeyVaultRoleAssignment -HsmName $hsmName -ObjectId $auditorsGroupId -RoleDefinitionName "Managed HSM Crypto Auditor" -Scope "/"

# Assign the "Crypto User" role to the VM's managed identity
$vmIdentityId = (Get-AzVM -Name "vmname" -ResourceGroupName "ContosoAppRG").Identity.PrincipalId
New-AzKeyVaultRoleAssignment -HsmName $hsmName -ObjectId $vmIdentityId -RoleDefinitionName "Managed HSM Crypto User" -Scope "/"
```

---

For more information on securing your Managed HSM, see [Secure your Managed HSM](access-control.md).

## Next steps
- [Access control for Managed HSM](access-control.md)
- [Secure your Managed HSM](secure-managed-hsm.md)
- [Azure RBAC documentation](/azure/role-based-access-control/overview)
- [Azure RBAC: Built-in roles](/azure/role-based-access-control/built-in-roles)
- [Manage Azure RBAC with Azure CLI](/azure/role-based-access-control/role-assignments-cli)
