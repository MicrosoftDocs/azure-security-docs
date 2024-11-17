---
title: Support and prerequisites for deploying the Defender for APIs plan
description: Learn about the requirements for Defender for APIs deployment in Microsoft Defender for Cloud
author: dcurwin
ms.author: dacurwin
ms.service: defender-for-cloud
ms.topic: conceptual
ms.date: 12/03/2023
ms.custom: references_regions
---
# Support and prerequisites for Defender for APIs deployment

Review the requirements on this page before setting up [Microsoft Defender for APIs](defender-for-apis-introduction.md).

## Cloud and region support

Defender for APIs is available in the Azure commercial cloud, in these regions:

- Asia (Southeast Asia, EastAsia)
- Australia (Australia East, Australia Southeast, Australia Central, Australia Central 2)
- Brazil (Brazil South, Brazil Southeast)
- Canada (Canada Central, Canada East)
- Europe (West Europe, North Europe)
- India (Central India, South India, West India)
- Japan (Japan East, Japan West)
- UK (UK South, UK West)
- US (East US, East US 2, West US, West US 2, West US 3, Central US, North Central US, South Central US, West Central US, East US 2 EUAP, Central US EUAP)

Review the latest cloud support information for Defender for Cloud plans and features in the [cloud support matrix](support-matrix-cloud-environment.md).

## API support

**Feature** | **Supported**
--- | ---
Availability | This feature is available in the Premium, Standard, Basic, and Developer tiers of Azure API Management.
API gateways | Azure API Management<br/><br/> Defender for APIs currently doesn't onboard APIs that are exposed using the API Management [self-hosted gateway](/azure/api-management/self-hosted-gateway-overview), or managed using API Management [workspaces](/azure/api-management/workspaces-overview).
API types | Currently, Defender for APIs discovers and analyzes REST APIs.

## Defender CSPM integration

To explore API security risks using Cloud Security Explorer, the Defender Cloud Security Posture Management (CSPM) plan must be enabled. [Learn more](concept-cloud-security-posture-management.md).

## Onboarding requirements

Onboarding requirements for Defender for APIs are as follows.

**Requirement** | **Details**
--- | ---
API Management instance | At least one API Management instance in an Azure subscription. Defender for APIs is enabled at the level of a subscription.<br/><br/> One or more supported APIs must be imported to the API Management instance.
Azure account | You need an Azure account to sign in to the Azure portal.
Onboarding permissions | To enable and onboard Defender for APIs, you will need [API Management Service Contributor](/azure/api-management/api-management-role-based-access-control#built-in-service-roles) role access, along with the permissions outlined in the [User roles and permissions](permissions.md#roles-and-allowed-actions) for enabling Microsoft Defender plans.
Onboarding location | You can [enable Defender for APIs in the Defender for Cloud portal](defender-for-apis-deploy.md), or in the [Azure API Management portal](/azure/api-management/protect-with-defender-for-apis).

## Next steps

[Enable and onboard](defender-for-apis-deploy.md) Defender for APIs.
