---
title: Secure your Azure Payment HSM
description: Learn how to secure Azure Payment HSM, with best practices for network security, identity management, monitoring, and key management.
author: msmbaldwin
ms.author: mbaldwin
ms.service: azure-payment-hsm
ms.topic: best-practice
ms.date: 03/31/2026
ms.custom: horz-security
ai-usage: ai-assisted
---

# Secure your Azure Payment HSM

Azure Payment HSM is a "BareMetal" service that provides cryptographic key operations for real-time, critical payment transactions in the Azure cloud using Thales payShield 10K payment hardware security modules (HSMs). Because this service protects sensitive financial cryptographic keys and operations, implementing robust security controls is essential to maintain the integrity and confidentiality of your payment infrastructure.

This article provides guidance on how to best secure your Azure Payment HSM deployment. Following these recommendations helps you maintain a strong security posture and protect your cryptographic assets.

[!INCLUDE [Security horizontal Zero Trust statement](~/reusable-content/ce-skilling/azure/includes/security/zero-trust-security-horizontal.md)]

## Network security

Azure Payment HSM requires a properly secured network environment for its operation, especially when connecting to on-premises environments or implementing high availability scenarios.

- **Deploy HSMs in an isolated subnet with proper delegation**: Create a dedicated subnet for your HSM devices with delegation set to `Microsoft.HardwareSecurityModules/dedicatedHSMs`. This provides necessary isolation and ensures proper resource provisioning. For more information, see [Create a payment HSM](create-payment-hsm.md).

- **Enable the FastPathEnabled feature flag and VNet tag**: Register the `FastPathEnabled` feature flag on every subscription that needs access to Azure Payment HSM, and add the `fastpathenabled` tag to every virtual network connecting to the payment HSM's delegated subnet. Both steps must be completed before resources can connect to HSM devices. For more information, see [Fastpathenabled](fastpathenabled.md).

- **Implement traffic inspection with appropriate architecture**: Network Security Groups (NSGs) and User-Defined Routes (UDRs) aren't supported on payment HSM subnets. To inspect and filter traffic to your HSM devices, implement a firewall with source network address translation (SNAT) or a reverse proxy. For more information, see [Azure Payment HSM traffic inspection](inspect-traffic.md).

- **Implement secure connectivity to on-premises resources**: Use Site-to-Site VPN or ExpressRoute to create secure connections between your on-premises environment and Azure-based HSMs. Virtual network peering doesn't support cross-region communication with payment HSM instances without the use of ExpressRoute or a VPN gateway. For more information, see [Solution design for Azure Payment HSM](solution-design.md).

## Identity and access management

Proper identity and access management is crucial for securing your HSM devices, which store and process your most sensitive cryptographic keys.

- **Secure your administrative cards and readers**: Protect the smart cards and readers required for HSM administration. A minimum of five smart cards (three cards for a CTA set, a Left Key Card, and a Right Key Card) and one reader are needed. Follow Thales's payShield 10K Installation and User Guide for detailed instructions. For more information, see [Access the payShield manager](access-payshield-manager.md).

- **Change default passwords immediately after provisioning**: When you first access a newly provisioned HSM device, change any default credentials to strong, complex passwords. Store these passwords securely following your organization's credential management policies. For more information, see [Access the payShield manager](access-payshield-manager.md).

- **Apply the least privilege principle to HSM roles**: Assign the minimum necessary permissions to users through Thales payShield's role-based system. Use separate accounts for different administrative tasks and cryptographic operations as needed. For more information, see [Azure Payment HSM Lifecycle Management](lifecycle-management.md).

- **Understand HSM management responsibilities**: Microsoft handles physical security of the HSM while customers are responsible for the logical security, configuration, and maintenance of allocated HSMs. Microsoft doesn't maintain console-access once an HSM is allocated. For more information, see [Azure Payment HSM Lifecycle Management](lifecycle-management.md).

- **Carefully manage key component distribution**: When keys need to be distributed to third parties in the form of key components, use a Trusted Management Device (TMD) or similar solution to export keys securely from your customer's location. For more information, see [Azure Payment HSM Lifecycle Management](lifecycle-management.md#key-management--customer-scenarios).

## Data protection

Protecting the cryptographic keys and sensitive data stored in your HSM is the primary purpose of using Azure Payment HSM.

- **Ensure proper commissioning and LMK generation**: Follow Thales's payShield 10K Installation and User Guide for HSM commissioning, HSM configuration, and loading Local Master Keys (LMKs). For more information, see [Access the payShield manager](access-payshield-manager.md).

- **Apply firmware updates regularly**: Microsoft allocates Payment HSMs with a base image that includes approved firmware. You're responsible for applying security patches and firmware updates to allocated HSMs. For more information, see [Azure Payment HSM Lifecycle Management](lifecycle-management.md#hsm-firmware-management).

- **Properly decommission HSMs when no longer needed**: When decommissioning an HSM, release the device through the payShield Manager before deleting it in Azure. For firmware versions earlier than 2.1a (1.15.0), also use the `SV` command to view loaded certificates and the `SD` command to delete any remaining certificate data before releasing the device. This ensures all customer content and secrets are removed before the HSM is returned to Microsoft. For more information, see [Remove a commissioned payment HSM](remove-payment-hsm.md) and [Known issues](known-issues.md#tls-certificates-arent-removed-during-hsm-release).

## Logging and monitoring

Comprehensive logging and monitoring are essential for maintaining visibility into HSM operations, detecting potential security incidents, and ensuring compliance.

- **Monitor HSM operational health**: Take responsibility for monitoring your allocated HSM's operational health, including HSM error logs and audit logs. You can use all payShield monitoring solutions. For more information, see [Azure Payment HSM Lifecycle Management](lifecycle-management.md#hsm-monitoring).

- **Implement real-time log replication and backup**: Back up your HSM audit and error logs regularly. Implement real-time log replication to ensure that log data is preserved and available for incident investigation and compliance auditing. For more information, see [Azure Payment HSM service support guide](support-guide.md).

- **Implement SNMP monitoring**: Configure Simple Network Management Protocol (SNMP) to monitor the health and status of your HSM devices. This allows you to proactively detect hardware issues before they affect operations. For more information, see [Getting started with Azure Payment HSM](getting-started.md).

- **Consider using payShield Monitor**: For continuous monitoring of HSMs, consider using payShield Monitor, which is available for licensing from Thales. For more information, see [Getting started with Azure Payment HSM](getting-started.md).

## Compliance and governance

Azure Payment HSM meets stringent compliance standards for payment security. Understanding the certification landscape helps you maintain audit compliance for your payment infrastructure.

- **Use Azure Payment HSM to simplify PCI compliance**: Azure Payment HSM can be deployed as part of a validated PCI P2PE or PCI PIN component or solution, helping to simplify ongoing security audit compliance. Microsoft maintains PCI DSS, PCI PIN, and PCI 3DS compliant Azure data centers, including those that house Azure Payment HSM solutions. For more information, see [Certification and compliance](certification-compliance.md).

- **Verify HSM device certifications**: Thales payShield 10K HSMs deployed in the security infrastructure are certified to FIPS 140-2 Level 3 and PCI HSM v3. The service also meets ISO 27001, SOC 1/2/3, and other compliance standards. For more information, see [Certification and compliance](certification-compliance.md).

## Backup and recovery

Implementing proper backup and recovery mechanisms is crucial for maintaining service availability and protecting against data loss.

- **Design proper high availability to maintain resiliency**: Implement high availability by provisioning HSMs across different stamps in the same region, and deploy additional HSMs in alternate regions for disaster recovery. Without a high availability setup in your production environment, you can't receive Severity A support from Microsoft. For more information, see [Deployment scenarios](deployment-scenarios.md).

- **Implement key backup procedures**: Regularly back up your cryptographic keys using approved methods. Since Payment HSMs operate in a stateless server model, proper key backup is essential to prevent data loss. For more information, see [FAQ](faq.yml) and [Lifecycle Management](lifecycle-management.md).

- **Test recovery procedures regularly**: Periodically test your recovery procedures to ensure they work properly when needed. Document these procedures and keep them updated as your environment changes. For more information, see [Deployment scenarios](deployment-scenarios.md).

## Next steps

- [Overview of Azure Payment HSM](overview.md)
- [Getting started with Azure Payment HSM](getting-started.md)
- [Create a payment HSM](create-payment-hsm.md)
- [Solution design for Azure Payment HSM](solution-design.md)
- [Azure Payment HSM frequently asked questions](faq.yml)

