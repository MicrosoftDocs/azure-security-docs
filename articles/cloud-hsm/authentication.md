---
title: Authentication in Azure Cloud HSM
description: Learn about various authentication methods and best practices for securing and optimizing your Azure Cloud HSM deployment.
author: msmbaldwin
ms.service: azure-cloud-hsm
ms.topic: conceptual
ms.date: 03/20/2025
ms.author: mbaldwin
# Customer intent: As a Cloud HSM administrator, I want to learn how to secure and optimize my Cloud HSM deployment so that I can ensure the highest level of security and performance.
---

# Authentication in Azure Cloud HSM

Authentication is a crucial aspect of securely accessing and operating within Azure Cloud HSM. This article outlines various authentication methods, including CLI, PKCS#11, JCE, and OpenSSL, as well as best practices for multi-threading and session handling.

## Cloud HSM CLI Authentication

You can authenticate with CLI utilities like `azcloudhsm_util` in either single command mode or interactive mode. In interactive mode, use the `login` command. For single command mode, include the `singlecmd` and parameters for `loginHSM`. Azure Cloud HSM advises securely storing your HSM credentials when not in use by your application.

### Interactive Mode

```sh
./azcloudhsm_util
loginHSM -u CU -s cu1 -p user1234
```

### Single Command Mode

```sh
sudo ./azcloudhsm_util singlecmd loginHSM -u CU -s cu1 -p user1234 findKey
```

## PKCS#11 Authentication

In PKCS#11, you login using the `C_Login` API after opening a session using `C_OpenSession`. You only need to perform one `C_Login` per slot (Cloud HSM Cluster). After you have successfully logged in, you can open additional sessions using `C_OpenSession` without the need to perform additional login operations. For examples on authenticating to PKCS#11, please see the Azure Cloud HSM PKCS#11 Integration Guide for code samples.

```c
char pPin[256] = "cu1:user1234";
…
rv = (func_list->C_Initialize) (NULL);
rv = (func_list->C_GetTokenInfo) (slot, &token_info);
rv = (func_list->C_OpenSession) (slot, CKF_SERIAL_SESSION | CKF_RW_SESSION, NULL, NULL, &session_rw);
rv = (func_list->C_Login) (session_rw, CKU_USER, (CK_UTF8CHAR_PTR) pPin, n_pin);
…
```

## JCE Authentication

The Azure Cloud HSM JCE Provider offers support for both implicit and explicit login methods, each suitable for different use cases. **Implicit Login** is recommended whenever possible because the SDK autonomously manages authentication. This is particularly beneficial if your application encounters disconnecting from the cluster and requires re-authentication. Implicit login also facilitates providing credentials to your application when integrating with platforms where direct control over application code isn't feasible. For further details on login methods, please refer to the Azure Cloud HSM JCE Integration Guide.

```java
LoginManager lm = LoginManager.getInstance();
lm.login("PARTITION_1","cu1", "user1234");
…
lm.logout();
…
```

## OpenSSL Authentication

When utilizing the Azure Cloud HSM OpenSSL Engine, credentials are supplied via environmental variables. Microsoft advises securely storing your HSM credentials when not in use by your application. Ideally, configure your environment to automatically retrieve and set these environment variables to avoid manual entry. For authentication details with OpenSSL, please consult the Azure Cloud HSM OpenSSL Integration Guide.

```sh
export azcloudhsm_password="cu1:user1234" 
export azcloudhsm_openssl_conf=/usr/local/bin/AzureCloudHSM-ClientSDK-1.0.4.0/azcloudhsm_openssl_dynamic.conf
export LD_LIBRARY_PATH=/usr/local/lib64/AzureCloudHSM-ClientSDK-1.0.4.0/:$LD_LIBRARY_PATH
…
sudo ./azcloudhsm_client azcloudhsm_client.cfg > /dev/null 2>&1 &
openssl genpkey -algorithm RSA -out private_key.pem -engine azcloudhsm_openssl
…
```

## Employing Multi-threading Techniques

Azure Cloud HSM supports multi-threaded applications, but there are considerations for handling them:

- **For PKCS#11**, initialize the PKCS#11 library using `C_Initialize` only once. Assign each thread its own session using `C_OpenSession`. Avoid using the same session across multiple threads.
- **For JCE**, initialize the Azure Cloud HSM provider only once. Avoid sharing instances of SPI objects across threads. For instance, `Cipher`, `Signature`, `Digest`, `Mac`, `KeyFactory`, or `KeyGenerator` objects should only be used within their respective thread context.

## Incorporate Retries for HSM Operations Integration

Microsoft may swap out an HSM in your Azure Cloud HSM cluster for operational or maintenance purposes, like if a device fails. To prepare your application for such scenarios, Microsoft advises adding client-side retry logic to all operations sent to your cluster. This setup anticipates that subsequent retries on failed operations, whether due to replacements or temporary maintenance outages, will be successful.

## Cloud HSM Client Session Handling

The Azure Cloud HSM client logs in and out of all HSM sessions whenever any application performs a login or logout. As a result, if a second application uses the `azurecloudhsm_client`, it shares the same sessions and inherits the same login credentials if running from the same host. The `azurecloudhsm_client` keeps track of which applications have attempted to log in and only allows properly logged-in applications to execute commands that require authentication. 

For example, if you're logged in with `azurecloudhsm_util` and attempt to execute your application or key tool in another terminal window with the Azure Cloud HSM provider, you will encounter an error because an active session is already open. You must close the `azurecloudhsm_util` session for your application to create the session with the `azurecloudhsm_client` running on your host for it to authenticate.

## Next steps

- [Secure your Azure Cloud HSM](secure-cloud-hsm.md)
- [Network security](network-security.md)
- [Key Management in Azure Cloud HSM](key-management.md)
- [User Management in Azure Cloud HSM](user-management.md)
- [Authentication in Azure Cloud HSM](authentication.md)
- [Overview of Azure Cloud HSM](overview.md)
