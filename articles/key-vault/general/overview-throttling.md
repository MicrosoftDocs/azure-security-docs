---
title: Azure Key Vault throttling guidance
description: Key Vault throttling limits the number of concurrent calls to prevent overuse of resources.
services: key-vault
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: general
ms.topic: conceptual
ms.date: 04/15/2025
ms.author: mbaldwin

---

# Azure Key Vault throttling guidance

Throttling is a process you initiate that limits the number of concurrent calls to the Azure service to prevent overuse of resources. Azure Key Vault (AKV) is designed to handle a high volume of requests. If an overwhelming number of requests occurs, throttling your client's requests helps maintain optimal performance and reliability of the AKV service.

Throttling limits vary based on the scenario. For example, if you are performing a large volume of writes, the possibility for throttling is higher than if you are only performing reads.

## How does Key Vault handle its limits?

Service limits in Key Vault prevent misuse of resources and ensure quality of service for all of Key Vault's clients. When a service threshold is exceeded, Key Vault limits any further requests from that client, returns HTTP status code 429 (Too many requests), and the request fails. Failed requests that return a 429 do not count towards the throttle limits tracked by Key Vault.

Key Vault was originally designed to store and retrieve your secrets at deployment time. As technology evolved, Key Vault is now increasingly used at runtime to store and retrieve secrets. Many applications and services use Key Vault similar to a database. However, the current service limits are not designed to support such high throughput scenarios.

Key Vault was originally created with the limits specified in [Azure Key Vault service limits](service-limits.md). To maximize your Key Vault throughput rates, here are some recommended guidelines/best practices for maximizing your throughput:
1. Ensure that throttling is in place. Client must honor exponential back-off policies for 429s and ensure you are doing retries per the guidance.
2. Divide your Key Vault traffic among multiple vaults and different regions.   Use a separate vault for each security/availability domain. If you have five apps, each in two regions, then we recommend 10 vaults each containing the secrets unique to app and region. A subscription-wide limit for all transaction types is five times the individual key vault limit. For example, HSM-other transactions per subscription are limited to 5,000 transactions in 10 seconds per subscription. Consider caching the secret within your service or app to also reduce the RPS directly to key vault and/or handle burst based traffic.  You can also divide your traffic among different regions to minimize latency and use a different subscription/vault. Do not send more than the subscription limit to the Key Vault service in a single Azure region.
3. Cache the secrets you retrieve from Azure Key Vault in memory, and reuse from memory whenever possible.  Re-read from Azure Key Vault only when the cached copy stops working (for example, because it got rotated at the source). 
4. Key Vault is designed for your own services secrets. If you are storing your customers' secrets (especially for high-throughput key storage scenarios), consider putting the keys in a database or storage account with encryption, and storing just the primary key in Azure Key Vault.
5. For public-key operations such as encryption, wrapping, and verification, perform these operations locally without accessing Key Vault by caching the public key material. This approach not only reduces the risk of throttling but also improves reliability of your application.
6. If you use Key Vault to store credentials for a service, check if that service supports Microsoft Entra authentication to authenticate directly. This reduces the load on Key Vault, improves reliability, and simplifies your code since Key Vault can now use the Microsoft Entra token. Many services now use Microsoft Entra authentication. See the current list at [Services that support managed identities for Azure resources](/azure/active-directory/managed-identities-azure-resources/services-support-managed-identities#azure-services-that-support-managed-identities-for-azure-resources).
7. Consider staggering your load/deployment over a longer period of time to stay under the current RPS limits.
8. If your app comprises multiple nodes that need to read one or more same secrets, then consider using a fan-out pattern, where one entity reads the secret from Key Vault, and fans out to all nodes. Cache the retrieved secrets only in memory.


## How to throttle your app in response to service limits

The following are **best practices** you should implement when your service is throttled:
- Reduce the number of operations per request.
- Reduce the frequency of requests.
- Avoid immediate retries. 
    - All requests accrue against your usage limits.

When you implement your app's error handling, use the HTTP error code 429 to detect the need for client-side throttling. If the request fails again with an HTTP 429 error code, you are still encountering an Azure service limit. Continue to use the recommended client-side throttling method, retrying the request until it succeeds.

Here is code that implements exponential backoff: 

```
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
    var client = new SecretClient(new Uri("https://keyVaultName.vault.azure.net"), new DefaultAzureCredential(),options);
                                 
    //Retrieve Secret
    secret = client.GetSecret(secretName);
```


Using this code in a client C# application is straightforward. 

### Recommended client-side throttling method

On HTTP error code 429, begin throttling your client using an exponential backoff approach:

1. Wait 1 second, retry request
2. If still throttled wait 2 seconds, retry request
3. If still throttled wait 4 seconds, retry request
4. If still throttled wait 8 seconds, retry request
5. If still throttled wait 16 seconds, retry request

At this point, you should not be getting HTTP 429 response codes.

## See also

For a deeper orientation of throttling on the Microsoft Cloud, see [Throttling Pattern](/azure/architecture/patterns/throttling).
