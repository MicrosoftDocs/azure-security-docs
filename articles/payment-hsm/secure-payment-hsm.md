---
title: Secure your Azure Payment HSM
description: Learn how to secure Azure Payment HSM, with best practices for network security, identity management, monitoring, and key management.
author: msmbaldwin
ms.author: mbaldwin
ms.service: azure-payment-hsm
ms.topic: conceptual
ms.date: 04/10/2025
ms.custom: horz-security
---

# Secure your Azure Payment HSM

Azure Payment HSM is a "BareMetal" service that provides cryptographic key operations for real-time, critical payment transactions in the Azure cloud using Thales payShield 10K payment hardware security modules (HSMs). Because this service protects sensitive financial cryptographic keys and operations, implementing robust security controls is essential to maintain the integrity and confidentiality of your payment infrastructure.

This article provides guidance on how to best secure your Azure Payment HSM deployment. Following these recommendations will help you maintain a strong security posture and protect your cryptographic assets.

## Network security

Azure Payment HSM requires a properly secured network environment for its operation, especially when connecting to on-premises environments or implementing high availability scenarios.

- **Deploy HSMs in an isolated subnet with proper delegation**: Create a dedicated subnet for your HSM devices with delegation set to "Microsoft.HardwareSecurityModules/dedicatedHSMs". This provides necessary isolation and ensures proper resource provisioning. For more information, see [Create a payment HSM](create-payment-hsm.md).

- **Implement traffic inspection with appropriate architecture**: Since Network Security Groups (NSGs) and User-Defined Routes (UDRs) aren't supported on payment HSM subnets, consider implementing a firewall with source network address translation (SNAT) or a reverse proxy to inspect traffic. This enables you to monitor and filter traffic to your HSM devices. For more information, see [Azure Payment HSM traffic inspection](inspect-traffic.md).

- **Implement secure connectivity to on-premises resources**: Use Site-to-Site VPN or ExpressRoute to create secure connections between your on-premises environment and Azure-based HSMs. This enables secure management and operations while protecting sensitive traffic. For more information, see [Solution design for Azure Payment HSM](solution-design.md).

## Identity management

Proper identity and access management is crucial for securing your HSM devices, which store and process your most sensitive cryptographic keys.

- **Secure your administrative cards and readers**: Protect the smart cards and readers required for HSM administration. A minimum of five smart cards (three cards for a CTA set, a Left Key Card, and a Right Key Card) and one reader are needed. Follow Thales's payShield 10K Installation and User Guide for detailed instructions. For more information, see [Access the payShield manager](access-payshield-manager.md).

- **Change default passwords immediately after provisioning**: When you first access a newly provisioned HSM device, change any default credentials to strong, complex passwords. Store these passwords securely following your organization's credential management policies.

- **Implement strong authentication for HSM access**: Configure the HSM to use strong authentication mechanisms for all accounts. Be aware of the different thresholds for login attempts to avoid accidental zeroization of your HSM.

- **Carefully manage key component distribution**: When keys need to be distributed to third parties in the form of key components, use a Trusted Management Device (TMD) or similar solution to export keys securely from your customer's location. For more information, see [Azure Payment HSM Lifecycle Management](lifecycle-management.md#key-management--customer-scenarios).

## Privileged access

Controlling privileged access to HSM devices is essential for maintaining the security of your cryptographic operations and sensitive key material.

- **Apply least privilege principle to HSM roles**: Assign the minimum necessary permissions to users through Thales payShield's role-based system. Use separate accounts for different administrative tasks and cryptographic operations as needed.

- **Be aware of HSM management responsibilities**: Understand that Microsoft handles physical security of the HSM while customers are responsible for the logical security, configuration, and maintenance of allocated HSMs. Microsoft does not maintain console-access once an HSM is allocated. For more information, see [Azure Payment HSM Lifecycle Management](lifecycle-management.md).

## Backup and Recovery

Implementing proper backup and recovery mechanisms is crucial for maintaining service availability and protecting against data loss.

- **Design proper high availability to maintain resiliency**: Implement high availability by provisioning HSMs across different stamps in the same region, and deploy additional HSMs in alternate regions for disaster recovery. This ensures continuous availability of your payment infrastructure. For more information, see [Deployment scenarios](deployment-scenarios.md).

- **Implement key backup procedures**: Regularly backup your cryptographic keys using approved methods. Since Payment HSMs operate in a stateless server model, proper key backup is essential to prevent data loss. For more information, see [FAQ](faq.yml) and [Lifecycle Management](lifecycle-management.md).

- **Test recovery procedures regularly**: Periodically test your recovery procedures to ensure they work properly when needed. Document these procedures and keep them updated as your environment changes.

## Logging and monitoring

Comprehensive logging and monitoring are essential for maintaining visibility into HSM operations, detecting potential security incidents, and ensuring compliance.

- **Monitor HSM operational health**: Take responsibility for monitoring your allocated HSM's operational health, including HSM error logs and audit logs. You can utilize all payShield monitoring solutions. For more information, see [Azure Payment HSM Lifecycle Management](lifecycle-management.md#hsm-monitoring).

- **Implement SNMP monitoring**: Configure Simple Network Management Protocol (SNMP) to monitor the health and status of your HSM devices. This allows you to proactively detect hardware issues before they impact operations. For more information, see [Getting started with Azure Payment HSM](getting-started.md).

- **Consider using payShield Monitor**: For continuous monitoring of HSMs, consider using payShield Monitor, which is available for licensing from Thales. For more information, see [Getting started with Azure Payment HSM](getting-started.md).

- **Set up alerts for critical events**: Configure alerting based on critical log events, such as failed login attempts, configuration changes, or hardware issues. This enables prompt response to potential security incidents.

## Data protection

Protecting the cryptographic keys and sensitive data stored in your HSM is the primary purpose of using Azure Payment HSM.

- **Ensure proper commissioning and LMK generation**: Follow Thales's payShield 10K Installation and User Guide for HSM commissioning, HSM configuration, and loading Local Master Keys (LMKs). For more information, see [Access the payShield manager](access-payshield-manager.md).

- **Apply firmware updates regularly**: Microsoft allocates Payment HSMs with a base image that includes approved firmware. You are responsible for applying security patches and firmware updates to allocated HSMs. For more information, see [Azure Payment HSM Lifecycle Management](lifecycle-management.md#hsm-firmware-management).

- **Properly decommission HSMs when no longer needed**: When decommissioning an HSM, ensure you properly release the device through the payShield Manager before deleting it in Azure. This ensures all customer content/secrets are removed before it is returned to Microsoft. For more information, see [Remove a commissioned payment HSM](remove-payment-hsm.md).

- **Understand that zeroization occurs automatically**: When deallocating HSMs from customers, all encryption material is erased from the device as part of the mechanism that re-enables Microsoft's administrative access. For more information, see [Azure Payment HSM Lifecycle Management](lifecycle-management.md#security-and-compliance).


## Next steps

- [Overview of Azure Payment HSM](overview.md)
- [Getting started with Azure Payment HSM](getting-started.md)
- [Create a payment HSM](create-payment-hsm.md)
- [Solution design for Azure Payment HSM](solution-design.md)
- [Azure Payment HSM frequently asked questions](faq.yml)

