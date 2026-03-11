---
title: User defined functions in Azure confidential ledger (preview)
description: Run custom logic in Azure confidential ledger.
author: andpiccione
ms.author: apiccione
ms.service: azure-confidential-ledger
ms.topic: conceptual
ms.date: 04/18/2025

---

# User defined functions in Azure confidential ledger (preview)

Azure confidential ledger provides _user defined functions_ (UDFs) to run custom logic inside the ledger. This feature enables you to extend the functionality of the ledger by writing custom code that can be executed inside a confidential environment and interact with the ledger's data.

The new programming capabilities allow users to create _simple UDFs_ and _advanced UDFs_ to perform various operations with a different level of customization. Once defined, UDFs can be invoked from the Azure confidential ledger clients to execute directly inside the ledger engine.  

> [!IMPORTANT]
> User defined functions are currently in PREVIEW.
> You can request access for this preview via [this sign-up form](https://aka.ms/ACL2025Preview).
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

## Simple and advanced UDFs

Azure confidential ledger offers two types of UDF features:

- **Simple UDFs**: Simple UDFs are simple custom functions that can be defined and executed inside the ledger. They allow you to perform simple operations using the JavaScript API and can be integrated with the existing ledger write API. Learn more about simple UDFs [here](./user-defined-functions.md). 

- **Advanced UDFs**: Advanced UDFs are custom endpoints that can be defined and exposed on a ledger application. These endpoints can be used to expose more sophisticated APIs with customizable parameters, data formats, and authentication mechanisms. Learn more about advanced UDFs [here](./user-defined-endpoints.md).

Here's a comparison of the main features and capabilities of simple and advanced UDFs:  

| Capability                                     | Simple UDFs                                                                 | Advanced UDFs                                                               |
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


## Switching between simple and advanced UDFs

Simple and advanced UDFs are mutually exclusive features. You can't create or run simple UDFs if advanced UDFs are defined, and vice versa. It's possible to switch between the two features using the following steps.

> [!IMPORTANT]
> Switching between simple and advanced UDFs is a destructive operation. All existing simple or advanced UDFs are deleted when switching to the other feature. Make sure to back up any important data before proceeding.

### From simple to advanced UDFs

To switch from simple to advanced UDFs, you need to delete all existing UDFs:

1. [List all UDFs](./user-defined-functions.md#list-udfs) stored in the ledger using the `GET /app/userDefinedFunctions` API.

2. [Delete each UDF](./user-defined-functions.md#delete-a-udf), one by one, using the `DELETE /app/userDefinedFunctions/{functionId}` API.

### From advanced to simple UDFs

To switch from advanced to simple UDFs, you need to define advanced UDFs with empty modules and endpoints definitions.

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

Refer to the following articles to get started with UDFs features in Azure confidential ledger:

- [Simple user defined functions in Azure confidential ledger](./user-defined-functions.md)
- [Advanced user defined functions in Azure confidential ledger](./user-defined-endpoints.md)

## Related content

- [Overview of Microsoft Azure confidential ledger](overview.md)
