---
title: Authentication in Azure Cloud HSM
description: Learn about various authentication methods and best practices for securing and optimizing your Azure Cloud HSM deployment.
author: msmbaldwin
ms.service: azure-cloud-hsm
ms.topic: conceptual
ms.date: 03/20/2025
ms.author: mbaldwin
#customer intent: As a Cloud HSM administrator, I want to learn how to secure and optimize my Cloud HSM deployment so that I can ensure the highest level of security and performance.
---

# Authentication in Azure Cloud HSM

Authentication is a crucial aspect of securely accessing and operating within Azure Cloud HSM. This article outlines authentication methods, including command-line interface (CLI), PKCS#11, Java Cryptography Extension (JCE), and OpenSSL. This article also provides best practices for multithreading and session handling.

## Cloud HSM CLI authentication

You can authenticate by using CLI tools like `azcloudhsm_util` in either interactive mode or single-command mode. In interactive mode, use the `login` command. For single-command mode, include `singlecmd` and parameters for `loginHSM`. We advise you to securely store your HSM credentials when your application isn't using them.

### Interactive mode

```sh
./azcloudhsm_util
loginHSM -u CU -s cu1 -p user1234
```

### Single-command mode

```sh
sudo ./azcloudhsm_util singlecmd loginHSM -u CU -s cu1 -p user1234 findKey
```

## PKCS#11 authentication

In PKCS#11, you sign in by using the `C_Login` API after you open a session by using `C_OpenSession`. You need to use `C_Login` only once per slot (Cloud HSM cluster). After you successfully sign in, you can open additional sessions by using `C_OpenSession` without signing in again.

```c
char pPin[256] = "cu1:user1234";
…
rv = (func_list->C_Initialize) (NULL);
rv = (func_list->C_GetTokenInfo) (slot, &token_info);
rv = (func_list->C_OpenSession) (slot, CKF_SERIAL_SESSION | CKF_RW_SESSION, NULL, NULL, &session_rw);
rv = (func_list->C_Login) (session_rw, CKU_USER, (CK_UTF8CHAR_PTR) pPin, n_pin);
…
```

To get code examples for authenticating to PKCS#11, see the [guide for integrating PKCS#11 with Azure Cloud HSM](https://github.com/microsoft/MicrosoftAzureCloudHSM/blob/main/IntegrationGuides/Azure%20Cloud%20HSM%20PKCS11%20Integration%20Guide.pdf).

## JCE authentication

The JCE provider for Azure Cloud HSM offers support for both implicit and explicit sign-in methods. Each is suitable for different use cases.

We recommend that you use implicit sign-in whenever possible, because the SDK autonomously manages authentication. This method is particularly beneficial if your application disconnects from the cluster and requires reauthentication. Implicit sign-in also facilitates providing credentials to your application during integration with platforms where direct control over application code isn't feasible.

```java
LoginManager lm = LoginManager.getInstance();
lm.login("PARTITION_1","cu1", "user1234");
…
lm.logout();
…
```

For more details on sign-in methods, refer to the [guide for integrating JCE with Azure Cloud HSM](https://github.com/microsoft/MicrosoftAzureCloudHSM/blob/main/IntegrationGuides/Azure%20Cloud%20HSM%20JCE%20Integration%20Guide.pdf).

## OpenSSL authentication

When you use an OpenSSL engine for Azure Cloud HSM, environmental variables supply the credentials. We advise you to securely store your HSM credentials when your application isn't using them. Ideally, configure your environment to automatically retrieve and set these environment variables to avoid manual entry.

```sh
export azcloudhsm_password="cu1:user1234" 
export azcloudhsm_openssl_conf=/usr/local/bin/AzureCloudHSM-ClientSDK-1.0.4.0/azcloudhsm_openssl_dynamic.conf
export LD_LIBRARY_PATH=/usr/local/lib64/AzureCloudHSM-ClientSDK-1.0.4.0/:$LD_LIBRARY_PATH
…
sudo ./azcloudhsm_client azcloudhsm_client.cfg > /dev/null 2>&1 &
openssl genpkey -algorithm RSA -out private_key.pem -engine azcloudhsm_openssl
…
```

For authentication details with OpenSSL, consult the [guide for integrating OpenSSL with Azure Cloud HSM](https://github.com/microsoft/MicrosoftAzureCloudHSM/blob/main/IntegrationGuides/Azure%20Cloud%20HSM%20OpenSSL%20Integration%20Guide.pdf).

## Multithreading techniques

Azure Cloud HSM supports multithreaded applications, but there are considerations for handling them:

- **PKCS#11**: Initialize the PKCS#11 library by using `C_Initialize` only once. Assign each thread its own session by using `C_OpenSession`. Avoid using the same session across multiple threads.
- **JCE**: Initialize the Azure Cloud HSM provider only once. Avoid sharing instances of Service Provider Interface (SPI) objects across threads. For example, use `Cipher`, `Signature`, `Digest`, `Mac`, `KeyFactory`, and `KeyGenerator` objects only within their respective thread contexts.

## Retries for integration of HSM operations

Microsoft might swap out an HSM in your Azure Cloud HSM cluster for operational or maintenance purposes, like if a device fails. To prepare your application for such scenarios, we advise that you add client-side retry logic to all operations sent to your cluster. This setup anticipates that subsequent retries on failed operations, whether due to replacements or temporary maintenance outages, will be successful.

## Cloud HSM client session handling

The Azure Cloud HSM client signs in and out of all HSM sessions whenever any application performs a sign-in or sign-out. As a result, if a second application uses `azurecloudhsm_client`, it shares the same sessions and inherits the same sign-in credentials if it's running from the same host. The `azurecloudhsm_client` tool keeps track of which applications attempt to sign in. It allows properly signed-in applications to run commands that require authentication.

For example, if you're signed in with `azurecloudhsm_util` and you attempt to run your application or key tool in another terminal window with the Azure Cloud HSM provider, you encounter an error because an active session is already open. You must close the `azurecloudhsm_util` session for your application to create the session with `azurecloudhsm_client` running on your host for it to authenticate.

## Related content

- [Secure your Azure Cloud HSM deployment](secure-cloud-hsm.md)
- [Network security in Azure Cloud HSM](network-security.md)
- [Key management in Azure Cloud HSM](key-management.md)
- [User management in Azure Cloud HSM](user-management.md)
- [Overview of Azure Cloud HSM](overview.md)
