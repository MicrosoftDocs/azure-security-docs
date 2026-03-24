---
title: Quickstart - Azure Key Vault Managed HSM client library for .NET
description: Learn how to access keys in Azure Managed HSM using the .NET client library
author: msmbaldwin
ms.author: mbaldwin
ms.date: 03/13/2026
ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: quickstart
ms.devlang: csharp
ms.custom: devx-track-csharp, mode-api, devx-track-dotnet
#Customer intent: As a developer, I want to programmatically access keys in Managed HSM using the .NET SDK
---

# Quickstart: Azure Key Vault Managed HSM client library for .NET

Get started with the Azure Key Vault Managed HSM client library for .NET. [!INCLUDE [Managed HSM description](../includes/managed-hsm/intro.md)]

In this quickstart, you learn how to access and perform cryptographic operations on keys in a Managed HSM using the .NET client library.

Managed HSM client library resources:

[API reference documentation](/dotnet/api/azure.security.keyvault.keys) | [Library source code](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/keyvault) | [Package (NuGet)](https://www.nuget.org/packages/Azure.Security.KeyVault.Keys/)

## Prerequisites

[!INCLUDE [Managed HSM SDK prerequisites](../includes/managed-hsm/sdk-prereqs.md)]
- [.NET 6 SDK or later](https://dotnet.microsoft.com/download)

## Set up your local environment

This quickstart uses the Azure Identity library with Azure CLI to authenticate to Azure services. Developers can also use Visual Studio or Visual Studio Code to authenticate their calls. For more information, see [Authenticate the client with Azure Identity client library](/dotnet/api/overview/azure/identity-readme).

### Sign in to Azure

Run the `az login` command to sign in:

```azurecli
az login
```

## Create a new .NET console app

1. In a command shell, run the following command to create a project named `mhsm-console-app`:

    ```dotnetcli
    dotnet new console --name mhsm-console-app
    ```

1. Change to the newly created *mhsm-console-app* directory:

    ```dotnetcli
    cd mhsm-console-app
    ```

### Install the packages

Install the Azure Identity and Key Vault Keys client libraries:

```dotnetcli
dotnet add package Azure.Identity
dotnet add package Azure.Security.KeyVault.Keys
```

### Create the sample code

Replace the contents of `Program.cs` with the following code. Replace `<your-managed-hsm-name>` with your Managed HSM name, and `<your-key-name>` with an existing key name.

```csharp
using Azure.Identity;
using Azure.Security.KeyVault.Keys;
using Azure.Security.KeyVault.Keys.Cryptography;

// Use DefaultAzureCredential for automatic credential selection
// Works with managed identities in Azure, and developer credentials locally
var credential = new DefaultAzureCredential();

// Connect to Managed HSM - replace with your HSM URI
var hsmUri = new Uri("https://<your-managed-hsm-name>.managedhsm.azure.net");
var keyClient = new KeyClient(hsmUri, credential);

// Get a key reference
string keyName = "<your-key-name>";
Console.WriteLine($"Retrieving key '{keyName}' from Managed HSM...");
KeyVaultKey key = await keyClient.GetKeyAsync(keyName);
Console.WriteLine($"Key retrieved. Key type: {key.KeyType}");

// Perform cryptographic operations
var cryptoClient = new CryptographyClient(key.Id, credential);

// Encrypt data
byte[] plaintext = System.Text.Encoding.UTF8.GetBytes("Hello, Managed HSM!");
Console.WriteLine($"\nOriginal text: {System.Text.Encoding.UTF8.GetString(plaintext)}");

EncryptResult encryptResult = await cryptoClient.EncryptAsync(EncryptionAlgorithm.RsaOaep256, plaintext);
Console.WriteLine($"Encrypted (base64): {Convert.ToBase64String(encryptResult.Ciphertext)}");

// Decrypt data
DecryptResult decryptResult = await cryptoClient.DecryptAsync(EncryptionAlgorithm.RsaOaep256, encryptResult.Ciphertext);
Console.WriteLine($"Decrypted text: {System.Text.Encoding.UTF8.GetString(decryptResult.Plaintext)}");

Console.WriteLine("\nDone!");
```

### Run the application

Run the application with the `dotnet run` command:

```dotnetcli
dotnet run
```

You should see output similar to:

```output
Retrieving key 'myrsakey' from Managed HSM...
Key retrieved. Key type: RSA-HSM

Original text: Hello, Managed HSM!
Encrypted (base64): <encrypted-data>
Decrypted text: Hello, Managed HSM!

Done!
```

## Understanding the code

### Authentication with DefaultAzureCredential

[!INCLUDE [DefaultAzureCredential explanation](../includes/managed-hsm/sdk-default-credential.md)]

### Key operations

The `KeyClient` class provides methods to:
- Create, get, update, and delete keys
- List keys and key versions
- Backup and restore keys

The `CryptographyClient` class provides cryptographic operations:
- Encrypt and decrypt data
- Sign and verify signatures
- Wrap and unwrap keys

[!INCLUDE [Assign Managed HSM roles](../includes/managed-hsm/sdk-role-assignment.md)]

## Clean up resources

When no longer needed, delete the resource group and all related resources:

```azurecli
az group delete --name ContosoResourceGroup
```

[!INCLUDE [Managed HSM cleanup warning](../includes/managed-hsm/cleanup-warning.md)]

## Next steps

[!INCLUDE [SDK next steps](../includes/managed-hsm/sdk-next-steps.md)]
