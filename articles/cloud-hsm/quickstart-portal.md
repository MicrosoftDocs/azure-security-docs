---
title: Deploy Azure Cloud HSM using the Azure portal
description: Learn how to deploy and configure Azure Cloud HSM instances using the Azure portal, including managed identity setup and private endpoint configuration.
author: msmbaldwin
ms.service: azure-cloud-hsm
ms.topic: how-to
ms.date: 05/15/2024
ms.author: mbaldwin

# Customer intent: As a cloud administrator, I want to deploy Azure Cloud HSM instances securely through the Azure portal.
---

# Deploy Azure Cloud HSM using the Azure portal

[Azure Cloud HSM](overview.md) is a highly available, FIPS 140-3 Level 3 validated single-tenant HSM service that enables you to deploy hardware security modules (HSMs) using various methods including Azure CLI, PowerShell, ARM templates, Terraform, or the Azure portal. This article guides you through the step-by-step deployment process using the Azure portal.

## Prerequisites

Before deploying Azure Cloud HSM with a private endpoint:
- An Azure account with an active subscription. If you don't have one, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.
- An existing virtual network (VNet) and subnet. If you need to create these resources, see [Create a virtual network](/azure/virtual-network/quick-create-portal).
- Appropriate permissions to create resources in your subscription, including the ability to create HSM resources and managed identities.
- Network connectivity permissions to configure [private endpoints](/azure/private-link/private-endpoint-overview) if securing your HSM with Private Link.

> [!NOTE]
> If you don't have a VNet and subnet ready, you can still create the HSM first and add network connectivity later, but using private endpoints is strongly recommended for production environments, as described in [Network security](network-security.md).

## Create an Azure Cloud HSM

To create an Azure Cloud HSM via the Azure portal:

1. Sign in to the [Azure portal](https://portal.azure.com).
2. Search for and select **Cloud HSM**.
3. Click **Create**.

:::image type="content" source="./media/cloud-hsm-portal-1.png" lightbox="./media/cloud-hsm-portal-1.png" alt-text="Azure Portal interface showing the Cloud HSM resource creation option":::

> [!IMPORTANT]  
> Managed Identity is supported by Azure Cloud HSM for backup and restore operations only. For more information about backup and restore, see [Back up and restore Azure Cloud HSM resources](backup-restore.md).

## Enter Cloud HSM details

1. Select your Azure subscription.
2. Choose an existing resource group or create a new one. It's recommended to deploy your Cloud HSM resources in a separate resource group from your related client virtual network and virtual machine resources for better management and security isolation.
3. Specify the HSM name, region, and SKU.
4. Keep the default Domain Name Reuse setting as "Tenant Reuse" to prevent malicious subdomain takeover and ensure the FQDN can only be reused within your tenant.

:::image type="content" source="./media/cloud-hsm-portal-2.png" lightbox="./media/cloud-hsm-portal-2.png" alt-text="Cloud HSM basic configuration details form showing subscription, resource group, name, region, and SKU options":::

## Configure managed identity

By default, Azure Cloud HSM is set to "No Identity" since it primarily uses password-based authentication with user management handled directly on the HSM. However, for backup and restore operations, you'll need a Managed Service Identity (MSI).

Consider the following options:

1. **No identity (default)** - Choose this if you don't plan to use backup and restore operations.

   :::image type="content" source="./media/cloud-hsm-portal-3.png" lightbox="./media/cloud-hsm-portal-3.png" alt-text="Identity configuration page showing the No Identity (default) option selected for Cloud HSM":::

2. **User-assigned identity** - Recommended for business continuity and disaster recovery (BCDR). Each Cloud HSM cluster can have only one MSI, but you can use the same MSI for multiple HSMs or assign different ones.

   :::image type="content" source="./media/cloud-hsm-portal-4.png" lightbox="./media/cloud-hsm-portal-4.png" alt-text="Identity configuration page showing User-Assigned Identity option for Cloud HSM backup and restore operations":::

For detailed instructions on configuring a user-assigned identity for backup and restore operations, see [Apply a managed identity and create a storage account](backup-restore.md#apply-a-managed-identity-and-create-a-storage-account).

## Set up networking

For secure connectivity, establish a private endpoint to your Azure Cloud HSM. This requires an existing VNet.

1. Select the subscription containing your VNet.
2. Select your VNet and subnet.
3. Configure the DNS integration settings as needed.

:::image type="content" source="./media/cloud-hsm-portal-5.png" lightbox="./media/cloud-hsm-portal-5.png" alt-text="Networking configuration showing private endpoint setup for secure connectivity to Cloud HSM":::

> [!TIP]
> Private endpoints are crucial for security as they enable secure connections to your Azure Cloud HSM through a private link, ensuring traffic between your virtual network and the service traverses the Microsoft backbone network. This eliminates exposure to the public internet, as described in [Network security](network-security.md).

## Add tags (optional)

Tags are name/value pairs that help organize and categorize resources for management and reporting. This step is optional but recommended for resource organization, especially in enterprise environments.

:::image type="content" source="./media/cloud-hsm-portal-6.png" lightbox="./media/cloud-hsm-portal-6.png" alt-text="Tags configuration interface for organizing Cloud HSM resources with name/value pairs":::

## Review and create HSM

1. Verify all HSM details, including managed identity and networking settings.
2. Click **Create** to begin provisioning your Azure Cloud HSM resource.
3. The portal will display "Deployment is in progress" while resources are being created.

:::image type="content" source="./media/cloud-hsm-portal-7.png" lightbox="./media/cloud-hsm-portal-7.png" alt-text="Azure portal deployment progress screen for Cloud HSM showing 'Deployment is in progress'":::

## Validate and configure your Azure Cloud HSM

When the deployment finishes, you'll see "Your deployment is complete" in the portal.

:::image type="content" source="./media/cloud-hsm-portal-8.png" lightbox="./media/cloud-hsm-portal-8.png" alt-text="Azure portal deployment completion screen showing 'Your deployment is complete' for Cloud HSM":::

> [!IMPORTANT]  
> Azure Cloud HSM activation and configuration cannot be performed through the portal. You must use the Azure Cloud HSM SDK and Client Tools, which you can download from GitHub at [microsoft/MicrosoftAzureCloudHSM: Azure Cloud HSM SDK](https://github.com/microsoft/MicrosoftAzureCloudHSM/releases).

After deployment, you'll need to follow these steps:

1. **Download and install the Azure Cloud HSM SDK** on a VM that has network connectivity to your HSM.
2. **Initialize and configure your HSM** following the detailed steps in the [Azure Cloud HSM Onboarding Guide](onboarding-guide.md).
3. **Establish user management** with appropriate crypto officers and crypto users, as described in [User management in Azure Cloud HSM](user-management.md).
4. **Implement proper key management practices** to ensure optimal security and performance, as outlined in [Key management in Azure Cloud HSM](key-management.md).

## Troubleshooting common deployment issues

If you encounter issues during deployment:

- **Network connectivity problems**: Ensure your VM has proper network access to the HSM. Review [Network security](network-security.md) for best practices.
- **Authentication failures**: Verify you're using the correct credentials format as detailed in [Authentication in Azure Cloud HSM](authentication.md).
- **Client connection errors**: Check if the Azure Cloud HSM client is running and properly configured. See [Troubleshoot Azure Cloud HSM](troubleshoot.md) for common client connection issues.

## Next steps

- [Azure Cloud HSM Onboarding Guide](onboarding-guide.md)
- [Configure backup and restore for Cloud HSM](backup-restore.md) 
- [Secure your Cloud HSM](secure-cloud-hsm.md)
- [Network security for Cloud HSM](network-security.md)
- [Deploy Azure Cloud HSM using Azure PowerShell](quickstart-powershell.md)