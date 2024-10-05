---
title: Azure Payment HSM known issues
description: Azure Payment HSM known issues
services: payment-hsm
author: msmbaldwin

ms.service: azure-payment-hsm
ms.topic: article
ms.date: 01/31/2024
ms.author: mbaldwin
---

# Azure Payment HSM known issues

This article describes some known issues with Azure Payment HSM.

## The PayShield fan is running too fast

Sporadic problems have been observed with the PS10K HSM, where the error log indicates that one of the fans is running too fast. Once this error occurs, it's replicated once every 24 hours to the unit's error log. The error is benign and doesn't affect the HSMs operational functionalities. Clearing the specific error entry from the HSM involves a hard-reboot to the unit. The fan error problem will be fixed with Thales payShield firmware release version v1.8a and 1.6a.  

If Azure Payment HSM customers observe the fan too fast error and want to do a hard-reboot to the unit, contact Microsoft support.

See details in [Thales support portal KB0026952](https://supportportal.thalesgroup.com/csm?sys_kb_id=6fe423cec319259063ec26359901310c&id=kb_article_view&sysparm_rank=1&sysparm_tsqueryId=18143570dba96d544f917828f496190c&sysparm_article=KB0026952). 

## Shared memory error logged in Hosted HSM after reboot

After a reboot, either manual or as a result of a firmware upgrade, some Hosted HSMs have experienced the following shared memory permission errors:

```bash
,bullsharkprod,ntfn:,mem.c:81,Failed to open/create shared memory because Permission denied
,bullsharkprod,ntfn:,misc.c:356,Unable to create shared pConsoleDisabledByGUI because No such file or directory
,bullsharkprod,ntfn:,mem.c:81,Failed to open/create shared memory because Permission denied
```

These errors are logged under several circumstances: when accessing the payShield manager landing page, during sign-in or sign-out of payShield manager, and when using the JK host command. In the case of the JK host command, the error will repeat after each attempt until a workaround is applied.

It's important to note that this issue is limited in scope. The problem only affects HSMs in a HOSTED HSM environment, and specifically those using SNMP or the JK host command. Hosted HSMs with SNMP disabled or those not utilizing the JK command will not experience these errors or related issues.

The impact of this problem is minimal. While it does cause entries to appear in the payShield error log, it does not affect the operation of the payShield 10k in any way. Essentially, the issue is confined to log entries and does not compromise the functionality or performance of the system.

A fix is currently being worked on and will be released in a future payShield firmware.

For more information and a workaround, see [Thales support portal KB0028943](https://supportportal.thalesgroup.com/csm?id=kb_article_view&sys_kb_id=ae8f0d9283b41a10fc177e126daad306&sysparm_article=KB0028943) (sign-in required). If you have questions regarding the issue or workaround, open a support ticket with Thales.

## Next steps

- Learn more about [Azure Payment HSM](overview.md)
- See some common [deployment scenarios](deployment-scenarios.md)
- Learn about [Certification and compliance](certification-compliance.md)
- Read the [frequently asked questions](faq.yml)
