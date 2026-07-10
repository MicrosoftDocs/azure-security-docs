---
title: Secure your Azure Dedicated HSM
description: Learn the best practices for securing Azure Dedicated HSM, including network security, identity management, monitoring, and backup strategies.
author: msmbaldwin
ms.author: mbaldwin
ms.service: azure-dedicated-hsm
ms.topic: best-practice
ms.date: 07/10/2026
ms.custom: horz-security
ai-usage: ai-assisted
---

# Secure your Azure Dedicated HSM

Azure Dedicated HSM provides cryptographic key storage in Azure with FIPS 140-2 Level 3 validated Thales Luna 7 HSM appliances and complete customer control of the appliance. Because this service handles your most sensitive cryptographic keys and operations, implementing strong security controls is essential.

> [!IMPORTANT]
> Azure Dedicated HSM is being retired. Microsoft will fully support existing Dedicated HSM customers until **July 31, 2028**, but no new customer onboardings are accepted. Plan a migration to [Azure Cloud HSM](/azure/cloud-hsm/overview) or [Azure Key Vault Managed HSM](/azure/key-vault/managed-hsm/overview). For more information, see [Migrate from Azure Dedicated HSM to Managed HSM or Cloud HSM](migration-guide.md). This article documents the security controls available for the remaining service lifetime; new deployments should target the successor services and their respective security articles.

[!INCLUDE [Security horizontal Zero Trust statement](~/reusable-content/ce-skilling/azure/includes/security/zero-trust-security-horizontal.md)]

This article provides security recommendations to help protect your existing Azure Dedicated HSM deployment through the end of its supported lifetime.

## Network security

Azure Dedicated HSM requires a highly secure networking environment, whether you're connecting from Azure back to your on-premises environment, using distributed applications, or implementing high-availability scenarios.

- **Deploy HSMs in an isolated subnet with proper delegation**: Create a dedicated subnet for your HSM devices with delegation set to `Microsoft.HardwareSecurityModules/dedicatedHSMs`. This delegation provides necessary isolation and ensures proper resource provisioning. For more information, see [Azure Dedicated HSM networking](networking.md).

- **Implement secure connectivity to on-premises resources**: Use Site-to-Site VPN or Point-to-Site VPN to create secure connections between your on-premises environment and Azure-based HSMs. This approach enables secure management and backup operations while protecting sensitive traffic. For more information, see [Connecting your on-premises IT to Azure](networking.md#connecting-your-on-premises-it-to-azure).

- **Connect virtual networks by using VNet peering**: Use virtual network peering to create secure communication channels between virtual networks within the same region. This approach provides low-latency, high-bandwidth connections between HSM devices and the applications that use them. For more information, see [Connecting virtual networks](networking.md#connecting-virtual-networks).

- **Implement cross-region connectivity for disaster recovery**: For globally distributed applications or high-availability scenarios across regions, set up VPN Gateway connections between virtual networks in different regions. This configuration creates a secure tunnel for HSM communication during regional failover scenarios. For more information, see [Connecting across Azure regions](networking.md#connecting-across-azure-regions).

- **Use Network Virtual Appliance proxies as an alternative to NSGs**: Network security groups aren't supported on dedicated HSM subnets. Implement a Network Virtual Appliance (NVA) proxy tier in a transit or DMZ hub to provide similar security controls. This approach helps you place logical firewalls in front of HSM devices. For more information, see [Networking restrictions](networking.md#networking-restrictions).

- **Configure proper NAT settings when you use NVA proxies**: When you use NVAs for traffic control, configure network address translation and disable client IP check in the Luna HSM configuration. This configuration ensures correct traffic routing while maintaining security. For more information, see [Alternative to UDRs](networking.md#alternative-to-udrs).

## Identity and access management

Proper identity and access management is crucial for securing HSM devices, which store and process your most sensitive cryptographic keys. The appliance's own role-based system (HSM SO, Partition SO, Crypto Officer, Crypto User) controls access to the underlying Thales Luna 7 appliance; Azure RBAC controls only the Dedicated HSM Azure resource.

- **Change default passwords immediately after provisioning**: When you first access a newly provisioned HSM device, change the default tenant admin password to a strong, complex password. Store this password securely according to your organization's credential management policies. Losing this password can require a device reset and loss of key material.

- **Implement strong authentication for HSM access**: Configure the HSM to use strong authentication mechanisms for all accounts. Be aware of the different thresholds for sign-in attempts across different roles to avoid accidental zeroization. For more information, see [Failed logins](troubleshoot.md#failed-logins).

- **Limit local authentication methods**: Avoid using local authentication methods or accounts wherever possible. Disable them when they aren't required, to reduce the attack surface. For more information, see [Failed logins](troubleshoot.md#failed-logins).

- **Protect HSM credentials**: Store HSM credentials securely according to your organization's policies. Protect both shell and HSM credentials: loss of credentials can result in complete loss of access and key material. For more information, see [Lost credentials](troubleshoot.md#lost-credentials).

- **Apply the least-privilege principle to HSM roles**: Assign the minimum necessary permissions to users through the HSM's role-based system. Use separate accounts for administrative tasks (HSM SO), partition management (Partition SO), and cryptographic operations (Crypto Officer, Crypto User). For more information, see the [Thales Luna 7 HSM documentation](https://thalesdocs.com/gphsm/luna/7.2/docs/network/Content/Home_network.htm).

- **Understand role lockout thresholds**: The HSM SO role is zeroized after three failed sign-in attempts, while the Partition SO role is zeroized after 10 failed attempts. Design your operational procedures with these thresholds in mind. For more information, see [Failed logins](troubleshoot.md#failed-logins).

- **Monitor and audit privileged access**: Configure logging to track all administrative and privileged operations performed on the HSM, and review the logs regularly to detect unauthorized access attempts. For more information, see [Customer monitoring](monitoring.md#customer-monitoring).

- **Decide whether to disable Microsoft's monitor account**: Microsoft uses a nonadministrative monitor role to retrieve basic telemetry for device health. You can disable this account if your security requirements demand complete isolation, but doing so means Microsoft has no visibility into device health problems. For more information, see [Microsoft monitoring](monitoring.md#microsoft-monitoring).

## Data protection

The primary purpose of using a dedicated HSM service is to protect the cryptographic keys and sensitive data stored in your HSM.

- **Implement proper key backup procedures**: Regularly back up your HSM keys to secure backup devices. This practice protects against data loss in case of hardware failure or other incidents. Use a peer-to-peer or site-to-site VPN connection to an on-premises workstation connected to an HSM backup device. For more information, see the [Dedicated HSM FAQ](faq.yml).

- **Ensure secure key transport**: When you transfer keys between HSMs or to backup devices, use secure key wrapping techniques to protect the keys during transit. For more information, see the Thales documentation through the [Thales customer support portal](https://supportportal.thalesgroup.com/csm).

- **Enable encrypted communications**: Ensure all communications with the HSM use secure, encrypted channels. The HSM supports data-in-transit encryption by default. For more information, see [Networking](networking.md#connecting-your-on-premises-it-to-azure).

- **Zeroize HSMs before deprovisioning**: Before requesting deprovisioning of an HSM, zeroize the device to remove all key material. You can do this action by getting the HSM admin password wrong three times or by using the appropriate zeroization commands. For more information, see [How to delete an HSM resource](troubleshoot.md#how-to-delete-an-hsm-resource).

## Logging and monitoring

Comprehensive logging and monitoring are critical for maintaining visibility into HSM operations, detecting potential security incidents, and ensuring compliance.

- **Implement SNMP monitoring**: Configure Simple Network Management Protocol (SNMP) to monitor the health and status of your HSM devices. This monitoring helps you proactively detect hardware problems before they affect operations. For more information, see [Customer monitoring](monitoring.md#customer-monitoring).

- **Configure syslog for audit logging**: Set up syslog to collect and centralize audit logs from your HSM devices. These logs provide valuable information about administrative actions, cryptographic operations, and potential security events. For more information, see [Customer monitoring](monitoring.md#customer-monitoring).

- **Set up alerts for critical events**: Configure alerting based on critical log events, such as failed sign-in attempts, configuration changes, or hardware problems, to enable prompt response to potential security incidents. For more information, see [Customer monitoring](monitoring.md#customer-monitoring).

- **Regularly review logs**: Establish a process for regular review of HSM logs to identify unusual patterns or potential security problems. Regular review is particularly important if you disabled Microsoft's monitoring. For more information, see [Customer monitoring](monitoring.md#customer-monitoring).

## Compliance and governance

Regular compliance review helps you demonstrate that Azure Dedicated HSM continues to meet your regulatory obligations through the end of its supported lifetime, and helps you plan the migration to a successor service.

- **Verify HSM device certifications**: The Thales Luna 7 HSM appliances used by Azure Dedicated HSM are certified to FIPS 140-2 Level 3. Confirm this level meets your regulatory requirements. If you need FIPS 140-3, plan migration to Azure Cloud HSM or Managed HSM (both FIPS 140-3 Level 3).

- **Align with the Microsoft cloud security benchmark**: Use the horizontal controls in [Microsoft cloud security benchmark (MCSB) v2](/security/benchmark/azure/overview) to structure network, identity, data, and monitoring controls consistently across your Dedicated HSM deployment and the successor service you migrate to.

- **Track patching and updates as part of governance**: Regularly check for and apply security patches for your HSM devices, especially for critical vulnerabilities. Test patches in a nonproduction environment first, and keep the HSM client software running on your application servers up to date. For more information, see [HSM device reboot](troubleshoot.md#hsm-device-reboot) and [HSM documentation and software](troubleshoot.md#hsm-documentation-and-software).

## Backup and recovery

Implement proper backup, high-availability, and disaster-recovery strategies to maintain continuous access to cryptographic services.

- **Deploy HSMs in pairs for high availability**: Provision at least two HSM devices in each region and configure them as a high-availability pair by using Thales software. This configuration ensures continued availability of keys if a single device fails. For more information, see [High availability](high-availability.md).

- **Implement cross-region replication**: To protect against regional failures, deploy more HSM pairs in a secondary region. Configure cross-region replication of key material to ensure business continuity during disaster scenarios. For more information, see [High availability example](high-availability.md#high-availability-example).

- **Ensure failed HA group members recover properly**: Configure retry counts for high-availability groups to enable automatic recovery. By default, an HA group doesn't attempt to recover a member automatically. For more information, see [Failed HA group member doesn't recover](troubleshoot.md#failed-ha-group-member-doesnt-recover).

- **Test recovery procedures regularly**: Periodically test your recovery procedures to ensure you can restore operations quickly in case of device or regional failures. This testing includes key restoration from backups and failover between high-availability pairs. For more information, see the [Thales Luna 7 HSM documentation](https://thalesdocs.com/gphsm/luna/7.2/docs/network/Content/Home_network.htm).

## Next steps

- [Migrate from Azure Dedicated HSM to Managed HSM or Cloud HSM](migration-guide.md)
- [Secure your Azure Cloud HSM deployment](/azure/cloud-hsm/secure-cloud-hsm)
- [Secure your Azure Key Vault Managed HSM deployment](/azure/key-vault/managed-hsm/secure-managed-hsm)
- [High availability](high-availability.md) configurations for Azure Dedicated HSM
- [Networking considerations](networking.md) for Azure Dedicated HSM
- [Monitoring options](monitoring.md) for Azure Dedicated HSM
- [Troubleshooting guide](troubleshoot.md) for common issues and solutions

