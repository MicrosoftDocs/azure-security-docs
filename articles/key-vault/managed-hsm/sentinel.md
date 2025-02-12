---
title: Setting up Microsoft Sentinel for Azure Managed HSM
description: Learn how to set up Microsoft Sentinel to monitor and detect suspicious activity in your Azure Managed HSM.
services: key-vault
author: msmbaldwin
ms.author: mbaldwin
ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: tutorial
ms.date: 01/30/2025
#Customer intent: As a security administrator, I want to set up Microsoft Sentinel for Azure Managed HSM so I can monitor and detect suspicious activity effectively.
---

# Setting up Microsoft Sentinel for Azure Managed HSM

You can use Microsoft Sentinel to automatically detect suspicious activity on your Azure resources. Microsoft Sentinel comes with many out-of-the-box connectors for Microsoft services, which integrate in real time. You can find the specific "Solution Package" for protecting Azure Key Vaults in the Microsoft Sentinel's Content Hub. You can use this for Managed HSM as well. However, there are a few key steps to take to ensure you are using it for Managed HSM properly.

1. Follow the instructions found in [Quickstart: Onboard to Microsoft Sentinel | Microsoft Learn](/azure/sentinel/quickstart-onboard) to enable Microsoft Sentinel.
2. Navigate to your Microsoft Sentinel workspace, and then select **Content hub** under **Content management**.
  
   :::image type="content" source="./media/sentinel-1.png" alt-text="Content hub under Content management in Microsoft Sentinel workspace" lightbox="./media/sentinel-1.png":::
   
1. Search for **Azure Key Vault** in the Content Hub and select it.
   
   :::image type="content" source="./media/sentinel-2.png" alt-text="Search for Azure Key Vault in the Content Hub" lightbox="./media/sentinel-2.png":::
   
1. Select **Install** on the sidebar that appears.
   
   :::image type="content" source="./media/sentinel-3.png" alt-text="Install option in the sidebar for Azure Key Vault" lightbox="./media/sentinel-3.png":::
   
1. Select **Analytics** under **Configuration**.
   
   :::image type="content" source="./media/sentinel-4.png" alt-text="Analytics under Configuration in Microsoft Sentinel" lightbox="./media/sentinel-4.png":::
   
1. Select **Rule templates**, and then search for **Azure Key Vault** or use the filter to filter **Data sources** to **Azure Key Vault**.
  
   :::image type="content" source="./media/sentinel-5.png" alt-text="Rule templates filtered by Azure Key Vault data source" lightbox="./media/sentinel-5.png":::
   
1. Use the rule template that matches your use case best. In this example, we select **Sensitive Key Vault operations**. In the sidebar that appears, select **Create rule**.
   
   :::image type="content" source="./media/sentinel-6.png" alt-text="Create rule option for Sensitive Key Vault operations" lightbox="./media/sentinel-6.png":::
   
1. In the **Set rule logic** tab, edit the rule query. Change "VAULTS" to "MANAGEDHSMS". In this example, we also changed the `SensitiveOperationList` to include key-related operations only.
   
   :::image type="content" source="./media/sentinel-7.png" alt-text="Set rule logic tab with rule query for Managed HSM" lightbox="./media/sentinel-7.png":::
   
1. In this example, we schedule the query to run once every hour.
   
   :::image type="content" source="./media/sentinel-8.png" alt-text="Schedule query to run every hour" lightbox="./media/sentinel-8.png":::
   
1. Review and save the rule. You should now see the rule you created on the **Analytics** page.
   
   :::image type="content" source="./media/sentinel-9.png" alt-text="Created rule on the Analytics page" lightbox="./media/sentinel-9.png":::

1.  You can test the rule by creating and deleting a key. The `KeyDelete` operation is one of the sensitive operations searched by the Analytic Rule named "Sensitive Azure Managed HSM operations".

## Next Steps

- [Monitor Azure Managed HSM](logging-azure-monitor.md)
- [Configure Managed HSM alerts](configure-alerts.md)
- [Azure Key Vault monitoring data reference](../general/monitor-key-vault-reference.md)
- [Create a log query alert for an Azure resource](/azure/azure-monitor/platform/alerts-log)