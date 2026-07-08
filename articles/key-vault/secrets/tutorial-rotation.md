---
title: Rotation tutorial for resources with one set of authentication credentials stored in Azure Key Vault
description: Use this tutorial to learn how to automate the rotation of a secret for resources that use one set of authentication credentials.
services: key-vault

ms.service: azure-key-vault
ms.subservice: secrets
ms.topic: tutorial
ms.date: 07/07/2026

ms.devlang: csharp
ms.custom: devx-track-csharp, sfi-image-nochange, copilot-scenario-highlight
---
# Automate the rotation of a secret for resources that use one set of authentication credentials

The best way to authenticate to Azure services is with a [managed identity](../general/authentication.md), but some scenarios require an access key, password, or other secret. Rotate those secrets regularly to reduce the impact of a leaked credential.

This tutorial shows how to automate periodic secret rotation for databases and services that use one set of authentication credentials. For an overview of autorotation across different asset types, see [Understanding autorotation in Azure Key Vault](../general/autorotation.md).

Specifically, this tutorial rotates a SQL Server password that's stored in Azure Key Vault, by using a function that's triggered by an Azure Event Grid notification:

> [!TIP]
> For Azure SQL Database and Azure SQL Managed Instance, prefer [Microsoft Entra-only authentication](/azure/azure-sql/database/authentication-azure-ad-only-authentication) over SQL logins. Use the password-rotation pattern in this tutorial only when a workload requires SQL authentication.

:::image type="content" source="../media/rotate-1.png" alt-text="Diagram of rotation solution":::

1. Thirty days before the expiration date of a secret, Key Vault publishes the "near expiry" event to Event Grid.
2. Event Grid checks the event subscriptions and uses HTTP POST to call the function app endpoint subscribed to the event.
3. The function app receives the secret information, generates a new random password, and creates a new version for the secret with the new password in Key Vault.
4. The function app updates SQL Server with the new password.

> [!NOTE]
> There can be a lag between steps 3 and 4. During that window, the secret in Key Vault can't authenticate to SQL Server.
> If any step fails, Event Grid retries for two hours.

## Prerequisites

* An Azure subscription - [create one for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
* Azure Key Vault.
* SQL Server.

If you don't already have a key vault and SQL Server instance, use this deployment link:

[![Image showing a button labeled "Deploy to Azure".](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2FKeyVault-Rotation-SQLPassword-Csharp%2Fmain%2FARM-Templates%2FInitial-Setup%2Fazuredeploy.json)

1. Under **Resource group**, select **Create new**, and give the group a name. This tutorial uses **akvrotation**.
1. Under **SQL Admin Login**, enter the SQL administrator login name.
1. Select **Review + create**.
1. Select **Create**.

:::image type="content" source="../media/rotate-2.png" alt-text="Create a resource group":::

You now have a key vault and a SQL Server instance. Verify this setup in the Azure CLI:

```azurecli
az resource list -o table -g akvrotation
```

The result looks similar to the following output:

```console
Name                     ResourceGroup         Location    Type                               Status
-----------------------  --------------------  ----------  ---------------------------------  --------
akvrotation-kv           akvrotation      eastus      Microsoft.KeyVault/vaults
akvrotation-sql          akvrotation      eastus      Microsoft.Sql/servers
akvrotation-sql/master   akvrotation      eastus      Microsoft.Sql/servers/databases
akvrotation-sql2         akvrotation      eastus      Microsoft.Sql/servers
akvrotation-sql2/master  akvrotation      eastus      Microsoft.Sql/servers/databases
```

## Create and deploy the SQL Server password rotation function

> [!IMPORTANT]
> This template requires the key vault, SQL Server instance, and function app to be in the same resource group.

Next, create a function app with a system-assigned managed identity along with the other required components, and deploy the SQL Server password rotation function.

The function app requires these components:
- An Azure App Service plan.
- A function app with a SQL password rotation function that has an Event Grid trigger and an HTTP trigger.
- A storage account for function app trigger management.
- An Azure role assignment that lets the function app identity access secrets in the key vault.
- An Event Grid event subscription for the **SecretNearExpiry** event.

1. Select the Azure template deployment link:

   [![Image showing a button labeled "Deploy to Azure".](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2FKeyVault-Rotation-SQLPassword-Csharp%2Fmain%2FARM-Templates%2FFunction%2Fazuredeploy.json)

1. In the **Resource group** list, select **akvrotation**.
1. In **SQL Server Name**, enter the name of the SQL Server instance whose password you want to rotate.
1. In **Key Vault Name**, enter the key vault name.
1. In **Function App Name**, enter the function app name.
1. In **Secret Name**, enter the secret name that stores the password.
1. In **Repo URL**, enter the GitHub location of the function code: `https://github.com/Azure-Samples/KeyVault-Rotation-SQLPassword-Csharp.git`.
1. Select **Review + create**.
1. Select **Create**.

> [!NOTE]
> This ARM template uses App Service source-control (Kudu) deployment to pull the function code from GitHub. For a production workload, we recommend a package-based deployment such as [`func azure functionapp publish`](/azure/azure-functions/functions-run-local#project-file-deployment) or [zip deploy with `WEBSITE_RUN_FROM_PACKAGE`](/azure/azure-functions/run-functions-from-deployment-package) instead.

:::image type="content" source="../media/rotate-3.png" alt-text="Select Review+create":::
  

After you complete the preceding steps, you have a storage account, a server farm, and a function app. Verify this setup in the Azure CLI:

```azurecli
az resource list -o table -g akvrotation
```

The result looks similar to the following output:

```console
Name                     ResourceGroup         Location    Type                               Status
-----------------------  --------------------  ----------  ---------------------------------  --------
akvrotation-kv           akvrotation       eastus      Microsoft.KeyVault/vaults
akvrotation-sql          akvrotation       eastus      Microsoft.Sql/servers
akvrotation-sql/master   akvrotation       eastus      Microsoft.Sql/servers/databases
cfogyydrufs5wazfunctions akvrotation       eastus      Microsoft.Storage/storageAccounts
akvrotation-fnapp        akvrotation       eastus      Microsoft.Web/serverFarms
akvrotation-fnapp        akvrotation       eastus      Microsoft.Web/sites
akvrotation-fnapp        akvrotation       eastus      Microsoft.insights/components
```

For more information about creating a function app and using a managed identity to access Key Vault, see [Create a function app from the Azure portal](/azure/azure-functions/functions-create-function-app-portal), [How to use managed identity for App Service and Azure Functions](/azure/app-service/overview-managed-identity), and [Provide access to Key Vault with Azure RBAC](../general/rbac-guide.md).

> [!NOTE]
> The sample function targets the .NET in-process model for Azure Functions. For new development, use the [.NET isolated worker model](/azure/azure-functions/dotnet-isolated-process-guide), which is the supported model going forward.

### Rotation function

The function deployed in the previous step uses an event to trigger secret rotation. It updates Key Vault and the SQL database.

#### Function trigger event

This function reads event data and runs the rotation logic:

```csharp
public static class SimpleRotationEventHandler
{
   [FunctionName("AKVSQLRotation")]
   public static void Run([EventGridTrigger]EventGridEvent eventGridEvent, ILogger log)
   {
      log.LogInformation("C# Event trigger function processed a request.");
      var secretName = eventGridEvent.Subject;
      var secretVersion = Regex.Match(eventGridEvent.Data.ToString(), "Version\":\"([a-z0-9]*)").Groups[1].ToString();
      var keyVaultName = Regex.Match(eventGridEvent.Topic, ".vaults.(.*)").Groups[1].ToString();
      log.LogInformation($"Key Vault Name: {keyVaultName}");
      log.LogInformation($"Secret Name: {secretName}");
      log.LogInformation($"Secret Version: {secretVersion}");

      SecretRotator.RotateSecret(log, secretName, keyVaultName);
   }
}
```

#### Secret rotation logic
This rotation method reads database information from the secret, creates a new version of the secret, and updates the database with the new secret:

```csharp
    public class SecretRotator
    {
		private const string CredentialIdTag = "CredentialId";
		private const string ProviderAddressTag = "ProviderAddress";
		private const string ValidityPeriodDaysTag = "ValidityPeriodDays";

		public static void RotateSecret(ILogger log, string secretName, string keyVaultName)
        {
            //Retrieve Current Secret
            var kvUri = "https://" + keyVaultName + ".vault.azure.net";
            var client = new SecretClient(new Uri(kvUri), new DefaultAzureCredential());
            KeyVaultSecret secret = client.GetSecret(secretName);
            log.LogInformation("Secret Info Retrieved");

            //Retrieve Secret Info
            var credentialId = secret.Properties.Tags.ContainsKey(CredentialIdTag) ? secret.Properties.Tags[CredentialIdTag] : "";
            var providerAddress = secret.Properties.Tags.ContainsKey(ProviderAddressTag) ? secret.Properties.Tags[ProviderAddressTag] : "";
            var validityPeriodDays = secret.Properties.Tags.ContainsKey(ValidityPeriodDaysTag) ? secret.Properties.Tags[ValidityPeriodDaysTag] : "";
            log.LogInformation($"Provider Address: {providerAddress}");
            log.LogInformation($"Credential Id: {credentialId}");

            //Check Service Provider connection
            CheckServiceConnection(secret);
            log.LogInformation("Service Connection Validated");
            
            //Create new password
            var randomPassword = CreateRandomPassword();
            log.LogInformation("New Password Generated");

            //Add secret version with new password to Key Vault
            CreateNewSecretVersion(client, secret, randomPassword);
            log.LogInformation("New Secret Version Generated");

            //Update Service Provider with new password
            UpdateServicePassword(secret, randomPassword);
            log.LogInformation("Password Changed");
            log.LogInformation($"Secret Rotated Successfully");
        }
}
```
You can find the complete code on [GitHub](https://github.com/Azure-Samples/KeyVault-Rotation-SQLPassword-Csharp).

## Add the secret to Key Vault

Assign yourself the **Key Vault Secrets Officer** role so that you can manage secrets in the vault:

```azurecli
az role assignment create --role "Key Vault Secrets Officer" --assignee <email-address-of-user> --scope /subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.KeyVault/vaults/akvrotation-kv
```

Create a secret with tags that contain the SQL Server resource ID, the SQL Server login name, and the secret validity period in days. Set the secret name, the initial password from the SQL database (`Simple123` in this example), and an expiration date set to tomorrow so rotation triggers immediately.

# [Bash](#tab/bash)

```azurecli
tomorrowDate=$(date -u -d "+1 day" +"%Y-%m-%dT%H:%M:%SZ")
az keyvault secret set --name sqlPassword --vault-name akvrotation-kv --value "Simple123" --tags "CredentialId=sqlAdmin" "ProviderAddress=<sql-database-resource-id>" "ValidityPeriodDays=90" --expires $tomorrowDate
```

# [PowerShell](#tab/powershell)

```azurecli
$tomorrowDate = (Get-Date).AddDays(1).ToString("yyyy-MM-ddTHH:mm:ssZ")
az keyvault secret set --name sqlPassword --vault-name akvrotation-kv --value "Simple123" --tags "CredentialId=sqlAdmin" "ProviderAddress=<sql-database-resource-id>" "ValidityPeriodDays=90" --expires $tomorrowDate
```

---

Setting a short expiration date publishes a `SecretNearExpiry` event within 15 minutes, which triggers the function to rotate the secret.

## Test and verify

To verify that the secret rotated, go to **Key Vault** > **Secrets**:

:::image type="content" source="../media/rotate-8.png" alt-text="Screenshot that shows how to access Key Vault > Secrets.":::

Open the **sqlPassword** secret and view the original and rotated versions:

:::image type="content" source="../media/rotate-9.png" alt-text="Go to Secrets":::

### Create a web app

To verify the SQL credentials, create a web app. The web app retrieves the secret from Key Vault, extracts the SQL database information and credentials from the secret, and tests the connection to SQL Server.

The web app requires these components:
- A web app with a system-assigned managed identity.
- A role assignment that lets the web app's managed identity read secrets in the key vault.

1. Select the Azure template deployment link: 

   [![Image showing a button labeled "Deploy to Azure".](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2FKeyVault-Rotation-SQLPassword-Csharp-WebApp%2Fmain%2FARM-Templates%2FWeb-App%2Fazuredeploy.json)

1. Select the **akvrotation** resource group.
1. In **SQL Server Name**, enter the SQL Server name whose password you rotated.
1. In **Key Vault Name**, enter the key vault name.
1. In **Secret Name**, enter the secret name that stores the password.
1. In **Repo URL**, enter the GitHub location of the web app code: `https://github.com/Azure-Samples/KeyVault-Rotation-SQLPassword-Csharp-WebApp.git`.
1. Select **Review + create**.
1. Select **Create**.

### Open the web app

Go to the deployed application URL: `https://akvrotation-app.azurewebsites.net/`.

When the application opens in the browser, the page shows the **Generated Secret Value** and a **Database Connected** value of `true`.

## Use AI to customize the rotation function for your database

This tutorial demonstrates secret rotation for SQL Server, but you can adapt the rotation function for other database types. GitHub Copilot can help you modify the rotation function code to work with your database or credential type.

```copilot-prompt
I'm using the Azure Key Vault secret rotation tutorial for SQL Server. Help me modify the rotation function to work with PostgreSQL instead. The function should:
1. Generate a new secure password
2. Update the PostgreSQL database user password
3. Store the new password in Key Vault
Show me the changes needed to the C# function code, including the correct PostgreSQL connection library and password update command.
```

Copilot can also help you adapt this pattern for other credential types like API keys, connection strings, or service account passwords.

[!INCLUDE [copilot-highlights-disclaimer](~/reusable-content/ce-skilling/azure/includes/copilot-highlights-disclaimer.md)]

## Learn more

- [Rotation for resources with two sets of credentials](tutorial-rotation-dual.md)
- [Understanding autorotation in Azure Key Vault](../general/autorotation.md)
- [Monitoring Key Vault with Azure Event Grid](../general/event-grid-overview.md)
- [Receive email when a key vault secret changes](../general/event-grid-logicapps.md)
- [Azure Event Grid event schema for Azure Key Vault](/azure/event-grid/event-schema-key-vault)