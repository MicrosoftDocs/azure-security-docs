---
title: Deploy Azure Cloud HSM by Using the Azure Portal
description: Learn how to deploy and configure Azure Cloud HSM by using the Azure portal, including managed identity setup and private endpoint configuration.
author: msmbaldwin
ms.service: azure-cloud-hsm
ms.topic: quickstart
ms.date: 05/15/2024
ms.author: mbaldwin

#customer intent: As a cloud administrator, I want to deploy Azure Cloud HSM instances securely through the Azure portal.
---

# Quickstart: Deploy Azure Cloud HSM by using the Azure portal

[Azure Cloud HSM](overview.md) is a highly available, FIPS 140-3 Level 3 validated single-tenant service that enables you to deploy hardware security modules (HSMs) by using various methods. These methods include the Azure CLI, Azure PowerShell, Azure Resource Manager templates (ARM templates), Terraform, or the Azure portal. This quickstart guides you through the deployment process in the Azure portal.

## Prerequisites

- An Azure account with an active subscription. If you don't have one, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.
- Appropriate permissions to create resources in your subscription, including the ability to create HSM resources and managed identities.
- For production environments, an existing virtual network and subnet for configuring [private endpoints](/azure/private-link/private-endpoint-overview).

> [!NOTE]
> If you don't have a virtual network and subnet ready, you can still create the HSM first and add network connectivity later. We strongly recommend that you use private endpoints for production environments, as described in [Network security for Azure Cloud HSM](network-security.md).

## Create an Azure Cloud HSM resource

To create an Azure Cloud HSM resource via the Azure portal:

1. Sign in to the [Azure portal](https://portal.azure.com).
2. Search for and select **Azure Cloud HSMs**.
3. Select **Create**.

:::image type="content" source="./media/cloud-hsm-portal-1.png" lightbox="./media/cloud-hsm-portal-1.png" alt-text="Screenshot of the Azure portal that shows selections for creating a Cloud HSM resource.":::

## Configure basic settings

On the **Basics** tab:

1. Select your Azure subscription.
2. Choose an existing resource group or create a new one.

   We recommend that you deploy your Cloud HSM resources in a separate resource group from your related client virtual network and virtual machine (VM) resources. This separation provides for better management and security isolation.
3. Specify the HSM **Name**, **Region**, and **Sku** (product tier) values.

   The HSM name must be unique. If you specify an HSM resource name that already exists in the chosen region, your deployment will fail.
4. Keep the default **Domain Name Reuse** setting as **Tenant Reuse**. This setting helps prevent malicious subdomain takeover and ensure that the fully qualified domain name (FQDN) can be reused only within your tenant.

:::image type="content" source="./media/cloud-hsm-portal-2.png" lightbox="./media/cloud-hsm-portal-2.png" alt-text="Screenshot of the Cloud HSM basic configuration settings for subscription, resource group, name, region, and product tier.":::

## Configure a managed identity (optional)

On the **Managed Identity** tab, consider the following options:

- **No identity**: Select this option if you don't plan to use backup and restore operations.

   By default, Azure Cloud HSM is set to **No Identity** because it primarily uses password-based authentication with user management handled directly on the HSM. However, for backup and restore operations, you need a managed identity.

   :::image type="content" source="./media/cloud-hsm-portal-3.png" lightbox="./media/cloud-hsm-portal-3.png" alt-text="Screenshot of the tab for identity configuration that shows the default No Identity option selected for Cloud HSM.":::

- **User-Assigned Identity**: We recommend that you select this option for business continuity and disaster recovery (BCDR). Each Cloud HSM cluster can have only one managed identity, but you can use the same managed identity for multiple HSMs or assign different ones.

   :::image type="content" source="./media/cloud-hsm-portal-4.png" lightbox="./media/cloud-hsm-portal-4.png" alt-text="Screenshot of the tab for identity configuration that shows the User-Assigned Identity option selected for Cloud HSM backup and restore operations.":::

For detailed instructions on configuring a user-assigned identity for backup and restore operations, see [Apply a managed identity and create a storage account](backup-restore.md#apply-a-managed-identity-and-create-a-storage-account).

## Set up networking (recommended)

For secure connectivity, establish a private endpoint to Azure Cloud HSM. This task requires an existing virtual network.

On the **Networking** tab:

1. Select the subscription that contains your virtual network.
2. Select your virtual network and subnet.
3. Configure the DNS integration settings as needed.

:::image type="content" source="./media/cloud-hsm-portal-5.png" lightbox="./media/cloud-hsm-portal-5.png" alt-text="Screenshot of the tab for network configuration, with the section for setting up a private endpoint for secure connectivity to Cloud HSM.":::

> [!TIP]
> Private endpoints are crucial for security. They enable secure connections to Azure Cloud HSM through a private link. These connections ensure that traffic between your virtual network and the service traverses the Microsoft backbone network. This configuration eliminates exposure to the public internet, as described in [Network security for Azure Cloud HSM](network-security.md).

## Add tags (optional)

Tags are name/value pairs that help you organize and categorize resources for management and reporting. On the **Tags** tab, you can enter details to create tags. This step is optional but recommended for resource organization, especially in enterprise environments.

:::image type="content" source="./media/cloud-hsm-portal-6.png" lightbox="./media/cloud-hsm-portal-6.png" alt-text="Screenshot of the tab for creating tags to organize Cloud HSM resources with name/value pairs.":::

## Deploy your Cloud HSM resource

On the **Review + submit** tab:

1. Review all HSM details, including managed identity and network settings.

2. Select **Create** to begin provisioning your Azure Cloud HSM resource.

:::image type="content" source="./media/cloud-hsm-portal-7.png" lightbox="./media/cloud-hsm-portal-7.png" alt-text="Screenshot of the Azure portal that shows the tab for reviewing resource details, along with the Create button.":::

The portal displays **Deployment is in progress** while it creates the resource. When the deployment finishes, the portal displays **Your deployment is complete**.

:::image type="content" source="./media/cloud-hsm-portal-8.png" lightbox="./media/cloud-hsm-portal-8.png" alt-text="Screenshot of the Azure portal pane that shows the successfully completed deployment of a Cloud HSM resource.":::

## Initialize and configure your HSM

You can't activate or configure Azure Cloud HSM through the portal. You must use the Azure Cloud HSM SDK and client tools.

After you deploy your Cloud HSM resource, follow these steps:

1. Download and install the Azure Cloud HSM SDK from [GitHub](https://github.com/microsoft/MicrosoftAzureCloudHSM/releases) on a VM that has network connectivity to your HSM.
2. Initialize and configure your HSM by following the detailed steps in the [Azure Cloud HSM onboarding guide](onboarding-guide.md).
3. Establish user management with appropriate cryptographic officers and users, as described in [User management in Azure Cloud HSM](user-management.md).
4. Implement proper key management practices to help ensure optimal security and performance, as outlined in [Key management in Azure Cloud HSM](key-management.md).

## Clean up resources

If you created a resource group solely for this quickstart and you don't need to keep these resources, you can delete the entire resource group:

1. In the Azure portal, go to the resource group that contains your Cloud HSM resources.
2. Select **Delete resource group**.
3. Enter the resource group name to confirm deletion, and then select **Delete**.

## Troubleshoot common deployment problems

If you encounter problems during deployment:

- **Resource name conflicts**: Ensure that your HSM name is unique in the region. If deployment fails with a naming conflict, try a different name.
- **Network connectivity problems**: Ensure that your VM has proper network access to the HSM. For best practices, review [Network security for Azure Cloud HSM](network-security.md).
- **Authentication failures**: Verify that you're using the correct format for credentials, as detailed in [Authentication in Azure Cloud HSM](authentication.md).
- **Client connection errors**: Check that the Azure Cloud HSM client is running and properly configured. For common client connection problems, see [Troubleshoot Azure Cloud HSM](troubleshoot.md).

## Related content

- [Azure Cloud HSM onboarding guide](onboarding-guide.md)
- [Back up and restore Azure Cloud HSM resources](backup-restore.md)
- [Secure your Azure Cloud HSM deployment](secure-cloud-hsm.md)
- [Network security for Azure Cloud HSM](network-security.md)
- [Deploy Azure Cloud HSM by using Azure PowerShell](quickstart-powershell.md)
