---
title: PKCS#11 API for Certificate Storage in Azure Cloud HSM
description: Learn how to use PKCS#11 APIs to manage X.509 certificates in Azure Cloud HSM, including creating, copying, deleting, and retrieving certificate attributes.
author: keithp
manager: keithp
ms.service: azure-cloud-hsm
ms.topic: tutorial
ms.date: 03/20/2025
ms.author: keithp
ms.custom: pkcs11, certificate-management, x509-certificates, azure-cloud-hsm

#Customer Intent: As an IT pro, I want to set up and use certificate storage for Azure Cloud HSM to help ensure security and compliance.

---

# PKCS#11 API for certificate storage

Azure Cloud HSM provides robust support for certificate storage using the PKCS#11 API. This tutorial explains how to use the PKCS#11 API to manage X.509 certificates, including creating, copying, deleting, and retrieving certificate attributes. For a detailed overview of certificate storage setup, including prerequisites and configuration, refer to the [Azure Cloud HSM Certificate Storage Tutorial](tutorial-certificate-storage.md).

## Using PKCS#11 API for X.509 certificate storage

The following existing APIs in PKCS#11 for Azure Cloud HSM have been expanded to add support for X.509 Public Key Certificates.

- C_CreateObject: Creates a new certificate object.
- C_DestroyObject: Deletes an existing certificate object.
- C_CopyObject: Copies an existing certificate object.
- C_GetAttributeValue: Gets the value of one or more attributes of a certificate object.
- C_SetAttributeValue: Updates the value of one or more attributes of a certificate object.
- C_FindObjectsInit: Starts a search for certificate objects.
- C_FindObjects: Continues a search for certificate objects.
- C_FindObjectsFinal: Ends a search for certificate objects.

### C_CreateObject

The C_CreateObject API functions similarly for both keys and certificates. It expects an array of attributes, the number of attributes, and a pointer to an object handle where the generated handle will be stored.

Below is a sample of how to use the C_CreateObject.

```c
int create_cert(CK_SESSION_HANDLE session_rw, CK_OBJECT_HANDLE_PTR cert_handle)
{
    // Dummy certificate data
    CK_BYTE certData[] = { 0x30, 0x82, 0x03, 0x08, 0x30, 0x82, 0x02, 0xD0 }; // Sample DER-encoded cert
    CK_ULONG certSize = sizeof(certData);

    CK_OBJECT_CLASS objClass = CKO_CERTIFICATE;
    CK_CERTIFICATE_TYPE certType = CKC_X_509;
    CK_BBOOL trueValue = CK_TRUE;
    CK_BYTE id[] = {123};

    // Dummy DER-encoded Subject Name (adjust as needed)
    CK_BYTE subjectData[] = { 0x30, 0x1D, 0x31, 0x1B, 0x30, 0x19, 0x06, 0x03,
                              0x55, 0x04, 0x03, 0x0C, 0x12, 'M', 'y', 'C', 'e',
                              'r', 't', 'i', 'f', 'i', 'c', 'a', 't', 'e', '-', 'B', 'b', 'j' };
    CK_ULONG subjectSize = sizeof(subjectData);

    CK_ATTRIBUTE certTemplate[] = {
        { CKA_CLASS, &objClass, sizeof(objClass) },
        { CKA_CERTIFICATE_TYPE, &certType, sizeof(certType) },
        { CKA_TOKEN, &trueValue, sizeof(trueValue) },
        { CKA_LABEL, "MyCertificate", 13 },
        { CKA_SUBJECT, subjectData, subjectSize },
        { CKA_ID, id, sizeof(id) },
        { CKA_VALUE, certData, certSize }
    };

    int n_attr = sizeof(certTemplate) / sizeof(CK_ATTRIBUTE);

    if ((func_list->C_CreateObject)(session_rw, certTemplate,
                                    n_attr, cert_handle)) {
        return FAILED;
    }
#ifdef DEBUG
    printf("The cert handle created is : %lu \n", *cert_handle);
#endif

    return CKR_OK;
}
```

The following attributes represent the minimum required set to create an X.509 certificate in PKCS#11.

| Layer                     | Attribute                  | Data Type              | Description                                                                 |
|---------------------------|----------------------------|------------------------|-----------------------------------------------------------------------------|
| Common Attributes         | CKA_CLASS                 | CK_OBJECT_CLASS        | Object class (type)                                                        |
| Certificate Objects       | CKA_CERTIFICATE_TYPE      | CK_CERTIFICATE_TYPE    | Type of certificate, CKC_X_509 for X.509 public key certificates           |
| X.509 Public Key Certificate Objects | CKA_SUBJECT | Byte array             | DER-encoding of the certificate subject name                               |
| X.509 Public Key Certificate Objects | CKA_VALUE   | Byte array             | BER-encoding of the certificate                                            |

The following attributes are applicable to X.509 public key certificates.

| Layer                     | Attribute                  | Data Type              | Description                                                                 |
|---------------------------|----------------------------|------------------------|-----------------------------------------------------------------------------|
| Common Attributes         | CKA_CLASS                 | CK_OBJECT_CLASS        | Object class (type)                                                        |
| Storage Objects           | CKA_TOKEN                 | CK_BBOOL               | CK_TRUE if object is a token object; CK_FALSE if object is a session object. Default is CK_FALSE. |
| Storage Objects           | CKA_PRIVATE               | CK_BBOOL               | CK_TRUE if object is a private object; CK_FALSE if object is a public object. Default value is token-specific and may depend on the values of other attributes of the object. |
| Storage Objects           | CKA_MODIFIABLE            | CK_BBOOL               | CK_TRUE if object can be modified, Default is CK_TRUE.                     |
| Storage Objects           | CKA_LABEL                 | RFC2279 string         | Description of the object (default empty).                                 |
| Storage Objects           | CKA_COPYABLE              | CK_BBOOL               | CK_TRUE if object can be copied using C_CopyObject. Defaults to CK_TRUE. Can’t be set to TRUE once it is set to FALSE. |
| Storage Objects           | CKA_DESTROYABLE           | CK_BBOOL               | CK_TRUE if the object can be destroyed using C_DestroyObject. Default is CK_TRUE. |
| Certificate Objects       | CKA_CERTIFICATE_TYPE      | CK_CERTIFICATE_TYPE    | Type of certificate, CKC_X_509 for X.509 public key certificates           |
| Certificate Objects       | CKA_TRUSTED               | CK_BBOOL               | The certificate can be trusted for the application that it was created.    |
| Certificate Objects       | CKA_CERTIFICATE_CATEGORY  | CKA_CERTIFICATE_CATEGORY | (default CK_CERTIFICATE_CATEGORY_UNSPECIFIED)                              |
| Certificate Objects       | CKA_CHECK_VALUE           | Byte array             | Checksum                                                                   |
| Certificate Objects       | CKA_START_DATE            | CK_DATE                | Start date for the certificate (default empty)                             |
| Certificate Objects       | CKA_END_DATE              | CK_DATE                | End date for the certificate (default empty)                               |
| Certificate Objects       | CKA_PUBLIC_KEY_INFO       | Byte Array             | DER-encoding of the SubjectPublicKeyInfo for the public key contained in this certificate (default empty) |
| X.509 Public Key Certificate Objects | CKA_SUBJECT | Byte array             | DER-encoding of the certificate subject name                               |
| X.509 Public Key Certificate Objects | CKA_ID      | Byte array             | Key identifier for public/private key pair (default empty)                 |
| X.509 Public Key Certificate Objects | CKA_ISSUER  | Byte array             | DER-encoding of the certificate issuer name (default empty)                |
| X.509 Public Key Certificate Objects | CKA_SERIAL_NUMBER | Byte array           | DER-encoding of the certificate serial number (default empty)              |
| X.509 Public Key Certificate Objects | CKA_VALUE   | Byte array             | BER-encoding of the certificate                                            |
| X.509 Public Key Certificate Objects | CKA_URL     | RFC2279 string         | If not empty this attribute gives the URL where the complete certificate can be obtained (default empty) |
| X.509 Public Key Certificate Objects | CKA_HASH_OF_SUBJECT_PUBLIC_KEY | Byte array | Hash of the subject public key (default empty). Hash algorithm is defined by CKA_NAME_HASH_ALGORITHM |
| X.509 Public Key Certificate Objects | CKA_HASH_OF_ISSUER_PUBLIC_KEY | Byte array | Hash of the issuer public key (default empty). Hash algorithm is defined by CKA_NAME_HASH_ALGORITHM |
| X.509 Public Key Certificate Objects | CKA_JAVA_MIDP_SECURITY_DOMAIN | CK_JAVA_MIDP_SECURITY_DOMAIN | Java MIDP security domain. (default CK_SECURITY_DOMAIN_UNSPECIFIED)         |
| X.509 Public Key Certificate Objects | CKA_NAME_HASH_ALGORITHM | CK_MECHANISM_TYPE | Defines the mechanism used to calculate CKA_HASH_OF_SUBJECT_PUBLIC_KEY and CKA_HASH_OF_ISSUER_PUBLIC_KEY. If the attribute is not present, then the type defaults to SHA-1. |

### C_DestroyObject

The C_DestroyObject API takes a session handle, and the object handle associated with the certificate you want to delete. Invoking this function removes the specified certificate from the Azure Blob Storage Account by deleting the corresponding JWS blob named pkcs11_certificate_<cert_handle>.

Below is a code snippet demonstrating how to call C_DestroyObject for certificates (the same approach applies to keys).

```c
int delete_cert(CK_SESSION_HANDLE session_rw, CK_OBJECT_HANDLE cert_handle)
{
    CK_RV rv = 0;

    rv = (func_list->C_DestroyObject)(session_rw, cert_handle);

    if(rv != CKR_OK) {
        printf("Deleting Certificate failed \n");
        return rv;
    }

    return rv;
}
```

### C_CopyObject

The C_CopyObject API takes a session handle, the handle of the object to be copied, and a pointer to receive the handle of the newly created object. To maintain parity with the C_CopyObject implementation for key objects in Azure Cloud HSM, the certificate implementation does not support modifying attributes during the copy operation.

Below is a sample snippet demonstrating how to use C_CopyObject for storing certificates.

```c
int copy_cert(CK_SESSION_HANDLE session_rw, CK_OBJECT_HANDLE cert_handle,
                   CK_OBJECT_HANDLE_PTR copied_cert_handle)
{
    CK_RV rv = 0;

    rv = (func_list->C_CopyObject)(session_rw, cert_handle, NULL, 0, copied_cert_handle);

    if(rv != CKR_OK) {
        printf("Copying Certificate failed \n");
        return rv;
    }

    return rv;
}
```

### C_GetAttributeValue

The C_GetAttributeValue API allows retrieval of all attributes listed in the C_CreateObject API section. This API is typically invoked twice. The first call determines the size of attributes with unknown lengths such as CKA_SUBJECT, which contains the DER-encoded certificate subject.

Below is an example of how to call C_GetAttributeValue to obtain the sizes of the specified attributes.

```c
int get_cert_attribute(CK_SESSION_HANDLE session_rw, CK_OBJECT_HANDLE_PTR cert_handle)
{
    CK_RV rv = 0;

    CK_ULONG cka_class = 0;
    CK_CERTIFICATE_TYPE cka_cert_type = 0;
    CK_BBOOL cka_token = 0;
    char* cka_label = NULL;
    char* cka_subject = NULL;
    CK_BYTE* cka_id = NULL;
    CK_BYTE* cka_value = NULL;

    // Determine size needed for each attribute by calling C_GetAttributeValue with NULL pointers
    // and zero as the length.

    CK_ATTRIBUTE cert_template[] = {
        { CKA_CLASS, NULL, 0 },
        { CKA_CERTIFICATE_TYPE, NULL, 0 },
        { CKA_TOKEN, NULL, 0 },
        { CKA_LABEL, NULL, 0 },
        { CKA_ID, NULL, 0 },
    };

    int n_attr = sizeof(cert_template) / sizeof(CK_ATTRIBUTE);

    rv = (func_list->C_GetAttributeValue)(session_rw, *cert_handle, cert_template, n_attr);

    if (rv != CKR_OK) {
        printf("C_GetAttributeValue failed with %ld\n", rv);
        return FAILED;
    }

Once the attribute sizes are known, memory can be allocated accordingly. A second call to the C_GetAttributeValue API is then made to retrieve the attribute values and store them in the allocated memory.

The image below shows a code snippet demonstrating this process based on the previous example:

    cka_label = (char*)malloc(cert_template[3].ulValueLen);
    if (cka_label == NULL) {
        printf("Memory allocation failed for CKA_LABEL.\n");
        rv = FAILED;
        goto end_test_get_cert_attribute;
    }

    cert_template[3].pValue = cka_label;

    if (cert_template[4].ulValueLen <= 0) {
        printf("CKA_ID size must be > 0.\n");
        rv = FAILED;
        goto end_test_get_cert_attribute;
    }

    cka_id = (CK_BYTE*)malloc(cert_template[4].ulValueLen);
    if (cka_id == NULL) {
        printf("Memory allocation failed for CKA_ID.\n");
        rv = FAILED;
        goto end_test_get_cert_attribute;
    }

    cert_template[4].pValue = cka_id;

    rv = (func_list->C_GetAttributeValue)(session_rw, *cert_handle, cert_template, n_attr);
    if (rv != CKR_OK) {
        printf("C_GetAttributeValue failed with %ld\n", rv);
        rv = FAILED;
        goto end_test_get_cert_attribute;
    }
```

### C_SetAttributeValue

The C_SetAttributeValue API now supports updating certificate objects. It requires the session handle, the handle of the certificate to be updated, an array of attributes and their new values, and the number of attributes to update. Only attributes listed in the C_CreateObject API Usage table are supported for updates—attempting to modify unsupported attributes will result in a failed API call.

Below is a snippet showing how C_SetAttributeValue can be used with certificate objects.

```c
int set_cert_attribute(CK_SESSION_HANDLE session_rw, CK_OBJECT_HANDLE_PTR cert_handle)
{
    CK_RV rv = CKR_OK;
    CK_BBOOL falseValue = CK_FALSE;
    CK_BYTE subjectData[] = { 0x40, 0x41, 0x42, 0x43, 0x44 };
    CK_BYTE id[] = {254};
    CK_BYTE certData[] = { 0x10, 0x20, 0x30, 0x40, 0x50, 0xAA, 0xBB, 0xCC, 0xDD, 0xEE, 0xFF };

    CK_ATTRIBUTE certTemplateValid1[] = {
        { CKA_TOKEN, &falseValue, sizeof(falseValue) },
        { CKA_LABEL, "This is a new label", strlen("This is a new label") },
        { CKA_SUBJECT, subjectData, sizeof(subjectData) },
        { CKA_ID, id, sizeof(id) },
        { CKA_VALUE, certData, sizeof(certData) }
    };

    int n_attr = sizeof(certTemplateValid1) / sizeof(CK_ATTRIBUTE);
    rv = (func_list->C_SetAttributeValue)(session_rw, *cert_handle, certTemplateValid1, n_attr);
    if (rv != CKR_OK) {
        printf("test_set_cert_attribute failed when updating attribute values.\n");
        return FAILED;
    }

    return rv;
}
```

### C_FindObjectsInit

The C_FindObjects* API now supports locating certificate objects in addition to key objects. A search operation can return both key and certificate handles if the search template includes attributes common to both object types. The C_FindObjectsInit API has been enhanced to support all certificate-related attributes listed in the C_CreateObject API Usage table.

Below is an example of a C_FindObjectsInit call that performs a certificate search using the CKA_CLASS, CKA_CERTIFICATE_TYPE, and CKA_LABEL attributes to find all matching certificate objects.

```c
int find_cert(CK_SESSION_HANDLE session_rw, CK_OBJECT_HANDLE cert_handle)
{
    CK_RV rv;
    CK_OBJECT_CLASS objClass = CKO_CERTIFICATE;
    CK_CERTIFICATE_TYPE certType = CKC_X_509;

    CK_ATTRIBUTE certTemplate[] = {
        { CKA_CLASS, &objClass, sizeof(objClass) },
        { CKA_CERTIFICATE_TYPE, &certType, sizeof(certType) },
        { CKA_LABEL, "MyCertificate", 13 }
    };

    // Step 1: Initialize the search
    rv = (func_list->C_FindObjectsInit)(session_rw, certTemplate, sizeof(certTemplate) / sizeof(CK_ATTRIBUTE));
    if (rv != CKR_OK) {
        printf("C_FindObjectsInit failed: 0x%lX\n", rv);
        return rv;
    }
```

### C_FindObjects

After initializing the search parameters, the C_FindObjects API is used to retrieve the matching object handles. It also returns the number of objects found. This API takes the session handle, an array to store the resulting object handles, the maximum number of objects to retrieve, and an output parameter indicating how many objects were found.

The snippet below shows a call to C_FindObjects following the search template setup in the C_FindObjectsInit example above.

```c
    // Step 2: Call C_FindObjects
    CK_OBJECT_HANDLE_PTR foundObjects = NULL;
    CK_ULONG maxObjects = 50;

    foundObjects = (CK_OBJECT_HANDLE_PTR)malloc(sizeof(CK_OBJECT_HANDLE) * maxObjects);
    if (!foundObjects) {
        printf("Memory allocation failed\n");
        return CKR_HOST_MEMORY;
    }
    CK_ULONG foundCount = 0;

    rv = (func_list->C_FindObjects)(session_rw, foundObjects, maxObjects, &foundCount);
    if (rv != CKR_OK) {
        printf("C_FindObjects failed: 0x%lX\n", rv);
        (func_list->C_FindObjectsFinal)(session_rw); // Ensure cleanup
        free(foundObjects);
        return rv;
    }
```

### C_FindObjectsFinal

The C_FindObjectsFinal API behaves the same for both key and certificate objects. It takes the current session handle as an argument and performs cleanup of all search-related structures and memory allocated during the C_FindObjectsInit call.

Below is a snippet showing how to call C_FindObjectsFinal to complete and clean up the search process initiated by the C_FindObjectsInit and C_FindObjects APIs.

```c
    // Step 3: Finalize the search
    rv = (func_list->C_FindObjectsFinal)(session_rw);
    if (rv != CKR_OK) {
        printf("C_FindObjectsFinal failed: 0x%lX\n", rv);
        free(foundObjects);
        return rv;
    }
}
```

## Configure and run your PKCS#11 application with Azure Cloud HSM

Azure Cloud HSM includes sample application code to help validate certificate storage, available in the Azure Cloud HSM Certificate Storage Integration Guide within the Azure Cloud HSM SDK on GitHub.

## Certificate structure in storage

### Verify certificates in storage

After a successful call to the C_CreateObject() API, the newly created certificate object will appear in your Azure Blob Storage account, as specified in the azcloudhsm_application.cfg file. The blob will be named using the format pkcs11_certificate_\<ObjectHandle\>, as shown below. Certificate objects are assigned object handles ranging from 0xFFF00000 to 0xFFFFFFFF (decimal range: 4,293,918,720 to 4,294,967,295), allowing support for up to 1,048,575 certificates.

From both Azure portal as well as from your Azure VM you can see the certificates stored.

### Verify from Azure portal

:::image type="content" source="./media/pkcs-verify-portal.png" alt-text="Screenshot showing certificate blobs stored in Azure portal for Azure Cloud HSM." lightbox="./media/pkcs-verify-portal.png":::

### Verify from Azure VM with AZ CLI installed

```bash
chsmVMAdmin@AdminVM:~$ az login --identity
[
  {
    "environmentName": "AzureCloud",
    "homeTenantId": "",
    "id": "",
    "isDefault": true,
    "managedByTenants": [],
    "name": "Test Subscription",
    "state": "Enabled",
    "tenantId": "",
    "user": {
      "assignedIdentityInfo": "MSI",
      "name": "systemAssignedIdentity",
      "type": "servicePrincipal"
    }
  }
]

chsmVMAdmin@AdminVM:~$ az storage blob list \
  --account-name chsmstorage \
  --container-name certificates \
  --auth-mode login \
  --output table

Name                                  Blob Type    Blob Tier    Length    Content Type              Last Modified
-----------------------------------  -----------  -----------  --------  ------------------------  -------------------------
pkcs11_certificate_4293918720        BlockBlob    Hot          1305      application/octet-stream  2025-05-16T22:43:31+00:00
pkcs11_certificate_4293918721        BlockBlob    Hot          1305      application/octet-stream  2025-05-16T22:47:25+00:00
pkcs11_certificate_4293918722        BlockBlob    Hot          1305      application/octet-stream  2025-05-16T22:47:25+00:00
pkcs11_certificate_4293918723        BlockBlob    Hot          3452      application/octet-stream  2025-05-16T22:56:28+00:00
```

Downloading the blob or viewing it in the Azure portal and inspecting its contents will reveal that the certificate is stored as a JWS (JSON Web Signature) token. The token follows the standard JWS structure, which is divided into the following format:

```bash
chsmVMAdmin@AdminVM:~$ az storage blob list \
  --account-name chsmstorage \
  --container-name certificates \
  --auth-mode login \
  --output table

Name                                  Blob Type    Blob Tier    Length    Content Type              Last Modified
-----------------------------------  -----------  -----------  --------  ------------------------  -------------------------
pkcs11_certificate_4293918720        BlockBlob    Hot          1305      application/octet-stream  2025-05-16T22:43:31+00:00
pkcs11_certificate_4293918721        BlockBlob    Hot          1305      application/octet-stream  2025-05-16T22:47:25+00:00
pkcs11_certificate_4293918722        BlockBlob    Hot          1305      application/octet-stream  2025-05-16T22:47:25+00:00
pkcs11_certificate_4293918723        BlockBlob    Hot          3452      application/octet-stream  2025-05-16T22:56:28+00:00

chsmVMAdmin@AdminVM:~$ az storage blob download \
  --account-name chsmstorage \
  --container-name certificates \
  --name pkcs11_certificate_4293918723 \
  --file pkcs11_certificate_4293918723.crt \
  --auth-mode login
Finished[########################################] 100.0000%
{
  "container": "certificates",
  "content": ""
}

chsmVMAdmin@AdminVM:~$ cat pkcs11_certificate_4293918723.crt
eyJhbgGciOiJSUzUxMiIsImp... (base64-encoded certificate continues)
```

## Next steps

- [Azure Cloud HSM Certificate Storage Tutorial](tutorial-certificate-storage.md): Learn how to set up certificate storage prerequisites and configure Azure Blob Storage for PKCS#11 applications.
- [Overview of Azure Cloud HSM](overview.md)
- [Secure your Azure Cloud HSM](secure-cloud-hsm.md)