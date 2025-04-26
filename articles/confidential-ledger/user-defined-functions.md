---
title: Simple user defined functions in Azure confidential ledger
description: Learn about user defined functions (UDFs) in Azure confidential ledger, a simple way to run custom code in a confidential environment.
author: andpiccione
ms.author: apiccione
ms.service: azure-confidential-ledger
ms.topic: conceptual
ms.date: 04/10/2025

---

# Simple user defined functions in Azure confidential ledger (preview)

Simple user defined functions (UDFs) in Azure confidential ledger allow you to create custom JavaScript functions that can be executed inside the ledger trust boundary. This feature is designed to be simple and easy to use, allowing you to extend the functionality of the ledger API without the need for complex application development.

Using the [built-in JavaScript API](https://microsoft.github.io/CCF/main/build_apps/js_app_bundle.html#javascript-api), you can run custom code to achieve various tasks, such as custom queries and computations, conditional checks, post-processing tasks, and more. This feature is suitable for scenarios where you need a direct integration with the existing ledger API or run lightweight custom logic in a confidential environment.

> [!VIDEO cc413f21-53da-4e2a-9a3e-7e2dc8fc948c]

> [!IMPORTANT]
> User defined functions are currently in PREVIEW under API version `2024-12-09-preview`.
> You can request access for this preview via [this sign-up form](https://aka.ms/ACL2025Preview).
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

> [!TIP]
> For more advanced scenarios, such as custom Role-Based Access Control (RBAC) or integration with external confidential workloads, see [advanced user defined functions in Azure confidential ledger](./user-defined-endpoints.md).

## Use cases

Azure confidential ledger UDFs allow you to extend the functionality of the ledger by running custom logic. Some common use cases for UDFs include:

* **Custom computations and queries**: run standalone UDFs to read or write data into any ledger application table according to your business logic.

* **Data validation and input checks**: use UDFs as _pre-hooks_ to run pre-processing actions before a ledger entry is written to the ledger, for example to sanitize input data or check for pre-conditions.

* **Data enrichment and smart contracts**: use UDFs as _post-hooks_ to run post-processing actions after a ledger entry is written, for example to add custom metadata into the ledger or trigger post-write workflows.

## Writing UDFs 

An Azure confidential ledger UDF is an entity stored in the ledger with a unique ID and contains the JavaScript code that is executed when the UDF is called. This section describes how to write UDF code and use the JavaScript API for achieving different tasks.

### Function structure

The code of a UDF requires an exported function that is the entry point of the script at the time of execution. A basic UDF code template looks like this:

```javascript
export function main() {
    // Your JavaScript code here
}
```

> [!NOTE]
> The name of the exported entrypoint function that is called during execution can be modified with the `exportedFunctionName` argument when running the UDF. If not specified, the default name is `main`.

> [!NOTE]
> Lambda functions are supported, but they require the exported function name to be explicitly defined and match the entrypoint function name. For example:
> ```javascript
> export const main = () => { 
>     // Your JavaScript code here 
> };
> ```

### Function arguments

You can specify any optional runtime arguments accepted by the UDF. The arguments' values can be passed at runtime when running the UDF using the `arguments` parameter.

The arguments are always passed as an array of strings. It's the user responsibility to ensure that the arguments specified in the UDF code match the arguments passed when running the UDF. The user should also ensure that the arguments are properly parsed to the expected data type at runtime.

```javascript
export function main(arg1, arg2) {
    // Your JavaScript code here
}
```

### JavaScript API

The JavaScript code of a UDF is executed inside a sandbox environment that provides a [limited set of APIs](https://microsoft.github.io/CCF/main/build_apps/js_app_bundle.html#javascript-api).

All the [JavaScript standard global functions, objects, and values](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects) can be used. A global object called `ccf` can be used to access specific functionalities and utilities provided by the [Confidential Consortium Framework (CCF)](https://microsoft.github.io/CCF/main/) (for example, cryptography helper functions, ledger tables accessors, etc.). The full API of the `ccf` global object is documented [here](https://microsoft.github.io/CCF/main/js/ccf-app/interfaces/global.CCF.html).

You can also access contextual information of the current request by using the `context` global object. This object provides access to the request metadata that originated the function execution (`context.request`) and the user ID of the function caller (`context.userId`). For transaction hooks, the collection ID and the transaction contents associated to the write operation are also added to the `context` object (`context.collectionId` and `context.contents` respectively). 
 
The following snippet shows some basic examples of the use of the JavaScript API:

```javascript
export function main(args) {
    
    // Basic instructions
    const a = 1 + 1;

    // Basic statements
    if (a > 0) {
        console.log("a is positive");
    } else {
        console.log("a is negative or zero");
    }

    // Parse the string argument as a JSON object
    JSON.parse(args);

    // Logging utilities
    console.log("Hello world");
    
    // Math utilities
    Math.random();
    
    // CCF cryptography utilities
    ccf.crypto.digest("SHA-256", ccf.strToBuf("Hello world"));
    
    // Write to a custom ledger table
    ccf.kv["public:mytable"].set(ccf.strToBuf("myKey"), ccf.strToBuf("myValue"));

    // Read from a custom ledger table
    ccf.bufToStr(ccf.kv["public:mytable"].get(ccf.strToBuf("myKey")));

    // Read from the ledger entry table
    ccf.kv["public:confidentialledger.logs"].get(ccf.strToBuf("subledger:0"));

    // Get the request metadata that originated the function execution
    const requestMetadata = context.request;
    
    // Get the collection ID and transaction content (for transaction hooks only)
    const collectionId = context.collectionId;
    const contents = context.contents;

    // Throw exceptions
    throw new Error("MyCustomError");
}
```

> [!TIP]
> For more information on how ledger maps can be used to store and retrieve data, see [the CCF documentation on the Key-Value Store API](https://microsoft.github.io/CCF/main/build_apps/kv/kv_how_to.html).

> [!NOTE]
> Importing modules isn't supported in UDFs. The JavaScript code must be self-contained and can't rely on external libraries or modules. [Web APIs](https://developer.mozilla.org/en-US/docs/Web/API) also aren't supported at the moment.

## Managing UDFs

Azure confidential ledger applications provide a dedicated CRUD API to create, read, update, and delete UDF entities. UDFs are securely stored in the ledger and are accessible only to the ledger application. 

> [!NOTE]
> Simple user defined functions and [advanced user defined functions](./user-defined-endpoints.md) are mutually exclusive features. You can't create or run simple UDFs if advanced UDFs are defined, and vice versa. To switch between the two, follow the instructions in the [UDF overview page](./server-side-programming.md#switching-between-simple-and-advanced-udfs).

### Create or update a UDF

```http
PUT /app/userDefinedFunctions/myFunction
{
    "code": "export function main() { return "Hello World"; }",
}
```

> [!IMPORTANT]
> Administrator role is required to create or update a UDF.

### Get a UDF

```http
GET /app/userDefinedFunctions/myFunction
```

### List UDFs

```http
GET /app/userDefinedFunctions
```

### Delete a UDF

```http
DELETE /app/userDefinedFunctions/myFunction
```

> [!IMPORTANT]
> Administrator role is required to delete a UDF.

> [!NOTE]
> Deleting a UDF only removes the entity from the current state of the ledger. Any deleted UDF is always retained in the immutable ledger history (as any committed transaction). 

## Running UDFs 

Once created, Azure confidential ledger users can execute a UDF either as a standalone function or as a transaction hook associated to a write operation. Each UDF execution runs on a separate runtime environment and sandbox, meaning that the UDF execution is isolated from other UDFs or other ledger operations.

The UDF execution can be controlled using optional properties that can be specified in the request body. The properties currently supported are:

* `arguments`: an array of strings that represent the arguments to be passed to the UDF. The arguments are passed in the same order as they're defined in the UDF code. The default value is an empty array.

* `exportedFunctionName`: the name of the exported function to be called during execution. If not specified, the default value is `main`.

* `runtimeOptions`: an object that specifies the runtime options for the UDF execution. The following options are available:

    * `max_heap_bytes`: the maximum heap size in bytes. The default value is 10,485,760 (10 MB).

    * `max_stack_bytes`: the maximum stack size in bytes. The default value is 1,048,576 (1 MB).

    * `max_execution_time_ms`: the maximum execution time in milliseconds. The default value is 1000 (1 second).

    * `log_exception_details`: a boolean value that specifies whether to log exception details. The default value is `true`.

    * `return_exception_details`: a boolean value that specifies whether to return exception details in the response. The default value is `true`. 

### Standalone functions

A UDF can be directly executed using the `POST /app/userDefinedFunctions/{functionId}:execute` API. 

```http
POST /app/userDefinedFunctions/myFunction:execute
{}
```

The request body can be used to specify optional execution parameters, such as function arguments and JavaScript runtime properties.

```http
POST /app/userDefinedFunctions/myFunction:execute
{
    "arguments": ["arg1", "arg2"],
    "exportedFunctionName": "myMainFunction",
    "runtimeOptions": {
        "max_heap_bytes": 5,
        "max_stack_bytes": 1024,
        "max_execution_time_ms": 5000,
        "log_exception_details": true,
        "return_exception_details": true
    }
}
```

The response indicates the outcome of the UDF execution (succeeded or failed). If the UDF succeeded, the response includes the function returned value in string format (if any). 

```json
{
    "result": 
        {
            "returnValue": "MyReturnValue"
        }, 
    "status": "Succeeded"
}
```

If the UDF failed, the response includes the error message with the detailed stack trace.

```json
{
    "error": {
        "message": "Error while executing function myFunction: Error: MyCustomError\n    at myMainFunction (myFunction)\n"
    }, 
    "status": "Failed"
}
```


> [!IMPORTANT]
> Contributor role is required to execute a UDF.

### Transaction hooks

A UDF can alternatively be executed as a hook before (_pre-hook_) or after (_post-hook_) an entry is written to the ledger as part of the ledger write API (`POST /app/transactions`). Hooks run in the same context of the write operation; this means that any data written to the ledger by the hooks is automatically included in the same write transaction.

The request body of the write request can be used to specify any UDF IDs to be executed as pre-hooks and post-hooks respectively.

```http
POST /app/transactions?collectionId=myCollection
{
  "contents": "myValue", 
  "preHooks": [ 
    { 
        "id": "myPreHook"
    } 
  ], 
  "postHooks": [ 
    { 
        "id": "myPostHook" 
    }
  ] 
} 
```

> [!IMPORTANT]
> Hooks must be explicitly defined in the request body of the write operation. In general, UDFs can't run automatically for every write operation after being created.  

For each hook, it's possible to specify any optional execution properties. For example:

```http
POST /app/transactions?collectionId=myCollection
{
  "contents": "myValue", 
  "preHooks": [ 
    { 
        "id": "myPreHook", 
        "properties": { 
            "arguments": [ 
                "arg1",
                "arg2"
            ], 
            "exportedFunctionName": "myMainFunction", 
            "runtimeOptions": { 
                "max_heap_bytes": 5,
                "max_stack_bytes": 1024,
                "max_execution_time_ms": 5000,
                "log_exception_details": true,
                "return_exception_details": true
            } 
        } 
    } 
  ], 
  "postHooks": [ 
    { 
        "id": "myPostHook", 
        "properties": { 
            "arguments": [ 
                "arg1"
            ], 
            "exportedFunctionName": "myMainFunction", 
            "runtimeOptions": { 
                "max_heap_bytes": 5,
                "max_stack_bytes": 1024,
                "max_execution_time_ms": 5000,
                "log_exception_details": true,
                "return_exception_details": true
            } 
        } 
    }
  ] 
} 
```

You can specify up to 5 pre-hooks and post-hooks in the request body, with any combination. The hooks are always executed in the order they're provided in the request body.

If a pre-hook or post-hook fails, the entire transaction is aborted. In that case, the response contains the error message with the reason for the failure:

```json
{
    "error": {
        "code": "InternalError",
        "message": "Error while executing function myPreHook: Error: MyCustomError\n    at myMainFunction (myPreHook)\n",
    }
}
```

> [!NOTE]
> Even if multiple hooks succeed, the transaction can still fail if any of the defined pre-hooks or post-hooks doesn't run successfully to completion.

> [!TIP]
> A UDF can be reused as a pre-hook and a post-hook in the same request and called multiple times.

## Examples

This section walks through some practical examples of how to use UDFs in Azure confidential ledger. For the following example scenarios, we assume using Azure confidential ledger to store banking transactions for different bank users.

### Context

To store a banking transaction for a user, the existing ledger write API can be used: the value of the transaction is the content of the ledger entry and the user ID can be the collection, or key, where the content is written under.

```http
POST /app/transactions?collectionId=John
{
    "contents": "10"
}

HTTP/1.1 200 OK
```

Since there's no validation on the input contents, it's possible to write a non-numeric value as a contents. For example, this request succeeds even if the contents value isn't a number:

```http
POST /app/transactions?collectionId=Mark
{
    "contents": "This is not a number"
}

HTTP/1.1 200 OK
```

### Pre-hooks for data validation

To ensure the transaction contents is always a number, a UDF can be created to check the input contents. The following pre-hook check if the contents value is a number and throws an error if not.

```http
PUT /app/userDefinedFunctions/validateTransaction
{
    "code": "export function main() { if (isNaN(context.contents)) { throw new Error('Contents is not a number'); } }"
}

HTTP/1.1 201 CREATED
```

Using the pre-hook in the write request, it's possible to enforce that the input data corresponds to the expected format. The previous request now fails as expected:

```http
POST /app/transactions?collectionId=Mark
{
    "contents": "This is not a number",
    "preHooks": [
        {
            "functionId": "validateTransaction"
        }
    ]
}

HTTP/1.1 500 INTERNAL_SERVER_ERROR
{
  "error": {
    "code": "InternalError",
    "message": "Error while executing function validateTransaction: Error: Contents is not a number\n    at main (validateTransaction)\n"
  }
}
```

Valid requests containing numeric values would instead succeed as expected:

```http
POST /app/transactions?collectionId=Mark
{
    "contents": "30",
    "preHooks": [
        {
            "functionId": "validateTransaction"
        }
    ]
}

HTTP/1.1 200 OK
```

### Post-hooks for data enrichment

As users perform new banking transactions, we want to record when a transaction is higher than a certain threshold for audit reasons. A post-hook can be used to write custom metadata in a ledger after a write operation to indicate whether the transaction was higher than a certain threshold. 

For example, a UDF can be created to check the transaction value and write a dummy message ("Alert" for high values, "Normal" otherwise) under the input user into a custom ledger table (`payment_metadata`) if the value is higher than 50. 

```http
PUT /app/userDefinedFunctions/detectHighTransaction
{
    "code": "export function main() { let value = 'Normal'; if (context.contents > 50) { value = 'Alert' } ccf.kv['public:payment_metadata'].set(ccf.strToBuf(context.collectionId), ccf.strToBuf(value)); }"
}

HTTP/1.1 201 CREATED
```

Once the UDF is successfully created, the post-hook can be used in new write requests:

```http
POST /app/transactions?collectionId=Mark
{
    "contents": "100",
    "preHooks": [
        {
            "functionId": "validateTransaction"
        }
    ],
    "postHooks": [
        {
            "functionId": "detectHighTransaction"
        }
    ]
}

HTTP/1.1 200 OK
```

```http
POST /app/transactions?collectionId=John
{
    "contents": "20",
    "preHooks": [
        {
            "functionId": "validateTransaction"
        }
    ],
    "postHooks": [
        {
            "functionId": "detectHighTransaction"
        }
    ]
}

HTTP/1.1 200 OK
```

### Standalone UDFs for custom queries

To inspect the latest values written to the custom table `payment_metadata` using the post-hook, a UDF can be created to read the values from the table given an input user ID:

```http
PUT /app/userDefinedFunctions/checkPaymentMetadataTable
{
    "code": "export function main(user) { const value = ccf.kv['public:payment_metadata'].get(ccf.strToBuf(user)); if (value === undefined) { throw new Error('UnknownUser'); } return ccf.bufToStr(value); }"
}

HTTP/1.1 201 CREATED
```

By running the UDF directly, it's possible to check the latest value recorded in the custom metadata table for a given user.

For users with a recent high transaction, the UDF returns value "Alert" as expected.

```http
POST /app/userDefinedFunctions/checkPaymentMetadataTable:execute
{
    "arguments": [
        "Mark"
    ]
}

HTTP/1.1 200 OK
{
  "result": {
    "returnValue": "Alert"
  },
  "status": "Succeeded"
}
```

For users with a recent low transaction, the UDF returns value "Normal" instead.

```http
POST /app/userDefinedFunctions/checkPaymentMetadataTable:execute
{
    "arguments": [
        "John"
    ]
}

HTTP/1.1 200 OK
{
  "result": {
    "returnValue": "Normal"
  },
  "status": "Succeeded"
}
```

For users that don't have any entry in the custom table, the UDF throws an error as defined in the UDF code.

```http
POST /app/userDefinedFunctions/checkPaymentMetadataTable:execute
{
    "arguments": [
        "Jane"
    ]
}

HTTP/1.1 200 OK
{
  "error": {
    "message": "Error while executing function checkPaymentMetadataTable: Error: UnknownUser\n    at main (checkPaymentMetadataTable)\n"
  },
  "status": "Failed"
}
```

## Considerations

* Transaction hooks are currently only supported for the `POST /app/transactions` API, when adding a new entry to the ledger.

* UDFs and hooks are always executed on the primary replica of the ledger, to ensure transaction ordering and strong consistency.

* UDF code execution is always wrapped in a single atomic transaction. If the JavaScript logic in a UDF completes without any exceptions, all the operations within the UDF are committed to the ledger. If any exception is thrown, the entire transactions are rolled back. Similarly, pre-hooks and post-hooks are executed in the same context of the write operation they're registered to. If a pre-hook or a post-hook fails, the entire transaction is aborted and no entry is added to the ledger.

* UDFs can only access CCF application tables and can't access [the ledger's internal and governance tables](https://microsoft.github.io/CCF/main/audit/read_write_restrictions.html#table-namespaces) or other [built-in tables](https://microsoft.github.io/CCF/main/audit/builtin_maps.html) for security reasons. The ledger tables where entries are written to (`public:confidentialledger.logs` for public ledgers and `private:confidentialledger.logs` for private ledgers) are read-only.

* The maximum number of pre-hooks and post-hooks that can be registered for a single write transaction is 5.

* UDF and hook execution is limited to 5 seconds. If a function takes longer than 5 seconds to execute, the operation is aborted and an error is returned.

## Related content

- [User defined functions overview in Azure confidential ledger](server-side-programming.md)
- [Advanced user defined functions in Azure confidential ledger](./user-defined-endpoints.md)
- [Overview of Microsoft Azure confidential ledger](overview.md)
