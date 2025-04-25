---
title: Server-side programming in Azure confidential ledger (preview)
description: Run custom logic in Azure confidential ledger.
author: andpiccione
ms.author: apiccione
ms.service: azure-confidential-ledger
ms.topic: conceptual
ms.date: 04/18/2025

---

# Server-side programming in Azure confidential ledger (preview)

Azure confidential ledger provides a server-side programming model that allows you to run custom logic inside the ledger. This feature enables you to extend the functionality of the ledger by writing custom code that can be executed inside a confidential environment and interact with the ledger's data.

The new server-side programming capabilities allow users to create _user defined functions (UDFs)_ and _user defined endpoints (UDEs)_ to perform various operations with a different level of customization. Once defined, UDFs and UDEs can be invoked from the Azure confidential ledger clients to execute directly inside the ledger engine.  

> [!IMPORTANT]
> Server-side programming is currently in PREVIEW.
> You can request access for this preview via [this sign-up form](https://aka.ms/ACL2025Preview).
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

## User defined functions and user defined endpoints

Azure confidential ledger offers two types of server-side programming features:

- **User defined functions (UDFs)**: UDFs are simple custom functions that can be defined and executed inside the ledger. They allow you to perform simple operations using the JavaScript API and can be integrated with the existing ledger write API. Learn more about UDFs in the [user defined functions](./user-defined-functions.md) article. 

- **User defined endpoints (UDEs)**: UDEs are custom endpoints that can be defined and exposed on a ledger application. These endpoints can be used to expose more sophisticated APIs with customizable parameters, data formats, and authentication mechanisms. Learn more about UDEs in the [user defined endpoints](./user-defined-endpoints.md) article.

Here's a comparison of the main features and capabilities of UDFs and UDEs:  

| Capability                                     | User defined functions (UDFs)                                                                 | User defined endpoints (UDEs)                                                              |
|--------------------------------------|---------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------|
| Run custom business logic            | Yes                                                                                   | Yes                                                                                     |
| Customization of runtime arguments / parameters  | Yes                                                                                   | Yes                                                                                     |
| Expose new API / endpoints           | No                                                                                   | Yes                                                                                     |
| Run individual functions             | Yes                                                                                   | No                                                                                     |
| Run hooks before / after a ledger entry write operation  | Yes                                                                                    | No                                    |
| Creation and update of custom code   | Yes (granular update/delete of each function)                                         | Yes (single application bundle update for all endpoints)                                                                         |
| Custom JavaScript runtime options    | Yes (per each execution)                                                              | Yes (globally)                                                                                     |
| Execution on backup nodes        | No                                                                                    | Yes                                                                                    |
| Support multiple data formats for input and output  | No                                                                                   | Yes                                                                                     |
| Custom authentication and authorization  | No                                                                                   | Yes                                                                                     |
| **Target scenarios**                     | Execution of simple custom logic and integration with the ledger API                  | Advanced use cases requiring custom APIs and control over advanced JavaScript parameters    |


## Switching between UDFs and UDEs

UDFs and UDEs are mutually exclusive features. You can't create or run UDFs if UDEs are defined, and vice versa. It's possible to switch between the two features using the following steps.

> [!IMPORTANT]
> Switching between UDFs and UDEs is a destructive operation. All existing UDFs or UDEs are deleted when switching to the other feature. Make sure to back up any important data before proceeding.

### From UDFs to UDEs

To switch from UDFs to UDEs, you need to delete all existing UDFs:

1. [List all UDFs](./user-defined-functions.md#list-udfs) stored in the ledger using the `GET /app/userDefinedFunctions` API.

2. [Delete each UDF](./user-defined-functions.md#delete-a-udf), one by one, using the `DELETE /app/userDefinedFunctions/{functionId}` API.

### From UDEs to UDFs

To switch from UDEs to UDFs, you need to define user defined endpoints with empty modules and endpoints definitions.

1. Create a new application bundle with the following structure:
   
    ```json
    {
        "metadata": {
            "endpoints": {}
        }, 
       "modules": []
    }
    ```

2. [Deploy the application bundle](./user-defined-endpoints.md#deploy-the-application) with the request body defined in step 1 using the `PUT /app/userDefinedEndpoints` API.

## Next steps

Refer to the following articles to get started with server-side programming features in Azure confidential ledger:

- [User defined functions in Azure confidential ledger](./user-defined-functions.md)
- [User defined endpoints in Azure confidential ledger](./user-defined-endpoints.md)

## Related content

- [Overview of Microsoft Azure confidential ledger](overview.md)
