---
title: How to secure access to your managed HSMs
description: Learn how to implement access control for Managed HSM using Azure RBAC and Managed HSM local RBAC.
services: key-vault
author: msmbaldwin
ms.custom: devx-track-azurecli
ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: how-to
ms.date: 03/13/2026
ms.author: mbaldwin
# Customer intent: As a managed HSM administrator, I want to set access control and configure the Managed HSM, so that I can ensure it's secure and auditors can properly monitor all activities for this Managed HSM.
---

# Secure access to your managed HSMs

This tutorial provides a practical implementation example of access control for Azure Key Vault Managed HSM. You learn how to implement separation of duties by using Azure RBAC and Managed HSM local RBAC through a realistic scenario.

> [!IMPORTANT]
> Before proceeding with this tutorial, ensure you understand the Managed HSM access control model, including the differences between control plane and data plane access. For this conceptual foundation, see [Managed HSM access control](access-control.md).

## Prerequisites

[!INCLUDE [Azure subscription prerequisite](../includes/azure-subscription-prerequisite.md)]

You also need:

* The Azure CLI version 2.25.0 or later. Run `az --version` to find the version. If you need to install or upgrade, see [Install the Azure CLI](/cli/azure/install-azure-cli).
* A managed HSM in your subscription. See [Quickstart: Provision and activate a managed HSM using Azure CLI](quick-create-cli.md) to provision and activate a managed HSM.

[!INCLUDE [cloud-shell-try-it.md](~/reusable-content/ce-skilling/azure/includes/cloud-shell-try-it.md)]

## Sign in to Azure

To sign in to Azure by using the CLI, enter:

```azurecli
az login
```

For more information about authentication options through the CLI, see [sign in with Azure CLI](/cli/azure/authenticate-azure-cli).

## Understand the example scenario

In this example, you're developing an application that uses an RSA 2,048-bit key for sign operations. Your application runs in an Azure virtual machine (VM) with a [managed identity](/entra/identity/managed-identities-azure-resources/overview). The RSA key used for signing is stored in your managed HSM.

### Define roles and responsibilities

Identify the roles that manage, deploy, and audit your application:

- **Security team**: IT staff from the office of the CSO (Chief Security Officer) or similar contributors. The security team is responsible for the proper safekeeping of keys. The keys include RSA or EC keys for signing, and RSA or oct keys for data encryption.
- **Developers and operators**: The staff who develop the application and deploy it in Azure. The members of this team aren't part of the security staff. They shouldn't have access to sensitive data like RSA keys. Only the application that they deploy should have access to this sensitive data.
- **Auditors**: Contributors who aren't members of the development or general IT staff. They review the use and maintenance of certificates, keys, and secrets to ensure compliance with security standards.

There's another role that's outside the scope of your application: the subscription (or resource group) administrator. The subscription admin sets up initial access permissions for the security team. They grant access to the security team by using a resource group that has the resources required by the application.

### Define required operations per role

Authorize the following operations for each role:

**Security team**
- Create the managed HSM
- Download the managed HSM security domain for disaster recovery
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
- Monitor role assignments to ensure only authorized users and applications can access keys
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

To assign control plane roles (Azure RBAC), use Azure portal or any of the other management interfaces such as Azure CLI or Azure PowerShell. To assign managed HSM data plane roles, use Azure CLI or Azure REST API. 

For more information on control plane roles, see [Azure built-in roles](/azure/role-based-access-control/built-in-roles). For more information on Managed HSM data plane roles, see [Managed HSM local RBAC built-in roles for Managed HSM](built-in-roles.md).

The following Azure CLI snippets demonstrate how to implement the role assignments described earlier:

### Assumptions

- The Microsoft Entra administrator created security groups to represent the three roles: Security Team, App DevOps, and App Auditors. The admin added users to their respective groups.
- All resources are located in the `<resource-group>` resource group.
- The managed HSM logs are stored in the `<storage-account-name>` storage account.
- The `<hsm-name>` managed HSM and the `<storage-account-name>` storage account are in the same Azure location.

### Assign control plane roles

The subscription admin assigns the `Managed HSM Contributor` role to the security team. This role grants the security team permission to manage existing managed HSMs and create new ones.

```azurecli-interactive
# This role assignment allows Security Team to create new Managed HSMs
az role assignment create --assignee-object-id $(az ad group show -g '<security-team-group>' --query 'id' -o tsv) --assignee-principal-type Group --role "Managed HSM Contributor"
```

### Assign data plane roles

For existing managed HSMs, assign the `Managed HSM Administrator` role to the security team so they can manage the HSMs:

```azurecli-interactive
# This role assignment allows Security Team to become administrator of existing managed HSM
az keyvault role assignment create --hsm-name <hsm-name> --assignee $(az ad group show -g '<security-team-group>' --query 'id' -o tsv) --scope / --role "Managed HSM Administrator"
```

### Configure logging and assign additional roles

The security team sets up logging and assigns roles to auditors and the VM application:

```azurecli-interactive
# Enable logging
hsmresource=$(az keyvault show --hsm-name <hsm-name> --query id -o tsv)
storageresource=$(az storage account show --name <storage-account-name> --query id -o tsv)
az monitor diagnostic-settings create --name MHSM-Diagnostics --resource $hsmresource --logs '[{"category": "AuditEvent","enabled": true}]' --storage-account $storageresource

# Assign the "Crypto Auditor" role to App Auditors group. It only allows them to read.
az keyvault role assignment create --hsm-name <hsm-name> --assignee $(az ad group show -g '<auditor-group>' --query 'id' -o tsv) --scope / --role "Managed HSM Crypto Auditor"

# Grant the "Crypto User" role to the VM's managed identity. It allows to use keys.
az keyvault role assignment create --hsm-name <hsm-name> --assignee $(az vm identity show --name "<vm-name>" --resource-group "<resource-group>" --query principalId -o tsv) --scope / --role "Managed HSM Crypto User"

# Assign "Managed HSM Crypto Service Encryption User" role to the Storage account identity
storage_account_principal=$(az storage account show --id $storageresource --query identity.principalId -o tsv)
# (if no identity exists), then assign a new one
[ "$storage_account_principal" ] || storage_account_principal=$(az storage account update --assign-identity --id $storageresource --query identity.principalId -o tsv)

az keyvault role assignment create --hsm-name <hsm-name> --role "Managed HSM Crypto Service Encryption User" --assignee $storage_account_principal
```

## Configure Privileged Identity Management for just-in-time access

For highly sensitive environments, use [Microsoft Entra Privileged Identity Management (PIM)](/entra/id-governance/privileged-identity-management/pim-configure) to enforce just-in-time access for the Managed HSM Administrator role. PIM reduces the attack surface by eliminating standing administrative privileges.

### Prerequisites for PIM integration

- Microsoft Entra ID P2 or Microsoft Entra ID Governance license
- Privileged Role Administrator or Global Administrator role in Microsoft Entra ID

### Enable PIM for Managed HSM Administrator role

1. Sign in to the [Microsoft Entra admin center](https://entra.microsoft.com).

1. Navigate to **Identity governance** > **Privileged Identity Management** > **Microsoft Entra roles**.

1. Select **Roles** and search for roles that include "Managed HSM". While the data plane roles (Managed HSM Administrator, Crypto User, etc.) are managed through Managed HSM local RBAC, you can use PIM for the control plane **Managed HSM Contributor** role.

1. Select the role and configure:
   - **Activation maximum duration**: Set to a limited time window (for example, 4-8 hours)
   - **Require justification**: Enable to require users to provide a reason for activation
   - **Require approval**: Enable and specify approvers from your security team
   - **Require MFA**: Enable for an additional security layer

### Use Microsoft Entra security groups with PIM

For data plane roles managed through Managed HSM local RBAC, combine PIM with Microsoft Entra security groups:

1. Create a Microsoft Entra security group for HSM administrators (for example, "HSM Admins").

1. Assign the Managed HSM Administrator role to this security group:

   ```azurecli-interactive
   az keyvault role assignment create --hsm-name <hsm-name> \
     --assignee $(az ad group show -g '<hsm-admins-group>' --query 'id' -o tsv) \
     --scope / --role "Managed HSM Administrator"
   ```

1. Configure the security group as PIM-eligible in Microsoft Entra admin center:
   - Navigate to **Identity governance** > **Privileged Identity Management** > **Groups**
   - Select **Discover groups** and add "HSM Admins"
   - Configure activation settings (duration, approval, MFA)

1. When administrators need access, they activate their group membership through PIM, which temporarily grants them the Managed HSM Administrator role.

### Monitor PIM activations

Configure alerts for PIM role activations to maintain visibility:

1. In Microsoft Entra admin center, navigate to **Privileged Identity Management** > **Microsoft Entra roles** > **Alerts**.

1. Configure alerts for:
   - Roles being activated too frequently
   - Roles being assigned outside of PIM
   - Eligible assignments being created

For comprehensive security monitoring, integrate these alerts with [Microsoft Sentinel](sentinel.md) alongside your Managed HSM audit logs.

## Considerations for production environments

This tutorial demonstrates a simplified scenario to illustrate access control implementation.

Adjust permissions for your managed HSM based on your specific requirements. In this example, the security team provides key and secret references (URIs and thumbprints) to the DevOps staff for use in their applications. Developers and operators don't require direct data plane access. While this tutorial focuses on securing your managed HSM, apply similar security measures to other Azure resources, such as [virtual machines](https://azure.microsoft.com/services/virtual-machines/security/), [storage accounts](/azure/storage/blobs/security-recommendations), and others.

## Next steps

- For a conceptual understanding of the Managed HSM access control model, see [Managed HSM access control](access-control.md).
- For a getting-started tutorial for an administrator, see [What is Managed HSM?](overview.md)
- For more information about usage logging for Managed HSM logging, see [Managed HSM logging](logging.md).
- To learn about managing roles in Managed HSM, see [Managed HSM local RBAC](role-management.md).
- Learn about [Microsoft Entra Privileged Identity Management](/entra/id-governance/privileged-identity-management/pim-configure).
- Review [Secure your Azure Managed HSM deployment](secure-managed-hsm.md).
- See [Azure RBAC documentation](/azure/role-based-access-control/overview).
- See [Azure RBAC: Built-in roles](/azure/role-based-access-control/built-in-roles).
- See [Manage Azure RBAC with Azure CLI](/azure/role-based-access-control/role-assignments-cli).
