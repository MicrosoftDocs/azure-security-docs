---
title: Tutorial - Use Azure Key Vault with an Azure web app in .NET 
description: In this tutorial, you'll configure an Azure web app in an ASP.NET Core application to read a secret from your key vault.
services: key-vault
ms.service: azure-key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 07/07/2026
ms.devlang: csharp
ms.custom: devx-track-csharp, devx-track-azurecli, devx-track-dotnet
#Customer intent: As a developer, I want to use Azure Key Vault to store secrets for my app to help keep them secure.
---

# Tutorial: Use a managed identity to connect Key Vault to an Azure web app in .NET

[Azure Key Vault](./overview.md) provides a way to store credentials and other secrets with increased security. But your code needs to authenticate to Key Vault to retrieve them. [Managed identities for Azure resources](/entra/identity/managed-identities-azure-resources/overview) solve this problem by giving Azure services an automatically managed identity in Microsoft Entra ID. Your code uses this identity to authenticate to any service that supports Microsoft Entra authentication, including Key Vault, without embedding credentials in the code.

In this tutorial, you create and deploy an Azure web application to [Azure App Service](/azure/app-service/overview), then use a managed identity to authenticate the app to a key vault by using the [Azure Key Vault secret client library for .NET](/dotnet/api/overview/azure/key-vault) and the [Azure CLI](/cli/azure/get-started-with-azure-cli). The same principles apply if you use another language, Azure PowerShell, or the Azure portal.

For more information about App Service web apps and deployment, see:
- [App Service overview](/azure/app-service/overview)
- [Create an ASP.NET Core web app in Azure App Service](/azure/app-service/quickstart-dotnetcore)
- [Deploy files to App Service](/azure/app-service/deploy-zip)

## Prerequisites

To complete this tutorial, you need:

* An Azure subscription. [Create one for free.](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn)
* The [.NET 8.0 SDK or later](https://dotnet.microsoft.com/download/dotnet/8.0).
* The [Azure CLI](/cli/azure/install-azure-cli) or [Azure PowerShell](/powershell/azure/).
* [Azure Key Vault](./overview.md). You can create a key vault by using the [Azure portal](quick-create-portal.md), the [Azure CLI](quick-create-cli.md), or [Azure PowerShell](quick-create-powershell.md).
* A Key Vault [secret](../secrets/about-secrets.md). You can create a secret by using the [Azure portal](../secrets/quick-create-portal.md), [PowerShell](../secrets/quick-create-powershell.md), or the [Azure CLI](../secrets/quick-create-cli.md).

If you already have a web app deployed in Azure App Service, skip to [Configure the web app to connect to Key Vault](#configure-the-web-app-to-connect-to-key-vault) and [Modify the app to access your key vault](#modify-the-app-to-access-your-key-vault).

## Create a .NET Core app

In this step, set up the local .NET project.

In a terminal window, create a directory named `akvwebapp` and switch to it:

```bash
mkdir akvwebapp
cd akvwebapp
```

Create a .NET web app by using the [dotnet new web](/dotnet/core/tools/dotnet-new) command:

```bash
dotnet new web
```

Run the app locally to see how it looks before you deploy it to Azure:

```bash
dotnet run
```

In a web browser, open the app at `http://localhost:5000`. You see the "Hello World!" message from the sample app.

For more information about creating web apps, see [Create an ASP.NET Core web app in Azure App Service](/azure/app-service/quickstart-dotnetcore).

## Deploy the app to Azure

In this step, deploy the .NET app to Azure App Service by using zip deploy. Zip deploy is the recommended package-based deployment mechanism for App Service. For continuous delivery from source control, use [GitHub Actions](/azure/app-service/deploy-github-actions) or [Azure DevOps Pipelines](/azure/app-service/deploy-azure-pipelines) instead.

### Create a resource group

Create a resource group to contain the key vault and web app by using [az group create](/cli/azure/group#az-group-create):

```azurecli-interactive
az group create --name "<resource-group>" --location "EastUS"
```

### Create an App Service plan

Create an [App Service plan](/azure/app-service/overview-hosting-plans) by using [az appservice plan create](/cli/azure/appservice/plan). The following example creates a plan named `myAppServicePlan` in the free (`FREE`) tier:

```azurecli-interactive
az appservice plan create --name myAppServicePlan --resource-group <resource-group> --sku FREE
```

### Create a web app

Create an [Azure web app](/azure/app-service/overview) in the `myAppServicePlan` plan.

> [!IMPORTANT]
> Like a key vault, an Azure web app must have a globally unique name. Replace `<webapp-name>` with your web app's name.

```azurecli-interactive
az webapp create --resource-group "<resource-group>" --plan "myAppServicePlan" --name "<webapp-name>"
```

Go to the new app to confirm it's running:

```
https://<webapp-name>.azurewebsites.net
```

You see the default page for a new Azure web app.

### Deploy your local app with zip deploy

From the `akvwebapp` project directory, build the project and create a deployment zip:

```bash
dotnet publish -c Release -o ./publish
cd publish
zip -r ../akvwebapp.zip .
cd ..
```

> [!TIP]
> On Windows without `zip`, use PowerShell: `Compress-Archive -Path .\publish\* -DestinationPath .\akvwebapp.zip`.

Deploy the zip to your web app by using [az webapp deploy](/cli/azure/webapp#az-webapp-deploy):

```azurecli-interactive
az webapp deploy --resource-group "<resource-group>" --name "<webapp-name>" --src-path ./akvwebapp.zip --type zip
```

Refresh the deployed app in your web browser:

```
https://<webapp-name>.azurewebsites.net
```

You see the same "Hello World!" message you saw at `http://localhost:5000`.

## Configure the web app to connect to Key Vault

In this section, you enable the web app to access Key Vault and update your app code to retrieve a secret.

### Create and assign access to a managed identity

Use a [managed identity](/entra/identity/managed-identities-azure-resources/overview) to authenticate the web app to Key Vault. A managed identity removes the need for you to manage credentials in code.

Create the identity for the app by using [az webapp identity assign](/cli/azure/webapp/identity#az-webapp-identity-assign):

```azurecli-interactive
az webapp identity assign --name "<webapp-name>" --resource-group "<resource-group>"
```

The command returns a JSON snippet similar to this:

```json
{
  "principalId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "type": "SystemAssigned"
}
```

[!INCLUDE [Using RBAC to provide access to a key vault](~/reusable-content/ce-skilling/azure/includes/key-vault/rbac/upn-secrets-user-pivot.md)]

### Modify the app to access your key vault

This tutorial uses the [Azure Key Vault secret client library](/dotnet/api/overview/azure/security.keyvault.secrets-readme). You can also use the [Azure Key Vault certificate client library](/dotnet/api/overview/azure/security.keyvault.certificates-readme) or the [Azure Key Vault key client library](/dotnet/api/overview/azure/security.keyvault.keys-readme).

#### Install the packages

From the terminal window, install the Azure Key Vault secret client library and the Azure Identity client library:

```console
dotnet add package Azure.Identity
dotnet add package Azure.Security.KeyVault.Secrets
```

#### Update the code

Open `Program.cs` in your `akvwebapp` project.

Add these `using` directives at the top of the file:

```csharp
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;
using Azure.Core;
```

Add the following lines before the `app.MapGet` call, updating `<vault-name>` to your key vault name. This code uses [DefaultAzureCredential](/dotnet/api/azure.identity.defaultazurecredential) to authenticate to Key Vault through the web app's managed identity. For more information, see the [developer's guide](./developers-guide.md#authenticate-to-key-vault-in-code). The code also configures exponential backoff for retries in case Key Vault is throttled. For details on transaction limits, see [Azure Key Vault throttling guidance](./overview-throttling.md).

```csharp
SecretClientOptions options = new SecretClientOptions()
    {
        Retry =
        {
            Delay= TimeSpan.FromSeconds(2),
            MaxDelay = TimeSpan.FromSeconds(16),
            MaxRetries = 5,
            Mode = RetryMode.Exponential
         }
    };
var client = new SecretClient(new Uri("https://<vault-name>.vault.azure.net/"), new DefaultAzureCredential(), options);

KeyVaultSecret secret = client.GetSecret("<secret-name>");

string secretValue = secret.Value;
```

Update the line `app.MapGet("/", () => "Hello World!");` to:

```csharp
app.MapGet("/", () => secretValue);
```

Save your changes.

#### Redeploy your web app

Rebuild the deployment package and redeploy:

```bash
dotnet publish -c Release -o ./publish
cd publish
zip -r ../akvwebapp.zip .
cd ..
az webapp deploy --resource-group "<resource-group>" --name "<webapp-name>" --src-path ./akvwebapp.zip --type zip
```

## Go to your completed web app

```
https://<webapp-name>.azurewebsites.net
```

Where you previously saw "Hello World!", you now see the value of your secret.

## Next steps

- [Use Azure Key Vault with applications deployed to a virtual machine in .NET](./tutorial-net-virtual-machine.md)
- Learn more about [managed identities for Azure resources](/entra/identity/managed-identities-azure-resources/overview).
- Review the [Key Vault developer's guide](./developers-guide.md).
- [Secure access to a key vault](./secure-key-vault.md)
