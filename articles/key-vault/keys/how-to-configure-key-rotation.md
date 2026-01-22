---
title: Configure cryptographic key auto-rotation in Azure Key Vault
description: Use this guide to learn how to configure automated the rotation of a key in Azure Key Vault
services: key-vault
author: msmbaldwin
tags: 'rotation'
ms.custom: devx-track-arm-template, sfi-image-nochange
ms.service: azure-key-vault
ms.subservice: keys
ms.topic: how-to
ms.date: 11/19/2025
ms.author: mbaldwin
---

# Configure cryptographic key auto-rotation in Azure Key Vault

By automating cryptographic key rotation in [Key Vault](../general/overview.md), you can set up Key Vault to automatically create a new key version at a chosen frequency. To set up rotation, use the key rotation policy, which you can define for each individual key. 

To follow cryptographic best practices, rotate encryption keys at least every two years.

For more information about how Key Vault versions objects, see [Key Vault objects, identifiers, and versioning](../general/about-keys-secrets-certificates.md#objects-identifiers-and-versioning). For a comprehensive understanding of autorotation concepts across different asset types in Azure Key Vault, see [Understanding autorotation in Azure Key Vault](../general/autorotation.md).

## Integration with Azure services

This feature enables end-to-end zero-touch rotation for encryption at rest for Azure services with customer-managed key (CMK) stored in Azure Key Vault. Check the specific Azure service documentation to see if the service supports end-to-end rotation. 

> [!NOTE]
> When you rotate customer-managed keys that Azure services use, the time it takes for each service to detect and apply the new key version varies (from one hour to 24 hours or more). Consult the specific Azure service documentation for guidance on when you can safely disable the old key version after rotation.

For more information about data encryption in Azure, see:
- [Azure Encryption at Rest](/azure/security/fundamentals/encryption-atrest#azure-encryption-at-rest-components)
- [Azure services data encryption support table](/azure/security/fundamentals/encryption-models#supporting-services)

## Pricing

There's an extra cost for each scheduled key rotation. For more information, see the [Azure Key Vault pricing page](https://azure.microsoft.com/pricing/details/key-vault/).

## Permissions required

The Key Vault key rotation feature requires key management permissions. Assign the **Key Vault Crypto Officer** role to manage the rotation policy and on-demand rotation.

For more information on how to use Azure RBAC for Key Vault and assign Azure roles, see [Use an Azure RBAC to control access to keys, certificates and secrets](../general/rbac-guide.md).

> [!NOTE]
> If you use the access policies permission model, set the 'Rotate', 'Set Rotation Policy', and 'Get Rotation Policy' key permissions to manage the rotation policy on keys. 

## Key rotation policy

The key rotation policy allows you to configure rotation and Event Grid notifications for near expiry notification.

Key rotation policy settings:

-   Expiry time: key expiration interval. It sets the expiration date on the newly rotated key. It doesn't affect the current key.
-   Enabled/disabled: flag to enable or disable rotation for the key.
-   Rotation types:
    -   Automatically renew at a given time after creation (default).
    -   Automatically renew at a given time before expiry. It requires 'Expiry Time' set on rotation policy and 'Expiration Date' set on the key.
-   Rotation time: key rotation interval. The minimum value is seven days from creation and seven days from expiration time.
-   Notification time: key near expiry event interval for Event Grid notification. It requires 'Expiry Time' set on rotation policy and 'Expiration Date' set on the key. 

> [!IMPORTANT]
> Key rotation generates a new key version of an existing key with new key material. Target services should use versionless key URI to automatically refresh to the latest version of the key. Ensure that your data encryption solution stores versioned key URI with data to point to the same key material for decrypt/unwrap as was used for encrypt/wrap operations to avoid disruption to your services. All Azure services currently follow that pattern for data encryption.

:::image type="content" source="../media/keys/key-rotation/key-rotation-1.png" alt-text="Rotation policy configuration":::

## Configure key rotation policy

Configure the key rotation policy during key creation.

:::image type="content" source="../media/keys/key-rotation/key-rotation-2.png" alt-text="Configure rotation during key creation":::

Configure the rotation policy on existing keys.

:::image type="content" source="../media/keys/key-rotation/key-rotation-3.png" alt-text="Configure rotation on existing key":::

### Azure CLI

Save the key rotation policy to a file. Here's an example of a key rotation policy:

```json
{
  "lifetimeActions": [
    {
      "trigger": {
        "timeAfterCreate": "P18M",
        "timeBeforeExpiry": null
      },
      "action": {
        "type": "Rotate"
      }
    },
    {
      "trigger": {
        "timeBeforeExpiry": "P30D"
      },
      "action": {
        "type": "Notify"
      }
    }
  ],
  "attributes": {
    "expiryTime": "P2Y"
  }
}
```

Set rotation policy on a key passing previously saved file using Azure CLI [az keyvault key rotation-policy update](/cli/azure/keyvault/key/rotation-policy) command. 

```azurecli
az keyvault key rotation-policy update --vault-name <vault-name> --name <key-name> --value </path/to/policy.json>
```

### Azure PowerShell

Set the rotation policy by using the Azure PowerShell [Set-AzKeyVaultKeyRotationPolicy](/powershell/module/az.keyvault/set-azkeyvaultkeyrotationpolicy) cmdlet. 

```powershell
Set-AzKeyVaultKeyRotationPolicy -VaultName <vault-name> -KeyName <key-name> -ExpiresIn (New-TimeSpan -Days 720) -KeyRotationLifetimeAction @{Action="Rotate";TimeAfterCreate= (New-TimeSpan -Days 540)}
```

## Rotation on demand

You can manually invoke key rotation.

### Portal

Select **Rotate Now** to start the rotation process.

:::image type="content" source="../media/keys/key-rotation/key-rotation-4.png" alt-text="Rotation on-demand":::

### Azure CLI

Use the Azure CLI [az keyvault key rotate](/cli/azure/keyvault/key#az-keyvault-key-rotate) command to rotate a key.

```azurecli
az keyvault key rotate --vault-name <vault-name> --name <key-name>
```

### Azure PowerShell

Use the Azure PowerShell [Invoke-AzKeyVaultKeyRotation](/powershell/module/az.keyvault/invoke-azkeyvaultkeyrotation) cmdlet to rotate a key.

```powershell
Invoke-AzKeyVaultKeyRotation -VaultName <vault-name> -Name <key-name>
```

## Configure key near expiry notification

You can configure an expiry notification for the Event Grid key near expiry event. If automated rotation can't be used, like when a key is imported from local HSM, you can configure near expiry notification as a reminder for manual rotation or as a trigger to custom automated rotation through integration with Event Grid. You can set the notification to trigger the near expiry event days, months, or years before the key expires. 

:::image type="content" source="../media/keys/key-rotation/key-rotation-5.png" alt-text="Configure Notification":::

For more information about Event Grid notifications in Key Vault, see
[Azure Key Vault as Event Grid source](/azure/event-grid/event-schema-key-vault?tabs=event-grid-event-schema).

## Configure key rotation by using ARM template

You can configure the key rotation policy by using ARM templates.

> [!NOTE]
> To deploy a key through the control plane, you need the **Key Vault Contributor** role on the Key Vault configured with Azure RBAC.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "vaultName": {
            "type": "String",
            "metadata": {
                "description": "The name of the key vault to be created."
            }
        },
        "keyName": {
            "type": "String",
            "metadata": {
                "description": "The name of the key to be created."
            }
        },
        "rotatationTimeAfterCreate": {
            "defaultValue": "P18M",
            "type": "String",
            "metadata": {
                "description": "Time duration to trigger key rotation. i.e. P30D, P1M, P2Y"
            }
        },
        "expiryTime": {
            "defaultValue": "P2Y",
            "type": "String",
            "metadata": {
                "description": "The expiry time for new key version. i.e. P90D, P2M, P3Y"
            }
        },
        "notifyTime": {
            "defaultValue": "P30D",
            "type": "String",
            "metadata": {
                "description": "Near expiry Event Grid notification. i.e. P30D"
            }
        }

    },
    "resources": [
        {
            "type": "Microsoft.KeyVault/vaults/keys",
            "apiVersion": "2021-06-01-preview",
            "name": "[concat(parameters('vaultName'), '/', parameters('keyName'))]",
            "location": "[resourceGroup().location]",
            "properties": {
                "vaultName": "[parameters('vaultName')]",
                "kty": "RSA",
                "rotationPolicy": {
                    "lifetimeActions": [
                        {
                            "trigger": {
                                "timeAfterCreate": "[parameters('rotatationTimeAfterCreate')]",
                                "timeBeforeExpiry": ""
                            },
                            "action": {
                                "type": "Rotate"
                            }
                        },
                        {
                            "trigger": {
                                "timeBeforeExpiry": "[parameters('notifyTime')]"
                            },
                            "action": {
                                "type": "Notify"
                            }
                        }

                    ],
                    "attributes": {
                        "expiryTime": "[parameters('expiryTime')]"
                    }
                }
            }
        }
    ]
}

```

## Configure key rotation policy governance

By using the Azure Policy service, you can govern the key lifecycle and ensure that all keys are configured to rotate within a specified number of days.

### Create and assign policy definition

1. Go to the Policy resource.
1. Select **Assignments** under **Authoring** on the left side of the Azure Policy page.
1. Select **Assign policy** at the top of the page. This button opens to the Policy assignment page.
1. Enter the following information:
    - Define the scope of the policy by choosing the subscription and resource group over which the policy is enforced. Select by clicking the three-dot button at on **Scope** field.
    - Select the name of the policy definition: "[Keys should have a rotation policy ensuring that their rotation is scheduled within the specified number of days after creation.
](https://ms.portal.azure.com/#view/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd8cf8476-a2ec-4916-896e-992351803c44)"
    - Go to the **Parameters** tab at the top of the page.
        - Set **The maximum days to rotate** parameter to desired number of days for example, 730.
        - Define the desired effect of the policy (Audit, or Disabled). 
1. Fill out any additional fields. Navigate the tabs by clicking on **Previous** and **Next** buttons at the bottom of the page.
1. Select **Review + create**.
1. Select **Create**.

After you assign the built-in policy, it can take up to 24 hours to complete the scan. When the scan finishes, you can see compliance results like the following.

:::image type="content" source="../media/keys/key-rotation/key-rotation-policy.png" alt-text="Screenshot of key rotation policy compliance." lightbox="../media/keys/key-rotation/key-rotation-policy.png":::

## Resources

- [Monitoring Key Vault with Azure Event Grid](../general/event-grid-overview.md)
- [Understanding autorotation in Azure Key Vault](../general/autorotation.md)
- [Use an Azure RBAC to control access to keys, certificates and secrets](../general/rbac-guide.md)
- [Azure Data Encryption At Rest](/azure/security/fundamentals/encryption-atrest)
- [Azure Storage Encryption](/azure/storage/common/storage-service-encryption)
- [Azure Disk Encryption](/azure/virtual-machines/disk-encryption)
- [Automatic key rotation for transparent data encryption](/azure/azure-sql/database/transparent-data-encryption-byok-key-rotation#automatic-key-rotation)
