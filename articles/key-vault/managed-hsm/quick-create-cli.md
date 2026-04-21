---
title: Quickstart - Provision and activate an Azure Managed HSM
description: Quickstart showing how to provision and activate a managed HSM using Azure CLI
services: key-vault
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: quickstart
ms.date: 04/14/2026
ms.author: mbaldwin
ms.custom: mode-api, devx-track-azurecli 
ms.devlang: azurecli
#Customer intent: As a security admin who is new to Azure, I want to provision and activate a managed HSM
---

# Quickstart: Provision and activate a Managed HSM using Azure CLI

In this quickstart, you create and activate an Azure Key Vault Managed HSM (Hardware Security Module) by using Azure CLI. [!INCLUDE [Managed HSM description](~/reusable-content/ce-skilling/azure/includes/managed-hsm/intro.md)]

## Prerequisites

[!INCLUDE [Azure subscription prerequisite](~/reusable-content/ce-skilling/azure/includes/azure-subscription-prerequisite.md)]

You also need:

* Azure CLI version 2.25.0 or later. Run `az --version` to find the version. If you need to install or upgrade, see [Install the Azure CLI]( /cli/azure/install-azure-cli).

[!INCLUDE [cloud-shell-try-it.md](~/reusable-content/ce-skilling/azure/includes/cloud-shell-try-it.md)]

[!INCLUDE [Sign in to Azure](~/reusable-content/ce-skilling/azure/includes/managed-hsm/cli-sign-in.md)]

## Create a resource group

[!INCLUDE [Create a resource group](~/reusable-content/ce-skilling/azure/includes/create-resource-group-cli.md)]

## Create a Managed HSM

Creating a Managed HSM is a two-step process:
1. Provision a Managed HSM resource.
1. Activate your Managed HSM by downloading an artifact called the *security domain*.

### Provision a Managed HSM

Use the `az keyvault create` command to create a Managed HSM. This script has three mandatory parameters: a resource group name, an HSM name, and the geographic location.

To create a Managed HSM resource, provide the following inputs:
- Managed HSM name: A string of 3 to 24 characters that can contain only numbers (0-9), letters (a-z, A-Z), and hyphens (-).

  > [!Important]
  > Each Managed HSM must have a unique name. Replace `<hsm-name>` with your own unique Managed HSM name in the following examples.

- Resource group name: **myResourceGroup**.
- Location: **EastUS**.
- A list of initial administrators.

The following example creates an HSM named `<hsm-name>` in the resource group *myResourceGroup*, residing in the *EastUS* location, with **the current signed in user** as the only administrator, and a **7-day retention period** for soft-delete. You continue to pay for the Managed HSM until it's purged in a **soft-delete period**. For more information, see [Managed HSM soft-delete and purge protection](recovery.md#what-are-soft-delete-and-purge-protection) and read more about [Managed HSM soft-delete](soft-delete-overview.md).

```azurecli-interactive
oid=$(az ad signed-in-user show --query id -o tsv)
az keyvault create --hsm-name "<hsm-name>" --resource-group "myResourceGroup" --location "EastUS" --administrators $oid --retention-days 7
```

If you use Managed Identities as the initial admins of your Managed HSM, enter the OID/PrincipalID of the Managed Identities after `--administrators` and not the ClientID.

The create command can take a few minutes. When it returns successfully, you're ready to activate your HSM.

[!INCLUDE [Managed HSM billing warning](~/reusable-content/ce-skilling/azure/includes/managed-hsm/billing-warning.md)]

The output of this command shows properties of the Managed HSM that you created. The two most important properties are:

* **name**: The name you specified. You use this name for other commands.
* **hsmUri**: The URI for your HSM (for example, `https://<hsm-name>.managedhsm.azure.net`). Applications that use your HSM through its REST API must use this URI.

Your Azure account is now authorized to perform any operations on this Managed HSM. As of yet, nobody else is authorized.

### Activate your Managed HSM

[!INCLUDE [Activate HSM intro](~/reusable-content/ce-skilling/azure/includes/managed-hsm/activate-hsm-intro.md)]

[!INCLUDE [Security domain prerequisites](~/reusable-content/ce-skilling/azure/includes/managed-hsm/security-domain-prerequisites.md)]

Use the `az keyvault security-domain download` command to download the security domain and activate your Managed HSM. The following example uses three RSA key pairs (only public keys are needed for this command) and sets the quorum to two.

```azurecli-interactive
az keyvault security-domain download --hsm-name <hsm-name> --sd-wrapping-keys ./certs/cert_0.cer ./certs/cert_1.cer ./certs/cert_2.cer --sd-quorum 2 --security-domain-file <hsm-name>-SD.json
```

[!INCLUDE [Security domain storage](~/reusable-content/ce-skilling/azure/includes/managed-hsm/security-domain-storage.md)]

## Clean up resources

[!INCLUDE [Delete resource group](~/reusable-content/ce-skilling/azure/includes/delete-resource-group-cli.md)]
[!INCLUDE [Managed HSM cleanup warning](~/reusable-content/ce-skilling/azure/includes/managed-hsm/cleanup-warning.md)]

## Next steps

In this quickstart, you provisioned a Managed HSM and activated it. To learn more about Managed HSM and how to integrate it with your applications, continue on to these articles.

- Read an [Overview of Managed HSM](overview.md)
- Learn about [Managing keys in a managed HSM](key-management.md)
- Learn about [Role management for a managed HSM](role-management.md)
- Review [Secure your Azure Managed HSM deployment](secure-managed-hsm.md)
