---
title: Tutorial - Azure Cloud HSM Certificate Storage
description: Step-by-step guide to configure certificate storage for Azure Cloud HSM using PKCS#11 APIs, including prerequisites, Azure Blob Storage setup, and Managed Identity configuration.
author: keithp
manager: keithp
ms.service: azure-cloud-hsm
ms.topic: tutorial
ms.date: 03/20/2025
ms.author: keithp
ms.custom: certificate-storage, pkcs11, azure-blob-storage, managed-identity

#Customer Intent: As an IT pro, I want to set up and use certificate storage for Azure Cloud HSM to help ensure security and compliance.

---

# Azure Cloud HSM Certificate Storage

Azure Cloud HSM supports certificate storage via PKCS#11, enabling applications to manage X.509 certificates alongside keys. This tutorial provides step-by-step instructions for setting up certificate storage prerequisites, including Azure Blob Storage and Managed Identity configuration. For details on using PKCS#11 APIs to manage certificates, refer to the [PKCS#11 API for Certificate Storage](pkcs-api-certificate-storage.md).

## Prerequisite

The following prerequisites are required to support certificate storage with Azure Cloud HSM. Reference [the Azure Cloud HSM Onboarding Guide for SDK Installation and configuration](https://github.com/microsoft/MicrosoftAzureCloudHSM/blob/main/OnboardingGuides/Azure%20Cloud%20HSM%20Onboarding.pdf) if HSM deployment is not complete.

### System Requirements

- Azure Cloud HSM resource is deployed, initialized, and configured.
- Azure Cloud HSM Client SDK
- Copy of partition owner certificate "PO.crt" on application server.
- Known address of your HSM "hsm1.chsm-\<resourcename\>-\<uniquestring\>.privatelink.cloudhsm.azure.net".
- Knowledge of Crypto User credentials

### Certificate Storage Prerequisites

- Azure Blob Storage Account
- Managed Identity to access storage

> [!IMPORTANT]
> Customers using any version of Windows Server should install the most recent version of Visual C++ Redistributable.

## Setting up an Azure Blob Storage Account

Before you can use the PKCS#11 API for Certificate Storage, you must create an Azure Blob Storage Account. This storage account will hold the PKCS#11 certificate objects, which are saved and retrieved in JWS format.

1. To set up an Azure Blob Storage Account for PKCS#11 certificate storage, go to the Azure portal and create a new **Storage Account**.
2. After successfully creating the Storage Account, navigate to it in the Azure portal and select **Containers** under **Data storage**. Here, you create a new container to store the blobs.

   :::image type="content" source="./media/certificate-storage-add-container.png" lightbox="./media/certificate-storage-add-container.png" alt-text="Screenshot of creating a container in Azure Blob Storage.":::

3. After creating the container, locate the container endpoint URL by navigating to **Container properties**. This URL is needed later.

   :::image type="content" source="./media/certificate-storage-container-properties.png" lightbox="./media/certificate-storage-container-properties.png" alt-text="Screenshot of locating container properties in Azure Blob Storage.":::

4. In **Container properties**, you find the container URL listed. This URL is required later in the azcloudhsm_application.cfg file to enable PKCS#11 applications to locate the storage location for certificate objects.

   :::image type="content" source="./media/certificate-storage-container-url.png" lightbox="./media/certificate-storage-container-url.png" alt-text="Screenshot of container URL in Azure Blob Storage properties.":::

## Setting up User Assigned Managed Identity to access storage

The next prerequisite for certificate storage is to create a **User Assigned Managed Identity**. This identity is granted the necessary role to access the Azure Blob Storage Account and is used to authenticate from your designated Admin VM.

> [!NOTE]
> The following example creates and uses a User Assigned Managed Identity. A System Assigned Managed Identity can also be created and used on the VM.

1. To create a **User Assigned Managed Identity** for PKCS#11 certificate storage, navigate to the Azure portal and create a new identity.
2. After successfully creating the Managed Identity, make note of the **Client ID**, which is required later in the azcloudhsm_application.cfg file to enable authentication to the storage account from your VM.

   :::image type="content" source="./media/certificate-storage-client-id.png" lightbox="./media/certificate-storage-client-id.png" alt-text="Screenshot of Managed Identity Client ID in Azure portal.":::

3. The next step is to assign the appropriate Azure role to grant the Managed Identity permission to read and write to the previously created Blob Storage Account. Assign the **Storage Blob Data Contributor** role to the Managed Identity, setting the **Scope** to **Storage** and selecting the specific Storage Account resource.

   :::image type="content" source="./media/certificate-storage-role-assignement.png" lightbox="./media/certificate-storage-role-assignement.png" alt-text="Screenshot of assigning Storage Blob Data Contributor role in Azure portal.":::

   :::image type="content" source="./media/certificate-storage-role-assignement-2.png" lightbox="./media/certificate-storage-role-assignement-2.png" alt-text="Screenshot of setting scope for Storage Blob Data Contributor role.":::

4. The next step is to assign the **User Assigned Managed Identity** to the VM that will run your PKCS#11 certificate storage application. Navigate to your VM resource in the Azure portal, go to the **Security** section, select **Identity**, and add the User Assigned Identity.

   :::image type="content" source="./media/certificate-storage-add-user-assigned-identity.png" lightbox="./media/certificate-storage-add-user-assigned-identity.png" alt-text="Screenshot of adding User Assigned Managed Identity to VM in Azure portal.":::

   :::image type="content" source="./media/certificate-storage-user-assigned-managed-identity.png" lightbox="./media/certificate-storage-user-assigned-managed-identity.png" alt-text="Screenshot of VM Identity settings in Azure portal.":::

## Configure the Azure Cloud HSM Client Tools

### Create a Storage Signing Key

The following azcloudhsm_util command can be used to create an RSA signing key pair for PKCS#11 certificate storage in a single step. By default, it generates a 2048-bit RSA key with a public exponent of 65537. You may modify the key size as needed. Before running the command, ensure that the azcloudhsm_client is running as a service in the background.

Replace the placeholders as follows:

- PKCS11_S with your Crypto User username. (for example, cu1)
- PKCS11_P with your Crypto User password. (for example, user1234)
- SIGNING_KEY_ID with the desired key pair ID (this ID will also be used later in your azcloudhsm_application.cfg file)

**Signing Key ID** For this example, we are going to set Signing Key ID to a random value.

```bash
SIGNING_KEY_ID=$(tr -dc 'a-z' </dev/urandom | head -c 10)
```

**Linux:**

```bash
sudo ./azcloudhsm_util singlecmd loginHSM -u CU -s $PKCS11_S -p $PKCS11_P genRSAKeyPair -m 2048 -e 65537 -l $SIGNING_KEY_ID -id $SIGNING_KEY_ID
```

**Windows:**

```powershell
.\azcloudhsm_util.exe singlecmd loginHSM -u CU -s %PKCS11_S% -p %PKCS11_P% genRSAKeyPair -m 2048 -e 65537 -l %SIGNING_KEY_ID% -id %SIGNING_KEY_ID%
```

> [!IMPORTANT]
> Please ensure that each of the HSM partitions returns to success.

```bash
chsmVMAdmin@AdminVM:/opt/azurecloudhsm/bin$ sudo ./azcloudhsm_util singlecmd loginHSM -u CU -s cu1 -p user1234 genRSAKeyPair -m 2048 -e 65537 -l signkeyid -id signkeyid
Version info, Client Version: 2.09.07.02, SDK API Version: 2.09.07.02, SDK Package Version: 2.0.1.2

Cfm3Initialize() returned app id : 01000000

    session_handle 1000000

    Current FIPS mode is: 00000000

Cfm3LoginHSM returned: 0x00 : HSM Return: SUCCESS

Cluster Status:
Node id 1 status: 0x00000000 : HSM Return: SUCCESS
Node id 2 status: 0x00000000 : HSM Return: SUCCESS
Node id 3 status: 0x00000000 : HSM Return: SUCCESS
Command: genRSAKeyPair -m 2048 -e 65537 -L signkeyid -id signkeyid

Cfm3GenerateKeyPair returned: 0x00 : HSM Return: SUCCESS

Cfm3GenerateKeyPair:    public key handle: 262151    private key handle: 262152

Cluster Status:
Node id 1 status: 0x00000000 : HSM Return: SUCCESS
Node id 2 status: 0x00000000 : HSM Return: SUCCESS
Node id 3 status: 0x00000000 : HSM Return: SUCCESS
```

### Update Configuration Files

#### Update Application Config

You will need to update the following parameters in the azcloudhsm_application.cfg file.

**CERTSTORAGE_URL**: This field refers to the URL of the container within the customer's Blob Storage Account and is used to store certificate information. (e.g. <https://chsmstorage.blob.core.windows.net/certificates>)

**CERTSTORAGE_SIGNING_KEYID**: This field refers to the ID assigned to the key pair, which is used to perform integrity checks during read and write operations to storage (signing and verification).

**UAMI_CLIENT_ID**: This field refers to the Client ID of the User Assigned Managed Identity, which is used to authenticate to the customer's Blob Storage Account. If left blank, authentication will default to using a System Assigned Managed Identity.

> [!NOTE]
> These parameters only apply when running certificate operations in PKCS#11. It is not required for Key Operations.

```bash
DAEMON_ID=1  
SOCKET_TYPE=UNIXSOCKET  
PORT=1111  
USER_KEK_HANDLE=262150  
DEFAULT_WRAP_WITH_TRUSTED=1  
CERTSTORAGE_URL=https://chmsstorage.blob.core.windows.net/certificates  
CERTSTORAGE_SIGNING_KEYID=hjgrwvvofe  
UAMI_CLIENT_ID=25e659cc-6570-4f3b-9617-adfbc84b2565  
```

#### Validate PKCS#11 Configuration

Please refer to the PKCS#11 Integration Guide for sample-based validation of your PKCS#11 configuration.

```bash
chsmVMAdmin@AdminVM:/opt/azurecloudhsm$ sudo ./cust_p11_app -s cu1 -p user1234 -l /opt/azurecloudhsm/lib64/libazcloudhsm_pkcs11.so
[INFO] Azure Cloud HSM – Loading PKCS#11 library.
[INFO] Azure Cloud HSM – C_GetFunctionList
[INFO] Azure Cloud HSM – Preparing PIN with given username and password.
[INFO] Azure Cloud HSM – C_Initialize
[INFO] Azure Cloud HSM – C_GetInfo
[INFO] Azure Cloud HSM – Retrieve access token, C_GetTokenInfo
[INFO] Azure Cloud HSM – Start session with specified token, C_OpenSession
[INFO] Azure Cloud HSM – Login with PIN, C_Login
Add Your PKCS#11 Code Here
```

## Next Steps

- [PKCS#11 API for Certificate Storage](pkcs-api-certificate-storage.md): Learn how to use PKCS#11 APIs to manage X.509 certificates in Azure Cloud HSM.
- [Overview of Azure Cloud HSM](overview.md)
- [Secure your Azure Cloud HSM](secure-cloud-hsm.md)
