---
title: Update Managed HSM Firmware for enhanced security and compliance
description: Updating Azure Key Vault Managed HSM firmware to FIPS 140-3 level 3 enhances security and compliance, ensuring seamless transitions and continued support for users globally.
services: azure-key-vault
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: tutorial
ms.date: 08/23/2024
ms.author: mbaldwin
---

# Update Managed HSM Firmware for enhanced security and compliance

We are excited to announce that we are updating the firmware of Azure Key Vault Managed HSM to a modern version that is validated to [FIPS 140-3 level 3 standards](https://csrc.nist.gov/projects/cryptographic-module-validation-program/certificate/4700). This update represents a significant step forward in our commitment to providing best in class security to safeguard your valuable data.

Here's what you need to know about this update.

**FIPS 140-3 Level 3 Validation**: Azure Key Vault Managed HSM firmware update ensures compliance with the rigorous standards set forth by FIPS 140-3 level 3. This validation demonstrates our dedication to adhering to the highest security standards and industry best practices, giving you peace of mind knowing that your data is handled with the utmost care and security.

**Improved Compliance**: In addition to bolstering security, the updated firmware also enhances compliance capabilities. Whether you are subject to industry regulations or internal compliance policies, this firmware ensures that your HSMs meet or exceed the required standards, helping you maintain regulatory compliance effortlessly.

**Seamless Transition**: We understand that system updates can sometimes be disruptive, which is why we are committed to ensuring a smooth and seamless transition process.

**Continued Support**: Our dedication to providing exceptional customer support remains unwavered. Should you have any questions or concerns regarding the firmware upgrade or its impact on your operations, our knowledgeable support team is here to assist you every step of the way.

We are confident that this firmware upgrade improves the security and compliance posture of your infrastructure, allowing you to focus on your core business activities with confidence.

Here are some things to keep in mind:

1. The Managed HSM firmware update is rolled out in phases across all Azure data centers globally and expected to take approximately 6-8 weeks starting August 2024.

2. Restoring Security Domains:
   - **From Legacy to New Firmware:** Security domains from a Managed HSM with legacy firmware can be restored to a Managed HSM with new firmware. Most of our customers fall into this category and will not have any issues restoring a security domain.
   - **From New to Legacy Firmware:** Security domains from a Managed HSM with new firmware cannot be restored to a Managed HSM with legacy firmware. This will affects a few of customers. Here is our guidance to resolve the issue:
     - The expected error message is “This security domain cannot be restored at this time due to necessary service maintenance.”
  
  Full key backup, restore, and selective key restore operations are unaffected by this firmware upgrade.
- Guidance for Resolving New to Legacy Firmware Security Domain Restore:
   - **Open a Support Ticket:** Include the exact error message, including the HSM pool name and region. Our support team escalates the issue to our engineering on-call team.
   - **Check Region Status:** The on-call team verifies if the firmware update is complete in the region where the error occurred.
     - **If Completed:** The on-call team unblocks SD restore/upload for the pool where the customer attempted to restore.
     - **If Not Completed:** The on-call team informs the customer to wait and suggest alternative regions where they could restore the SD. There is a possibility that you may have to wait up to eight weeks; therefore, your only option may be to restore your security domain to an alternate region.

We understand this may be an inconvenience for a small percentage of our customers, but the benefits of upgrading to a modern FIPS 140-3 level 3 firmware far outweigh the temporary inconvenience. This update significantly enhances the security and compliance posture of your key management solution.

Upon completion of the firmware update of the Managed HSM in all Azure regions, this page will be revised.
