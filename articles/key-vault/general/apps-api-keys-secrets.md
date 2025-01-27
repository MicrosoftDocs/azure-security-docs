---
title: Apps, API Keys, and Azure Key Vault Secrets
description: Overview of configuring apps to use Azure Key Vault for API secrets.
services: key-vault
author: orin-thomas
ms.service: azure-key-vault
ms.subservice: general
ms.topic: overview
ms.date: 01/15/2025
ms.author: orthomas
---

# Apps, API Keys, and Azure Key Vault secrets

Azure Key Vault is an Azure service that safeguards cryptographic keys, secrets, and certificates. It provides a centralized, secure, and highly available repository for sensitive information like API keys. One  method of avoiding the insecure practice of embedding API keys directly in your application's source code is to configure your app to securely interact with API keys that are stored in Azure Key Vault.

In this article you learn how to create a Key Vault instance, add an API key as a secret to this key vault, and then configure the key vault using best practices. These best practices include restricting access using role based access control (RBAC), enabling monitoring, and restricting network access.

## Creating and securing an Azure Key Vault instance

Azure Key Vault allows you to securely store cryptographic keys, secrets, and certificates. You should deploy separate key vaults for different environments (development, staging, production) and applications. 

### Create an Resource Group and key vault instance

To create an key vault instance, you can use the following command from Azure CLI or Azure Cloud Shell:

[!INCLUDE [Create a resource group and key vault](../includes/key-vault-python-qs-rg-kv-creation.md)]

### Add an API key to Azure Key Vault as a secret

Once you have created the Azure Key Vault instance, you can add an API key as a secret to this Azure Key Vault instance.

# [Azure CLI](#tab/azure-cli)

The following uses the Azure CLI [az keyvault secret set](/cli/azure/keyvault/secret?view=azure-cli-latest#az-keyvault-secret-set) command to add a secret named MyApiKey to the keyvault and setsthe secret to expire after 180 days:

```azurecli
az keyvault secret set \
    --vault-name "<YourKeyVaultName>" \
    --name "MyApiKey" \
    --value "<YourSecretValue>"
    --expires "$(date -u -d '+180 days' +'%Y-%m-%dT%H:%M:%SZ')"
```

# [Azure PowerShell](#tab/azure-powershell)

The following uses the Azure PowerShell [Set-AzKeyVaultSecret](/powershell/module/az.keyvault/set-azkeyvaultsecret) cmdlet to add a secret named MyApiKey to the keyvault and setsthe secret to expire after 180 days:

```powershell
$secret = ConvertTo-SecureString -String "<YourSecretValue>" -AsPlainText -Force
Set-AzKeyVaultSecret -VaultName "<YourKeyVaultName>" -Name "MyApiKey" -SecretValue $secret -Expires (Get-Date).AddDays(180)
```

---

You should aim to rotate your API keys periodically. Depending on your organization's security needs, you may choose to rotate keys more or less frequently than every 180 days. You can configure an Event Grid subscription for the "SecretNearExpiry" event as a method of receiving notification about expiring API key secrets.

### Restrict access to the Key Vault using RBAC

You can restrict access to the Azure Key Vault instance so that only the application's identity has access to Azure Key Vault. 

# [Azure CLI](#tab/azure-cli)

To do this configure a Role Based Access Control (RBAC) role using the Azure CLI [az role assignment create](/cli/azure/role/assignment?view=azure-cli-latest#az-role-assignment-create) command:

```azurecli
az role assignment create --role "Key Vault Secrets User" \
  --assignee <object-id-of-app-or-user> \
  --scope /subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/Microsoft.KeyVault/vaults/<key-vault-name>
```

# [Azure PowerShell](#tab/azure-powershell)

To do this configure a Role Based Access Control (RBAC) role using the Azure PowerShell [New-AzRoleAssignment](/powershell/module/az.resources/new-azroleassignment) cmdlet:

```powershell
New-AzRoleAssignment -RoleDefinitionName "Key Vault Secrets User" `
    -ObjectId <object-id-of-app-or-user> `
    -Scope "/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/Microsoft.KeyVault/vaults/<key-vault-name>"
```

---

### Enable Key Vault logging and alerts

You should enable logging and alerts on your Key Vault instance as a method of detecting potentially suspicious activity. 

# [Azure CLI](#tab/azure-cli)

To enable Azure Key Vault Logging and Alerts, use the Azure CLI [az monitor diagnostic-settings create](/cli/azure/monitor/diagnostic-settings?view=azure-cli-latest#az-monitor-diagnostic-settings-create) command:

```azurecli
az monitor diagnostic-settings create \
    --name myDiagnosticSettings \
    --resource {key-vault-resource-id} \
    --logs '[{"category": "AuditEvent","enabled": true}]' \
    --workspace {log-analytics-workspace-id}
```

# [Azure PowerShell](#tab/azure-powershell)

To enable Azure Key Vault Logging and Alerts, use the Azure PowerShell [Set-AzDiagnosticSetting](/powershell/module/az.monitor/set-azdiagnosticsetting) cmdlet:

```powershell
Set-AzDiagnosticSetting -Name "myDiagnosticSettings" `
    -ResourceId {key-vault-resource-id} `
    -WorkspaceId {log-analytics-workspace-id} `
    -Category "AuditEvent" `
    -Enabled $true
```

---

This command creates a diagnostic setting named `myDiagnosticSettings`, configures it for the specified Azure Key Vault, enables the `AuditEvent` log category, which tracks security and access-related events and sends the logs to the specified Log Analytics workspace for monitoring, analysis, and alerting. This allows you to monitor access patterns, detect unauthorized access attempts, and configure alerts for critical security events (for example, someone tries to access a secret without the right permissions).

# [Azure CLI](#tab/azure-cli)

You can run the Azure CLI [az monitor scheduled-query create](/cli/azure/monitor/scheduled-query?view=azure-cli-latest#az-monitor-scheduled-query-create) command to monitor logs in the specified Log Analytics workspace for unauthorized access attempts to Azure Key Vault secrets and trigger an alert if any matching unauthorized access attempt is detected:

```azurecli
az monitor scheduled-query create \
    --name "Suspicious Access Alert" \
    --resource-group myResourceGroup \
    --scopes {log-analytics-workspace-resource-id} \
    --condition "AzureDiagnostics | where ResourceType == 'VAULTS' | where OperationName == 'SecretGet' | where ResultSignature == 'Unauthorized'"
```

# [Azure PowerShell](#tab/azure-powershell)

You can run the Azure PowerShell [New-AzScheduledQueryRule](/powershell/module/az.monitor/new-azscheduledqueryrule) cmdlet to monitor logs in the specified Log Analytics workspace for unauthorized access attempts to Azure Key Vault secrets and trigger an alert if any matching unauthorized access attempt is detected:

```powershell
New-AzScheduledQueryRule -ResourceGroupName "myResourceGroup" `
    -Location "eastus" `
    -Action `
        (New-AzScheduledQueryRuleAction -Severity 3 -Trigger `
            (New-AzScheduledQueryRuleTriggerCondition -ThresholdOperator GreaterThan -Threshold 0 -MetricTrigger `
                (New-AzScheduledQueryRuleMetricTrigger -MetricName "UnauthorizedAccess" -MetricResourceId {log-analytics-workspace-resource-id} -TimeAggregation "Count" -Operator "GreaterThan" -Threshold 0))) `
    -Source `
        (New-AzScheduledQueryRuleSource -Query "AzureDiagnostics | where ResourceType == 'VAULTS' | where OperationName == 'SecretGet' | where ResultSignature == 'Unauthorized'" -DataSourceId {log-analytics-workspace-resource-id}) `
    -Schedule `
        (New-AzScheduledQueryRuleSchedule -FrequencyInMinutes 5 -TimeWindowInMinutes 5) `
    -Description "Alert for unauthorized access attempts to Key Vault secrets"
```

---

### Restrict network access to Key Vault

You should restrict network access to Azure Key Vault so that the vault only accepts requests from known network locations. There are two general methods you  can use to do this:

- Azure Private Link. This creates a private endpoint within your virtual network, allowing your application to connect to Azure Key Vault without traversing the public internet. This option is the most secure as traffic remains within your network, but requires creating a private endpoint and configuring DNS.
- Firewall Rules. You can configure the Azure Key Vault firewall settings, located under Networks, with a list of allowed IP ranges. You can also use this method to allow access to existing virtual networks, but this requires that you enable a service endpoint for Microsoft.KeyVault on the selected subnet.

# [Azure CLI](#tab/azure-cli)

You can create a private endpoint using the Azure CLI [az network private-endpoint create](/cli/azure/network/private-endpoint?view=azure-cli-latest#az-network-private-endpoint-create) command:

```azurecli
az network private-endpoint create \
    --name myPrivateEndpoint \
    --resource-group myResourceGroup \
    --vnet-name myVNet \
    --subnet mySubnet \
    --private-connection-resource-id /subscriptions/{subscription}/resourceGroups/{rg}/providers/Microsoft.KeyVault/vaults/{key-vault-name} \
    --group-id vault \
    --connection-name myConnection

You can create firewall rules on the Azure Key Vault instance using the Azure CLI [az keyvault network-rule add](/cli/azure/keyvault/network-rule?view=azure-cli-latest#az-keyvault-network-rule-add) command, substituting the appropriate key vault names, resource groups, subnet, and subnet mask information:

```azurecli
az keyvault network-rule add \
    --name {key-vault-name} \
    --resource-group myResourceGroup \
    --ip-address {trusted-ip-address}/32
```

# [Azure PowerShell](#tab/azure-powershell)
You can create a private endpoint using the Azure PowerShell [New-AzPrivateEndpoint](/powershell/module/az.network/new-azprivateendpoint) cmdlet:

```powershell
$privateEndpoint = New-AzPrivateEndpoint -Name "myPrivateEndpoint" `
    -ResourceGroupName "myResourceGroup" `
    -Location "eastus" `
    -SubnetId "/subscriptions/{subscription}/resourceGroups/{rg}/providers/Microsoft.Network/virtualNetworks/{vnet-name}/subnets/{subnet-name}" `
    -PrivateLinkServiceConnection `
        (New-AzPrivateLinkServiceConnection -Name "myConnection" `
            -PrivateLinkServiceId "/subscriptions/{subscription}/resourceGroups/{rg}/providers/Microsoft.KeyVault/vaults/{key-vault-name}" `
            -GroupId "vault")
```
You can create firewall rules on the Azure Key Vault instance using the Azure PowerShell [Add-AzKeyVaultNetworkRule](/powershell/module/az.keyvault/add-azkeyvaultnetworkrule) cmdlet, substituting the appropriate key vault names, resource groups, subnet, and subnet mask information:

```powershell
Add-AzKeyVaultNetworkRule -VaultName {key-vault-name} `
    -ResourceGroupName myResourceGroup `
    -IPAddress {trusted-ip-address}/32
```

---

Azure Key Vault enforces HTTPS for all communications. This ensures that your API keys and other sensitive data are encrypted in transit, protecting them from eavesdropping and man-in-the-middle attacks.

## Interact with Azure Key Vault using Python

To interact with Azure Key Vault using Python, install the Azure identity library for Microsoft Entra ID and the Azure Key Vault secrets library:

```python
pip install azure-identity
pip install azure-keyvault-secrets
```

You can use the Azure Identity and Azure Key Vault Secrets client library to manage secrets programmatically:

```python
from azure.keyvault.secrets import SecretClient
from azure.identity import DefaultAzureCredential

key_vault_name = "<your-key-vault-name>"
KVUri = f"https://{key_vault_name}.vault.azure.net"
secret_name = "<your-secret-name>"

credential = DefaultAzureCredential()
client = SecretClient(vault_url=KVUri, credential=credential)

retrieved_secret = client.get_secret(secret_name)

# Now you can use the API key:

api_key = retrieved_secret.value
print(f"The API key is: {api_key}")
```

In this sample code:

- DefaultAzureCredential: This class attempts to authenticate using various methods (environment variables, managed identities, etc.), making it suitable for different Azure environments.
- SecretClient: This class provides methods to interact with secrets in Key Vault.
- get_secret(): Retrieves the secret from Key Vault.

## Next steps

- [About keys](../keys/about-keys.md)
- [About secrets](../secrets/about-secrets.md)
- [Authentication, requests, and responses](../general/authentication-requests-and-responses.md)
- [Key Vault Developer's Guide](../general/developers-guide.md)
