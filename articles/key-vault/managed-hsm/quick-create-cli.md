---
title: Quickstart - Provision and activate an Azure Managed HSM
description: Quickstart showing how to provision and activate a managed HSM using Azure CLI
services: key-vault
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: quickstart
ms.date: 03/13/2026
ms.author: mbaldwin
ms.custom: mode-api, devx-track-azurecli 
ms.devlang: azurecli
#Customer intent: As a security admin who is new to Azure, I want to provision and activate a managed HSM
---

# Quickstart: Provision and activate a Managed HSM using Azure CLI

In this quickstart, you create and activate an Azure Key Vault Managed HSM (Hardware Security Module) by using Azure CLI. [!INCLUDE [Managed HSM description](../includes/managed-hsm/intro.md)]

## Prerequisites

[!INCLUDE [Azure subscription prerequisite](../includes/azure-subscription-prerequisite.md)]

You also need:

* Azure CLI version 2.25.0 or later. Run `az --version` to find the version. If you need to install or upgrade, see [Install the Azure CLI]( /cli/azure/install-azure-cli).

[!INCLUDE [cloud-shell-try-it.md](~/reusable-content/ce-skilling/azure/includes/cloud-shell-try-it.md)]

## Sign in to Azure

To sign in to Azure by using the CLI, type:

```azurecli
az login
```

## Create a resource group

A resource group is a logical container into which you deploy and manage Azure resources. The following example creates a resource group named `<resource-group>` in the `<location>` location.

```azurecli-interactive
az group create --name "<resource-group>" --location <location>
```

## Create a Managed HSM

Creating a Managed HSM is a two-step process:
1. Provision a Managed HSM resource.
1. Activate your Managed HSM by downloading an artifact called the *security domain*.

### Provision a Managed HSM

Use the `az keyvault create` command to create a Managed HSM. This script has three mandatory parameters: a resource group name, an HSM name, and the geographic location.

To create a Managed HSM resource, provide the following inputs:
- A resource group where you place the Managed HSM in your subscription.
- An Azure location.
- A list of initial administrators.

The following example creates an HSM named `<hsm-name>` in the resource group `<resource-group>`, residing in the specified location, with **the current signed in user** as the only administrator, and a **7-day retention period** for soft-delete. You continue to pay for the Managed HSM until it's purged in a **soft-delete period**. For more information, see [Managed HSM soft-delete and purge protection](recovery.md#what-are-soft-delete-and-purge-protection) and read more about [Managed HSM soft-delete](soft-delete-overview.md).

```azurecli-interactive
oid=$(az ad signed-in-user show --query id -o tsv)
az keyvault create --hsm-name "<hsm-name>" --resource-group "<resource-group>" --location "<location>" --administrators $oid --retention-days 7
```

> [!NOTE]
> If you use Managed Identities as the initial admins of your Managed HSM, enter the OID/PrincipalID of the Managed Identities after `--administrators` and not the ClientID.

> [!NOTE]
> The create command can take a few minutes. When it returns successfully, you're ready to activate your HSM.

[!INCLUDE [Managed HSM billing warning](../includes/managed-hsm/billing-warning.md)]

The output of this command shows properties of the Managed HSM that you created. The two most important properties are:

* **name**: The name you specified. You use this name for other commands.
* **hsmUri**: The URI for your HSM (for example, `https://<hsm-name>.managedhsm.azure.net`). Applications that use your HSM through its REST API must use this URI.

Your Azure account is now authorized to perform any operations on this Managed HSM. As of yet, nobody else is authorized.

### Activate your Managed HSM

All data plane commands are disabled until you activate the HSM. For example, you can't create keys or assign roles. Only the designated administrators that you assign during the create command can activate the HSM. To activate the HSM, you must download the [Security Domain](security-domain.md).

[!INCLUDE [Security domain prerequisites](../includes/managed-hsm/security-domain-prerequisites.md)]

Use the `az keyvault security-domain download` command to download the security domain and activate your Managed HSM. The following example uses three RSA key pairs (only public keys are needed for this command) and sets the quorum to two.

```azurecli-interactive
az keyvault security-domain download --hsm-name <hsm-name> --sd-wrapping-keys ./certs/cert_0.cer ./certs/cert_1.cer ./certs/cert_2.cer --sd-quorum 2 --security-domain-file <hsm-name>-SD.json
```

Store the security domain file and the RSA key pairs securely. You need them for disaster recovery or for creating another Managed HSM that shares same security domain so the two can share keys.

After successfully downloading the security domain, your HSM will be in active state and ready for you to use.

## Clean up resources

Other quickstarts and tutorials in this collection build upon this quickstart. If you plan to continue on to work with subsequent quickstarts and tutorials, you may wish to leave these resources in place.

When no longer needed, you can use the [az group delete](/cli/azure/group) command to remove the resource group, and all related resources. You can delete the resources as follows:

```azurecli-interactive
az group delete --name <resource-group>
```
[!INCLUDE [Managed HSM cleanup warning](../includes/managed-hsm/cleanup-warning.md)]

## Next steps

In this quickstart, you provisioned a Managed HSM and activated it. To learn more about Managed HSM and how to integrate it with your applications, continue on to these articles.

- Read an [Overview of Managed HSM](overview.md)
- Learn about [Managing keys in a managed HSM](key-management.md)
- Learn about [Role management for a managed HSM](role-management.md)
- Review [Secure your Azure Managed HSM deployment](secure-managed-hsm.md)
