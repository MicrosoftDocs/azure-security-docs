---
title: Troubleshoot Dedicated HSM - Azure Dedicated HSM | Microsoft Docs
description: Overview of Azure Dedicated HSM, which provides key storage capabilities within Azure that meets FIPS 140-2 Level 3 certification
services: dedicated-hsm
author: msmbaldwin
ms.topic: how-to
ms.service: azure-dedicated-hsm
ms.date: 04/14/2025
ms.author: mbaldwin
#Customer intent: As an IT Pro, Decision maker I am looking for key storage capability within Azure Cloud that meets FIPS 140-2 Level 3 certification and that gives me exclusive access to the hardware.
---

# Troubleshooting the Azure Dedicated HSM service

The Azure Dedicated HSM service has two distinct facets. Firstly, the registration and deployment in Azure of the HSM devices with their underlying network components. Secondly, the configuration of the HSM devices in preparation for use/integration with a given workload or application. Although the [Thales Luna 7 HSM](https://cpl.thalesgroup.com/encryption/hardware-security-modules/network-hsms) devices are the same in Azure as you would purchase directly from Thales, the fact they are a resource in Azure creates some unique considerations. These considerations and any resulting troubleshooting insights or best practices are documented here to ensure high visibility and access to critical information. Once the service is in use, definitive information is available via support requests to either Microsoft or Thales directly. 

> [!NOTE]
> It should be noted that prior to performing any configuration on a newly deployed HSM device, it should be updated with any relevant patches. A specific required patch is [KB0019789](https://supportportal.gemalto.com/csm?id=kb_article_view&sys_kb_id=19a81c8bdb9a1fc8d298728dae96197d&sysparm_article=KB0019789) in Thales support portal which addresses a issue where the system becomes unresponsive during reboot.

## HSM Registration

Dedicated HSM is a valuable resource that delivers baremetal HSM capabilities in Azure, and is not freely available for use. To ensure proper utilization, we employ an allowlisting process to approve Azure subscriptions for onboarding and deployment. If you wish to proceed with onboarding to Dedicated HSM, contact your Microsoft Account Manager for further guidance.

### Getting access to Dedicated HSM

First, verify that your use cases cannot be addressed by [Azure Key Vault](../key-vault/general/overview.md) or [Azure Managed HSM](../key-vault/managed-hsm/overview.md). If then you believe only Dedicated HSM meets your key storage requirements, then contact your Microsoft Account Manager or Microsoft Customer Support for further guidance to request access. Outline your application and use cases, the regions you would like HSMs and the volume of HSMs you desire.

## HSM Provisioning

Provisioning an HSM device in Azure can be done via either CLI or PowerShell. When you register for the service, a sample ARM template is provided and assistance is given for initial customization.

### HSM Deployment Failure Information

Dedicated HSM supports CLI and PowerShell for deployment so portal based error information is limited and not verbose. Better information can be found by using the Resource Explorer. The portal home page has an icon for this and more detailed error information is available. This information helps a lot if pasted in when creating a support request related to deployment issues.

![Failure Information](./media/troubleshoot/failure-information.png)

### HSM Subnet Delegation
The number one reason for deployment failures is forgetting to set the appropriate delegation for the customer defined subnet on which the HSMs are provisioned. Setting that delegation is part of the virtual network and subnet prerequisites for deployment and more details can be found in the tutorials.

![Subnet Delegation](./media/troubleshoot/subnet-delegation.png)

### HSM Deployment Race Condition

The standard ARM template provided for deployment has HSM and [ExpressRoute gateway](/azure/expressroute/expressroute-howto-add-gateway-portal-resource-manager) related resources. Networking resources are a dependency for successful HSM deployment and timing can be crucial. Occasionally, we see deployment failures related to dependency issues and rerunning the deployment often solves the issue. If not, deleting resources and then redeploying is often successful. After attempting this and still finding issue, raise a support request in the Azure portal selecting the problem type of "Issues configuring the Azure setup".

### HSM Deployment Using Terraform

A few customers used Terraform as an automation environment instead of the ARM templates supplied when registering for this service. The HSMs cannot be deployed this way but the dependent networking resources can. Terraform has a module to call out to a minimal ARM template that just has the HSM deployment. In this situation, care should be taken to ensure networking resources such as the required [ExpressRoute gateway](/azure/expressroute/expressroute-howto-add-gateway-portal-resource-manager) are fully deployed before deploying HSMs. The following CLI command can be used to test for completed deployment and integrated as required. Replace the angle bracket place holders for your specific naming. You should look for a result of "provisioningState is Succeeded"

```azurecli
az resource show --ids /subscriptions/<subid>/resourceGroups/<myresourcegroup>/providers/Microsoft.Network/virtualNetworkGateways/<myergateway>
```

### Deployment failure based on quota

Dedicated HSM has an initial quota limit of 2 HSMs per stamp and 4 HSMs per region. Because deployments may fail if these limits are exceeded, you should delete resources from previous failed deployments before attempting new ones. You can check existing resources by referring to the [How do I see HSMs when provisioned](#how-do-i-see-hsms-when-provisioned). If you require more than 4 HSMs in a single region, submit a customer support ticket to request an increase in your quota limit.

### Deployment failure based on capacity

When a stamp or region is nearing capacity, with most of the available HSMs provisioned, deployment failures can occur. Each stamp provides 12 HSMs for customer use, totaling 24 per region, with two spares and one test device per stamp. If you suspect you reached this limit, submit a customer support ticket to inquire about available capacity in the region or the fill level of specific stamps.

### How do I see HSMs when provisioned?

Due to Dedicated HSM being an allowlisted service, it is considered a "Hidden Type" in the Azure portal. To see the HSM resources, you must check the "Show hidden types" check box. The NIC resource always follows the HSM and is a good place to find out the IP address of the HSM before using SSH to connect.

![Screenshot that highlights the Show hidden types check](./media/troubleshoot/hsm-provisioned.png)

## Networking Resources

Deployment of Dedicated HSM has a dependency on networking resources and some consequential limitations to be aware of.

### Provisioning ExpressRoute

Dedicated HSM uses [ExpressRoute gateway](/azure/expressroute/expressroute-howto-add-gateway-portal-resource-manager) as a "tunnel" for communication between the customer private IP address space and the physical HSM in an Azure datacenter. Considering there is a restriction of one gateway per virtual network, customers requiring connection to their on-premises resources via ExpressRoute must use another virtual network for that connection.  

### HSM Private IP Address

The sample templates provided for Dedicated HSM assume that the HSM IP is automatically taken from a given subnet range. You can specify an explicit IP address for the HSM via a "NetworkInterfaces" attribute in the ARM template.

![Screenshot that shows the sample template for Dedicated HSM.](./media/troubleshoot/private-ip-address.png)

## HSM Initialization

Initialization prepares a new HSM for use, or an existing HSM for reuse. Initialization of the HSM must be complete before you can generate or store objects, allow clients to connect, or perform cryptographic operations.

### Lost Credentials

Loss of the Shell administrator password will result in loss of HSM key material. A support request should be made to reset the HSM.
When initializing the HSM, securely store credentials. Shell and HSM credentials should be kept in accordance with your company's policies.

### Failed Logins

Providing incorrect credentials to HSMs can have destructive consequences. The following are default behaviors for HSM Roles.

| Role | Threshold (# of tries) | Result of too many bad sign in attempts | Recovery |
|--|--|--|--|
| HSM SO | 3 |  HSM is zeroized (all HSM objects identities, and all partitions are gone)  |  HSM must be reinitialized. Contents can be restored from backups. | 
| Partition SO | 10 |  Partition is zeroized. |  Partition must be reinitialized. Contents can be restored from backup. |  
| Audit | 10 | Lockout | Unlocked automatically after 10 minutes. |  
| Crypto Officer | 10 (can be decreased) | If HSM policy 15: Enable SO reset of partition PIN is set to 1 (enabled), the CO and CU roles are locked out.<br>If HSM policy 15: Enable SO reset of partition PIN is set to 0 (disabled), the CO and CU roles are permanently locked out and the partition contents are no longer accessible. Default setting. | CO role must be unlocked and the credential reset by the Partition SO, using `role resetpw -name co`.<br>The partition must be reinitialized, and key material restored from a backup device. |  

## HSM Configuration

The following items are situation where configuration errors are either common or have an impact that is worthy of calling out:

### HSM Documentation and Software
Software and documentation for the [Thales Luna 7 HSM](https://cpl.thalesgroup.com/encryption/hardware-security-modules/network-hsms) devices is not available from Microsoft and must be downloaded from Thales directly. Registration is required using the Thales Customer ID received during the registration process. The devices, as provided by Microsoft, have software version 7.2 and firmware version 7.0.3. Early in 2020 Thales made documentation public and it can be found [here](https://thalesdocs.com/gphsm/luna/7.2/docs/network/Content/Home_network.htm).  

### HSM Networking Configuration

Be careful when configuring the networking within the HSM. The HSM has a connection via the [ExpressRoute gateway](/azure/expressroute/expressroute-howto-add-gateway-portal-resource-manager) from a customer private IP address space directly to the HSM. This communication channel is for customer communication only and Microsoft has no access. If the HSM is configured in such a way that this network path is impacted, that means all communication with the HSM is removed.  In this situation, the only option is to raise a Microsoft support request via the Azure portal to have the device reset. This reset procedure sets the HSM back to its initial state and all configuration and key material is lost.  Configuration must be recreated and when the device joins the HA group the key material is replicated.  

### HSM Device Reboot

Some configuration changes require the HSM to be power cycled or rebooted. Microsoft testing of the HSM in Azure determined that on some occasions the reboot could stop responding. The implication is that a support request must be created in the Azure portal requesting hard-reboot and that could take up to 48 hours to complete considering it's a manual process in an Azure datacenter. To avoid this situation, ensure you deployed the reboot patch available from Thales directly. Refer to [KB0019789](https://supportportal.gemalto.com/csm?sys_kb_id=d66911e2db4ffbc0d298728dae9619b0&id=kb_article_view&sysparm_rank=1&sysparm_tsqueryId=d568c35bdb9a4850d6b31f3b4b96199e&sysparm_article=KB0019789) in the Thales Luna 7 HSM 7.2 Downloads for a recommended patch for an issue where the system becomes unresponsive during reboot (Note: you must register in the [Thales customer support portal](https://supportportal.thalesgroup.com/csm) to download.)

### NTLS Certificates out of sync

A client may lose connectivity to an HSM when a certificate expires or was overwritten through configuration updates. The certificate exchange client configuration should be reapplied with each HSM.
Example NTLS logging with invalid certificate:

> NTLS[8508]: info : 0 : Incoming connection request... : 192.168.50.2/59415
> NTLS[8508]: Error message from SSLAccept is : error:14094418:SSL routines:ssl3_read_bytes:tlsv1 alert unknown ca
> NTLS[8508]: Error during SSL accept ( RC_SSL_ERROR )
> NTLS[8508]: info : 0xc0000711 : Fail to establish a secure channel with client : 192.168.50.2/59415 : RC_SSL_FAILED_HANDSHAKE
> NTLS[8508]: info : 0 : NTLS Client "Unknown host name" Connection instance removed : 192.168.50.2/59415

### Failed TCP Communication

Communication from the Luna Client installation to the HSM requires at a minimum TCP port 1792. Consider this as any network configurations are changed in the environment.

### Failed HA Group Member Doesn't Recover

If a failed HA Group member doesn't recover, it must be manually recovered from the Luna client using the command hagroup recover.
It is necessary to configure a retry count for an HA group to enable auto recover. By default an HA group will not attempt to recover an HA member into the group when it recovers.

### HA Group Doesn't Sync

In the case where member partitions do not have the same cloning domain, the ha synchronize command will display the following:
Warning: Synchronize may fail.  The members in slot 0 and slot 1 have conflicting settings for private key cloning.
A new partition with the correct cloning domain should be added to the HA group, followed by removing the incorrectly configured partition.

## HSM Deprovisioning

Only when fully finished with an HSM can it be deprovisioned and then Microsoft will reset it and return it to a free pool. 

### How to delete an HSM resource

**DO NOT DELETE the Resource Group of your Dedicated HSM directly. It will not delete the HSM resource, you will continue to be billed as it places the HSM into a orphaned state. If did not follow correct procedures and end up in this situation, contact Microsoft Support.**

**Step 1:** Zeroize the HSM. The Azure resource for an HSM cannot be deleted unless the HSM is in a "zeroized" state. Hence, all key material must have been deleted prior to trying to delete it as a resource. The quickest way to zeroize is to get the HSM admin password wrong 3 times (note: this refers to the HSM admin and not appliance level admin). Use command 'hsm login' and enter wrong password three times. The Luna shell does have a hsm -factoryreset command that zeroizes the HSM but it can only be executed via console on the serial port and customers do not have access to this.

**Step 2:** Once HSM is zeroized, you can use either of the following commands to initiate the Delete Dedicated HSM resource
> **Azure CLI**: az dedicated-hsm delete --resource-group \<RG name\> –-name \<HSM name\> <br />
> **Azure PowerShell**: Remove-AzDedicatedHsm -Name \<HSM name\> -ResourceGroupName \<RG name\>

**Step 3:** Once **Step 2** is successful, you can delete the resource group to delete the other resources associated with the dedicated HSM by using either Azure CLI or Azure PowerShell.
> **Azure CLI**: az group delete --name \<RG name\> <br />
> **Azure PowerShell**: Remove-AzResourceGroup -Name \<RG name\>

## Next steps

This article has provided insight into areas across the HSM deployment lifecycle that may have issues or require troubleshooting or careful consideration. Hopefully this article helps you avoid unnecessary delays and frustration, and if you have relevant additions or changes, then raise a support request with Microsoft and let us know.
