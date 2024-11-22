---
title: ACL Programmability
description: Learn how to run custom business logic on Azure Confidential Ledger
author: settiy
ms.author: settiy
ms.date: 11/20/2024
ms.service: azure-confidential-ledger
ms.topic: how-to
---

# Introduction

Programmability is a new feature in Confidential Ledger that allows customers to run custom code in the same Trusted Compute Base (TCB) as the regular ACL transactions. The benefit of executing the custom code and transactions in the TCB is that it provides the same confidentiality and integrity guarantees to the custom code and the transactions produced by it. Programmability also supports Role Based Access Control (RBAC) through custom roles that are defined in ACL and used in the code.

A few scenarios that can be enabled through programmability are as follows:

 - ** Data aggregation and analytics: Sensitive information can be analyzed in the TCB and only aggregated information can be shared with the stakeholders.**
 - ** Attestation: Sensitive information like PII, credit score and health information can be shared with workloads running on other confidential compute offerings like Azure Confidential ACI and Confidential VM after attestation**

## Prerequisites

- [Azure CLI](/cli/azure/install-azure-cli) or [Azure PowerShell](/powershell/azure/install-azure-powershell).
- [!INCLUDE [Ensure subscription owner](./includes/ensure-subscription-owner.md)]

This tutorial assumes that a Confidential Ledger resource has already been created. If not, create one using these [instructions](quickstart-portal.md).

## Creating and deploying application

Applications are developed using TypeScript and rolled-up into a JavaScript bundle. To learn more about developing applications, refer to the Confidential Consortium Framework (CCF) documentation (https://microsoft.github.io/CCF/main/build_apps/js_app_ts.html).

> [!IMPORTANT]
> Only Administrator user(s) can deploy JavaScript applications and manage custom RBAC in Confidential Ledger.

The following section assumes that you have created a JavaScript application and are ready to deploy it to a Confidential Ledger instance. To demostrate the deployment process and explore the new features, we will use the sample banking application available at the azureconfidentialledger-app-samples GitHub repo (https://github.com/microsoft/azureconfidentialledger-app-samples).

> [!NOTE]
> The application demostrates how common banking scenarios like opening accounts, depositing and transferring funds can be realized through a JavaScript application deployed on a Confidential Ledger instance. 
> The application uses custom roles and RBAC to authorize user actions.

> [!IMPORTANT]
> The rest of the section assumes that the person executing the following steps has Administrator role on the Confidential Ledger instance.

## Login to Azure

> [!NOTE]
> Confidential Ledger supports Microsoft Entra out-of-the-box. No additional configuration is required.
> If your application needs to use JWT issued by other Identity providers, contact customer support to configure it on the Confidential Ledger instance.

Obtain a Microsoft Entra token to authenticate to the Confidential Ledger instance.

# [CLI](#tab/CLI)

```azurecli-interactive
az login --use-device-code
az account get-access-token --resource https://confidential-ledger.azure.com
```

# [PowerShell](#tab/PowerShell)

```azurepowershell-interactive

```
Connect-AzAccount
Set-AzContext -Subscription subscriptionid
Get-AzAccessToken -ResourceUrl "https://confidential-ledger.azure.com" 
---

Copy the raw token value from the command output.

## Deploy the application

Deploy the JavaScript application bundle by calling /app/userDefinedEndpoints.

```terminal
apiVersion="2024-08-22-preview"
content_type_application_json="Content-Type: application/json"
bundle="/path/to/bundle.json"
authorization="Authorization: Bearer raw_token_value"
curl -k -X PUT "https://contoso.confidential-ledger.azure.com/app/userDefinedEndpoints?api-version=$apiVersion" -H "$content_type_application_json" -H "$authorization" -d @$bundle

# View the deployed bundle
#
curl -k "https://contoso.confidential-ledger.azure.com/app/userDefinedEndpoints?api-version=$apiVersion" -H "$authorization"
```

## Create custom roles and users

The banking application involves two personas, namely, a 'manager' and a 'teller'. We will create two roles and users to represent the personas.

> [!NOTE]
> The user will be represented by a unique certificate.

> [!NOTE]
> Application users can be assigned the built-in roles, namely, Administrator, Contributor and Reader. 
> A user can be assigned multiple roles.

```terminal
apiVersion="2024-08-22-preview"
content_type_application_json="Content-Type: application/json"
content_type_merge_patch_json="Content-Type: application/merge-patch+json"
authorization="Authorization: Bearer raw_token_value"
curve="secp384r1"

# These actions must match (case-sensitive) the values defined in the application.
#
manager_role_actions='{"role":{"role_name":"manager","role_actions":["/banking/accounts/post","/banking/accounts/put","/banking/accounts/get","/banking/accounts/patch"]}}'
teller_role_actions='{"role":{"role_name":"teller","role_actions":["/banking/accounts/put","/banking/accounts/get","/banking/accounts/patch"]}}'

# Create the roles.
#
curl -k -X PUT "https://contoso.confidential-ledger.azure.com/app/roles?api-version=$apiVersion" -H "$content_type_application_json" -H "$authorization" -d $manager_role_actions
curl -k -X PUT "https://contoso.confidential-ledger.azure.com/app/roles?api-version=$apiVersion" -H "$content_type_application_json" -H "$authorization" -d $teller_role_actions

# View the roles
#
curl -k "https://contoso.confidential-ledger.azure.com/app/roles?api-version=$apiVersion" -H "$authorization"

# Create a certificate for the manager user.
#
openssl ecparam -out "manager_privk.pem" -name "$curve" -genkey
openssl req -new -key "manager_privk.pem" -x509 -nodes -days 365 -out "manager_cert.pem" -sha384 -subj=/CN="manager"
manager_cert_fingerprint=$(openssl x509 -in "manager_cert.pem" -noout -fingerprint -sha256 | cut -d "=" -f 2)
manager_user="{\"user_id\":\"$manager_cert_fingerprint\",\"assignedRoles\":[\"manager\"]}"

# Create a certificate for the teller user.
#
openssl ecparam -out "teller_privk.pem" -name "$curve" -genkey
openssl req -new -key "teller_privk.pem" -x509 -nodes -days 365 -out "teller_cert.pem" -sha384 -subj=/CN="teller"
teller_cert_fingerprint=$(openssl x509 -in "teller_cert.pem" -noout -fingerprint -sha256 | cut -d "=" -f 2)
teller_user="{\"user_id\":\"$teller_cert_fingerprint\",\"assignedRoles\":[\"teller\"]}"

# Create the manager user.
#
curl -k -X PATCH "https://contoso.confidential-ledger.azure.com/app/ledgerUsers/$manager_cert_fingerprint?api-version=$apiVersion" -H "$content_type_merge_patch_json" -H "$authorization" -d $manager_user

# Create the teller user.
#
curl -k -X PATCH "https://contoso.confidential-ledger.azure.com/app/ledgerUsers/$teller_cert_fingerprint?api-version=$apiVersion" -H "$content_type_merge_patch_json" -H "$authorization" -d $teller_user

# View the users
#
curl -k "https://contoso.confidential-ledger.azure.com/app/ledgerUsers?api-version=$apiVersion" -H "$authorization"
```

## Update the runtime configuration

> [!NOTE]
> This is an optional step.

The JavaScript runtime configuration can be updated by calling the /app/userDefinedEndpoints/runTimeOptions endpoint. To demostrate it, let us set the maximum execution time to 2000 ms.

```terminal
apiVersion="2024-08-22-preview"
content_type_merge_patch_json="Content-Type: application/merge-patch+json"
authorization="Authorization: Bearer raw_token_value"
runtime_options="{\"max_heap_bytes\":1024,\"max_stack_bytes\":1024,\"max_execution_time_ms\":2000,\"log_exception_details\":false,\"return_exception_details\":false,\"max_cached_interpreters\":1024}"

# Patch the runtime options
#
curl -k -X PATCH "https://contoso.confidential-ledger.azure.com/app/userDefinedEndpoints/runTimeOptions?api-version=$apiVersion" -H "$content_type_merge_patch_json" -H "$authorization" -d $runtime_options

# View the runtime options
#
curl -k "https://contoso.confidential-ledger.azure.com/app/userDefinedEndpoints/runTimeOptions?api-version=$apiVersion" -H "$authorization"
```

Now you are ready to call the application endpoints and submit transactions.

## Clean up resources

[!INCLUDE [Clean up resources](../../includes/cli-rg-delete.md)]

## Next steps

In this tutorial, you deployed a custom JavaScript application into a confidential ledger instance. To learn more about Azure confidential ledger and how to integrate it with your applications, continue on to these articles.

- [Overview of Microsoft Azure confidential ledger](overview.md)