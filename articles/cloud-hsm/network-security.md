---
title: Network Security for Azure Cloud HSM
description: Learn how to secure your network configuration for Azure Cloud HSM to prevent unauthorized access and enhance overall security.
author: msmbaldwin
ms.service: azure-cloud-hsm
ms.topic: conceptual
ms.date: 03/20/2025
ms.author: mbaldwin
#customer intent: As a security administrator, I need to understand how to secure my network configuration for Azure Cloud HSM so that I can prevent unauthorized access and enhance overall security.
---

# Network security for Azure Cloud HSM

Ensuring robust network security is crucial when you're using Azure Cloud HSM. This article provides guidance on securing your network configuration to prevent unauthorized access and enhance overall security.

## Private endpoint

To enhance the security of your Azure Cloud HSM deployment, employ private subnets and private endpoints. They enable secure connections to Azure Cloud HSM through a private link, to facilitate private IP address connectivity from your virtual network.

A private endpoint ensures that traffic between your virtual network and the Azure service traverses the Microsoft backbone network for security and private communication. This proactive approach strengthens your application's security posture by minimizing exposure to potential threats from the internet.

## Related content

- [Secure your Azure Cloud HSM deployment](secure-cloud-hsm.md)
- [Key management in Azure Cloud HSM](key-management.md)
- [User management in Azure Cloud HSM](user-management.md)
- [Authentication in Azure Cloud HSM](authentication.md)
- [Overview of Azure Cloud HSM](overview.md)
