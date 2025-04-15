---
title: What to do if there's an Azure service disruption that affects Managed HSM - Azure Key Vault | Microsoft Docs
description: Learn what to do if there's an Azure service disruption that affects Managed HSM.
services: key-vault
author: msmbaldwin

ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.custom: devx-track-azurecli
ms.topic: tutorial
ms.date: 02/23/2024
ms.author: mbaldwin
---

# Managed HSM disaster recovery

You may wish to create an exact replica of your HSM if the original is lost or unavailable due to any of these reasons:

- It was deleted and then purged.
- A catastrophic failure in the region resulted in all member partitions being destroyed.

You can re-create the HSM instance in the same or a different region if you have:
- The [Security Domain](security-domain.md) of the source HSM.
- The private keys (at least quorum number) that encrypt the security domain.
- The most recent full HSM [backup](backup-restore.md) from the source HSM.

Here are the steps of the disaster recovery procedure:

1. Create a new HSM Instance.
2. Activate "Security Domain recovery". A new RSA key pair (Security Domain Exchange Key) will be generated for Security Domain transfer and sent in response, which will be downloaded as a SecurityDomainExchangeKey (public key).
3. Create and then upload the "Security Domain Transfer File". You'll need the private keys that encrypt the security domain. The private keys are used locally, and never transferred anywhere in this process.
4. Take a backup of the new HSM. A backup is required before any restore, even when the HSM is empty. Backups allow for easy roll-back.
5. Restore the recent HSM backup from the source HSM.

These steps will enable you to manually replicate contents of the HSM to another region. The HSM name (and the service endpoint URI) will be different, so you will have to change your application configuration to make use of these keys from a different location.

## Create a new Managed HSM

Use the `az keyvault create` command to create a Managed HSM. This script has three mandatory parameters: a resource group name, an HSM name, and the geographic location.

You must provide the following inputs to create a Managed HSM resource:

- The name for the HSM.
- The resource group where it will be placed in your subscription.
- The Azure location.
- A list of initial administrators.

The following example creates an HSM named **ContosoMHSM2**, in the resource group  **ContosoResourceGroup**, residing in the **West US 3** location, with **the current signed in user** as the only administrator.

```azurecli-interactive
oid=$(az ad signed-in-user show --query objectId -o tsv)
az keyvault create --hsm-name "ContosoMHSM2" --resource-group "ContosoResourceGroup" --location "westus3" --administrators $oid
```

> [!NOTE]
> The create command can take a few minutes. Once it returns successfully, you are ready to activate your HSM.


> [!WARNING]
> Managed HSM instances are considered always-in-use. If you choose to enable purge protection using the `--enable-purge-protection` flag, you will be billed for the entirety of the retention period.

The output of this command shows properties of the Managed HSM that you've created. The two most important properties are:

* **name**: In the example, the name is ContosoMHSM. You'll use this name for other Key Vault commands.
* **hsmUri**: In the example, the URI is 'https://contosomhsm2.managedhsm.azure.net.' Applications that use your HSM through its REST API must use this URI.

Your Azure account is now authorized to perform any operations on this Managed HSM. As of yet, nobody else is authorized.

## Activate the Security Domain recovery mode

At this point in the normal creation process, we initialize and download the new HSM's Security Domain. However, since we're executing a disaster recovery procedure, we request the HSM to enter Security Domain Recovery Mode and download a Security Domain Exchange Key instead. The Security Domain Exchange Key is an RSA public key that will be used to encrypt the security domain before uploading it to the HSM. The corresponding private key is protected inside the HSM, to keep your Security Domain contents safe during the transfer.

```azurecli-interactive
az keyvault security-domain init-recovery --hsm-name ContosoMHSM2 --sd-exchange-key ContosoMHSM2-SDE.cer
```

## Create a Security Domain Upload blob of the source HSM

For this step you'll need:
- The Security Domain Exchange Key you downloaded in previous step.
- The Security Domain of the source HSM.
- At least quorum number of private keys that were used to encrypt the security domain.

The `az keyvault security-domain restore-blob` command performs following operations:
- Decrypt the source HSM's Security Domain with the private keys you supply.
- Create a Security Domain Upload blob encrypted with the Security Domain Exchange Key we downloaded in the previous step

This step can be performed offline.

In the following example, we use the Security Domain from the **ContosoMHSM**, the 3 of the corresponding private keys, and the Security Domain Exchange Key to create and download an encrypted blob which we will use to upload to **ContosoMHSM2**, which is waiting to receive a Security Domain. 

```azurecli-interactive 
az keyvault security-domain restore-blob --sd-exchange-key ContosoMHSM2-SDE.cer --sd-file ContosoMHSM-SD.json --sd-wrapping-keys cert_0.key cert_1.key cert_2.key --sd-file-restore-blob restore_blob.json 
``` 

## Upload Security Domain Upload blob to destination HSM

We now use the Security Domain Upload blob created in the previous step and upload it to the destination HSM to complete the security domain recovery. The `--restore-blob` flag is used to prevent exposing keys in an online environment.

```azurecli-interactive
az keyvault security-domain upload --hsm-name ContosoMHSM2 --sd-file restore_blob.json --restore-blob
```

Now both the source HSM (ContosoMHSM) and the destination HSM (ContosoMHSM2) have the same security domain. We can now restore a full backup from the source HSM into the destination HSM.

## Backup and restore

It's always a good idea to take a full backup before you execute a full HSM restore, so that you have a restore point in case something goes wrong with the restore. You can do this using one of two methods: user assigned managed identity, or SAS tokens.

### Create a backup (as a restore point) of your new HSM

To create an HSM backup, you'll need:
- A storage account where the backup will be stored
- A blob storage container in this storage account where the backup process will create a new folder to store encrypted backup
- A user assigned managed identity that has the Storage Blob Data Contributor role on the storage account OR storage container SAS token with permissions 'crdw'

We use az keyvault backup command to the HSM backup in the storage container **mhsmbackupcontainer**, which is in the storage account **mhsmdemobackup** in the following examples.

### [User assigned managed identity](#tab/uami)

If using the user assigned managed identity method, we specify the user assigned managed identity with the `--mi-user-assigned` parameter and associate that to the Managed HSM before writing the backup in the below example.

```azurecli-interactive
az keyvault update-hsm --hsm-name ContosoMHSM2 --mi-user-assigned "/subscriptions/subid/resourcegroups/mhsmrgname/providers/Microsoft.ManagedIdentity/userAssignedIdentities/userassignedidentityname"
az keyvault backup start --use-managed-identity true --hsm-name ContosoMHSM2 --storage-account-name mhsmdemobackup --blob-container-name mhsmbackupcontainer
```

### [SAS token](#tab/sas)

If using the SAS token method, we create a SAS token that expires in 30 minutes and provide that to Managed HSM to write the backup in the below example.

```azurecli-interactive
end=$(date -u -d "500 minutes" '+%Y-%m-%dT%H:%MZ')
skey=$(az storage account keys list --query '[0].value' -o tsv --account-name mhsmdemobackup)
az storage container create --account-name  mhsmdemobackup --name mhsmbackupcontainer  --account-key $skey
sas=$(az storage container generate-sas -n mhsmbackupcontainer --account-name mhsmdemobackup --permissions crdw --expiry $end --account-key $skey -o tsv)
az keyvault backup start --hsm-name ContosoMHSM2 --storage-account-name mhsmdemobackup --blob-container-name mhsmbackupcontainer --storage-container-SAS-token $sas
```

---

### Restore backup from source HSM

For this step you need:
- The storage account and the blob container in which the source HSM's backups are stored.
- The folder name from where you want to restore the backup. If you create regular backups, there will be many folders inside this container.

We use az keyvault restore command to the new HSM ContosoMHSM2, using the backup of the source MHSM we are trying to restore, which is in the folder name **mhsm-ContosoMHSM-2020083120161860** found in the storage container **mhsmdemobackupcontainer** of the storage account **ContosoBackup** in the following example.

### [User assigned managed identity](#tab/uami)

If using the user assigned managed identity method, we set the `--use-managed-identity` pramater to "true".

```azurecli-interactive
az keyvault restore start --hsm-name ContosoMHSM2 --storage-account-name ContosoBackup --blob-container-name mhsmdemobackupcontainer --backup-folder mhsm-ContosoMHSM-2020083120161860 --use-managed-identity true
```

### [SAS token](#tab/sas)

If using the SAS token method, we create a SAS token that expires in 30 minutes and provide that to Managed HSM to write the restore.

```azurecli-interactive
end=$(date -u -d "500 minutes" '+%Y-%m-%dT%H:%MZ')
skey=$(az storage account keys list --query '[0].value' -o tsv --account-name ContosoBackup)
sas=$(az storage container generate-sas -n mhsmdemobackupcontainer --account-name ContosoBackup --permissions rl --expiry $end --account-key $skey -o tsv)
az keyvault restore start --hsm-name ContosoMHSM2 --storage-account-name ContosoBackup --blob-container-name mhsmdemobackupcontainer --storage-container-SAS-token $sas --backup-folder mhsm-ContosoMHSM-2020083120161860
```

---

Now you've completed a full disaster recovery process. The contents of the source HSM when the backup was taken are copied to the destination HSM, including all the keys, versions, attributes, tags, and role assignments.


## Next steps

- Learn more about Security Domain see [About Managed HSM Security Domain](security-domain.md)
- Follow [Managed HSM best practices](best-practices.md)
