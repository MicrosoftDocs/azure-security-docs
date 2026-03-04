---
title: Synchronize users and keys across Azure Cloud HSM nodes
description: Learn how to identify and fix missing users or keys across Azure Cloud HSM cluster nodes to prevent data loss.
author: keithp
manager: davinune
ms.service: azure-cloud-hsm
ms.topic: how-to
ms.date: 03/20/2025
ms.author: keithp
---

# Synchronize users and keys across Azure Cloud HSM nodes

This article explains how to identify and resolve synchronization issues when users or keys are missing from one or more nodes in your Azure Cloud HSM cluster.

> [!CAUTION]
> **Missing users or keys can lead to unrecoverable data loss.** If a user or key exists on only one node and that node fails, you might be permanently locked out of your cryptographic operations with no way to recover. Always verify that all users and keys are synchronized across all three nodes in your cluster, and maintain regular backups.

## Why synchronization matters

Azure Cloud HSM operates as a cluster of three nodes. When you create users or keys, they should be replicated across all nodes. However, if creation fails on one or more nodes (for example, due to temporary network issues or node unavailability), the user or key might exist on only some nodes.

This partial synchronization creates serious risks:

- **Authentication failures**: Your application might intermittently fail to authenticate if it connects to a node that's missing the user.
- **Cryptographic operation failures**: Operations might fail if the required key doesn't exist on the node handling the request.
- **Permanent data loss**: If the only node containing a user or key fails and you don't have a backup, you lose access permanently with no recovery option.

## Prerequisites

- Access to a VM with the Azure Cloud HSM SDK installed
- The `azcloudhsm_mgmt_util` tool
- Cryptography officer (CO) credentials for your Azure Cloud HSM deployment

## User synchronization

All users in Azure Cloud HSM are fully managed by the customer. The service doesn't perform backend user synchronization if user creation fails. If user creation fails on one or more nodes, you must manually synchronize the user to the missing nodes.

> [!IMPORTANT]
> Ensure that all users are consistently created and present across every node in the cluster. If a user is missing or creation fails on any node, you must execute the required commands and perform validation steps to restore consistency.

### Identify missing users

1. Start the management utility:

   ```bash
   ./azcloudhsm_mgmt_util ./azcloudhsm_resource.cfg
   ```

1. Run the `listUsers` command to display the User ID, User Type, and Username under each node (server 0, 1, 2):

   ```bash
   listUsers
   ```

1. Observe the number of users found for each node and corresponding usernames. Compare the lists across all three servers to identify any missing users.

### Synchronize missing users

1. Sign in as a cryptography officer (CO):

   ```bash
   loginHSM CO admin <adminPassword>
   ```

   Verify that you successfully signed in to all three nodes:

   ```output
   loginHSM success on server 0
   loginHSM success on server 1
   loginHSM success on server 2
   ```

   > [!NOTE]
   > If sign-in fails on any node, the sync operation might fail. Ensure successful sign-in to all nodes before proceeding.

1. Identify the source node that has the user. In this example, server 0 has a user that's not available on server 1 and server 2:

   ```bash
   server 0
   ```

1. Run the `syncUser` command for each server where the user is missing. Replace `<UserID>` with the actual User ID:

   ```bash
   syncUser <UserID> 1
   syncUser <UserID> 2
   ```

   > [!NOTE]
   > If `syncUser` is executed against a node where the user already exists, the error message "user already created, unable to insert object" appears. If the user doesn't exist, the operation succeeds.

### Validate user synchronization

1. Exit the current server context:

   ```bash
   exit
   ```

1. Run `listUsers` to confirm that all User IDs, User Types, and Usernames are now equal and available under each node (server 0, 1, 2):

   ```bash
   listUsers
   ```

## Key synchronization

When you create keys, it's your responsibility to ensure keys are present on all nodes. Although Azure Cloud HSM supports service-side key synchronization and restore operations, you must verify that keys are available on any missing nodes before use.

> [!IMPORTANT]
> If a key is missing or creation fails on any node, you must execute the appropriate commands and perform validation steps to restore consistency. A key that exists on only one node is at risk of permanent loss if that node fails.

### Identify missing keys

1. Start the management utility:

   ```bash
   ./azcloudhsm_mgmt_util ./azcloudhsm_resource.cfg
   ```

1. Sign in as a cryptography officer (CO):

   ```bash
   loginHSM CO admin <adminPassword>
   ```

   Verify that you successfully signed in to all three nodes:

   ```output
   loginHSM success on server 0
   loginHSM success on server 1
   loginHSM success on server 2
   ```

1. Run the `findAllKeys` command to display the number of keys and key handle IDs under each node:

   ```bash
   findAllKeys 0 0
   ```

1. Observe the number of keys found for each node and corresponding key handle IDs. Compare the results across all three servers to identify any missing keys.

### Synchronize missing keys

1. Identify the source node that has the key. In this example, server 0 has a key handle that's not available on server 1 and server 2:

   ```bash
   server 0
   ```

1. Run the `syncKey` command for each server where the key is missing. Replace `<KeyHandle>` with the actual key handle ID:

   ```bash
   syncKey <KeyHandle> 1
   syncKey <KeyHandle> 2
   ```

   For example, to synchronize key handle 262150 to servers 1 and 2:

   ```bash
   syncKey 262150 1
   syncKey 262150 2
   ```

### Validate key synchronization

1. Exit the current server context:

   ```bash
   exit
   ```

1. Run `findAllKeys 0 0` to confirm that all key handles and the number of keys found are now equal and available under each node (server 0, 1, 2):

   ```bash
   findAllKeys 0 0
   ```

## Best practices

To prevent synchronization issues and potential data loss:

- **Verify after creation**: After creating any user or key, immediately verify that it exists on all three nodes.
- **Maintain regular backups**: Use the [backup and restore](backup-restore.md) functionality to protect against node failures.
- **Monitor for discrepancies**: Periodically run `listUsers` and `findAllKeys 0 0` to check for inconsistencies across nodes.
- **Act quickly on failures**: If you notice a user or key creation failure, synchronize it to the missing nodes before a node failure occurs.

## Related content

- [Troubleshoot Azure Cloud HSM](troubleshoot.md)
- [User management in Azure Cloud HSM](user-management.md)
- [Key management in Azure Cloud HSM](key-management.md)
- [Backup and restore in Azure Cloud HSM](backup-restore.md)
