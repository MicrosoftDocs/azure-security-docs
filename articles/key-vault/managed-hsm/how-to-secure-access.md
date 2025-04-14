---
title: How to secure access to your managed HSMs
description: Learn how to implement access control for Managed HSM using Azure RBAC and Managed HSM local RBAC.
services: key-vault
author: msmbaldwin
ms.custom: devx-track-azurecli
ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: how-to
ms.date: 01/30/2024
ms.author: mbaldwin
# Customer intent: As a managed HSM administrator, I want to set access control and configure the Managed HSM, so that I can ensure it's secure and auditors can properly monitor all activities for this Managed HSM.
---

# Secure access to your managed HSMs

This tutorial provides a practical implementation example of access control for Azure Key Vault Managed HSM. You'll learn how to implement separation of duties using Azure RBAC and Managed HSM local RBAC through a realistic scenario.

> [!IMPORTANT]
> Before proceeding with this tutorial, ensure you understand the Managed HSM access control model, including the differences between control plane and data plane access. For this conceptual foundation, see [Managed HSM access control](access-control.md).

## Prerequisites

* A subscription to Microsoft Azure. If you don't have one, you can sign up for a [free trial](https://azure.microsoft.com/pricing/free-trial).
* The Azure CLI version 2.25.0 or later. Run `az --version` to find the version. If you need to install or upgrade, see [Install the Azure CLI](/cli/azure/install-azure-cli).
* A managed HSM in your subscription. See [Quickstart: Provision and activate a managed HSM using Azure CLI](quick-create-cli.md) to provision and activate a managed HSM.

[!INCLUDE [cloud-shell-try-it.md](~/reusable-content/ce-skilling/azure/includes/cloud-shell-try-it.md)]

## Sign in to Azure

To sign in to Azure using the CLI you can type:

```azurecli
az login
```

For more information on login options via the CLI, see [sign in with Azure CLI](/cli/azure/authenticate-azure-cli)

## Understand the example scenario

In this example, we're developing an application that uses an RSA 2,048-bit key for sign operations. Our application runs in an Azure virtual machine (VM) with a [managed identity](/azure/active-directory/managed-identities-azure-resources/overview). The RSA key used for signing is stored in our managed HSM.

### Define roles and responsibilities

We have identified following roles who manage, deploy, and audit our application:

- **Security team**: IT staff from the office of the CSO (Chief Security Officer) or similar contributors. The security team is responsible for the proper safekeeping of keys. The keys include RSA or EC keys for signing, and RSA or AES keys for data encryption.
- **Developers and operators**: The staff who develop the application and deploy it in Azure. The members of this team aren't part of the security staff. They shouldn't have access to sensitive data like RSA keys. Only the application that they deploy should have access to this sensitive data.
- **Auditors**: This role is for contributors who aren't members of the development or general IT staff. They review the use and maintenance of certificates, keys, and secrets to ensure compliance with security standards.

There's another role that's outside the scope of our application: the subscription (or resource group) administrator. The subscription admin sets up initial access permissions for the security team. They grant access to the security team by using a resource group that has the resources required by the application.

### Define required operations per role

We need to authorize the following operations for our roles:

**Security team**
- Create the managed HSM
- Download the managed HSM security domain (for disaster recovery)
- Turn on logging
- Generate or import keys
- Create the managed HSM backups for disaster recovery
- Set Managed HSM local RBAC to grant permissions to users and applications for specific operations
- Roll the keys periodically

**Developers and operators**
- Get reference (key URI) from the security team for the RSA key used for signing
- Develop and deploy the application that accesses the key programmatically

**Auditors**
- Review keys expiry dates to ensure keys are up-to-date
- Monitor role assignments to ensure keys can only be accessed by authorized users/applications
- Review the managed HSM logs to confirm proper use of keys in compliance with data security standards

## Assign appropriate roles

The following table summarizes the role assignments to teams and resources to access the managed HSM.

| Role | Control plane role | Data plane role |
| --- | --- | --- |
| Security team | Managed HSM Contributor | Managed HSM Administrator |
| Developers and operators | None | None |
| Auditors | None | Managed HSM Crypto Auditor |
| Managed identity of the VM used by the Application| None | Managed HSM Crypto User |
| Managed identity of the Storage account used by the Application| None| Managed HSM Crypto Service Encryption |

The three team roles need access to other resources along with managed HSM permissions. To deploy VMs (or the Web Apps feature of Azure App Service), developers and operators need `Contributor` access to those resource types. Auditors need read access to the Storage account where the managed HSM logs are stored.

## Implement with Azure CLI

To assign control plane roles (Azure RBAC) you can use Azure portal or any of the other management interfaces such as Azure CLI or Azure PowerShell. To assign managed HSM data plane roles you must use Azure CLI or Azure REST API. 

For more information on control plane roles, see [Azure built-in roles](/azure/role-based-access-control/built-in-roles). For more information on Managed HSM data plane roles, see [Local RBAC built-in roles for Managed HSM](built-in-roles.md).

The Azure CLI snippets below demonstrate how to implement the role assignments described above:

### Assumptions

- The Microsoft Entra administrator has created security groups to represent the three roles: Contoso Security Team, Contoso App DevOps, and Contoso App Auditors. The admin has added users to their respective groups.
- All resources are located in the **ContosoAppRG** resource group.
- The managed HSM logs are stored in the **contosologstorage** storage account.
- The **ContosoMHSM** managed HSM and the **contosologstorage** storage account are in the same Azure location.

### Assign control plane roles

The subscription admin assigns the `Managed HSM Contributor` role to the security team. This role allows the security team to manage existing managed HSMs and create new ones.

```azurecli-interactive
# This role assignment allows Contoso Security Team to create new Managed HSMs
az role assignment create --assignee-object-id $(az ad group show -g 'Contoso Security Team' --query 'id' -o tsv) --assignee-principal-type Group --role "Managed HSM Contributor"
```

### Assign data plane roles

For existing managed HSMs, the security team needs to be assigned the "Managed HSM Administrator" role to manage them:

```azurecli-interactive
# This role assignment allows Contoso Security Team to become administrator of existing managed HSM
az keyvault role assignment create --hsm-name ContosoMHSM --assignee $(az ad group show -g 'Contoso Security Team' --query 'id' -o tsv) --scope / --role "Managed HSM Administrator"
```

### Configure logging and assign additional roles

The security team sets up logging and assigns roles to auditors and the VM application:

```azurecli-interactive
# Enable logging
hsmresource=$(az keyvault show --hsm-name ContosoMHSM --query id -o tsv)
storageresource=$(az storage account show --name contosologstorage --query id -o tsv)
az monitor diagnostic-settings create --name MHSM-Diagnostics --resource $hsmresource --logs '[{"category": "AuditEvent","enabled": true}]' --storage-account $storageresource

# Assign the "Crypto Auditor" role to Contoso App Auditors group. It only allows them to read.
az keyvault role assignment create --hsm-name ContosoMHSM --assignee $(az ad group show -g 'Contoso App Auditors' --query 'id' -o tsv) --scope / --role "Managed HSM Crypto Auditor"

# Grant the "Crypto User" role to the VM's managed identity. It allows to use keys.
az keyvault role assignment create --hsm-name ContosoMHSM --assignee $(az vm identity show --name "vmname" --resource-group "ContosoAppRG" --query principalId -o tsv) --scope / --role "Managed HSM Crypto User"

# Assign "Managed HSM Crypto Service Encryption User" role to the Storage account identity
storage_account_principal=$(az storage account show --id $storageresource --query identity.principalId -o tsv)
# (if no identity exists), then assign a new one
[ "$storage_account_principal" ] || storage_account_principal=$(az storage account update --assign-identity --id $storageresource --query identity.principalId -o tsv)

az keyvault role assignment create --hsm-name ContosoMHSM --role "Managed HSM Crypto Service Encryption User" --assignee $storage_account_principal
```

## Considerations for production environments

This tutorial demonstrates a simplified scenario to illustrate access control implementation.

Adjust permissions to your managed HSM based on your specific requirements. In this example, we assumed the security team provides key and secret references (URIs and thumbprints) to the DevOps staff for use in their applications. Developers and operators don't require direct data plane access. While this tutorial focuses on securing your managed HSM, apply similar security measures to other Azure resources, such as [virtual machines](https://azure.microsoft.com/services/virtual-machines/security/), [storage accounts](/azure/storage/blobs/security-recommendations), and others.

## Next steps

- For a conceptual understanding of the Managed HSM access control model, see [Managed HSM access control](access-control.md)
- For a getting-started tutorial for an administrator, see [What is Managed HSM?](overview.md)
- For more information about usage logging for Managed HSM logging, see [Managed HSM logging](logging.md)
- To learn about managing roles in Managed HSM, see [Managed HSM local RBAC](role-management.md)
- [Azure RBAC documentation](/azure/role-based-access-control/overview)
- [Azure RBAC: Built-in roles](/azure/role-based-access-control/built-in-roles)
- [Manage Azure RBAC with Azure CLI](/azure/role-based-access-control/role-assignments-cli)
