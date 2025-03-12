---
title: Advanced user defined function
description: Learn how to execute custom code in Azure confidential ledger
author: settiy
ms.author: settiy
ms.date: 03/12/2025
ms.service: azure-confidential-ledger
ms.topic: how-to
---

# Advanced user defined function

Advanced user defined function allows custom code to execute in the same Trusted Execution Environment (TEE) as the ledger. This extends the benefits of confidentiality and integrity guarantee to the custom code. Also, it supports custom Role Based Access Control (RBAC) for authorization.

A few scenarios that would benefit fom this feature are as follows:

 - **Data analysis and aggregation**: Sensitive information can be processed in the TEE and aggregated information can be shared with the stakeholders.
 - **Protecting confidential information**: Confidential information like PII, credit score, and health information can be shared with other confidential workloads after attestation.

## Prerequisites

- [Azure CLI](/cli/azure/install-azure-cli) or [Azure PowerShell](/powershell/azure/install-azure-powershell).

[!INCLUDE [Ensure subscription owner](./includes/ensure-subscription-owner.md)]

This tutorial assumes that you have created a ledger instance. You can create one using the [Azure portal](quickstart-portal.md), [Azure CLI](quickstart-cli.md), or [Azure PowerShell](quickstart-powershell.md).

## Developing applications

Ledger applications are developed using TypeScript and rolled-up into a JavaScript bundle. To learn more about the development process, refer to the [Confidential Consortium Framework (CCF) documentation](https://microsoft.github.io/CCF/main/build_apps/js_app_ts.html).

> [!IMPORTANT]
> Only Administrators can deploy applications and manage custom RBAC in the ledger.
> The rest of the section assumes that an Administrator executes the commands.

We will use the banking application available at the azureconfidentialledger-app-samples repo (https://github.com/microsoft/azureconfidentialledger-app-samples) to demostrate the feature.

> [!NOTE]
> The banking application exposes APIs for commonly used banking scenarios like opening accounts, depositing and transferring funds using custom roles and actions.

## Sign in to Azure

> [!NOTE]
> Confidential Ledger supports Microsoft Entra ID out-of-the-box. If your application integrates with other identity providers, contact customer support to have it configured in the ledger.

Obtain a Microsoft Entra ID token.

# [CLI](#tab/CLI)

```azurecli-interactive
az login --use-device-code
az account get-access-token --resource https://confidential-ledger.azure.com
```

# [PowerShell](#tab/PowerShell)

```azurepowershell-interactive
Connect-AzAccount
Set-AzContext -Subscription subscriptionid
Get-AzAccessToken -ResourceUrl "https://confidential-ledger.azure.com"
```
---

Copy the raw token value from the output.

## Download the ledger identity

A ledger is uniquely identified by a certificate called the service certificate. It is used to establish a secure connection to the ledger. Download it from a well-known endpoint and save it to servicer_cert.pem.

> [!NOTE]
> `contoso` is the name of the ledger. Replace it with the appropriate ledger name.

```terminal
curl https://identity.confidential-ledger.core.azure.com/ledgerIdentity/contoso --silent | jq ' .ledgerTlsCertificate' | xargs echo -e > service_cert.pem
```

## Deploy the application

Deploy the JavaScript application bundle by invoking the /app/userDefinedEndpoints endpoint.

```terminal
apiVersion="2024-08-22-preview"
content_type_application_json="Content-Type: application/json"
bundle="/path/to/bundle.json"
authorization="Authorization: Bearer raw_token_value"
server_identity="--cacert service_cert.pem"

# Deploy the application
#
curl $server_identity -X PUT "https://contoso.confidential-ledger.azure.com/app/userDefinedEndpoints?api-version=$apiVersion" -H "$content_type_application_json" -H "$authorization" -d @$bundle

# View the application
#
curl $server_identity "https://contoso.confidential-ledger.azure.com/app/userDefinedEndpoints?api-version=$apiVersion" -H "$authorization"
```

## Create roles and users

The banking application uses two personas, namely, a 'manager' and a 'teller'. We will create roles and users to represent them.

> [!NOTE]
> Each user is represented by a unique certificate.

> [!NOTE]
> Application users can be assigned the built-in roles, namely, Administrator, Contributor and Reader.
> Custom roles are case-sensitive and built-in roles are case-insensitive.
> A user can be assigned multiple roles.

```terminal
apiVersion="2024-08-22-preview"
content_type_application_json="Content-Type: application/json"
content_type_merge_patch_json="Content-Type: application/merge-patch+json"
authorization="Authorization: Bearer raw_token_value"
curve="secp384r1"
server_identity="--cacert service_cert.pem"

# These actions must match (case-sensitive) the values defined in the application.
#
role_actions='{"roles":[{"role_name":"manager","role_actions":["/banking/accounts/post","/banking/accounts/put","/banking/accounts/get","/banking/accounts/patch"]},{"role_name":"teller","role_actions":["/banking/accounts/put","/banking/accounts/get","/banking/accounts/patch"]}]}'

# Create the roles.
#
curl $server_identity -X PUT "https://contoso.confidential-ledger.azure.com/app/roles?api-version=$apiVersion" -H "$content_type_application_json" -H "$authorization" -d $role_actions

# View the roles
#
curl $server_identity "https://contoso.confidential-ledger.azure.com/app/roles?api-version=$apiVersion" -H "$authorization"

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
curl $server_identity -X PATCH "https://contoso.confidential-ledger.azure.com/app/ledgerUsers/$manager_cert_fingerprint?api-version=$apiVersion" -H "$content_type_merge_patch_json" -H "$authorization" -d $manager_user

# Create the teller user.
#
curl $server_identity -X PATCH "https://contoso.confidential-ledger.azure.com/app/ledgerUsers/$teller_cert_fingerprint?api-version=$apiVersion" -H "$content_type_merge_patch_json" -H "$authorization" -d $teller_user

# View the users
#
curl $server_identity "https://contoso.confidential-ledger.azure.com/app/ledgerUsers?api-version=$apiVersion" -H "$authorization"
```

## Update the runtime configuration (optional)

The JavaScript runtime configuration can be updated by calling the /app/userDefinedEndpoints/runTimeOptions endpoint. To demonstrate it, let us set the maximum execution time to 2000 ms.

```terminal
apiVersion="2024-08-22-preview"
content_type_merge_patch_json="Content-Type: application/merge-patch+json"
authorization="Authorization: Bearer raw_token_value"
runtime_options="{\"max_heap_bytes\":1024,\"max_stack_bytes\":1024,\"max_execution_time_ms\":2000,\"log_exception_details\":false,\"return_exception_details\":false,\"max_cached_interpreters\":1024}"
server_identity="--cacert service_cert.pem"

# Patch the runtime options
#
curl $server_identity -X PATCH "https://contoso.confidential-ledger.azure.com/app/userDefinedEndpoints/runTimeOptions?api-version=$apiVersion" -H "$content_type_merge_patch_json" -H "$authorization" -d $runtime_options

# View the runtime options
#
curl $server_identity "https://contoso.confidential-ledger.azure.com/app/userDefinedEndpoints/runTimeOptions?api-version=$apiVersion" -H "$authorization"
```

Now you are ready to call the application endpoints and submit transactions.

## Clean up resources

[!INCLUDE [Clean up resources](../../includes/cli-rg-delete.md)]

## Next steps

In this tutorial, you deployed a custom JavaScript application into a confidential ledger instance. To learn more about Azure confidential ledger and how to integrate it with your applications, continue on to these articles.

- [Overview of Microsoft Azure confidential ledger](overview.md)