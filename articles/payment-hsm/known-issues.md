---
title: Azure Payment HSM known issues
description: Azure Payment HSM known issues
services: payment-hsm
author: msmbaldwin

ms.service: azure-payment-hsm
ms.topic: article
ms.date: 04/10/2025
ms.author: mbaldwin
---

# Azure Payment HSM known issues

This article describes some known issues with Azure Payment HSM. Before deploying Azure Payment HSM, review the [deployment scenarios](deployment-scenarios.md) and [solution design](solution-design.md) to ensure proper configuration.

## The PayShield fan is running too fast

Sporadic problems occur with the PS10K HSM, where the error log indicates that one of the fans is running too fast. Once this error occurs, the unit's error log replicates it once every 24 hours. The error is benign and doesn't affect the HSM's operational functionalities. Clearing the specific error entry from the HSM involves a hard-reboot to the unit. The fan error problem is fixed with Thales payShield firmware release versions v1.8a and 1.6a.  

If Azure Payment HSM customers observe the fan too fast error and want to do a hard-reboot to the unit, contact [Microsoft support](support-guide.md#microsoft-support). For more information on rebooting HSMs, see [Lifecycle management: Managing unresponsive HSM devices](lifecycle-management.md#managing-unresponsive-hsm-devices).

See details in [Thales support portal KB0026952](https://supportportal.thalesgroup.com/csm?sys_kb_id=6fe423cec319259063ec26359901310c&id=kb_article_view&sysparm_rank=1&sysparm_tsqueryId=18143570dba96d544f917828f496190c&sysparm_article=KB0026952). 

## Shared memory error logged in Hosted HSM after reboot

After a reboot, either manual or as a result of a firmware upgrade, some Hosted HSMs experience the following shared memory permission errors:

```bash
,bullsharkprod,ntfn:,mem.c:81,Failed to open/create shared memory because Permission denied
,bullsharkprod,ntfn:,misc.c:356,Unable to create shared pConsoleDisabledByGUI because No such file or directory
,bullsharkprod,ntfn:,mem.c:81,Failed to open/create shared memory because Permission denied
```

These errors occur under several circumstances: when accessing the payShield manager landing page, during sign-in or sign-out of payShield manager, and when using the JK host command. In the case of the JK host command, the error repeats after each attempt until a workaround is applied.

This issue is limited in scope. The problem only affects HSMs in a HOSTED HSM environment, and specifically those HSMs use SNMP or the JK host command. Hosted HSMs with SNMP disabled or those not utilizing the JK command don't experience these errors or related problems.

The impact of this problem is minimal. While it does cause entries to appear in the payShield error log, it doesn't affect the operation of the payShield 10k in any way. Essentially, the issue is confined to log entries and doesn't compromise the functionality or performance of the system.

A fix is currently being worked on and will be released in a future payShield firmware.

For more information and a workaround, see [Thales support portal KB0028943](https://supportportal.thalesgroup.com/csm?id=kb_article_view&sys_kb_id=ae8f0d9283b41a10fc177e126daad306&sysparm_article=KB0028943) (sign-in required). If you have questions regarding the issue or workaround, open a support ticket with Thales.

## TLS certificates aren't removed during HSM release

When you execute the RELEASE function from the payShield Manager to fully zeroize the PayShield Cloud HSM to factory state, the process removes all HSM settings except for loaded TLS certificates. This exception occurs because of a product bug where the TLS certificate deletion function wasn't implemented.

> [!NOTE]
> The residual TLS certificates after the RELEASE/RECLAIM operation are public certificates and pose no security risk.

**Resolution**: The product team fixed this bug in payShield HSM firmware version 2.1a 2100 0000 (1.15.0) and later.

**Recommended actions**:
- For firmware versions earlier than 2.1a (1.15.0), run the `SV` command on the virtual console to view certificates, and then use the `SD` command to delete any remaining certificate data.
- Remove all certificate data before releasing or returning the HSM. For detailed steps, see [Tutorial: Remove a commissioned payment HSM](remove-payment-hsm.md).
- Plan a firmware upgrade to version 2.1a (1.15.0) or later. For information about firmware versions, see [Support guide: Firmware and license support](support-guide.md#firmware-and-license-support).

For more information, see the Thales Support Portal KB Article (sign-in required). If you need assistance, contact Thales Support.

## Next steps

- Learn more about [Azure Payment HSM](overview.md).
- See some common [deployment scenarios](deployment-scenarios.md).
- Review [Azure Payment HSM lifecycle management](lifecycle-management.md).
- Learn about [Certification and compliance](certification-compliance.md).
- Understand [Azure Payment HSM support](support-guide.md).
- Read the [frequently asked questions](faq.yml).
