---
title: Supportability - Azure Dedicated HSM | Microsoft Docs
description: Support options and areas of responsibility for Azure Dedicated HSM in different scenarios
services: dedicated-hsm
author: johndaw
ms.topic: conceptual
ms.service: azure-dedicated-hsm
ms.date: 02/20/2024
ms.author: mbaldwin
---

# Azure Dedicated HSM supportability

The Azure Dedicated HSM service provides a physical device for sole customer use with complete administrative control and management responsibility. The device is a [Thales Luna 7 HSM model A790](https://cpl.thalesgroup.com/encryption/hardware-security-modules/network-hsms). Microsoft has no administrative access once you provision the device, beyond physical serial port attachment as a monitoring role. Without access, Microsoft has no ongoing software-level maintenance or system administration responsibilities. As a result, customers are responsible for typical operational activities.
Customers are fully responsible for applications that use the HSMs and should work with Thales for support or consulting-based assistance. Due to the extent of customer ownership of operational hygiene, Microsoft can't offer any kind of high availability guarantee for this service. It's the customer’s responsibility to ensure their applications are correctly configured to achieve high availability. Microsoft monitors and maintains device health and network connectivity.

## Getting support

Customer support for Dedicated HSM is a joint effort between Microsoft and Thales. Microsoft addresses any hardware issues or network path issues. Thales addresses anything to do with the actual HSM, such as configuration, software, firmware, and application development. This support model ensures the quickest route to the most effective support. If you're in doubt about a particular issue, raise a support request with Microsoft and we direct you appropriately. Microsoft stays engaged in all support scenarios and strives for the best support experience for you.

## Thales support

Customers using the Dedicated HSM service qualify for support from Thales as per their Plus Support Plan. This support plan just requires a registration process by using the Thales support portal. A customer ID and instructions are provided for this as part of the initial engagement with Microsoft to gain access to the Dedicated HSM service. You get support from Thales through their [customer support portal](https://supportportal.thalesgroup.com/csm).
A key point of note is that Thales provides all software and documentation required to use the HSM (for example, client access software and SDKs) via download on the customer support portal.

### Software components

You use various software components in the configuration of HSM devices:

* Client software
* SDK
* Tools

### Guidance

Thales makes administration and configuration guidance available via the [Thales customer support portal](https://supportportal.thalesgroup.com/csm). Once signed in by using a valid customer ID, you can download these documents. Thales also provides a series of integration guides to help customers with different scenarios and software integrations. For more information, see the [Thales partner site for Microsoft](https://cpl.thalesgroup.com/partners/overview).

### Support

Address any software-level issue or question in relation to using the HSMs as part of the Dedicated HSM service to Thales support directly. Thales addresses all software components listed earlier, and any custom HSM configuration that is post-provisioning. For more information, see the [Thales customer support portal](https://supportportal.thalesgroup.com/csm).

### Consulting services

For assistance with the design, development, and deployment of custom applications that use the HSM, contact your Thales account representative.

## Microsoft support

Microsoft ensures physical HSM devices are network accessible and in an operational state for the exclusive use of a single customer. Customers are responsible for the configuration, administration, and management of the device. 
Microsoft responsibilities include:

* Making sure that the device has power and cooling
* Maintaining an operational state of the HSM (for example, break/fix scenarios)
* Ensuring the device is accessible over the network.

Report the following issues to Microsoft:

* Component failures
* Full device failure
* Network access problems
* Problems provisioning and deprovisioning.

Microsoft has physical serial port access to the device through a monitoring role (that is a non-administrative role) that enables basic health telemetry.  This access allows Microsoft to provide proactive notification of issues to the customer unless the customer chooses to restrict this permission. 

### Provisioning and decommissioning

After you register and are approved for the Dedicated HSM service, you can create HSM resources (currently via PowerShell or command-line interface and not the Azure portal). The resource goes through an allocation process that maps a physical device in a specified region to a customer’s pre-defined virtual network (VNet). Once visible on a VNet, you can access the device and configure it further as per requirements. You access your dedicated HSMs by using Thales client software and tools. Microsoft supports the resource creation process. Thales supports the custom configuration process and beyond (see Thales support above). When you finish using an HSM, you must reset (or zeroize) the HSM to ensure no persistence of data. The process of resetting the device removes all custom configuration and data. Microsoft deallocates the device and returns it to the pool in a pristine state. This process ensures that when the device is returned to the pool, there's no evidence of previous customer activity. 

### Hardware problems

The HSM device has redundant and replaceable power supplies and fan units. However, fan unit removal still causes a tamper event. When a component failure occurs, Microsoft uses the most appropriate process to address the component-level problem in a way that causes minimal interruption and lowest risk to your service availability.

#### Power supply events

The HSM uses dual-PSU redundancy. Transient single-PSU log messages during datacenter maintenance are expected and don't require action or support tickets. For details, see [Power supply redundancy](monitoring.md#power-supply-redundancy).

Any more serious failure of the device results in replacing that device with a new device from the free pool. To synchronize and return to full operational state, include the new device in the existing HA pair. The failed device has its data-bearing devices removed and shredded on site at the datacenter. 

### Networking problems

If you experience networking access problems to the HSM device, contact Microsoft support. A simple test for networking access is to use SSH to connect to the HSM device. If this test fails, contact Microsoft support.

## Service level expectations for support

For Microsoft support service levels, see the [Azure support plan](https://azure.microsoft.com/support/plans/).
For Thales support service levels, see the [Thales Support Essentials](https://azure.microsoft.com/support/plans/).

## Next steps

Before device provisioning and application design or deployment, make sure you understand key concepts such as high availability and security.

* [Deployment Architecture](deployment-architecture.md)
* [High Availability](high-availability.md)
* [Physical Security](physical-security.md)
* [Networking](networking.md)
