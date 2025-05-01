---
title: Network Security for Azure Cloud HSM
description: Learn how to secure your network configuration for Azure Cloud HSM to prevent unauthorized access and enhance overall security.
author: msmbaldwin
ms.service: azure-cloud-hsm
ms.topic: conceptual
ms.date: 03/20/2025
ms.author: mbaldwin
# Customer intent: As a security administrator, I need to understand how to secure my network configuration for Azure Cloud HSM to prevent unauthorized access and enhance overall security.
---

# Network security

Ensuring robust network security is crucial when using Azure Cloud HSM. This article provides guidance on securing your network configuration to prevent unauthorized access and enhance overall security.

## Private endpoint

To enhance the security of your Azure Cloud HSM, employ private subnets and private endpoints. These enable secure connections to your Azure Cloud HSM through a private link, facilitating private IP address connectivity from your virtual network. By doing so, you eliminate the need to expose your services to the public internet.  

Azure Private Endpoint ensures that traffic between your virtual network and the Azure service traverses the Microsoft backbone network, guaranteeing secure and private communication. This proactive approach strengthens your application's security posture by minimizing exposure to potential threats from the internet.

## Next steps

- [Secure your Azure Cloud HSM](secure-cloud-hsm.md)
- [Key Management in Azure Cloud HSM](key-management.md)
- [User Management in Azure Cloud HSM](user-management.md)
- [Authentication in Azure Cloud HSM](authentication.md)
- [Overview of Azure Cloud HSM](overview.md)
