---
title: Data Organization in Azure Confidential Ledger
description: How to organize data in Azure Confidential Ledger.
author: musabbirkhan
ms.author: musabbirkhan
ms.service: azure-confidential-ledger
ms.topic: conceptual
ms.date: 06/15/2025
---
# Data Organization
Data written to Azure Ledger is organized in two primary ways:
- Transaction ID 
- Collection ID (or Subledger ID)

## Transaction ID
Every write that is written to the ledger generates a transaction ID, which is returned in the response from the write endpoint.
It looks like the following example:

```python
sample_entry = {"contents": "Hello world!"}
append_result = ledger_client.create_ledger_entry(entry=sample_entry)
print(f"Transaction ID: {append_result['transactionId']}")
```

Response:
```
Transaction ID: 2.10
```

The transaction ID in the response can be used to retrieve the entry or do other operations such retrieve the receipt, etc. It's also possible to get a list of entries by providing a range of Transaction ID parameters similar to the following example.
```python
# Get the latest transaction to the ledger default/global table
get_result = ledger_client.get_current_ledger_entry()
```
```python
# Get one transaction
get_result = ledger_client.get_ledger_entry(transaction_id=2.10)
```
```python
# Get a list of transactions within a range
list_result = ledger_client.list_ledger_entries(from_transaction_id=2.1, to_transaction_id=2.50)
```

## Collection ID
A Collection ID (also known as Subledger ID) in Azure Ledger refers to a user-specified optional key for entries written to the ledger. It provides a way to add a key or marker to the data being written which can then be used to retrieve the data later on. 

Collection ID are strings and have no size limit at this time. It means the maximum size is dependent on the http request size limit. 

They can be utilized as unique keys for every entry written to the ledger or to group a subsection of entries.  
If no collection ID is specified, the data is written to the default table (known as subledger-0).

In order to specify a collection ID, the user has to specify the collectionID parameter during write request.

```python
sample_entry = {"contents": "Secret recipe ingredients...!"}
append_result = ledger_client.create_ledger_entry(entry=sample_entry, collection_id="icecream-flavors")
```
The transaction can then be retrieved by specifying the `collection ID` and the `transaction ID` during a read operation or by specifying just the collection ID in `list` operations.

```python
# Get the latest transaction in a collection
get_result = ledger_client.get_ledger_entry(collection_id="icecream-flavors")
```
```python
# Get one specific in a collection
list_result = ledger_client.get_ledger_entry(transaction_id=2.68,collection_id="icecream-flavors")
```
```python
# Get all entries in a collection
list_result = ledger_client.list_ledger_entries(collection_id="icecream-flavors")
for entry in list_result:
    print(f"Transaction ID: {entry['transactionId']}")
    print(f"Contents: {entry['contents']}")
```

It's also possible to get a list of entries within a collection by specifying both the collection ID and a range of Transaction ID parameters.

```python
list_result = ledger_client.list_ledger_entries(from_transaction_id=2.1, to_transaction_id=2.50, collection_id="icecream-flavors")
```

### Tags (Preview)
Adding Tags to a collection is a preview feature available with the latest preview versions of the ledger. Upgrade to SDK versions based on REST API Reference `2024-12-09-preview` or newer for access.

They allow for improved management of data within a collection by acting as secondary keys to a collection of data. Tags are limited to 64 character strings. Each transaction can support upto five tags. If an error is seen related to the number of tags supported and a higher limit is required, reach out to Support.  

In order to use tags inside a collection, tags parameter needs to be specified. 

```python
# multiple tags can be specified using commas.
append_result = ledger_client.create_ledger_entry(entry=sample_entry, collection_id="icecream-flavors", tags="chocolate,vanilla")
```

Retrieving entries can be done as before:
```python
# Get all entries in a collection which has a specified tag. Only one tag is supported per query during a Read operation. 
list_result = ledger_client.list_ledger_entries(collection_id="icecream-flavors", tag="chocolate")
for entry in list_result:
    print(f"Transaction ID: {entry['transactionId']}")
    print(f"Contents: {entry['contents']}")
```


## Next steps

- [Overview of Microsoft Azure confidential ledger](overview.md)
- [User defined functions in Azure confidential ledger](server-side-programming.md)
- [API reference documentation](https://azuresdkdocs.z19.web.core.windows.net/python/azure-confidentialledger/latest/azure.confidentialledger.html) 
