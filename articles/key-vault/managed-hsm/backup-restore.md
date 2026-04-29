---
title: Full backup/restore and selective restore for Azure Managed HSM
description: This document explains full backup/restore and selective restore.
services: key-vault
author: msmbaldwin
tags: azure-key-vault
ms.custom: devx-track-azurecli

ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: tutorial
ms.date: 04/28/2026
ms.author: mbaldwin
ai-usage: ai-assisted
# Customer intent: As a developer using Key Vault I want to know the best practices so I can implement them.
---
# Full backup and restore and selective key restore

> [!NOTE]
> This feature is only available for the resource type managed HSM.

Managed HSM supports creating a full backup of the entire contents of the HSM, including all keys, versions, attributes, tags, and role assignments. The backup process encrypts the data by using cryptographic keys associated with the HSM's security domain.

Backup is a data plane operation. The caller initiating the backup operation must have permission to perform dataAction **Microsoft.KeyVault/managedHsm/backup/start/action**.

Only the following built-in roles have permission to perform a full backup:
- Managed HSM Administrator
- Managed HSM Backup

To back up and restore your Managed HSM, assign a user-assigned managed identity (UAMI) to the Managed HSM service. You can use a UAMI regardless of whether your storage account has public network access or private network access enabled. If the storage account is behind a private endpoint, the UAMI method works with trusted service bypass to allow for backup and restore.

To execute a full backup, provide the following information:
- HSM name or URL
- Storage account name
- Storage account blob storage container
- User-assigned managed identity

[!INCLUDE [cloud-shell-try-it.md](~/reusable-content/ce-skilling/azure/includes/cloud-shell-try-it.md)]

#### Prerequisites for backing up and restoring by using user-assigned managed identity

1. Ensure you have Azure CLI version 2.56.0 or later. Run `az --version` to find the version. If you need to install or upgrade, see [Install the Azure CLI](/cli/azure/install-azure-cli).
1. Create a user-assigned managed identity.
1. Create a storage account (or use an existing storage account). The storage account can't have an immutability policy applied to it.
1. If public network access is disabled on your storage account, enable trusted service bypass on the storage account in the **Networking** tab, under **Exceptions**.
1. Provide **Storage Blob Data Contributor** role access to the user-assigned managed identity created in step 2, by going to the **Access Control** tab on the portal and selecting **Add Role Assignment**. Then select **managed identity** and select the managed identity created in step 2 -> **Review + Assign**
1. Create the Managed HSM and associate the managed identity:
   ```azurecli-interactive
   az keyvault create --hsm-name <hsm-name> -l <location> --retention-days 7 --administrators "<initial-admin>" --mi-user-assigned "/subscriptions/<subscription-id>/resourcegroups/<resource-group>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<managed-identity-name>" 
   ```
 If you have an existing Managed HSM, associate the managed identity by updating the MHSM with the following command. 
  ```azurecli-interactive
   az keyvault update-hsm --hsm-name <hsm-name> --mi-user-assigned "/subscriptions/<subscription-id>/resourcegroups/<resource-group>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<managed-identity-name>" 
   ```

## Full backup

Backup is a long running operation but it immediately returns a job ID. You can check the status of the backup process by using this job ID. The backup process creates a folder inside the designated container with the following naming pattern: **`mhsm-{HSM_NAME}-{YYYY}{MM}{DD}{HH}{mm}{SS}`**. In this pattern, `HSM_NAME` is the name of the managed HSM being backed up, and `YYYY`, `MM`, `DD`, `HH`, `mm`, and `SS` are the year, month, date, hour, minutes, and seconds of the date and time in UTC when the backup command was received.

While the backup is in progress, the HSM might not operate at full throughput as some HSM partitions are busy performing the backup operation.

> [!NOTE]
> Backups to storage accounts with an immutability policy applied aren't supported.

# [Azure portal](#tab/azure-portal)

1. In the [Azure portal](https://portal.azure.com), navigate to your Managed HSM resource.

1. In the left menu, under **Settings**, select **Backup**.

1. Provide the storage account and container details, then select **Start Backup** to initiate the backup.

   :::image type="content" source="media/backup-restore/managed-hsm-backup-blade.png" alt-text="Screenshot of the Managed HSM Backup blade in the Azure portal.":::

# [Azure CLI](#tab/azure-cli)

### Backup HSM
```azurecli-interactive
az keyvault backup start --use-managed-identity true --hsm-name <hsm-name> --storage-account-name <storage-account-name> --blob-container-name <container-name>
  ```

# [Azure PowerShell](#tab/azure-powershell)

### Backup HSM

```azurepowershell-interactive
Backup-AzKeyVault -HsmName <hsm-name> -StorageAccountName <storage-account-name> -StorageContainerName <container-name> -UseUserManagedIdentity
```

---

## Full restore

Full restore restores the contents of the HSM from a previous backup, including all keys, versions, attributes, tags, and role assignments. The process removes everything currently stored in the HSM and returns it to the same state it was in when the source backup was created.

> [!IMPORTANT]
> Full restore is a destructive and disruptive operation. Therefore, you must complete a full backup of the HSM you're restoring at least 30 minutes before a `restore` operation.

Restore is a data plane operation. The caller starting the restore operation must have permission to perform dataAction **Microsoft.KeyVault/managedHsm/restore/start/action**. The source HSM where you created the backup and the destination HSM where you perform the restore **must** have the same Security Domain. See more [about Managed HSM Security Domain](security-domain.md).

You can execute a full restore by using a user-assigned managed identity. To execute a full restore, provide the following information:
- HSM name or URL
- Storage account name
- Storage account blob container
- User-assigned managed identity
- Storage container folder name where the source backup is stored

Restore is a long running operation but it immediately returns a Job ID. You can check the status of the restore process by using this Job ID. When the restore process is in progress, the HSM enters a restore mode and all data plane commands (except check restore status) are disabled.

# [Azure portal](#tab/azure-portal)

1. In the [Azure portal](https://portal.azure.com), navigate to your Managed HSM resource.

1. In the left menu, under **Settings**, select **Restore**.

1. Provide the storage account, container, and backup folder details, then initiate the restore.

   :::image type="content" source="media/backup-restore/managed-hsm-restore-blade.png" alt-text="Screenshot of the Managed HSM Restore blade in the Azure portal.":::

# [Azure CLI](#tab/azure-cli)

### Restore HSM
```azurecli-interactive
az keyvault restore start --hsm-name <hsm-name> --storage-account-name <storage-account-name> --blob-container-name <container-name> --backup-folder <backup-folder> --use-managed-identity true
  ```

# [Azure PowerShell](#tab/azure-powershell)

### Restore HSM

```azurepowershell-interactive
Restore-AzKeyVault -HsmName <hsm-name> -StorageAccountName <storage-account-name> -StorageContainerName <container-name> -BackupFolder <backup-folder> -UseUserManagedIdentity
```

---

## Selective key restore

Selective key restore restores one key with all its key versions from a previous backup to an HSM. The key must be purged for selective key restore to work. If you're attempting to recover a soft-deleted key, use key recover. Learn more about [key recover](key-management.md).

# [Azure portal](#tab/azure-portal)

1. In the [Azure portal](https://portal.azure.com), navigate to your Managed HSM resource.

1. In the left menu, under **Settings**, select **Restore**, then choose the option to restore a single key from the backup.

# [Azure CLI](#tab/azure-cli)

### Selective key restore
```
az keyvault restore start --hsm-name <hsm-name> --storage-account-name <storage-account-name> --blob-container-name <container-name> --backup-folder <backup-folder> --use-managed-identity true --key-name <key-name>
  ```

# [Azure PowerShell](#tab/azure-powershell)

### Selective key restore

```azurepowershell-interactive
Restore-AzKeyVault -HsmName <hsm-name> -StorageAccountName <storage-account-name> -StorageContainerName <container-name> -BackupFolder <backup-folder> -UseUserManagedIdentity -KeyName <key-name>
```

---

## Next steps
- See [Manage a Managed HSM using the Azure CLI](key-management.md).
- Learn more about [Managed HSM Security Domain](security-domain.md).
