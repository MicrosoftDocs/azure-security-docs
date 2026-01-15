---
title: What's new for Azure Payment HSM
description: Recent updates for Azure Payment HSM
ms.service: azure-payment-hsm
author: msmbaldwin
ms.author: mbaldwin
ms.topic: tutorial
ms.date: 04/10/2025
---

# What's new for Azure Payment HSM

Here's what's new with Azure Payment HSM. 

## December 2025

Azure Payment HSM upgraded the base firmware to 2.2b. Please refer to the Thales release notes for new features and bug fixes.

## September 2024

Azure Payment HSM upgraded the base firmware to 1.9a. For new features and bug fixes in the firmware 1.8a release, see the [Thales payShield 10K](https://cpl.thalesgroup.com/encryption/hardware-security-modules/payment-hsms/payshield-10k) release notes.

## May 2023

Azure Payment HSM now supports two host IP network interfaces from payShield 10K. Customers using two host network interfaces can now have a maximum of 128 concurrent connections.

There are no changes to the payment HSM resource creation process, as the two host network interfaces are created by default when you set up the PHSM. Additionally, you can only create these host network interfaces within the same virtual network, but you can use either static or dynamic IP addresses for the host interfaces.

For more information, see:
- [Create a payment HSM with the host and management port in the same virtual network using Azure CLI or PowerShell](create-payment-hsm.md)
- [Create a payment HSM with the host and management port in the same virtual network using an ARM template](quickstart-template.md)
- [Create a payment HSM with the host and management port in different virtual networks using Azure CLI or PowerShell](create-different-vnet.md)
- [Create a payment HSM with the host and management port in different virtual networks using an ARM template](create-different-vnet-template.md)
- [Create HSM resource with host and management port with IP addresses in different virtual networks using ARM template](create-different-ip-addresses.md)

## April 2023

Azure Payment HSM doesn't support traffic inspection by using UDR and NSG.

Currently, Payment HSM isn't compatible with vWAN topologies or cross region virtual network peering, as listed in the [topology supported](solution-design.md#supported-topologies). Payment HSM comes with some [policy restrictions](solution-design.md#constraints) on these subnets: **Network Security Groups (NSGs) and User-Defined Routes (UDRs) are currently not supported**. You can bypass the current UDR restriction and inspect traffic destined to a Payment HSM. [This article](inspect-traffic.md) presents two ways: a [firewall with source network address translation (SNAT)](inspect-traffic.md#firewall-with-source-network-address-translation-snat) and a [firewall with reverse proxy](inspect-traffic.md#firewall-with-reverse-proxy).

 
## November 2022

Azure Payment HSM is generally available. 

For more information, see the GA announcement: [Secure your digital payment system in the cloud with Azure Payment HSM—now generally available](https://azure.microsoft.com/blog/secure-your-digital-payment-system-in-the-cloud-with-azure-payment-hsm-now-generally-available).


## Next steps

If you have questions, contact us through [support](https://azure.microsoft.com/support/options/).
