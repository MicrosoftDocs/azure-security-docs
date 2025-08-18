---
title: Microsoft Defender for Resource Manager - Benefits and Features
description: Learn about the benefits and features of Microsoft Defender for Resource Manager to protect your Azure resources from potential threats.
ms.date: 05/13/2025
ms.topic: overview
#customer intent: As a security operations team, I want to learn about the benefits and features of Microsoft Defender for Resource Manager so that I can protect the resource management layer of my Azure resources.
---

# Overview of Microsoft Defender for Resource Manager

[Azure Resource Manager](/azure/azure-resource-manager/management/overview) is the deployment and management service for Azure. It provides a management layer that enables you to create, update, and delete resources in your Azure account. You use management features, like access control, locks, and tags, to secure and organize your resources after deployment.

The cloud management layer is a crucial service connected to all your cloud resources. Because of this, it is also a potential target for attackers. Consequently, we recommend security operations teams monitor the resource management layer closely.

Microsoft Defender for Resource Manager automatically monitors the resource management operations in your organization, whether they're performed through the Azure portal, Azure REST APIs, Azure CLI, or other Azure programmatic clients. Defender for Cloud runs advanced security analytics to detect threats and alerts you about suspicious activity.

## Availability

**Microsoft Defender for Resource Manager** is billed as shown on the [pricing page](https://azure.microsoft.com/pricing/details/defender-for-cloud/). You can also [estimate costs with the Defender for Cloud cost calculator](cost-calculator.md).

For cloud availability, see the [Defender for Cloud support matrices for Azure commercial/other clouds](support-matrix-cloud-environment.md).

## What are the benefits of Microsoft Defender for Resource Manager?

Microsoft Defender for Resource Manager protects against issues including:

- **Suspicious resource management operations**, such as operations from malicious IP addresses, disabling antimalware, and suspicious scripts running in VM extensions
- **Use of exploitation toolkits** like Microburst or PowerZure (supported in certain versions)

- **Lateral movement** from the Azure management layer to the Azure resources data plane

:::image type="content" source="media/defender-for-resource-manager-introduction/consistent-management-layer-with-defender.png" alt-text="Azure Resource Manager overview diagram.":::

A full list of the alerts provided by Microsoft Defender for Resource Manager is on the [alerts reference page](alerts-resource-manager.md).

## Next steps

In this article, you learned about Microsoft Defender for Resource Manager.

> [!div class="nextstepaction"]
> [Enable enhanced protections](enable-enhanced-security.md)

For related material, see the following article:

- Security alerts might be generated or received by Defender for Cloud from different security products. To export all of these alerts to Microsoft Sentinel, any third-party SIEM, or any other external tool, follow the instructions in [Exporting alerts to a SIEM solution](continuous-export.md).
