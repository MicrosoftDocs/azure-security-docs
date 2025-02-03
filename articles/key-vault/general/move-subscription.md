---
title: Azure Key Vault moving a vault to a different subscription | Microsoft Docs
description: Guidance on moving a key vault to a different subscription.
services: key-vault
author: msmbaldwin

ms.service: azure-key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 01/30/2024
ms.author: mbaldwin 
ms.custom: devx-track-azurepowershell
# Customer intent: As a key vault administrator, I want to move my vault to another subscription.
---

# Moving an Azure Key Vault to another subscription

[!INCLUDE [updated-for-az](~/reusable-content/ce-skilling/azure/includes/updated-for-az.md)]

## Overview

> [!IMPORTANT]
> **Moving a key vault to another subscription will cause a breaking change to your environment.**
> Make sure you understand the impact of this change and follow the guidance in this article carefully before deciding to move key vault to a new subscription.
> If you are using Managed Service Identities (MSI) please read the post-move instructions at the end of the document. 

[Azure Key Vault](overview.md) is automatically tied to the default [Microsoft Entra ID](/azure/active-directory/fundamentals/active-directory-whatis) tenant ID for the subscription in which it is created. You can find tenant ID associated with your subscription by following this [guide](/azure/active-directory-b2c/tenant-management-read-tenant-name). All access policy entries and roles assignments are also tied to this tenant ID.  If you move your Azure subscription from tenant A to tenant B, your existing key vaults will be inaccessible by the service principals (users and applications) in tenant B. To fix this issue, you need to:

> [!NOTE]
> If Key Vault is created through [Azure Lighthouse](/azure/lighthouse/overview), it is tied to managing tenant id instead. Azure Lighthouse is only supported by vault access policy permission model.
> For more information about tenants in Azure Lighthouse, see [Tenants, users, and roles in Azure Lighthouse](/azure/lighthouse/concepts/tenants-users-roles).

* Change the tenant ID associated with all existing key vaults in the subscription to tenant B.
* Remove all existing access policy entries.
* Add new access policy entries associated with tenant B.

For more information about Azure Key Vault and Microsoft Entra ID, see
- [About Azure Key Vault](overview.md)
- [What is Microsoft Entra ID](/azure/active-directory/fundamentals/active-directory-whatis)
- [How to find tenant ID](/azure/active-directory-b2c/tenant-management-read-tenant-name)

## Limitations

> [!IMPORTANT]
> **Key Vaults used for disk encryption cannot be moved**
> If you are using key vault with disk encryption for a VM, the key vault cannot be moved to a different resource group or a subscription while disk encryption is enabled. You must disable disk encryption prior to moving the key vault to a new resource group or subscription. 

Some service principals (users and applications) are bound to a specific tenant. If you move your key vault to a subscription in another tenant, there's a chance that you won't be able to restore access to a specific service principal. Check to make sure that all essential service principals exist in the tenant where you are moving your key vault.

## Prerequisites

* [Contributor](/azure/role-based-access-control/built-in-roles#contributor) level access or higher to the current subscription where your key vault exists. You can assign role using the [Azure portal](/azure/role-based-access-control/role-assignments-portal), [Azure CLI](/azure/role-based-access-control/role-assignments-cli), or [PowerShell](/azure/role-based-access-control/role-assignments-powershell).
* [Contributor](/azure/role-based-access-control/built-in-roles#contributor) level access or higher to the subscription where you want to move your key vault. You can assign role using the [Azure portal](/azure/role-based-access-control/role-assignments-portal), [Azure CLI](/azure/role-based-access-control/role-assignments-cli), or [PowerShell](/azure/role-based-access-control/role-assignments-powershell).
* A resource group in the new subscription. You can create one using the [Azure portal](/azure/azure-resource-manager/management/manage-resource-groups-portal), [PowerShell](/azure/azure-resource-manager/management/manage-resource-groups-powershell), or [Azure CLI](/azure/azure-resource-manager/management/manage-resource-groups-cli).

You can check existing roles using the [Azure portal](/azure/role-based-access-control/role-assignments-list-portal), [PowerShell](/azure/role-based-access-control/role-assignments-list-powershell), [Azure CLI](/azure/role-based-access-control/role-assignments-list-cli), or [REST API](/azure/role-based-access-control/role-assignments-list-rest).


## Moving a key vault to a new subscription

1. Sign in to the [Azure portal](https://portal.azure.com).
2. Navigate to your [key vault](overview.md)
3. Select on the "Overview" tab
4. Select the "Move" button
5. Select "Move to another subscription" from the dropdown options
6. Select the resource group where you want to move your key vault
7. Acknowledge the warning regarding moving resources
8. Select "OK"

## Additional steps when subscription is in a new tenant

If you moved your subscription containing the key vault to a new tenant, you need to manually update the tenant ID and remove old access policies and role assignments. Here are tutorials for these steps in PowerShell and Azure CLI. If you are using PowerShell, you may need to run the Clear-AzContext command to allow you to see resources outside your current selected scope. 

### Update tenant ID in a key vault

```azurepowershell
Select-AzSubscription -SubscriptionId <your-subscriptionId>                # Select your Azure Subscription
$vaultResourceId = (Get-AzKeyVault -VaultName myvault).ResourceId          # Get your key vault's Resource ID 
$vault = Get-AzResource -ResourceId $vaultResourceId -ExpandProperties     # Get the properties for your key vault
$vault.Properties.TenantId = (Get-AzContext).Tenant.TenantId               # Change the Tenant that your key vault resides in
$vault.Properties.AccessPolicies = @()                                     # Access policies can be updated with real
                                                                           # applications/users/rights so that it does not need to be                             # done after this whole activity. Here we are not setting 
                                                                           # any access policies. 
Set-AzResource -ResourceId $vaultResourceId -Properties $vault.Properties  # Modifies the key vault's properties.

Clear-AzContext                                                            #Clear the context from PowerShell
Connect-AzAccount                                                          #Log in again to confirm you have the correct tenant id
````

```azurecli
az account set -s <your-subscriptionId>                                    # Select your Azure Subscription
tenantId=$(az account show --query tenantId)                               # Get your tenantId
az keyvault update -n myvault --remove Properties.accessPolicies           # Remove the access policies
az keyvault update -n myvault --set Properties.tenantId=$tenantId          # Update the key vault tenantId
```
### Update access policies and role assignments

> [!NOTE]
> If Key Vault is using [Azure RBAC](/azure/role-based-access-control/overview) permission model. You need to also remove key vault role assignments. You can remove role assignments using the [Azure portal](/azure/role-based-access-control/role-assignments-portal), [Azure CLI](/azure/role-based-access-control/role-assignments-cli), or [PowerShell](/azure/role-based-access-control/role-assignments-powershell). 

Now that your vault is associated with the correct tenant ID and old access policy entries or role assignments are removed, set new access policy entries or role assignments.

For assigning policies, see:
- [Assign an access policy using Portal](assign-access-policy-portal.md)
- [Assign an access policy using Azure CLI](assign-access-policy-cli.md)
- [Assign an access policy using PowerShell](assign-access-policy-powershell.md)

For adding role assignments, see:
- [Assign Azure roles using the Azure portal](/azure/role-based-access-control/role-assignments-portal)
- [Assign Azure roles using Azure CLI](/azure/role-based-access-control/role-assignments-cli)
- [Assign Azure roles using PowerShell](/azure/role-based-access-control/role-assignments-powershell)


### Update managed identities

If you are transferring entire subscription and using a managed identity for Azure resources, you will need to update it to the new Microsoft Entra tenant as well. For more information on managed identities, [Managed identity overview](/azure/active-directory/managed-identities-azure-resources/overview).

If you are using managed identity, you'll also have to update the identity because the old identity will no longer be in the correct Microsoft Entra tenant. See the following documents to help resolve this issue. 

* [Updating MSI](/azure/active-directory/managed-identities-azure-resources/known-issues#transferring-a-subscription-between-azure-ad-directories)
* [Transfer Subscription to New Directory](/azure/role-based-access-control/transfer-subscription)

## Next steps

- Learn more about [keys, secrets, and certificates](about-keys-secrets-certificates.md)
- For conceptual information, including how to interpret Key Vault logs, see [Key Vault logging](logging.md)
- [Key Vault Developer's Guide](../general/developers-guide.md)
- [Azure Key Vault security features](security-features.md)
- [Configure Azure Key Vault firewalls and virtual networks](network-security.md)
