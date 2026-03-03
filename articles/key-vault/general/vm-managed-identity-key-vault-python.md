---
title: Read a Key Vault secret from a VM with managed identity in Python
description: End-to-end guidance to read a secret from Azure Key Vault from an Azure VM using a system-assigned managed identity and the Azure SDK for Python.
author: garrodonnell
ms.author: godonnell
ms.date: 03/03/2026
ms.service: azure-key-vault
ms.subservice: general
ms.topic: how-to
ms.devlang: python
ai-usage: ai-assisted
#customer intent: As a developer I want to read secrets from Azure Key Vault on an Azure VM without storing credentials, by using a system-assigned managed identity and the Azure SDK for Python.
---

# Read a Key Vault secret from a VM with managed identity in Python

Use this guide when your code runs on an Azure virtual machine and needs to retrieve secrets from Azure Key Vault without storing credentials. With a system-assigned managed identity, your VM gets an identity in Microsoft Entra ID and your app authenticates by using `DefaultAzureCredential` with no secrets in code.

This approach is best for apps running on a single VM where the identity lifecycle should match the VM lifecycle. If you need to reuse an identity across multiple resources or pre-authorize across fleets, use a user-assigned managed identity instead.

## Prerequisites

- An Azure subscription. If you don't have one, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- An Azure VM that you can modify.
- A Python environment on the VM.
- Familiarity with [Key Vault basic concepts](basic-concepts.md) and [Azure role-based access control](/azure/role-based-access-control/overview) is helpful.

## Understand how managed identity works on VMs

Review what system-assigned and user-assigned identities are, how they're created, and their lifecycle. For more information, see [How managed identities for Azure resources work with Azure virtual machines](/entra/identity/managed-identities-azure-resources/how-managed-identities-work-vm).

## Implement the scenario end to end

Create a vault, enable managed identity on the VM, grant Key Vault permissions through RBAC, and retrieve a secret by using `azure-identity` and `azure-keyvault-secrets`. For the complete walkthrough, see [Tutorial: Use Azure Key Vault with a virtual machine in Python](tutorial-python-virtual-machine.md).

After you complete the tutorial, run the sample from the VM to read a known secret. A successful read with no credential prompts confirms that managed identity is configured correctly.

## Review the credential chain

Learn how `DefaultAzureCredential` resolves credentials locally versus in Azure to avoid surprises when you promote from development to your VM. For more information, see [Walkthrough: Integrated authentication for Python apps with Azure services](/azure/developer/python/walkthrough-tutorial-authentication-01).

> [!TIP]
> If you receive a 403 or unauthorized error, confirm that the VM's identity has a Key Vault data-plane role such as **Key Vault Secrets User** on the target vault through RBAC. The [tutorial](tutorial-python-virtual-machine.md) shows where to assign roles.

> [!TIP]
> If you encounter credential resolution issues, ensure managed identity is enabled on the VM and that no local development credentials override the VM's identity. The [Python identity walkthrough](/azure/developer/python/walkthrough-tutorial-authentication-01) explains the credential chain.

## Next steps

- [Tutorial: Use Azure Key Vault with a virtual machine in Python](tutorial-python-virtual-machine.md)
- [How managed identities for Azure resources work with Azure virtual machines](/entra/identity/managed-identities-azure-resources/how-managed-identities-work-vm)
- [Azure Key Vault developer's guide](developers-guide.md)