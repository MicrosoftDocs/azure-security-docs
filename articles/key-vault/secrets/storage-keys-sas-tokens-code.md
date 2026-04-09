---
title: Fetch shared access signature tokens in code | Azure Key Vault
description: The managed storage account feature provides a seamless integration between Azure Key Vault and an Azure storage account. This sample uses the Azure SDK for .NET to manage SAS tokens.
ms.topic: tutorial
ms.service: azure-key-vault
ms.subservice: secrets
author: msmbaldwin
ms.author: mbaldwin
ms.date: 04/09/2026

ms.devlang: csharp
ms.custom: devx-track-csharp, devx-track-dotnet
# Customer intent: As a developer I want storage credentials and SAS tokens to be managed securely by Azure Key Vault.
---
# Create SAS definition and fetch shared access signature tokens in code (legacy)

> [!CAUTION]
> Key Vault-managed storage account keys and SAS token generation is a **legacy feature that is deprecated**. The underlying SDK sample has been retired because this approach goes against current Microsoft security guidance. Instead, use [Azure role-based access control (Azure RBAC)](/azure/storage/blobs/authorize-access-azure-active-directory) to secure your storage account. For more information, see [Authorize access to data in Azure Storage](/azure/storage/common/authorize-data-access).

You can manage your storage account with shared access signature (SAS) tokens stored in your key vault. For more information, see [Grant limited access to Azure Storage resources using SAS](/azure/storage/common/storage-sas-overview).

This article provides samples of .NET code that creates a SAS definition and fetches SAS tokens. For information on how to create and store SAS tokens, see [Manage storage account keys with Key Vault and the Azure CLI](overview-storage-keys.md) or [Manage storage account keys with Key Vault and Azure PowerShell](overview-storage-keys-powershell.md).

## Code samples

In the following example, a SAS template is created:

```csharp
private static string BuildSasDefinitionTemplate(bool readOnly) =>
        new StringBuilder("sv=2018-03-28")  // service version
            .Append("&spr=https")           // HTTPS only
            .Append("&ss=bf")               // blobs and files only
            .Append("&srt=o")               // applies to objects only
            .Append(readOnly ? "&sp=r" : "&sp=rw")  // read-only or read-write
            .ToString();
```

Using this template, you can create a SAS definition by using the following code:

```csharp
string sasDefinitionName = BuildSasDefinitionName(Tag, readOnly, duration);
        SasDefinitionAttributes sasDefinitionAttributes = new SasDefinitionAttributes
        {
            Enabled = true,
        };

        Dictionary<string, string> tags = new Dictionary<string, string>
        {
            [Tag] = "1",
        };

        SasDefinitionBundle createdSasDefinition = await storageClient.SetSasDefinitionAsync(
            storageAccountName,
            sasDefinitionName,
            sasTemplate,
            SasTokenType.Account,
            duration,
            sasDefinitionAttributes,
            tags,
            s_cancellationTokenSource.Token);
```

Once the SAS definition is created, you can retrieve SAS tokens like secrets using a `SecretClient`. You need to preface the secret name with the storage account name followed by a dash:

```csharp
// Build our SAS template, get an existing SAS definition, or create a new one.
        string sasTemplate = BuildSasDefinitionTemplate(readOnly);
        string sasDefinitionName = await GetOrCreateSasDefinitionAsync(storageClient, storageAccountName, sasTemplate, days, readOnly);

        // Now we can create a SecretClient and generate a new SAS token from the storage account and SAS definition names.
        SecretClient secretClient = new SecretClient(vaultUri, credential, options);
        KeyVaultSecret sasToken = await secretClient.GetSecretAsync($"{storageAccountName}-{sasDefinitionName}", cancellationToken: s_cancellationTokenSource.Token);
```

If your shared access signature token is about to expire, you can fetch the same secret again to generate a new one.

For a guide on how to use a SAS token retrieved from Key Vault to access Azure Storage services, see [Use an account SAS to access Blob service](/azure/storage/common/storage-account-sas-create-dotnet#use-an-account-sas-from-a-client)

> [!NOTE]
> Your app needs to be prepared to refresh the SAS if it gets a 403 from Storage so that you can handle the case where a key was compromised and you need to rotate them faster than the normal rotation period. 

## Next steps
- Learn how to [Grant limited access to Azure Storage resources using SAS](/azure/storage/common/storage-sas-overview).
- Learn how to [Manage storage account keys with Key Vault and the Azure CLI](overview-storage-keys.md) or [Azure PowerShell](overview-storage-keys-powershell.md).
- See [Managed storage account key samples](https://github.com/Azure-Samples?utf8=%E2%9C%93&q=key+vault+storage&type=&language=)
