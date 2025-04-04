---
title: Secure your Azure Dedicated HSM
description: Learn the best practices for securing Azure Dedicated HSM, including network security, identity management, monitoring, and backup strategies.
author: msmbaldwin
ms.author: mbaldwin
ms.service: azure-dedicated-hsm
ms.topic: conceptual
ms.date: 03/20/2025
ms.custom: horz-security
---

# Secure your Azure Dedicated HSM

Azure Dedicated HSM provides cryptographic key storage in Azure with FIPS 140-2 Level 3-validated devices and complete customer control of the HSM appliance. As this service handles your most sensitive cryptographic keys and operations, implementing strong security controls is essential to protect these critical resources.

This article provides guidance on how to best secure your Azure Dedicated HSM deployment. Following these recommendations will help you maintain a strong security posture and protect your cryptographic assets.

## Network security

Azure Dedicated HSM requires a highly secure networking environment whether connecting from Azure cloud back to your on-premises environment, using distributed applications, or implementing high availability scenarios.

- **Deploy HSMs in an isolated subnet with proper delegation**: Create a dedicated subnet for your HSM devices with delegation set to "Microsoft.HardwareSecurityModules/dedicatedHSMs". This provides necessary isolation and ensures proper resource provisioning. For more information, see [Azure Dedicated HSM networking](networking.md).

- **Implement secure connectivity to on-premises resources**: Use Site-to-Site VPN or Point-to-Site VPN to create secure connections between your on-premises environment and Azure-based HSMs. This enables secure management and backup operations while protecting sensitive traffic. For more information, see [Connecting your on-premises IT to Azure](networking.md#connecting-your-on-premises-it-to-azure).

- **Connect virtual networks using VNet peering**: Use Virtual Network Peering to create secure communication channels between virtual networks within the same region. This provides low-latency, high-bandwidth connections between HSM devices and the applications that use them. For more information, see [Connecting virtual networks](networking.md#connecting-virtual-networks).

- **Implement cross-region connectivity for disaster recovery**: For globally distributed applications or high availability scenarios across regions, set up VPN Gateway connections between virtual networks in different regions. This creates a secure tunnel for HSM communication during regional failover scenarios. For more information, see [Connecting across Azure Regions](networking.md#connecting-across-azure-regions).

- **Use Network Virtual Appliance proxies as an alternative to NSGs**: Since Network Security Groups (NSGs) are not supported on dedicated HSM subnets, implement a Network Virtual Appliance (NVA) proxy tier in a transit/DMZ hub to provide similar security controls. This allows you to place logical firewalls in front of HSM devices. For more information, see [Networking Restrictions](networking.md#networking-restrictions).

- **Configure proper NAT settings when using NVA proxies**: When using Network Virtual Appliances for traffic control, configure proper Network Address Translation and disable client IP-check in Luna HSM configuration. This ensures correct traffic routing while maintaining security. For more information, see [Alternative to UDRs](networking.md#alternative-to-udrs).

## Identity management

Proper identity and authentication management is crucial for securing access to your HSM devices, which store and process your most sensitive cryptographic keys.

- **Change default passwords immediately after provisioning**: When you first access a newly provisioned HSM device, change the default "tenant admin" password to a strong, complex password. Store this password securely following your organization's credential management policies. Losing this password could require device reset and loss of key material.

- **Implement strong authentication for HSM access**: Configure the HSM to use strong authentication mechanisms for all accounts. Be aware of the different thresholds for login attempts across different roles to avoid accidental zeroization. For more information, see [Failed Logins](troubleshoot.md#failed-logins).

- **Limit local authentication methods**: Avoid using local authentication methods or accounts wherever possible. These should be disabled when not required to reduce the attack surface. For more information, see [Failed Logins](troubleshoot.md#failed-logins).

- **Be cautious with credential management**: Store HSM credentials securely according to your organization's policies. Both shell and HSM credentials should be protected, as loss of credentials can result in complete loss of access and key material. For more information, see [Lost Credentials](troubleshoot.md#lost-credentials).

## Privileged access

Controlling privileged access to HSM devices is essential for maintaining the security of your cryptographic operations and sensitive key material.

- **Apply least privilege principle to HSM roles**: Assign the minimum necessary permissions to users through the HSM's role-based system. Use separate accounts for administrative tasks (HSM SO), partition management (Partition SO), and cryptographic operations (Crypto Officer, Crypto User). For more information, see the [Thales Luna 7 HSM Documentation](https://thalesdocs.com/gphsm/luna/7.2/docs/network/Content/Home_network.htm).

- **Be aware of role lockout thresholds**: Understand the different thresholds for failed login attempts across HSM roles. For example, the HSM SO role is zeroized after 3 failed attempts, while the Partition SO role is zeroized after 10 failed attempts. For more information, see [Failed Logins](troubleshoot.md#failed-logins).

- **Monitor and audit privileged access**: Configure logging to track all administrative and privileged operations performed on the HSM. Monitor these logs regularly to detect unauthorized access attempts or suspicious activities. For more information, see [Customer monitoring](monitoring.md#customer-monitoring).

- **Disable Microsoft's monitor account if required**: While Microsoft uses a non-administrative "monitor" role to retrieve basic telemetry for device health, you can disable this account if your security requirements demand complete isolation. Be aware that this means Microsoft will have no visibility into device health issues. For more information, see [Microsoft monitoring](monitoring.md#microsoft-monitoring).

## Logging and monitoring

Comprehensive logging and monitoring are critical for maintaining visibility into HSM operations, detecting potential security incidents, and ensuring compliance.

- **Implement SNMP monitoring**: Configure SimOple Network Management Protocol (SNMP) to monitor the health and status of your HSM devices. This allows you to proactively detect hardware issues before they impact operations. For more information, see [Customer monitoring](monitoring.md#customer-monitoring).

- **Configure syslog for audit logging**: Set up syslog to collect and centralize audit logs from your HSM devices. These logs provide valuable information about administrative actions, cryptographic operations, and potential security events. For more information, see [Customer monitoring](monitoring.md#customer-monitoring).

- **Set up alerts for critical events**: Configure alerting based on critical log events, such as failed login attempts, configuration changes, or hardware issues. This enables prompt response to potential security incidents. For more information, see [Customer monitoring](monitoring.md#customer-monitoring).

- **Regularly review logs**: Establish a process for regular review of HSM logs to identify unusual patterns or potential security issues. This is particularly important if you've disabled Microsoft's monitoring capabilities. For more information, see [Customer monitoring](monitoring.md#customer-monitoring).

## Data protection

Protecting the cryptographic keys and sensitive data stored in your HSM is the primary purpose of using a dedicated HSM service.

- **Implement proper key backup procedures**: Regularly back up your HSM keys to secure backup devices. This protects against data loss in case of hardware failure or other incidents. Use a peer-to-peer or site-to-site VPN connection to an on-premises workstation connected to an HSM backup device. For more information, see the [FAQ on ensuring keys aren't lost](faq.yml).

- **Ensure secure key transport**: When transferring keys between HSMs or to backup devices, use secure key wrapping techniques to protect the keys during transit. For more information, refer to the Thales documentation available through the [Thales customer support portal](https://supportportal.thalesgroup.com/csm).

- **Enable encrypted communications**: Ensure that all communications with the HSM use secure, encrypted channels. The HSM supports data-in-transit encryption by default. For more information, see [Networking](networking.md#connecting-your-on-premises-it-to-azure).

- **Properly zeroize HSMs before deprovisioning**: Before requesting deprovisioning of an HSM, ensure you've properly zeroized the device to remove all key material. This can be done by getting the HSM admin password wrong three times or using the appropriate zeroization commands. For more information, see [How to delete an HSM resource](troubleshoot.md#how-to-delete-an-hsm-resource).

## High availability and disaster recovery

Implementing proper high availability and disaster recovery strategies is essential for maintaining continuous access to cryptographic services.

- **Deploy HSMs in pairs for high availability**: Provision at least two HSM devices in each region and configure them as a high availability pair using Thales software. This ensures continued availability of keys if a single device fails. For more information, see [High availability](high-availability.md).

- **Implement cross-region replication**: To protect against regional failures, deploy additional HSM pairs in a secondary region. Configure cross-region replication of key material to ensure business continuity during disaster scenarios. For more information, see [High availability example](high-availability.md#high-availability-example).

- **Regularly test recovery procedures**: Periodically test your recovery procedures to ensure that you can restore operations quickly in case of device or regional failures. This includes testing key restoration from backups and failover between high availability pairs. For more information, see the [Thales Luna 7 HSM Documentation](https://thalesdocs.com/gphsm/luna/7.2/docs/network/Content/Home_network.htm).

- **Ensure failed HA group members recover properly**: Configure retry counts for high availability groups to enable automatic recovery. By default, an HA group will not attempt to recover a member automatically. For more information, see [Failed HA Group Member Doesn't Recover](troubleshoot.md#failed-ha-group-member-doesnt-recover).

## Patching and updates

Keeping your HSM devices updated with the latest security patches is important for maintaining a strong security posture.

- **Apply critical security patches**: Regularly check for and apply security patches for your HSM devices, especially for critical vulnerabilities. For example, apply the KB0019789 patch to address issues where the system becomes unresponsive during reboot. For more information, see [HSM Device Reboot](troubleshoot.md#hsm-device-reboot).

- **Test patches in non-production environments first**: Before applying patches to production HSM devices, test them in a non-production environment to ensure compatibility and avoid unexpected downtime. For more information, refer to the Thales documentation available through the [Thales customer support portal](https://supportportal.thalesgroup.com/csm).

- **Keep client software updated**: Ensure that HSM client software running on your application servers is kept up-to-date with the latest security fixes. For more information, see [HSM Documentation and Software](troubleshoot.md#hsm-documentation-and-software).

## Next steps

- Review the [Azure Dedicated HSM security baseline](/azure/baselines/azure-dedicated-hsm-security-baseline) for additional security recommendations
- Learn about [high availability](high-availability.md) configurations for Azure Dedicated HSM
- Understand [networking considerations](networking.md) for Azure Dedicated HSM
- Explore [monitoring options](monitoring.md) for Azure Dedicated HSM
- Review the [troubleshooting guide](troubleshoot.md) for common issues and solutions

