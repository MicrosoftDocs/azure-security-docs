---
title: Troubleshoot Azure Cloud HSM
description: Learn how to troubleshoot common Azure Cloud HSM problems and errors.
author: keithp
manager: davinune
ms.service: azure-cloud-hsm
ms.topic: troubleshooting-general
ms.date: 03/20/2025
ms.author: keithp
---

# Troubleshoot Azure Cloud HSM

## Common error messages

### Why am I getting the error message "Certificate ERROR: [Certificate signature failure]" when running azcloudhsm_mgmt_util?

This error can occur when you:

1. Create and initialize your Azure Cloud HSM deployment from another virtual machine (VM).
1. Attempt to install the Cloud HSM Client SDK and run from another VM that's missing or doesn't have the correct `PO.crt` file from the admin VM that you initialized from.

If you copy the `PO.crt` file from your admin VM to your new VM and rerun `azcloudhsm_mgmt_util`, you should see a successful connection to your hardware security modules (HSM).

### Why am I getting the error message "INF: shutdown_ssl_socket: SSL_shutdown sent close_notify alert" when running azcloudhsm_client?

This error commonly occurs when you:

1. Create and initialize your Azure Cloud HSM deployment from another VM.
1. Attempt to install the Cloud HSM Client SDK and run from another VM that's missing or doesn't have the correct `PO.crt` file from the admin VM that you initialized from.

If you copy the `PO.crt` file from your admin VM to your new VM and rerun `azcloudhsm_client`, you should see a successful connection to your HSM.

### Why am I getting the error message "azcloudhsm_application.cfg is not present"?

To ensure the successful execution of your application, two conditions must be met:

- The `azcloudhsm_application.cfg` file must be present in the same directory as your application.
- The `azcloudhsm_client` tool should be running. If `azcloudhsm_client` is not active, you encounter a failed socket connection.

  Likewise, the absence of the `azcloudhsm_application.cfg` file results in an error message. Depending on the execution location of your application, you might need to copy the `azcloudhsm_application.cfg` file from the Azure Cloud HSM SDK directory to the directory where you're running your application.

## OpenSSL engine for Azure Cloud HSM

### Does the OpenSSL engine for Azure Cloud HSM support Windows?

No. The OpenSSL engine for Azure Cloud HSM supports Linux (Ubuntu 20.04, Ubuntu 22.04, RHEL 7, RHEL 8, CBL Mariner 2) only.

### Why am I getting the error message "invalid engine 'azcloudhsm_openssl' could not load the shared library"?

The error message suggests that the prerequisite environment variable `LD_LIBRARY_PATH` isn't configured, and this configuration is necessary. Set the `LD_LIBRARY_PATH` environment variable to the directory where the Azure Cloud HSM SDK is located.

You must update your environment variables to reflect the correct SDK version by running:

```bash
export LD_LIBRARY_PATH=/opt/azurecloudhsm/lib64/:$LD_LIBRARY_PATH
```

### Why am I getting the error message "environment variable azcloudhsm_openssl_conf is not set"?

The error message signals that the prerequisite environment variable `azcloudhsm_openssl_conf` is not configured. Set the `azcloudhsm_openssl_conf` environment variable to the file path location of your `azcloudhsm_openssl_dynamic.conf` file within the Azure Cloud HSM SDK directory.

You must update your environment variables to reflect the correct SDK version by running:

```bash
export azcloudhsm_openssl_conf=/opt/azurecloudhsm/bin/azcloudhsm_openssl_dynamic.conf  
```

### Why am I getting the error message "environment variable azcloudhsm_password is not set"?

The error message suggests that the prerequisite environment variable `azcloudhsm_password` is not configured. Set the `azcloudhsm_password` environment variable to the value of `cryptouser:password`.

You must update your environment variables to reflect the correct cryptography user and password:

```bash
export azcloudhsm_password="cu1:user1234"
```

## JCE provider for Azure Cloud HSM

### Does JCE integration with Azure Cloud HSM support Windows?

No. Java Cryptography Extension (JCE) integration with Azure Cloud HSM supports Linux (Ubuntu 20.04, Ubuntu 22.04, RHEL 7, RHEL 8, CBL Mariner 2) only.

### Why am I getting the error message "could not find credentials to login to the HSM"?

The error indicates a challenge in locating or authenticating the credentials required for logging in to Cloud HSM. When you're using the samples in this documentation or incorporating the JCE provider for Azure Cloud HSM into your applications, it's essential to explicitly log in with credentials:

```bash
// Explicit Login with Credentials
LoginManager lm=LoginManager.getInstance();
lm.login("PARTITION_1","cu1","user1234");

// Logout of the HSM
lm.logout();
```

### Why am I getting the error message "attribute with value 0 for this operation? Attribute should set for User KEK"?

The error indicates that a user-generated key encryption key (KEK) was not created and designated as extractable, and `wrap_with_trusted` is set to 1. For appropriate settings to use when you perform key import, key wrap, and key unwrap operations, see the section about creating a user-generated KEK in the [JCE integration guide](https://github.com/microsoft/MicrosoftAzureCloudHSM/blob/main/IntegrationGuides/Azure%20Cloud%20HSM%20JCE%20Integration%20Guide.pdf).

### Why am I getting the error message "Daemon socket connection error, API login failed"?

The error indicates a challenge in locating or authenticating the credentials required for logging in to Cloud HSM. When you're using the samples in the [JCE integration guide](https://github.com/microsoft/MicrosoftAzureCloudHSM/blob/main/IntegrationGuides/Azure%20Cloud%20HSM%20JCE%20Integration%20Guide.pdf) or incorporating the Azure Cloud HSM JCE provider into your applications, it's essential to explicitly sign in with credentials to ensure connection.

## PKCS#11 library for Azure Cloud HSM

### Why am I getting the error message "C_Login failed with error code: 0xa3"?

The `0xa3` error indicates a sign-in failure. The failure might be related to the format of the PIN parameter you're passing, or the PIN (password) might be incorrect.

The PIN should be provided in the following format: `char pPin[256] = "crypto_user:user123";`. The `pPin` value should follow the structure of "`<username>`:`<password>`". Verify that you're providing the `pPin` value in this format, because any deviation might result in a sign-in failure.

### Why am I getting the error message "C_Initialize() failed with 00000005 (Failed to connect socket)"?

A failed socket connection during `C_Initialize` might be related to the `azcloudhsm_client` tool not running on your system. For your PKCS#11 applications to run successfully, `azcloudhsm_client` must be running. You can start `azcloudhsm_client` by running one of the following commands in a separate terminal.

#### Linux (recommended)

If you installed the Azure Cloud HSM SDK by using DEB or RPM, the client isn't configured automatically to run as a service. The SDK, during installation, includes a service unit file under `/etc/systemd/system/azure-cloud-hsm.service`. To enable `azcloudhsm_client` to run as a service, use the predefined `azure-cloud-hsm.service` file.

Reload the `systemd` configuration and enable the service to ensure that it's continuously running. Performing the following steps:

1. Open a terminal window and change the directory to `/etc/systemd/system`, where the Cloud HSM service unit file is located:

   ```bash
   cd /etc/systemd/system
   ```

1. Start and enable the Cloud HSM client service:

   ```bash
   sudo systemctl start azure-cloud-hsm.service
   sudo systemctl enable azure-cloud-hsm.service
   ```

1. Check the status of the Cloud HSM client service:

   ```bash
   sudo systemctl status azure-cloud-hsm.service
   ```

1. Reload the `systemd` configuration:

   ```bash
   sudo systemctl daemon-reload
   sudo systemctl list-units --type=service --state=running
   ```

#### Linux (manual)

Start the client daemon if it isn't running:

```bash
sudo ./azcloudhsm_client azcloudhsm_client.cfg
```

You can also choose to run the client daemon in the background by using the following command. Or you can run the client daemon as a service to ensure that it's always running.

```bash
sudo ./azcloudhsm_client azcloudhsm_client.cfg > /dev/null 2>&1 &
```

#### Windows (recommended)

If you installed the Azure Cloud HSM SDK by using an MSI package, the client is already configured to run as a service. If the client isn't running, you can open `Services.msc`, right-click **Microsoft Azure Cloud HSM Client Service**, and then select **Start**.

#### Windows (manual)

Start the client daemon if it isn't running:

```bash
cd C:\Program Files\Microsoft Azure Cloud HSM Client SDK 
.\azcloudhsm_client.exe .\azcloudhsm_resource.cfg 
```

### How does the PKCS#11 library for Linux know how to find the client configuration?

The PKCS#11 library knows how to find the client configuration because you must have a copy of your partition owner certificate (`PO.crt`) on the application server that's running your application and using the PKCS#11 library. In addition to the partition owner certificate:

- You have to update `/azcloudhsm_client/azcloudhsm_client.cfg` on your application server that has the SDK installed to point to your Azure Cloud HSM deployment. (That is, point to `hsm1.chsm-<resourcename>-<uniquestring>.privatelink.cloudhsm.azure.net`.)
- The `azcloudhsm_client` tool must be running on the application server that connects to your Azure Cloud HSM deployment.
- You must specify a PIN within your PKCS#11 application by using the syntax `<username>:<password>`. This PIN is used for calling `C_Login` to your Azure Cloud HSM deployment.
- You must include `pkcs11_headers/include/cryptoki.h` and `pkcs11_headers/include/pkcs11t.h` in your PKCS#11 application to use the PKCS#11 library for Azure Cloud HSM.

### How does the PKCS#11 library (azcloudhsm_pkcs11.dll) for Windows know how to find the client configuration?

The `azcloudhsm_pkcs11.dll` file in the Azure Cloud HSM Windows SDK knows how to find the client configuration because you must have a copy of your partition owner certificate (`PO.crt`) on the application server that's running your application and using the PKCS#11 library. In addition to the partition owner certificate:

- You have to update `/azcloudhsm_client/azcloudhsm_client.cfg` on your application server that has the SDK installed to point to your Azure Cloud HSM deployment. (That is, point to `hsm1.chsm-<resourcename>-<uniquestring>.privatelink.cloudhsm.azure.net`.)
- The `azcloudhsm_client` tool must run on the application server that connects to your Azure Cloud HSM deployment.
- You must specify a PIN within your PKCS#11 application by using the syntax `<username>:<password>`. This PIN is used for calling `C_Login` to your Azure Cloud HSM deployment.
- You must include `pkcs11_headers\include\cryptoki.h` and `pkcs11_headers\include\pkcs11t.h` in your PKCS#11 application to use the PKCS#11 library for Azure Cloud HSM.

### Why do I get PKCS#11 errors when I try to set a key or keystore?

Ensure that you have the Azure Cloud HSM client running (`sudo ./azcloudhsm_client azcloudhsm_client.cfg`). An incorrect user, an incorrect password, a liquid security client not running, or a client failing to connect might result in the following error:

```bash
Error Message:ERROR at line 1:
ORA-28407: Hardware Security Module failed with PKCS#11 error
CKR_GENERAL_ERROR(5)
```

### How does the PKCS#11 library (azcloudhsm_pkcs11.dll) integrate with OpenSC?

You must use the `--sensitive` and `--private` arguments when you use  open-source smart card (OpenSC) tools, because the sensitive and private attributes can't be `0`. Without those arguments, OpenSC tooling sets the sensitive and private attributes to `0`. These attributes generate an exception, because Azure Cloud HSM always imposes sensitive and private behavior.

Here's an Advanced Encryption Standard (AES) wrapping example:

```bash
pkcs11-tool.exe --sensitive --private --module "C:\AzureCloudHSM-ClientSDK\AzureCloudHSM-ClientSDK-Windows-1.0.3.0\libs\pkcs11\azcloudhsm_pkcs11.dll" --login --login-type user --pin cu1:user1234 --keygen --key-type AES:32*
```

Here's an Elliptic Curve Cryptography (ECC) P521 example:

```bash
pkcs11-tool.exe --sensitive --private --module "C:\AzureCloudHSM-ClientSDK\AzureCloudHSM-ClientSDK-Windows-1.0.3.0\libs\pkcs11\azcloudhsm_pkcs11.dll" --login --login-type user --pin cu1:user1234 --keypairgen --key-type EC:prime256v1 --usage-sign
```

## SSL/TLS offloading for Azure Cloud HSM

### Does the OpenSSL engine for Azure Cloud HSM support PKCS#11 for SSL/TLS offloading?

No. The OpenSSL engine for Azure Cloud HSM doesn't support PKCS#11 for SSL/TLS offloading. Customers who require PKCS#11 for SSL/TLS offloading must use the [TLS Offload library for Azure Managed HSM](https://github.com/microsoft/AzureManagedHsmTLSOffload).

### Why am I getting the error message "Failed to connect socket, LIQUIDSECURITY: Daemon socket connection error" when running azcloudhsm_util?

The error message suggests that `azcloudhsm_client` isn't running. Ensure that `azcloudhsm_client` is running for the Azure Cloud HSM tools to run properly. You can start `azcloudhsm_client` by ensuring that it's running as a service or by manually running one of the following commands in a separate terminal.

#### Linux (recommended)

If you installed the Azure Cloud HSM SDK by using DEB or RPM, the client isn't configured automatically to run as a service. The SDK, during installation, includes a service unit file under `/etc/systemd/system/azure-cloud-hsm.service`. To enable `azcloudhsm_client` to run as a service, use the predefined `azure-cloud-hsm.service` file.

Reload the `systemd` configuration and enable the service to ensure that it's continuously running. Perform the following steps:

1. Open a terminal window and change the directory to `/etc/systemd/system`, where the Cloud HSM service unit file is located:

   ```bash
   cd /etc/systemd/system
   ```

1. Start and enable the Cloud HSM client service:

   ```bash
   sudo systemctl start azure-cloud-hsm.service
   sudo systemctl enable azure-cloud-hsm.service
   ```

1. Check the status of the Cloud HSM client service:

   ```bash
   sudo systemctl status azure-cloud-hsm.service
   ```

1. Reload the `systemd` configuration:

   ```bash
   sudo systemctl daemon-reload
   sudo systemctl list-units --type=service --state=running
   ```

#### Linux (manual)

Start the client daemon if it isn't running:

```bash
sudo ./azcloudhsm_client azcloudhsm_client.cfg
```

You can also choose to run the client daemon in the background by using the following command. Or you can run the client daemon as a service to ensure that it's always running.

```bash
sudo ./azcloudhsm_client azcloudhsm_client.cfg > /dev/null 2>&1 &
```

### Why am I getting the error message "environment variable azcloudhsm_password is not set"?

The error message suggests that the prerequisite environment variable `azcloudhsm_password` is not configured. To resolve this problem, set the `azcloudhsm_password` environment variable to the value of `cryptouser:password`.

Update your environment variables to reflect the correct cryptography user and password:

```bash
export azcloudhsm_password="cu1:user1234"
```

### Why am I getting the error message "Key/Token not found" or "Invalid key-handle/token" when I try to import a key?

The error message suggests that the specified wrapping key handle (the KEK handle ID) is incorrect. The parameter `-f` specifies the file name that contains the key to import, whereas `-w` specifies the wrapping key handle. During the creation of a user-generated KEK, the `-w` value should be the key handle ID of your user-generated KEK ID.

```bash
azcloudhsm_util singlecmd loginHSM -u CU -p user1234 -s cu1 importPrivateKey -l importTestKey -f key.pem -w 262150
```

### Why am I getting the error message "/configure: error: C compiler cc not found" when I try to configure the NGINX sources?

The error message suggests that you're missing a C compiler from your system when you run the `configure` command. Depending on your installation, after you run the following commands, you might also need to install `openssl libssl-dev` and `zlib1g zlib1g-dev`. These items are dependencies.

#### For Red Hat-based systems (using YUM)

```bash
sudo yum update
sudo yum groupinstall "Development Tools"
```

#### For Red Hat-based systems (using DNF)

```bash
sudo dnf update
sudo dnf install "Development Tools"
```

#### For Ubuntu-based systems (using APT)

```bash
sudo apt update  
sudo apt install build-essential 
```

### Why am I getting the error message "/configure: error: SSL modules require the OpenSSL library"?

The error message indicates that the OpenSSL library is required to enable SSL modules for NGINX. To resolve this problem, run the following command and then run `configure` again:

```bash
sudo apt-get install libssl-dev
```

### Why am I getting the error message "/configure: error: the HTTP gzip module requires the zlib library"?

The error message indicates that the zlib library is required for the HTTP gzip module in NGINX. To resolve this problem, run the following command and then run `configure` again:

```bash
sudo apt-get install zlib1g-dev
```

### Why am I getting the error message "HSM Error: RET_USER_LOGIN_FAILURE"?

The error message indicates that the specified user doesn't exist on one or more nodes where you're attempting a cryptographic operation. Although Azure Cloud HSM operates with a cluster of three nodes, the operation was directed to a node where the login failed.

In such cases, a `critical_err_info_` file is generated within the `AzureCloudHSM-ClientSDK` directory. The file shows "Status: HSM Error: This user doesn't exist."

To mitigate this error, you need to create a cryptography user that's replicated across all three nodes of your Azure Cloud HSM deployment. The following example shows starting `azcloudhsm_mgmt_util` and an administrator logging on as a Controlling (CO) user. It then proceeds to create a `cu1` user with the role of a cryptography user.

```bash
sudo ./azcloudhsm_mgmt_util ./azcloudhsm_mgmt_util.cfg
loginHSM CO admin adminpassword 
createUser CU cu1 user1234
logoutHSM
loginHSM CU cu1 user1234
```

### How do I display the contents of the CRT, CSR, and key files that I created?

To display the contents of the CRT, CSR, or key file that you created in your terminal, use this command:

```bash
cat filename.key
```

How to display detailed information about an X.509 certificate contained in the PEM file. If your PEM file contains a private key or other types of data, you might need to adjust the command accordingly:

```bash
openssl x509 -in filename.pem -text -noout
```

How to display detailed information about a CSR, including the subject, public key, and any attributes included in the request:

```bash
openssl req -in filename.csr -text -noout
```

How to display detailed information about an RSA private key, including its modulus, public exponent, and other parameters:

```bash
openssl rsa -in filename.key -text -noout
```

How to display detailed information about an EC private key, including the curve parameters, private key value, and other relevant information:

```bash
openssl ec -in filename.key -text -noout
```

### How do I generate an ED25519 key pair in Azure Cloud HSM for signing?

ED25519 keys are typically used for self-signed certificates or in certificate signing processes that directly use the private key. You can generate an ED25519 key pair using azcloudhsm_util or the Azure Cloud HSM OpenSSL engine.

> [!IMPORTANT]
> Microsoft does not recommend using openssl genpkey or openssl ecparam. Using openssl genpkey or openssl ecparam with the -engine azcloudhsm_openssl to generate an ED25519 or ECCKeyPair doesn't produce an HSM key; instead, it generates a software key. Customers needing ED25519 and other EC Key types must utilize azcloudhsm_util to create the key within the HSM.
>
> Openssl genpkey and openssl ecparam are both OpenSSL commands used for generating cryptographic keys, but they serve different purposes and support different key types. Use openssl genpkey when you need to generate a wide range of asymmetric cryptographic keys, including RSA, DSA, ECDSA, and ED25519 keys. Use openssl ecparam specifically for generating elliptic curve cryptographic keys (ECDSA), such as prime256v1, secp384r1, or secp521r1.

```bash
./azcloudhsm_util singlecmd loginHSM -u CU -p user1234 -s cu1 genECCKeyPair -i 28 -l labelED25519Test
```

### Can I use azcloudhsm_util to generate RSA and EC keys before using Azure Cloud HSM OpenSSL engine to generate CSR?

Yes. The following azcloudhsm_util commands can be run to create an RSA or EC key and then extract the private key to a fake pem format. Replace {PRIVATE_KEY_HANDLE} with the private key handle of the RSA or EC key you created. The private key meta file in PEM format doesn't contain any sensitive private key materials. It is metadata that identifies the private key, and this metafile is only understood by the Azure Cloud HSM OpenSSL engine.

RSA key:

```bash
./azcloudhsm_util singlecmd loginHSM -u CU -p user1234 -s cu1 genRSAKeyPair -m 2048 -e 65537 -l labelRSATest 
./azcloudhsm_util singlecmd loginHSM -u CU -p user1234 -s cu1 getCaviumPrivKey -k {PRIVATE_KEY_HANDLE} -out web_server_fake_PEM.key 
openssl req -new -key web_server_fake_PEM.key -out web_server.csr -engine azcloudhsm_openssl 
openssl x509 -req -days 365 -in web_server.csr -signkey web_server_fake_PEM.key -out web_server.crt -engine azcloudhsm_openssl 
```

EC key:

```bash
./azcloudhsm_util singlecmd loginHSM -u CU -p user1234 -s cu1 genECCKeyPair -i 2 -l labelECTest 
./azcloudhsm_util singlecmd loginHSM -u CU -p user1234 -s cu1 getCaviumPrivKey -k {PRIVATE_KEY_HANDLE} -out web_server_fake_PEM.key 
openssl req -new -key web_server_fake_PEM.key -out web_server.csr -engine azcloudhsm_openssl 
openssl x509 -req -days 365 -in web_server.csr -signkey web_server_fake_PEM.key -out web_server.crt -engine azcloudhsm_openssl 
```

## Azure Cloud HSM SQL EKM

### Does Azure Cloud HSM SQL EKM Provider support Symmetric Keys?

No. Microsoft SQL Server doesn't support symmetric keys for SQL TDE. Only asymmetric keys are supported.

### Does Azure Cloud HSM SQL EKM Provider support SQL Server on Linux?

No. Azure Cloud HSM SQL EKM provider only supports Windows Server.

### Does Azure Cloud HSM SQL EKM Provider support SQL Server SaaS/PaaS?

No. Azure Cloud HSM is IaaS only. Support for SaaS/PaaS customers should use Azure Managed HSM which supports SQL TDE for Azure SQL DB and Azure SQL Managed Instance.

### Does Azure Cloud HSM SQL EKM support multiple logins, credentials, connections?

Azure Cloud HSM SQL EKM the login or SQL connection is limited and can only have a single credential assigned.

### Why am I getting the error message "Error: Cannot initialize cryptographic provider. Provider error code: 1. (Failure - Consult EKM provider for details)"?

This error commonly occurs when the Azure Cloud HSM MSI package is run after SQL Server is installed. The error goes away when you restart SQL Server, when it picks up the environment variables and settings created when the Azure Cloud HSM MSI package was executed.

## Azure Cloud HSM ADCS Integration

### Why do I get the error message "Can't open openssl-ca.cnf file" when trying to migrate a certificate from Microsoft KSP from the provided example?

This error commonly occurs when the openssl-ca.cnf does not exist. You can create one yourself or obtain it from another source.

### Can we use our own signing servers in our corporate network that consumes Azure Cloud HSM as a SaaS solution?

Azure Cloud HSM is IaaS only. You can use your own signing servers in your corporate network, however. You can configure Site-to-Site or Point-to-Site VPN connection from your local network gateway. This is the most common method, and it's suitable for most use cases. Set up a VPN connection between your corporate network's on-premises and Azure to where Azure Cloud HSM is deployed.

- [Tutorial - Connect an on-premises network and a virtual network: S2S VPN: Azure portal](/azure/vpn-gateway/tutorial-site-to-site-portal)
- You can have AD CS for your signing servers on-premises. What is required is SDK must be on your on-premises host and the client be reachable to your Azure virtual network private IPs.
  - The customer needs to run the azcloudhsm_client on the same machine where Azure Cloud HSM CNG provider exists.
  - The azcloudhsm_client should be able to reach out to the Customers Azure virtual network private IPs.

When testing AD CS, we generated a RSAKeyPair then used CFM to sign.

### Are the only accepted methods to sign or verify through Cloud HSM providers (for instance, azcloudhsm_util)?

No. Our recommendation is to use the Azure Cloud HSM SDK and the interfaces (PKCS#11, CNG, KSP, JCE, OpenSSL Engine, Etc.) it provides. You can use the Sign Tool for sign/verify operations as well.

ADCS is configured to use Azure Cloud HSM CNG/KSP. Can we use the signing / verify from Cloud HSM via signtool.exe? Yes. The Sign Tool automatically goes to your Cloud HSM. There is no need to use azcloudhsm_util. We included azcloudhsm_util in the instructions as a way for customers to quickly test and validate their ADCS configuration.

### Can we use self-signed certs based on the key pair in Cloud HSM for testing?

Yes. You can use self-signed certificates based on the key-pair for testing purposes.

### Is there an integration or way to make the certs available via the local Windows certificate store?

There is a tool called certreq.exe that can be used for this purpose.

### What algorithms do Azure Cloud HSM CNG and KSP providers support?

- RSA encryption/decryption.
- RSA signing with SHA1, SHA256, SHA384, SHA512, and MD5 Hash algorithms.
- ECC curves ECDSA_P256, ECDSA_P384, and ECDSA_P521.

### How do I change or manage private key permissions for nonadministrative users on Windows?

When generating a key using Azure Cloud HSM KSP or CNG, only the administrator is granted access to the private key. You might encounter difficulties in viewing security information or modifying permissions via MMC. This behavior is intentional. By default, when using KSP or CNG, access to the private key file for a new key is restricted to SYSTEM and administrators. If you need to provide access to a nonadministrative user for a specific private key file, identify the private key filename and assign permissions directly. The private keys are in the directory C:\Users\Default\AppData\Roaming\Microsoft\Crypto\CaviumKSP on Windows for Cloud HSM keys.

### What does azcloudhsm_ksp_import_key.exe do?

azcloudhsm_ksp_import_key.exe allows customers to import or represent an asymmetric key/keys in Cloud HSM KSP.

Supported algorithms are RSA (2048 to 4096 in multiples of 256), ECDSA, and ECDH. For ECDSA and ECDH supported curves are P256, P384 and P521. After a key is imported or represented in Cloud HSM KSP, the key should only be managed from Cloud HSM KSP.

### What is the recommended way of connecting to Cloud HSM from the corporate network (operation/admin tasks)?

Customers can connect through their virtual network.

- azcloudhsm_mgmt_util executes operations (admin tasks on the HSM) which requires PCO (partition crypto officer) credentials.
- azcloudhsm_client (client daemon) connects via PCU (partition crypto user).
  - The azcloudhsm_client and customers application need to both run on the same VM within that virtual network. This is because the application connects to the client process via RPC.
  - The Azure Cloud HSM service listens on port 443 (azcloudhsm_client requests), 444 (azcloudhsm_mgmt_util requests) and 445 (server-server communication).
    - Front end ports are 2224, 2225
      - TCP over TLS protocol and Ports 2224 and 2225
      - Ports 2224 and 2225 are for customers only.
