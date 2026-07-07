---
title: Network security for Azure Cloud HSM
description: Learn how to secure your network configuration for Azure Cloud HSM, including private endpoints, DNS, required ports, NSG rules, and VM placement.
ms.service: azure-cloud-hsm
ms.author: mbaldwin
ms.topic: best-practice
ms.date: 07/06/2026
ai-usage: ai-assisted
#customer intent: As a security administrator, I need to understand how to secure my network configuration for Azure Cloud HSM so that I can prevent unauthorized access and enhance overall security.
---

# Network security for Azure Cloud HSM

Ensuring robust network security is crucial when you're using Azure Cloud HSM. This article provides guidance on private endpoint configuration, DNS setup, required ports, network security group (NSG) rules, and VM placement to help you prevent unauthorized access and enhance overall security.

## Private endpoint

To enhance the security of your Azure Cloud HSM deployment, use private subnets and private endpoints. They enable secure connections to Azure Cloud HSM through a private link, to facilitate private IP address connectivity from your virtual network.

A private endpoint ensures that traffic between your virtual network and the Azure service traverses the Microsoft backbone network for security and private communication. This proactive approach strengthens your application's security posture by minimizing exposure to potential threats from the internet.

For instructions on creating a private endpoint for Azure Cloud HSM, see [Deploy Azure Cloud HSM by using the Azure portal](quickstart-portal.md#set-up-networking) or [Deploy Azure Cloud HSM by using Azure PowerShell](quickstart-powershell.md#set-up-networking).

## Private DNS

When you create a private endpoint for Azure Cloud HSM, the service assigns each HSM node a private DNS hostname in the following format:

```
hsm1.chsm-<resource-name>-<unique-string>.privatelink.cloudhsm.azure.net
```

To resolve these hostnames from your virtual network, configure a private DNS zone for `privatelink.cloudhsm.azure.net` and link it to the virtual networks that contain your admin and application VMs. When you create a private endpoint through the Azure portal, you can enable automatic private DNS zone integration on the **DNS** tab.

> [!NOTE]
> Your admin VM and application VMs must be able to resolve the HSM hostname. If DNS resolution fails, the `azcloudhsm_client` and `azcloudhsm_mgmt_util` tools can't connect to the HSM cluster. Verify DNS resolution by running `nslookup <hsm-hostname>` from your VM before you run HSM tools.

## Required ports

Azure Cloud HSM uses the following ports. Ensure that your NSG rules and any firewalls allow traffic on these ports from your VM subnets to the private endpoint.

| Port | Direction | Tool | Purpose |
|------|-----------|------|---------|
| 2224 | Outbound (VM → HSM) | `azcloudhsm_client` | Client front-end (TCP over TLS) |
| 2225 | Outbound (VM → HSM) | `azcloudhsm_client` | Client front-end (TCP over TLS) |
| 443 | Outbound (VM → HSM) | `azcloudhsm_client` | Client requests |
| 444 | Outbound (VM → HSM) | `azcloudhsm_mgmt_util` | Management utility requests |
| 445 | Internal | Service | Server-to-server (HSM node) communication |

## NSG rules

When you deploy your admin and application VMs in a subnet with an NSG, add outbound rules to allow traffic to your HSM private endpoint on the required ports. For example:

| Priority | Name | Source | Destination | Port | Protocol | Action |
|----------|------|--------|-------------|------|----------|--------|
| 100 | Allow-HSM-Client | VirtualNetwork | `<private-endpoint-IP>` | 2224, 2225 | TCP | Allow |
| 110 | Allow-HSM-Mgmt | VirtualNetwork | `<private-endpoint-IP>` | 443, 444 | TCP | Allow |

Replace `<private-endpoint-IP>` with the private IP address assigned to your Cloud HSM private endpoint, or use the subnet address range that contains the private endpoint.

## VM placement

Azure Cloud HSM uses a client-daemon architecture. The `azcloudhsm_client` daemon must run on the same VM as your application, because the application communicates with the daemon over a local socket. This means:

- **Admin VM**: used for initial HSM setup, user management, key management, and backup operations. Runs `azcloudhsm_mgmt_util` and `azcloudhsm_util`.
- **Application VMs**: run your workloads (for example, Apache, NGINX, SQL Server, or custom apps) and must each run their own instance of `azcloudhsm_client`.

Both VM types must reside in a virtual network that has connectivity to the Cloud HSM private endpoint.

> [!TIP]
> We recommend that you place Cloud HSM resources in a separate resource group from the client VMs and virtual network. This improves management and security isolation. For details, see [Deploy Azure Cloud HSM by using the Azure portal](quickstart-portal.md#configure-basic-settings).

## On-premises connectivity

If your signing servers or applications run on-premises, you can connect them to Azure Cloud HSM by using Azure VPN Gateway:

- **Site-to-site VPN**: recommended for persistent on-premises connectivity. For setup instructions, see [Tutorial: Create a site-to-site VPN connection in the Azure portal](/azure/vpn-gateway/tutorial-site-to-site-portal).
- **Point-to-site VPN**: suitable for individual workstations or smaller deployments.

In both cases, the on-premises host must run the `azcloudhsm_client` daemon and must be able to reach the HSM private endpoint IP addresses over the VPN tunnel.

## Related content

- [Secure your Azure Cloud HSM deployment](secure-cloud-hsm.md)
- [Deploy Azure Cloud HSM by using the Azure portal](quickstart-portal.md)
- [Deploy Azure Cloud HSM by using Azure PowerShell](quickstart-powershell.md)
- [Key management in Azure Cloud HSM](key-management.md)
- [User management in Azure Cloud HSM](user-management.md)
- [Authentication in Azure Cloud HSM](authentication.md)
- [Overview of Azure Cloud HSM](overview.md)
