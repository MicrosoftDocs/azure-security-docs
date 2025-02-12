---
title: Configure Managed HSM alerts
description: Learn how to configure alerts for Azure Managed HSM to monitor its health and performance.
services: key-vault
author: msmbaldwin
ms.author: mbaldwin
ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: tutorial
ms.date: 01/30/2025
#Customer intent: As a Managed HSM administrator, I want to configure alerts so I can monitor the health and performance of my HSM. 
---

# Configure Managed HSM alerts

After you start to use Azure Managed HSM to store your production keys, it's important to monitor the health of your HSM to make sure that your service operates as intended.

As you start to scale your service, the number of requests sent to your HSM rises. This rise has a potential to increase the latency of your requests. In extreme cases, it can cause your requests to be throttled and affect the performance of your service. You also need to know if your HSM is sending an unusual number of error codes, so you can quickly handle any problems with an access policy or firewall configuration.

This article shows you how to configure alerts at specified thresholds so you can alert your team to take action immediately if your HSM is in an unhealthy state. You can configure alerts that send an email (preferably to a team distribution list), fire an Azure Event Grid notification, or call or text a phone number.

## Alert Types

You can choose between these alert types:

- **Static alert** based on a fixed value
- **Dynamic alert** that notifies you if a monitored metric exceeds the average limit of your HSM a certain number of times within a defined time range

> [!IMPORTANT]
> It can take up to 10 minutes for newly configured alerts to start sending notifications.

This article focuses on alerts for managed HSM.

## Configure an Action Group

An action group is a configurable list of notifications and properties. The first step in configuring alerts is to create an action group and choose an alert type:

1. Select your HSM resource in the Azure portal, and then select **Alerts** under **Monitoring**.
   
   :::image type="content" source="./media/configure-alerts-1.png" alt-text="A screenshot showing the selection of Alerts under Monitoring in the Azure portal." lightbox="./media/configure-alerts-1.png":::
   
1. Select **Create**.
   
   :::image type="content" source="./media/configure-alerts-2.png" alt-text="A screenshot showing the creation of a new alert." lightbox="./media/configure-alerts-2.png":::
   
1. Select **Action group**.
   
   :::image type="content" source="./media/configure-alerts-3.png" alt-text="A screenshot showing the selection of Action group." lightbox="./media/configure-alerts-3.png":::
   
1. Enter **Project** and **Instance** details, and then select **Next**.
   
   :::image type="content" source="./media/configure-alerts-4.png" alt-text="A screenshot showing the entry of Project and Instance details." lightbox="./media/configure-alerts-4.png":::
   
1. Choose the **Notification Type** for your action group. In this example, we create an email and SMS alert. Select **Email/SMS message/Push/Voice**.
   
   :::image type="content" source="./media/configure-alerts-5.png" alt-text="A screenshot showing the selection of Email/SMS message/Push/Voice as the Notification Type." lightbox="./media/configure-alerts-5.png":::
   
1. In the dialog, enter email and SMS details, and then select **OK**.
   
   :::image type="content" source="./media/configure-alerts-6.png" alt-text="A screenshot showing the entry of email and SMS details." lightbox="./media/configure-alerts-6.png":::
   
1. Enter a name for the notification time and select **Next**.
   
   :::image type="content" source="./media/configure-alerts-7.png" alt-text="A screenshot showing the entry of a name for the notification." lightbox="./media/configure-alerts-7.png":::
   
1. Select an **Action type** for your action group. In this example, we create an Event Hubs action. Select **Event Hub**.
   
   :::image type="content" source="./media/configure-alerts-8.png" alt-text="A screenshot showing the selection of Event Hub as the Action type." lightbox="./media/configure-alerts-8.png":::
   
1. Enter **Event Hub namespace** and **name** and select **OK**.
   
   :::image type="content" source="./media/configure-alerts-9.png" alt-text="A screenshot showing the entry of Event Hub namespace and name." lightbox="./media/configure-alerts-9.png":::
   
1. Enter a **Name** for the action.
   
   :::image type="content" source="./media/configure-alerts-10.png" alt-text="A screenshot showing the entry of a name for the action." lightbox="./media/configure-alerts-10.png":::

1. Select **Review + create** and select **Create**.

## Configure Alert Thresholds

Next, create a rule and configure the thresholds that trigger an alert:

1. Select your HSM resource in the Azure portal, and then select **Alerts** under **Monitoring**.
   
   :::image type="content" source="./media/configure-alerts-11.png" alt-text="A screenshot showing the selection of Alerts under Monitoring in the Azure portal." lightbox="./media/configure-alerts-11.png":::

1. Select **Alert rule** under **Create**.
   
   :::image type="content" source="./media/configure-alerts-12.png" alt-text="A screenshot showing the creation of a new alert rule." lightbox="./media/configure-alerts-12.png":::

1. Select the scope of your alert rule. You can select a single HSM or multiple HSMs.

   > [!IMPORTANT]
   > When you're selecting multiple HSMs for the scope of your alerts, all selected HSMs must be in the same region. You have to configure separate alert rules for HSMs in different regions.
   
   :::image type="content" source="./media/configure-alerts-13.png" alt-text="A screenshot showing the selection of the scope of your alert rule." lightbox="./media/configure-alerts-13.png":::
   
1. Select the thresholds that define the logic for your alerts. You can view all available signals by selecting **See all signals**. The Managed HSM team recommends configuring the following thresholds for most applications, but you can adjust them based on your application needs:
   - **Key Vault availability** drops below 100 percent (static threshold)
   - **Key Vault latency** is greater than 1000 ms (static threshold)
   
   > [!NOTE]
   > The intention of the 1000 ms threshold is to notify that the Key Vault service in this region has a workload higher than average. Our SLA for Key Vault operations is several times higher. See the [Service Level Agreement for Online Services](https://azure.microsoft.com/support/legal/sla/) for current SLA. To alert when Key Vault operations are out of SLA, use the thresholds from the SLA documents.
   
   - **Total error codes** are higher than average (dynamic threshold).

   :::image type="content" source="./media/configure-alerts-14.png" alt-text="A screenshot showing the configuration of alert thresholds." lightbox="./media/configure-alerts-14.png":::

1. Select an action to apply to the alert rule. In this example, we add an existing action group. Select the action group and select **Select**.
   
   :::image type="content" source="./media/configure-alerts-15.png" alt-text="A screenshot showing the selection of an action group for the alert rule." lightbox="./media/configure-alerts-15.png":::

1. Enter **Project** and **Alert rule** details, and then select **Next**.
1. Select **Create**.

### Example: Configure a static alert threshold for latency

1. Select **Overall Service Api Latency** as the signal name and select **Apply**.
   
   :::image type="content" source="./media/configure-alerts-16.png" alt-text="A screenshot showing the selection of Overall Service Api Latency as the signal name." lightbox="./media/configure-alerts-16.png":::

   1. Use the following configuration parameters:
      - Set **Threshold** to **Static**.
      - Set **Aggregation type** to **Average**.
      - Set **Operator** to **Greater than**.
      - Set **Threshold value** to **1000**.
      - Set **Check every** to **1 minute**.
      - Set **Lookback period** to **5 Minutes**.
      
      :::image type="content" source="./media/configure-alerts-17.png" alt-text="A screenshot showing the configuration of parameters for the static alert threshold." lightbox="./media/configure-alerts-17.png":::
     
   1. Select **Done**.

   ### Example: Configure an Azure Advisor alert

   To get alerted if a backup has not been taken in the last 30 days, the alert must be set up in Advisor.

   1. Search "Advisor" in the Azure portal and select the "Advisor" service.
      
      :::image type="content" source="./media/configure-alerts-18.png" alt-text="A screenshot showing the search for Advisor in the Azure portal." lightbox="./media/configure-alerts-18.png":::
      
   1. Select **Alerts** under **Monitoring**.
      
      :::image type="content" source="./media/configure-alerts-19.png" alt-text="A screenshot showing the selection of Alerts under Monitoring in Advisor." lightbox="./media/configure-alerts-19.png":::
      
   1. Select **New Advisor Alert**.
      
      :::image type="content" source="./media/configure-alerts-20.png" alt-text="A screenshot showing the creation of a new Advisor alert." lightbox="./media/configure-alerts-20.png":::
      
   1. Select the scope of your alert rule.
   1. Select **Recommendation Type** as the configuration condition.
   1. Search for "Create a backup of HSM" as the recommendation type and select it.
   1. Select an **Action Group**. In this example, we will select an existing action group. You can select up to 5 action groups to attach to an alert rule. Choose **Select existing** and a side panel will pop out. Select the existing action group.
      
      :::image type="content" source="./media/configure-alerts-21.png" alt-text="A screenshot showing the selection of an existing action group." lightbox="./media/configure-alerts-21.png":::
      
      :::image type="content" source="./media/configure-alerts-22.png" alt-text="A screenshot showing the selection of the existing action group from the side panel." lightbox="./media/configure-alerts-22.png":::
      
   1. Give the alert rule a name and select the resource group it applies to. Then, select **Create Alert**.
      
      :::image type="content" source="./media/configure-alerts-23.png" alt-text="A screenshot showing the entry of a name for the alert rule and the selection of the resource group." lightbox="./media/configure-alerts-23.png":::

   ## Next Steps

Use the tools that you set up in this article to actively monitor the health of your key vault:

- [Monitor Azure Managed HSM](logging-azure-monitor.md)
- [Azure Key Vault monitoring data reference](../general/monitor-key-vault-reference.md)
- [Setting up Sentinel for Azure Managed HSM](sentinel.md)
- [Create a log query alert for an Azure resource](/azure/azure-monitor/platform/alerts-log)
