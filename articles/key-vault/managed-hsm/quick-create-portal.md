---
title: Quickstart - Provision and activate an Azure Managed HSM using the Azure portal
description: Quickstart showing how to provision and activate a managed HSM using the Azure portal
services: key-vault
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: quickstart
ms.date: 04/28/2026
ms.author: mbaldwin
ai-usage: ai-assisted
#Customer intent: As a security admin who is new to Azure, I want to provision and activate a managed HSM using the portal
---

# Quickstart: Provision and activate a Managed HSM using the Azure portal

In this quickstart, you create and activate an Azure Key Vault Managed HSM (Hardware Security Module) by using the Azure portal. [!INCLUDE [Managed HSM description](~/reusable-content/ce-skilling/azure/includes/managed-hsm/intro.md)]

## Prerequisites

[!INCLUDE [Azure subscription prerequisite](~/reusable-content/ce-skilling/azure/includes/azure-subscription-prerequisite.md)]

## Create a Managed HSM

Creating a Managed HSM is a two-step process:
1. Provision a Managed HSM resource.
1. Activate your Managed HSM by downloading an artifact called the *security domain*.

### Provision a Managed HSM

1. Sign in to the [Azure portal](https://portal.azure.com).

1. In the search box, enter **Managed HSM** and select **Managed HSM Pools** from the results.

1. Select **Create**.

1. On the **Basics** tab, provide the following information:

   - **Subscription**: Select the subscription you want to use.
   - **Resource group**: Select **Create new** and enter *myResourceGroup*.
   - **Managed HSM name**: Enter a name for your Managed HSM.

     > [!Important]
     > Each Managed HSM must have a unique name.

   - **Region**: Select **East US** (or your preferred region).
   - **Initial administrator(s)**: Search for and select the Microsoft Entra users or groups to designate as initial administrators.

   :::image type="content" source="media/quick-create-portal/managed-hsm-create-basics.png" alt-text="Screenshot of the Create Azure Key Vault Managed HSM Basics tab in the Azure portal.":::

1. Adjust settings on the **Advanced**, **Networking**, and **Tags** tabs as needed.

1. Select **Review + create**, then select **Create**.

   The deployment takes a few minutes to complete. When it's finished, navigate to the resource to see its Overview page.

   :::image type="content" source="media/quick-create-portal/managed-hsm-overview.png" alt-text="Screenshot of the Managed HSM Overview page in the Azure portal.":::

> [!NOTE]
> The provisioning process can take a few minutes. When it completes successfully, you're ready to activate your HSM.

[!INCLUDE [Managed HSM billing warning](~/reusable-content/ce-skilling/azure/includes/managed-hsm/billing-warning.md)]

### Activate your Managed HSM

[!INCLUDE [Activate HSM intro](~/reusable-content/ce-skilling/azure/includes/managed-hsm/activate-hsm-intro.md)]

[!INCLUDE [Security domain prerequisites](~/reusable-content/ce-skilling/azure/includes/managed-hsm/security-domain-prerequisites.md)]

1. In the Azure portal, navigate to your Managed HSM resource.

1. In the left menu, under **Settings**, select **Security domain**.

1. Follow the portal prompts to upload your RSA public key certificates (minimum three) and set the quorum value.

1. Download the encrypted security domain file.

   <!-- Security domain download screenshot will be added when an activated HSM is available. -->

> [!IMPORTANT]
> Store the security domain file and the RSA private keys in a secure, separate location. You need them to recover the Managed HSM in a disaster recovery scenario. Loss of the security domain can result in permanent loss of access.

[!INCLUDE [Security domain storage](~/reusable-content/ce-skilling/azure/includes/managed-hsm/security-domain-storage.md)]

## Clean up resources

When no longer needed, you can delete the resource group, which deletes the Managed HSM and all related resources:

1. In the Azure portal, search for and select **Resource groups**.
1. Select the resource group (for example, *myResourceGroup*).
1. Select **Delete resource group**.
1. Enter the resource group name and select **Delete**.

[!INCLUDE [Managed HSM cleanup warning](~/reusable-content/ce-skilling/azure/includes/managed-hsm/cleanup-warning.md)]

## Next steps

In this quickstart, you provisioned a Managed HSM and activated it. To learn more about Managed HSM and how to integrate it with your applications, continue on to these articles.

- Read an [Overview of Managed HSM](overview.md)
- Learn about [Managing keys in a managed HSM](key-management.md)
- Learn about [Role management for a managed HSM](role-management.md)
- Review [Secure your Azure Managed HSM deployment](secure-managed-hsm.md)
