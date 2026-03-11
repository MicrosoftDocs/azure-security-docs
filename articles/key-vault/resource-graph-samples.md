---
title: Azure Resource Graph sample queries for Azure Key Vault
description: Sample Azure Resource Graph queries for Azure Key Vault showing use of resource types and tables to access Azure Key Vault related resources and properties.
ms.date: 01/08/2026
ms.topic: sample
author: msmbaldwin
ms.author: mbaldwin
ms.service: azure-key-vault
ms.custom: subject-resourcegraph-sample
---
# Azure Resource Graph sample queries for Azure Key Vault

This page is a collection of [Azure Resource Graph](/azure/governance/resource-graph/overview) sample queries for Azure Key Vault.

## Sample queries

### Count key vault resources

This query uses `count` instead of `summarize` to count the number of records returned. Only key vaults are included in the count.

```kusto
Resources
| where type =~ 'microsoft.keyvault/vaults'
| count
```

# [Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type =~ 'microsoft.keyvault/vaults' | count"
```

# [Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'microsoft.keyvault/vaults' | count"
```

# [Portal](#tab/azure-portal)



- Azure portal: <a href="https://portal.azure.com/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20where%20type%20%3d%7e%20%27microsoft.keyvault%2fvaults%27%0a%7c%20count" target="_blank">portal.azure.com</a>
- Azure Government portal: <a href="https://portal.azure.us/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20where%20type%20%3d%7e%20%27microsoft.keyvault%2fvaults%27%0a%7c%20count" target="_blank">portal.azure.us</a>
- Microsoft Azure operated by 21Vianet portal: <a href="https://portal.azure.cn/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20where%20type%20%3d%7e%20%27microsoft.keyvault%2fvaults%27%0a%7c%20count" target="_blank">portal.azure.cn</a>

---

### Key vaults without soft delete enabled

This query identifies key vaults that don't have soft delete enabled, which is a critical security feature that allows recovery of deleted key vault objects.

```kusto
Resources
| where type =~ 'microsoft.keyvault/vaults'
| where properties.enableSoftDelete == false or isnull(properties.enableSoftDelete)
| project name, resourceGroup, subscriptionId, location, properties.enableSoftDelete
```

# [Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type =~ 'microsoft.keyvault/vaults' | where properties.enableSoftDelete == false or isnull(properties.enableSoftDelete) | project name, resourceGroup, subscriptionId, location, properties.enableSoftDelete"
```

# [Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'microsoft.keyvault/vaults' | where properties.enableSoftDelete == false or isnull(properties.enableSoftDelete) | project name, resourceGroup, subscriptionId, location, properties.enableSoftDelete"
```

# [Portal](#tab/azure-portal)



- Azure portal: <a href="https://portal.azure.com/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20where%20type%20%3d%7e%20%27microsoft.keyvault%2fvaults%27%0a%7c%20where%20properties.enableSoftDelete%20%3d%3d%20false%20or%20isnull%28properties.enableSoftDelete%29%0a%7c%20project%20name%2c%20resourceGroup%2c%20subscriptionId%2c%20location%2c%20properties.enableSoftDelete" target="_blank">portal.azure.com</a>
- Azure Government portal: <a href="https://portal.azure.us/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20where%20type%20%3d%7e%20%27microsoft.keyvault%2fvaults%27%0a%7c%20where%20properties.enableSoftDelete%20%3d%3d%20false%20or%20isnull%28properties.enableSoftDelete%29%0a%7c%20project%20name%2c%20resourceGroup%2c%20subscriptionId%2c%20location%2c%20properties.enableSoftDelete" target="_blank">portal.azure.us</a>
- Microsoft Azure operated by 21Vianet portal: <a href="https://portal.azure.cn/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20where%20type%20%3d%7e%20%27microsoft.keyvault%2fvaults%27%0a%7c%20where%20properties.enableSoftDelete%20%3d%3d%20false%20or%20isnull%28properties.enableSoftDelete%29%0a%7c%20project%20name%2c%20resourceGroup%2c%20subscriptionId%2c%20location%2c%20properties.enableSoftDelete" target="_blank">portal.azure.cn</a>

---

### Key vaults without purge protection enabled

This query finds key vaults that don't have purge protection enabled. Purge protection is recommended for production environments to prevent permanent deletion of key vault objects during the retention period.

```kusto
Resources
| where type =~ 'microsoft.keyvault/vaults'
| where properties.enablePurgeProtection != true
| project name, resourceGroup, subscriptionId, location, softDeleteEnabled = properties.enableSoftDelete, purgeProtectionEnabled = properties.enablePurgeProtection
```

# [Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type =~ 'microsoft.keyvault/vaults' | where properties.enablePurgeProtection != true | project name, resourceGroup, subscriptionId, location, softDeleteEnabled = properties.enableSoftDelete, purgeProtectionEnabled = properties.enablePurgeProtection"
```

# [Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'microsoft.keyvault/vaults' | where properties.enablePurgeProtection != true | project name, resourceGroup, subscriptionId, location, softDeleteEnabled = properties.enableSoftDelete, purgeProtectionEnabled = properties.enablePurgeProtection"
```

# [Portal](#tab/azure-portal)



- Azure portal: <a href="https://portal.azure.com/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20where%20type%20%3d%7e%20%27microsoft.keyvault%2fvaults%27%0a%7c%20where%20properties.enablePurgeProtection%20%21%3d%20true%0a%7c%20project%20name%2c%20resourceGroup%2c%20subscriptionId%2c%20location%2c%20softDeleteEnabled%20%3d%20properties.enableSoftDelete%2c%20purgeProtectionEnabled%20%3d%20properties.enablePurgeProtection" target="_blank">portal.azure.com</a>
- Azure Government portal: <a href="https://portal.azure.us/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20where%20type%20%3d%7e%20%27microsoft.keyvault%2fvaults%27%0a%7c%20where%20properties.enablePurgeProtection%20%21%3d%20true%0a%7c%20project%20name%2c%20resourceGroup%2c%20subscriptionId%2c%20location%2c%20softDeleteEnabled%20%3d%20properties.enableSoftDelete%2c%20purgeProtectionEnabled%20%3d%20properties.enablePurgeProtection" target="_blank">portal.azure.us</a>
- Microsoft Azure operated by 21Vianet portal: <a href="https://portal.azure.cn/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20where%20type%20%3d%7e%20%27microsoft.keyvault%2fvaults%27%0a%7c%20where%20properties.enablePurgeProtection%20%21%3d%20true%0a%7c%20project%20name%2c%20resourceGroup%2c%20subscriptionId%2c%20location%2c%20softDeleteEnabled%20%3d%20properties.enableSoftDelete%2c%20purgeProtectionEnabled%20%3d%20properties.enablePurgeProtection" target="_blank">portal.azure.cn</a>

---

### Key vaults using RBAC authorization

This query lists key vaults that use Azure RBAC for authorization instead of access policies. RBAC authorization is the recommended approach for managing access to key vault data plane.

```kusto
Resources
| where type =~ 'microsoft.keyvault/vaults'
| extend rbacEnabled = properties.enableRbacAuthorization
| project name, resourceGroup, subscriptionId, location, rbacEnabled
| where rbacEnabled == true
```

# [Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type =~ 'microsoft.keyvault/vaults' | extend rbacEnabled = properties.enableRbacAuthorization | project name, resourceGroup, subscriptionId, location, rbacEnabled | where rbacEnabled == true"
```

# [Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'microsoft.keyvault/vaults' | extend rbacEnabled = properties.enableRbacAuthorization | project name, resourceGroup, subscriptionId, location, rbacEnabled | where rbacEnabled == true"
```

# [Portal](#tab/azure-portal)



- Azure portal: <a href="https://portal.azure.com/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20where%20type%20%3d%7e%20%27microsoft.keyvault%2fvaults%27%0a%7c%20extend%20rbacEnabled%20%3d%20properties.enableRbacAuthorization%0a%7c%20project%20name%2c%20resourceGroup%2c%20subscriptionId%2c%20location%2c%20rbacEnabled%0a%7c%20where%20rbacEnabled%20%3d%3d%20true" target="_blank">portal.azure.com</a>
- Azure Government portal: <a href="https://portal.azure.us/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20where%20type%20%3d%7e%20%27microsoft.keyvault%2fvaults%27%0a%7c%20extend%20rbacEnabled%20%3d%20properties.enableRbacAuthorization%0a%7c%20project%20name%2c%20resourceGroup%2c%20subscriptionId%2c%20location%2c%20rbacEnabled%0a%7c%20where%20rbacEnabled%20%3d%3d%20true" target="_blank">portal.azure.us</a>
- Microsoft Azure operated by 21Vianet portal: <a href="https://portal.azure.cn/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20where%20type%20%3d%7e%20%27microsoft.keyvault%2fvaults%27%0a%7c%20extend%20rbacEnabled%20%3d%20properties.enableRbacAuthorization%0a%7c%20project%20name%2c%20resourceGroup%2c%20subscriptionId%2c%20location%2c%20rbacEnabled%0a%7c%20where%20rbacEnabled%20%3d%3d%20true" target="_blank">portal.azure.cn</a>

---

### Key vaults with public network access enabled

This query identifies key vaults that allow public network access. For enhanced security, consider restricting access to specific networks or using private endpoints.

```kusto
Resources
| where type =~ 'microsoft.keyvault/vaults'
| extend networkDefaultAction = tostring(properties.networkAcls.defaultAction)
| where networkDefaultAction =~ 'Allow' or isnull(networkDefaultAction)
| project name, resourceGroup, subscriptionId, location, networkDefaultAction
```

# [Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type =~ 'microsoft.keyvault/vaults' | extend networkDefaultAction = tostring(properties.networkAcls.defaultAction) | where networkDefaultAction =~ 'Allow' or isnull(networkDefaultAction) | project name, resourceGroup, subscriptionId, location, networkDefaultAction"
```

# [Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'microsoft.keyvault/vaults' | extend networkDefaultAction = tostring(properties.networkAcls.defaultAction) | where networkDefaultAction =~ 'Allow' or isnull(networkDefaultAction) | project name, resourceGroup, subscriptionId, location, networkDefaultAction"
```

# [Portal](#tab/azure-portal)



- Azure portal: <a href="https://portal.azure.com/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20where%20type%20%3d%7e%20%27microsoft.keyvault%2fvaults%27%0a%7c%20extend%20networkDefaultAction%20%3d%20tostring%28properties.networkAcls.defaultAction%29%0a%7c%20where%20networkDefaultAction%20%3d%7e%20%27Allow%27%20or%20isnull%28networkDefaultAction%29%0a%7c%20project%20name%2c%20resourceGroup%2c%20subscriptionId%2c%20location%2c%20networkDefaultAction" target="_blank">portal.azure.com</a>
- Azure Government portal: <a href="https://portal.azure.us/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20where%20type%20%3d%7e%20%27microsoft.keyvault%2fvaults%27%0a%7c%20extend%20networkDefaultAction%20%3d%20tostring%28properties.networkAcls.defaultAction%29%0a%7c%20where%20networkDefaultAction%20%3d%7e%20%27Allow%27%20or%20isnull%28networkDefaultAction%29%0a%7c%20project%20name%2c%20resourceGroup%2c%20subscriptionId%2c%20location%2c%20networkDefaultAction" target="_blank">portal.azure.us</a>
- Microsoft Azure operated by 21Vianet portal: <a href="https://portal.azure.cn/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20where%20type%20%3d%7e%20%27microsoft.keyvault%2fvaults%27%0a%7c%20extend%20networkDefaultAction%20%3d%20tostring%28properties.networkAcls.defaultAction%29%0a%7c%20where%20networkDefaultAction%20%3d%7e%20%27Allow%27%20or%20isnull%28networkDefaultAction%29%0a%7c%20project%20name%2c%20resourceGroup%2c%20subscriptionId%2c%20location%2c%20networkDefaultAction" target="_blank">portal.azure.cn</a>

---

### Key vaults security configuration summary

This query provides a comprehensive security posture summary for all key vaults, including soft delete status, purge protection, RBAC authorization, and network settings.

```kusto
Resources
| where type =~ 'microsoft.keyvault/vaults'
| extend 
    softDeleteEnabled = tobool(properties.enableSoftDelete),
    purgeProtectionEnabled = tobool(properties.enablePurgeProtection),
    rbacEnabled = tobool(properties.enableRbacAuthorization),
    networkDefaultAction = tostring(properties.networkAcls.defaultAction),
    sku = tostring(properties.sku.name)
| project name, resourceGroup, subscriptionId, location, sku, softDeleteEnabled, purgeProtectionEnabled, rbacEnabled, networkDefaultAction
| order by name asc
```

# [Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type =~ 'microsoft.keyvault/vaults' | extend softDeleteEnabled = tobool(properties.enableSoftDelete), purgeProtectionEnabled = tobool(properties.enablePurgeProtection), rbacEnabled = tobool(properties.enableRbacAuthorization), networkDefaultAction = tostring(properties.networkAcls.defaultAction), sku = tostring(properties.sku.name) | project name, resourceGroup, subscriptionId, location, sku, softDeleteEnabled, purgeProtectionEnabled, rbacEnabled, networkDefaultAction | order by name asc"
```

# [Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'microsoft.keyvault/vaults' | extend softDeleteEnabled = tobool(properties.enableSoftDelete), purgeProtectionEnabled = tobool(properties.enablePurgeProtection), rbacEnabled = tobool(properties.enableRbacAuthorization), networkDefaultAction = tostring(properties.networkAcls.defaultAction), sku = tostring(properties.sku.name) | project name, resourceGroup, subscriptionId, location, sku, softDeleteEnabled, purgeProtectionEnabled, rbacEnabled, networkDefaultAction | order by name asc"
```

# [Portal](#tab/azure-portal)



- Azure portal: <a href="https://portal.azure.com/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20where%20type%20%3d%7e%20%27microsoft.keyvault%2fvaults%27%0a%7c%20extend%20softDeleteEnabled%20%3d%20tobool%28properties.enableSoftDelete%29%2c%20purgeProtectionEnabled%20%3d%20tobool%28properties.enablePurgeProtection%29%2c%20rbacEnabled%20%3d%20tobool%28properties.enableRbacAuthorization%29%2c%20networkDefaultAction%20%3d%20tostring%28properties.networkAcls.defaultAction%29%2c%20sku%20%3d%20tostring%28properties.sku.name%29%0a%7c%20project%20name%2c%20resourceGroup%2c%20subscriptionId%2c%20location%2c%20sku%2c%20softDeleteEnabled%2c%20purgeProtectionEnabled%2c%20rbacEnabled%2c%20networkDefaultAction%0a%7c%20order%20by%20name%20asc" target="_blank">portal.azure.com</a>
- Azure Government portal: <a href="https://portal.azure.us/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20where%20type%20%3d%7e%20%27microsoft.keyvault%2fvaults%27%0a%7c%20extend%20softDeleteEnabled%20%3d%20tobool%28properties.enableSoftDelete%29%2c%20purgeProtectionEnabled%20%3d%20tobool%28properties.enablePurgeProtection%29%2c%20rbacEnabled%20%3d%20tobool%28properties.enableRbacAuthorization%29%2c%20networkDefaultAction%20%3d%20tostring%28properties.networkAcls.defaultAction%29%2c%20sku%20%3d%20tostring%28properties.sku.name%29%0a%7c%20project%20name%2c%20resourceGroup%2c%20subscriptionId%2c%20location%2c%20sku%2c%20softDeleteEnabled%2c%20purgeProtectionEnabled%2c%20rbacEnabled%2c%20networkDefaultAction%0a%7c%20order%20by%20name%20asc" target="_blank">portal.azure.us</a>
- Microsoft Azure operated by 21Vianet portal: <a href="https://portal.azure.cn/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20where%20type%20%3d%7e%20%27microsoft.keyvault%2fvaults%27%0a%7c%20extend%20softDeleteEnabled%20%3d%20tobool%28properties.enableSoftDelete%29%2c%20purgeProtectionEnabled%20%3d%20tobool%28properties.enablePurgeProtection%29%2c%20rbacEnabled%20%3d%20tobool%28properties.enableRbacAuthorization%29%2c%20networkDefaultAction%20%3d%20tostring%28properties.networkAcls.defaultAction%29%2c%20sku%20%3d%20tostring%28properties.sku.name%29%0a%7c%20project%20name%2c%20resourceGroup%2c%20subscriptionId%2c%20location%2c%20sku%2c%20softDeleteEnabled%2c%20purgeProtectionEnabled%2c%20rbacEnabled%2c%20networkDefaultAction%0a%7c%20order%20by%20name%20asc" target="_blank">portal.azure.cn</a>

---

### Key vaults by location

This query counts key vaults grouped by Azure region, which is useful for understanding the geographic distribution of your key management resources.

```kusto
Resources
| where type =~ 'microsoft.keyvault/vaults'
| summarize count() by location
| order by count_ desc
```

# [Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type =~ 'microsoft.keyvault/vaults' | summarize count() by location | order by count_ desc"
```

# [Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'microsoft.keyvault/vaults' | summarize count() by location | order by count_ desc"
```

# [Portal](#tab/azure-portal)



- Azure portal: <a href="https://portal.azure.com/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20where%20type%20%3d%7e%20%27microsoft.keyvault%2fvaults%27%0a%7c%20summarize%20count%28%29%20by%20location%0a%7c%20order%20by%20count_%20desc" target="_blank">portal.azure.com</a>
- Azure Government portal: <a href="https://portal.azure.us/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20where%20type%20%3d%7e%20%27microsoft.keyvault%2fvaults%27%0a%7c%20summarize%20count%28%29%20by%20location%0a%7c%20order%20by%20count_%20desc" target="_blank">portal.azure.us</a>
- Microsoft Azure operated by 21Vianet portal: <a href="https://portal.azure.cn/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20where%20type%20%3d%7e%20%27microsoft.keyvault%2fvaults%27%0a%7c%20summarize%20count%28%29%20by%20location%0a%7c%20order%20by%20count_%20desc" target="_blank">portal.azure.cn</a>

---

### Key vaults by SKU type

This query groups key vaults by their SKU tier (standard or premium). Premium SKUs are required for HSM-protected keys.

```kusto
Resources
| where type =~ 'microsoft.keyvault/vaults'
| extend skuName = tostring(properties.sku.name)
| summarize count() by skuName
```

# [Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type =~ 'microsoft.keyvault/vaults' | extend skuName = tostring(properties.sku.name) | summarize count() by skuName"
```

# [Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'microsoft.keyvault/vaults' | extend skuName = tostring(properties.sku.name) | summarize count() by skuName"
```

# [Portal](#tab/azure-portal)



- Azure portal: <a href="https://portal.azure.com/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20where%20type%20%3d%7e%20%27microsoft.keyvault%2fvaults%27%0a%7c%20extend%20skuName%20%3d%20tostring%28properties.sku.name%29%0a%7c%20summarize%20count%28%29%20by%20skuName" target="_blank">portal.azure.com</a>
- Azure Government portal: <a href="https://portal.azure.us/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20where%20type%20%3d%7e%20%27microsoft.keyvault%2fvaults%27%0a%7c%20extend%20skuName%20%3d%20tostring%28properties.sku.name%29%0a%7c%20summarize%20count%28%29%20by%20skuName" target="_blank">portal.azure.us</a>
- Microsoft Azure operated by 21Vianet portal: <a href="https://portal.azure.cn/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20where%20type%20%3d%7e%20%27microsoft.keyvault%2fvaults%27%0a%7c%20extend%20skuName%20%3d%20tostring%28properties.sku.name%29%0a%7c%20summarize%20count%28%29%20by%20skuName" target="_blank">portal.azure.cn</a>

---

### Key vaults with subscription name

The following query shows a complex use of `join` with **kind** as _leftouter_. The query limits the joined table to subscriptions resources and with `project` to include only the original field _subscriptionId_ and the _name_ field renamed to _SubName_. The field rename avoids `join` adding it as _name1_ since the field already exists in _resources_. The original table is filtered with `where` and the following `project` includes columns from both tables. The query result is all key vaults displaying type, the name of the key vault, and the name of the subscription it's in.

```kusto
Resources
| join kind=leftouter (ResourceContainers | where type=='microsoft.resources/subscriptions' | project SubName=name, subscriptionId) on subscriptionId
| where type == 'microsoft.keyvault/vaults'
| project type, name, SubName
```

# [Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | join kind=leftouter (ResourceContainers | where type=='microsoft.resources/subscriptions' | project SubName=name, subscriptionId) on subscriptionId | where type == 'microsoft.keyvault/vaults' | project type, name, SubName"
```

# [Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | join kind=leftouter (ResourceContainers | where type=='microsoft.resources/subscriptions' | project SubName=name, subscriptionId) on subscriptionId | where type == 'microsoft.keyvault/vaults' | project type, name, SubName"
```

# [Portal](#tab/azure-portal)



- Azure portal: <a href="https://portal.azure.com/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20join%20kind%3dleftouter%20(ResourceContainers%20%7c%20where%20type%3d%3d%27microsoft.resources%2fsubscriptions%27%20%7c%20project%20SubName%3dname%2c%20subscriptionId)%20on%20subscriptionId%0a%7c%20where%20type%20%3d%3d%20%27microsoft.keyvault%2fvaults%27%0a%7c%20project%20type%2c%20name%2c%20SubName" target="_blank">portal.azure.com</a>
- Azure Government portal: <a href="https://portal.azure.us/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20join%20kind%3dleftouter%20(ResourceContainers%20%7c%20where%20type%3d%3d%27microsoft.resources%2fsubscriptions%27%20%7c%20project%20SubName%3dname%2c%20subscriptionId)%20on%20subscriptionId%0a%7c%20where%20type%20%3d%3d%20%27microsoft.keyvault%2fvaults%27%0a%7c%20project%20type%2c%20name%2c%20SubName" target="_blank">portal.azure.us</a>
- Azure operated by 21Vianet portal: <a href="https://portal.azure.cn/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20join%20kind%3dleftouter%20(ResourceContainers%20%7c%20where%20type%3d%3d%27microsoft.resources%2fsubscriptions%27%20%7c%20project%20SubName%3dname%2c%20subscriptionId)%20on%20subscriptionId%0a%7c%20where%20type%20%3d%3d%20%27microsoft.keyvault%2fvaults%27%0a%7c%20project%20type%2c%20name%2c%20SubName" target="_blank">portal.azure.cn</a>

---

## Next steps

- Learn more about the [query language](/azure/governance/resource-graph/concepts/query-language).
- Learn more about how to [explore resources](/azure/governance/resource-graph/concepts/explore-resources).
