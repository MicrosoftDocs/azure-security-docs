---
title: Troubleshoot Azure Cloud HSM
description: Learn how to troubleshoot common Azure Cloud HSM issues and errors.
author: keithp
manager: davinune
ms.service: azure-dedicated-hsm
ms.topic: overview
ms.date: 03/20/2025
ms.author: keithp
---

# Troubleshooting Azure Cloud HSM

## Common error messages

### Why am I getting the error message "Certificate ERROR: [Certificate signature failure]" when running azcloudhsm_mgmt_util?

This error commonly occurs when a customer created and initialized their Azure Cloud HSM from another virtual machine (VM) and attempts to install the Cloud HSM Client SDK and run from another VM that is missing or does not have the correct PO.crt from the Admin VM you initialized from. If you copy the PO.crt from your Admin VM to your new VM and rerun the azcloudhsm_mgmt_util, you should see a successful connection to your HSM.

### Why am I getting the error message "INF: shutdown_ssl_socket: SSL_shutdown sent close_notify alert" when running azcloudhsm_client?

This error commonly occurs when a customer created and initialized their Azure Cloud HSM from another VM and attempts to install the Cloud HSM Client SDK and run from another VM that is missing or does not have the correct PO.crt from the Admin VM you initialized from. If you copy the PO.crt from your Admin VM to your new VM and rerun the azcloudhsm_client, you should see a successful connection to your HSM.

### Why am I getting the error message "azcloudhsm_application.cfg is not present"?

To ensure the successful execution of your application, two conditions must be met. Firstly, the azcloudhsm_application.cfg file must be present in the same directory as your application. Secondly, the azcloudhsm_client should be running. If the azcloudhsm_client is not active, you encounter a failed socket connection. Likewise, the absence of the azcloudhsm_application.cfg file results in an error message. Depending on the execution location of your application, you may need to copy the azcloudhsm_application.cfg file from the Azure Cloud HSM SDK directory to the directory where you are running your application.

## Azure Cloud HSM OpenSSL Engine

### Does Azure Cloud HSM OpenSSL engine support Windows?

No. Azure Cloud HSM OpenSSL engine supports Linux (Ubuntu 20.04, Ubuntu 22.04, RHEL 7, RHEL 8, CBL Mariner 2) only.

### Why am I getting the error message "invalid engine 'azcloudhsm_openssl' could not load the shared library"?

The error message suggests that the prerequisite environment variable `LD_LIBRARY_PATH` isn't configured, and this configuration is necessary. Set the `LD_LIBRARY_PATH` environment variable to the directory where the Azure Cloud HSM SDK is located.

You must update your environment variables to reflect the correct SDK version running:

```bash
export LD_LIBRARY_PATH=/opt/azurecloudhsm/lib64/:$LD_LIBRARY_PATH
```

### Why am I getting the error message "environment variable azcloudhsm_openssl_conf is not set"?

The error message signals that the prerequisite environment variable `azcloudhsm_openssl_conf` is not configured. Set the `azcloudhsm_openssl_conf` environment variable to the file path location of your `azcloudhsm_openssl_dynamic.conf` file within the Azure Cloud HSM SDK directory.

You must update your environment variables to reflect the correct SDK version running:

```bash
export azcloudhsm_openssl_conf=/opt/azurecloudhsm/bin/azcloudhsm_openssl_dynamic.conf  
```

### Why am I getting the error message "environment variable azcloudhsm_password is not set"?

The provided error message suggests that the prerequisite environment variable `azcloudhsm_password` is not configured. Set the `azcloudhsm_password` environment variable to the value of your `cryptouser:password`.

You must update your environment variables to reflect the correct crypto user and password:

```bash
export azcloudhsm_password="cu1:user1234"
```

## Azure Cloud HSM JCE Provider

### Does Azure Cloud HSM JCE support Windows?

No. Azure Cloud HSM JCE supports Linux (Ubuntu 20.04, Ubuntu 22.04, RHEL 7, RHEL 8, CBL Mariner 2) only.

### Why am I getting the error message "could not find credentials to login to the HSM"?

The error indicates a challenge in locating or authenticating the credentials required for logging into Cloud HSM. When utilizing the samples in this documentation or incorporating the Azure Cloud HSM JCE provider into your applications, it is essential to explicitly log in with credentials:

```bash
// Explicit Login with Credentials
LoginManager lm=LoginManager.getInstance();
lm.login("PARTITION_1","cu1","user1234");

// Logout of the HSM
lm.logout();
```

### Why am I getting the error message "attribute with value 0 for this operation? Attribute should set for User KEK"?

The error indicates that a user-generated KEK was not created and designated as extractable and wrap_with_trusted set to 1. See "Creating a user-generated KEK" in the JCE integration guide for appropriate settings when performing Key Import / Keywrap / Keyunwrap operations.

### Why am I getting the error message "Daemon socket connection error, API login failed"?

The error indicates a challenge in locating or authenticating the credentials required for logging into Cloud HSM. When utilizing the samples in the JCE integration guide or incorporating the Azure Cloud HSM JCE provider into your applications, it is essential to explicitly sign in with credentials to ensure connection.

## Azure Cloud HSM PKCS#11 Library

### Why am I getting the error message "C_Login failed with error code: 0xa3"?

Encountering (0xa3) indicates a sign in failure that the issue might be related to the format of the PIN parameter you're passing or incorrect PIN (password).

The PIN should be provided in the following format: `char pPin[256] = "crypto_user:user123";` The pPin value should follow the structure of "`<username>`:`<password>`". Verify that you are providing the pPin in this format, as any deviation might result in a sign-in failure.

### Why am I getting the error message "C_Initialize() failed with 00000005 (Failed to connect socket)"?

Encountering a failed socket connection during C_Initialize might be related to the azcloudhsm_client utility not running on your system. For your PKCS#11 applications to run successfully, the azcloudhsm_client must be running. You can start the azcloudhsm_client by running the following command in a separate terminal.

#### Linux (Recommended)

If you installed the Azure Cloud HSM SDK using deb or rpm, the client isn't configured automatically to run as a service. The SDK during installation includes a service unit file under /etc/systemd/system/azure-cloud-hsm.service. To enable azcloudhsm_client to run as a service, use the predefined azure-cloud-hsm.service file. Reload the Systemd configuration and enable the service to ensure it's continuously running by performing the following steps.

1. Open a terminal window and change the directory to /etc/systemd/system where the Cloud HSM service unit file is located.
  
  ```bash
  cd /etc/systemd/system 
  ```
  
1. Start and enable the Cloud HSM client service.

  ```bash
  sudo systemctl start azure-cloud-hsm.service 
  sudo systemctl enable azure-cloud-hsm.service 
  ```
  
1. Check the status of the Cloud HSM client service.

  ```bash
  sudo systemctl status azure-cloud-hsm.service 
  ```
  
1. Reload the Systemd configuration.

  ```bash
  sudo systemctl daemon-reload 
  sudo systemctl list-units --type=service --state=running 
  ```
  
#### Linux (Manual)

Start the client daemon if it isn't running.

```bash
sudo ./azcloudhsm_client azcloudhsm_client.cfg
```

You can also run the client daemon in the background using the following command or run the client daemon as a service to ensure it is always running.

```bash
sudo ./azcloudhsm_client azcloudhsm_client.cfg > /dev/null 2>&1 &
```

#### Windows (Recommended)

If you installed the Azure Cloud HSM SDK using MSI, the client is already configured to run as a service. If the client isn't running, you can open Services.msc, right-click on the **Microsoft Azure Cloud HSM Client Service**, and select **Start**.

#### Windows (Manual)

Start the client daemon if it isn't running.

```bash
cd C:\Program Files\Microsoft Azure Cloud HSM Client SDK 
.\azcloudhsm_client.exe .\azcloudhsm_resource.cfg 
```

### How does the PKCS#11 library know how to find the client configuration for Linux?

The PKCS#11 library knows how to find the client configuration because you must have a copy of your partition owner certificate "PO.crt" on the application server that is running your application and using the PKCS#11 library. In addition to the PO certificate, you have to update /azcloudhsm_client/azcloudhsm_client.cfg on your application server that has the SDK installed to point to your Azure Cloud HSM (that is, hsm1.chsm-`<resourcename>`-`<uniquestring>`.privatelink.cloudhsm.azure.net). The azcloudhsm_client utility must be running on your application server which connects to your Azure Cloud HSM. Finally, you must specify a PIN within your PKCS#11 application using the syntax `<username>`:`<password>` which is used when calling C_Login to your Azure Cloud HSM. You must #include pkcs11_headers/include/cryptoki.h and pkcs11_headers/include/pkcs11t.h in your PKCS#11 application to use the Azure Cloud HSM PKCS#11 library.

### How does the PKCS#11 library (azcloudhsm_pkcs11.dll) for Windows know how to find the client configuration?

The azcloudhsm_pkcs11.dll under Azure Cloud HSM Windows SDK knows how to find the client configuration because you must have a copy of your partition owner certificate "PO.crt" on the application server that is running your application and using the PKCS#11 library. In addition to the PO certificate, you have to update /azcloudhsm_client/azcloudhsm_client.cfg on your application server that has the SDK installed to point to your Azure Cloud HSM (that is, hsm1.chsm-`<resourcename>`-`<uniquestring>`.privatelink.cloudhsm.azure.net). The azcloudhsm_client must run on your application server which connects to your Azure Cloud HSM. Finally, you must specify a PIN within your PKCS#11 application using the syntax `<username>`:`<password>` which is used when calling C_Login to your Azure Cloud HSM. You will also need to #include pkcs11_headers\include\cryptoki.h and pkcs11_headers\include\pkcs11t.h in your PKCS#11 application to use the Azure Cloud HSM PKCS#11 library.

### Why do I get PKCS#11 errors when trying to set key or keystore?

Ensure that you have the Azure Cloud HSM client running [sudo ./azcloudhsm_client azcloudhsm_client.cfg]. An incorrect user, password, or liquid security client not running or client failing to connect might result in the following error.

```bash
Error Message:ERROR at line 1:
ORA-28407: Hardware Security Module failed with PKCS#11 error
CKR_GENERAL_ERROR(5)
```

### How does the PKCS#11 library (azcloudhsm_pkcs11.dll) integrate with OpenSC (Open-Source Smart Card Tools)?

Customers must use the "--sensitive" and "--private" arguments when using OpenSC tools because the sensitive and private attributes can't be 0. Without using those arguments, the sensitive and private attributes are set to 0 by OpenSC tooling and generate an exception, as Azure Cloud HSM always imposes sensitive and private behavior.

- AES Wrapping Example:
```bash
pkcs11-tool.exe --sensitive --private --module "C:\AzureCloudHSM-ClientSDK\AzureCloudHSM-ClientSDK-Windows-1.0.3.0\libs\pkcs11\azcloudhsm_pkcs11.dll" --login --login-type user --pin cu1:user1234 --keygen --key-type AES:32*
```
- ECC P521 Example:
```bash
pkcs11-tool.exe --sensitive --private --module "C:\AzureCloudHSM-ClientSDK\AzureCloudHSM-ClientSDK-Windows-1.0.3.0\libs\pkcs11\azcloudhsm_pkcs11.dll" --login --login-type user --pin cu1:user1234 --keypairgen --key-type EC:prime256v1 --usage-sign
```

## Azure Cloud HSM SSL/TLS Offloading

### Does Azure Cloud HSM OpenSSL engine support PKCS#11 for SSL/TLS offloading?

No. OpenSSL supporting PKCS#11 for SSL/TLS offloading isn't supported by Azure Cloud HSM. Customers that require PKCS#11 for SSL/TLS offloading must use the [TLS Offload Library for Azure Managed HSM](https://github.com/microsoft/AzureManagedHsmTLSOffload).
### Why am I getting the error message "Failed to connect socket, LIQUIDSECURITY: Daemon socket connection error" when running azcloudhsm_util?

The error message suggests that azcloudhsm_client isn't running. Ensure that azcloudhsm_client is running for the Azure Cloud HSM utilities to execute properly. You can start azcloudhsm_client by ensuring it's running as a service or by manually running the following command in a separate terminal.

#### Linux (Recommended)

If you installed the Azure Cloud HSM SDK using deb or rpm, the client isn't configured automatically to run as a service. The SDK during installation includes a service unit file under /etc/systemd/system/azure-cloud-hsm.service. To enable azcloudhsm_client to run as a service, use the predefined azure-cloud-hsm.service file. Reload the systemd configuration and enable the service to ensure it's continuously running by performing the following steps.

1. Open a terminal window and change directory to /etc/systemd/system where the Cloud HSM service unit file is located.

  ```bash
  cd /etc/systemd/system 
  ```

1. Start and enable the Cloud HSM Client service.

  ```bash
  sudo systemctl start azure-cloud-hsm.service 
  sudo systemctl enable azure-cloud-hsm.service 
  ```

1. Check the status of the Cloud HSM Client service.

  ```bash
  sudo systemctl status azure-cloud-hsm.service 
  ```

1. Reload the Systemd configuration.

  ```bash
  sudo systemctl daemon-reload 
  sudo systemctl list-units --type=service --state=running 
  ```

#### Linux (manual)

Start the client daemon if it isn't running.

```bash
sudo ./azcloudhsm_client azcloudhsm_client.cfg
```

You may also choose as an option to run the client daemon in the background using the following command or run the client daemon as a service to ensure it is always running.

```bash
sudo ./azcloudhsm_client azcloudhsm_client.cfg > /dev/null 2>&1 &
```

### Why am I getting the error message "environment variable azcloudhsm_password is not set"?

The provided error message suggests that the prerequisite environment variable azcloudhsm_password is not configured. To resolve this issue, set the azcloudhsm_password environment variable to the value of your cryptouser:password.

Update your environment variables to reflect the correct crypto user and password:

```bash
export azcloudhsm_password="cu1:user1234"
```

### Why am I getting the error message "Key/Token not found" or "Invalid key-handle/token" when attempting to import key?

The provided error message suggests that the specified wrapping key handle (the KEK handle ID) is incorrect. The parameters `-f` specify the filename containing the key to import, while `-w` specifies the wrapping key handle. During the creation of a user-generated KEK, the `-w` value should be the key handle ID of your user-generated KEK ID.

```bash
azcloudhsm_util singlecmd loginHSM -u CU -p user1234 -s cu1 importPrivateKey -l importTestKey -f key.pem -w 262150
```

### Why am I getting the error message "/configure: error: C compiler cc not found" when trying to configure the Nginx sources?

The error message suggests that you're missing a C compiler from your system when running the configure command. Depending on your installation, after running the following commands you might also need to install 'openssl libssl-dev' and 'zlib1g zlib1g-dev' which are dependencies.

#### Red Hat based (using yum)

```bash
sudo yum update
sudo yum groupinstall "Development Tools"
```

#### For Red Hat based systems (using dnf)

```bash
sudo dnf update
sudo dnf install "Development Tools”
```

#### For Ubuntu based systems (using apt)

```bash
sudo apt update  
sudo apt install build-essential 
```

### Why am I getting the error message "/configure: error: SSL modules require the OpenSSL library"?

The error message indicates that the OpenSSL library is required to enable SSL modules for Nginx. To resolve this issue, run the following command and then run configure again.

```bash
sudo apt-get install libssl-dev
```

### Why am I getting the error message "/configure: error: the HTTP gzip module requires the zlib library"?

The error message indicates that the zlib library is required for the HTTP gzip module in Nginx. To resolve this issue, run the following command and then run configure again.

```bash
sudo apt-get install zlib1g-dev
```

### Why am I getting the error message "HSM Error: RET_USER_LOGIN_FAILURE"?

The error message indicates that the specified user doesn't exist on one or more nodes where you're attempting a cryptographic operation. Although Azure Cloud HSM operates with a cluster of three nodes, the operation was directed to a node where the login failed. In such cases, a critical_err_info_ file is generated within the AzureCloudHSM-ClientSDK directory that shows "Status: HSM Error: This user doesn't exist".

To mitigate this error, you need to create a 'user' with the Crypto User role that is replicated across all three nodes of your Azure Cloud HSM. This example shows starting azcloudhsm_mgmt_util and administrator logging on as CO, then proceeds to create 'cu1' user with crypto user role. 

```bash
sudo ./azcloudhsm_mgmt_util ./azcloudhsm_mgmt_util.cfg
loginHSM CO admin adminpassword 
createUser CU cu1 user1234
logoutHSM
loginHSM CU cu1 user1234
```

### How do I display the contents of the CRT, CSR, and key files I created?

How to display the contents of the CRT, CSR, or key file you created in your terminal:

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

RSA Key:

```bash
./azcloudhsm_util singlecmd loginHSM -u CU -p user1234 -s cu1 genRSAKeyPair -m 2048 -e 65537 -l labelRSATest 
./azcloudhsm_util singlecmd loginHSM -u CU -p user1234 -s cu1 getCaviumPrivKey -k {PRIVATE_KEY_HANDLE} -out web_server_fake_PEM.key 
openssl req -new -key web_server_fake_PEM.key -out web_server.csr -engine azcloudhsm_openssl 
openssl x509 -req -days 365 -in web_server.csr -signkey web_server_fake_PEM.key -out web_server.crt -engine azcloudhsm_openssl 
```

EC Key:

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

This error commonly occurs when the "Azure Cloud HSM MSI" is run after SQL Server is installed. The error goes away when you restart SQL Server, when it picks up the environment variables and settings created when the Azure Cloud HSM MSI was executed.

## Azure Cloud HSM ADCS Integration

### Why do I get the error message "Can't open openssl-ca.cnf file" when trying to migrate a certificate from Microsoft KSP from the provided example?

This error commonly occurs when the openssl-ca.cnf does not exist. You can create one yourself or obtain it from another source.

### Can we use our own signing servers in our corporate network that consumes Azure Cloud HSM as a SaaS solution?

Azure Cloud HSM is IaaS only. You can use your own signing servers in your corporate network, however. You can configure Site-to-Site or Point-to-Site VPN connection from your local network gateway. This is the most common method, and it's suitable for most use cases. Set up a VPN connection between your corporate network's on-premises and Azure to where Azure Cloud HSM is deployed.

- [Tutorial - Connect an on-premises network and a virtual network: S2S VPN: Azure portal](/azure/vpn-gateway/tutorial-site-to-site-portal)
- You can have ADCS for your signing servers on-premises. What is required is SDK must be on your on-premises host and the client be reachable to your Azure VNET Private IPs.
  - The customer needs to run the azcloudhsm_client on the same machine where Azure Cloud HSM CNG provider exists.
  - The azcloudhsm_client should be able to reach out to the Customers Azure vNET private IPs.

When testing ADCS, we generated a RSAKeyPair then used CFM to sign.

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

Supported algorithms are RSA (2048 to 4096 in multiples of 256), ECDSA, and ECDH. For ECDSA and ECDH supported curves are P256, P384 and P521. Once a key is imported or represented in Cloud HSM KSP, the key should only be managed from Cloud HSM KSP.

### What is the recommended way of connecting to Cloud HSM from the corporate network (operation/admin tasks)?

Customers can connect through their VNET.

- azcloudhsm_mgmt_util executes operations (admin tasks on the HSM) which requires PCO (partition crypto officer) credentials.
- azcloudhsm_client (client daemon) connects via PCU (partition crypto user).
  - The azcloudhsm_client and customers application need to both run on the same VM within that VNET. This is because the application connects to the client process via RPC.
  - The Azure Cloud HSM service listens on port 443 (azcloudhsm_client requests), 444 (azcloudhsm_mgmt_util requests) and 445 (server-server communication).
    - Front end ports are 2224, 2225
      - TCP over TLS protocol and Ports 2224 and 2225
      - Ports 2224 and 2225 are for customers only.
