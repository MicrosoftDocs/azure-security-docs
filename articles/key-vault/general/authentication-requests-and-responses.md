---
title: Authentication, requests, and responses
description: Learn how Azure Key Vault uses JSON-formatted requests and responses and about required authentication for using a key vault.
services: key-vault
author: msmbaldwin
manager: msmbaldwin

ms.service: azure-key-vault
ms.subservice: general
ms.topic: conceptual
ms.date: 04/16/2025
ms.author: mbaldwin

---

# Authentication, requests, and responses

Azure Key Vault provides two types of containers to store and manage secrets for your cloud applications:

|Container type|Supported object types|Data-plane endpoint|
|--|--|--|
| **Vaults**|<ul><li>Software-protected keys</li><li>HSM-protected keys (with Premium SKU)</li><li>Certificates</li><li>Storage account keys</li></ul> | https://{vault-name}.vault.azure.net
|**Managed HSM** |<ul><li>HSM-protected keys</li></ul> | https://{hsm-name}.managedhsm.azure.net

Here are the suffixes of the URLs used to access each type of object

|Object type|URL suffix|
|--|--|
|Software-protected keys| /keys |
|HSM-protected keys| /keys |
|Secrets|/secrets|
|Certificates| /certificates|
|Storage account keys|/storageaccounts

Azure Key Vault supports JSON formatted requests and responses. Requests to the Azure Key Vault are directed to a valid Azure Key Vault URL using HTTPS with some URL parameters and JSON encoded request and response bodies.

This article covers specifics for the Azure Key Vault service. For general information on using Azure REST interfaces, including authentication/authorization and how to acquire an access token, see [Azure REST API Reference](/rest/api/azure).

## Request URL structure

Key management operations use HTTP verbs including DELETE, GET, PATCH, and PUT. Cryptographic operations on existing key objects use HTTP POST.

For clients that cannot support specific HTTP verbs, Azure Key Vault allows using HTTP POST with the `X-HTTP-REQUEST` header to specify the intended verb. When using POST as a substitute (for example, instead of DELETE), include an empty body for requests that normally don't require one.

 To work with objects in the Azure Key Vault, the following are example URLs:  

- To CREATE a key called TESTKEY in a Key Vault use - `PUT /keys/TESTKEY?api-version=<api_version> HTTP/1.1`  

- To IMPORT a key called IMPORTEDKEY into a Key Vault use - `POST /keys/IMPORTEDKEY/import?api-version=<api_version> HTTP/1.1`  

- To GET a secret called MYSECRET in a Key Vault use - `GET /secrets/MYSECRET?api-version=<api_version> HTTP/1.1`  

- To SIGN a digest using a key called TESTKEY in a Key Vault use - `POST /keys/TESTKEY/sign?api-version=<api_version> HTTP/1.1`  

- The authority for a request to a Key Vault is always as follows,
  - For vaults: `https://{keyvault-name}.vault.azure.net/`
  - For Managed HSMs: `https://{HSM-name}.managedhsm.azure.net/`
  Keys are always stored under the /keys path, while Secrets are always stored under the /secrets path.  

## Supported API versions

The Azure Key Vault Service supports protocol versioning to provide compatibility with down-level clients, although not all capabilities are available to those clients. Clients must use the `api-version` query string parameter to specify the version of the protocol that they support as there is no default.  

Azure Key Vault protocol versions follow a date numbering scheme using a {YYYY}.{MM}.{DD} format.  

## Request body requirements

As per the HTTP specification, GET operations must NOT have a request body, and POST and PUT operations must have a request body. The body in DELETE operations is optional in HTTP.  

Unless otherwise noted in operation description, the request body content type must be application/json and must contain a serialized JSON object conformant to content type.  

Unless otherwise noted in operation description, the Accept request header must contain the application/json media type.  

## Response body format

Unless otherwise noted in operation description, the response body content type of both successful and failed operations are application/json and contains detailed error information.  

## Using HTTP POST as an alternative

Some clients may not be able to use certain HTTP verbs, such as PATCH or DELETE. Azure Key Vault supports HTTP POST as an alternative for these clients if the client also includes the “X-HTTP-METHOD” header to specific the original HTTP verb. Support for HTTP POST is noted for each of the API defined in this document.  

## Handling error responses

Error handling uses HTTP status codes. Typical results are:  

- 2xx – Success: Used for normal operation. The response body contains the expected result  
- 3xx – Redirection: The 304 "Not Modified" may be returned to fulfill a conditional GET. Other 3xx codes may be used in the future to indicate DNS and path changes.  
- 4xx – Client Error: Used for bad requests, missing keys, syntax errors, invalid parameters, authentication errors, etc. The response body contains detailed error explanation.  
- 5xx – Server Error: Used for internal server errors. The response body contains summarized error information.  

  The system is designed to work behind a proxy or firewall. Therefore, a client might receive other error codes.  

  Azure Key Vault also returns error information in the response body when a problem occurs. The response body is JSON formatted and takes the form:  

```

{  
  "error":  
  {  
    "code": "BadArgument",  
    "message":  

      "’Foo’ is not a valid argument for ‘type’."  
    }  
  }  
}  

```  

## Authentication requirements

All requests to Azure Key Vault MUST be authenticated. Azure Key Vault supports Microsoft Entra access tokens that may be obtained using OAuth2 [[RFC6749](https://tools.ietf.org/html/rfc6749)]. 

For more information on registering your application and authenticating to use Azure Key Vault, see [Register your client application with Microsoft Entra ID](/rest/api/azure/index#register-your-client-application-with-azure-ad).

Access tokens must be sent to the service using the HTTP Authorization header:  

```
PUT /keys/MYKEY?api-version=<api_version>  HTTP/1.1  
Authorization: Bearer <access_token>  

```  

When an access token is not supplied, or when the service does not accept a token, an HTTP 401 error is returned to the client and includes the WWW-Authenticate header, for example:  

```
401 Not Authorized  
WWW-Authenticate: Bearer authorization="…", resource="…"  

```  

The parameters on the WWW-Authenticate header are:  

- authorization: The address of the OAuth2 authorization service that may be used to obtain an access token for the request.  

- resource: The name of the resource (`https://vault.azure.net`) to use in the authorization request.

> [!NOTE]
> Key Vault SDK clients for secrets, certificates, and keys in the first call to Key Vault do not provide an access token to retrieve tenant information. It’s expected to receive an HTTP 401 using Key Vault SDK client where the Key Vault shows to the application the WWW-Authenticate header containing the resource and the tenant where it needs to go and ask for the token. If everything is configured correctly, the second call from the application to Key Vault will contain a valid token and will succeed.
