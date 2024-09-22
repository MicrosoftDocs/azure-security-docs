---
title: Cross-tenant management 
description: Learn how to set up cross-tenant management to manage the security posture of multiple tenants in Defender for Cloud using Azure Lighthouse.
ms.topic: concept-article
ms.author: dacurwin
author: dcurwin
ms.date: 05/30/2024
---

# Cross-tenant management in Defender for Cloud

Cross-tenant management enables you to view and manage the security posture of multiple tenants in Defender for Cloud by using [Azure Lighthouse](/azure/lighthouse/overview). Manage multiple tenants efficiently, from a single view, without having to sign in to each tenant's directory.

- Service providers can manage the security posture of resources, for multiple customers, from within their own tenant.

- Security teams of organizations with multiple tenants can view and manage their security posture from a single location.

## Set up cross-tenant management

[Azure delegated resource management](/azure/lighthouse/concepts/architecture) is one of the key components of Azure Lighthouse. Set up cross-tenant management by delegating access to resources of managed tenants to your own tenant using these instructions from Azure Lighthouse's documentation: [Onboard a customer to Azure Lighthouse](/azure/lighthouse/how-to/onboard-customer).

## How cross-tenant management works in Defender for Cloud

You're able to review and manage subscriptions across multiple tenants in the same way that you manage multiple subscriptions in a single tenant.

From the top menu bar, select the filter icon, and select the subscriptions, from each tenant's directory, you'd like to view.

:::image type="content" source="media/cross-tenant-management/cross-tenant-filter.png" alt-text="Screenshot that shows where the cross tenant filter button is located.":::

The views and actions are basically the same. Here are some examples:

- **Manage security policies**: From one view, manage the security posture of many resources with [policies](tutorial-security-policy.md), take actions with security recommendations, and collect and manage security-related data.
- **Improve Secure Score and compliance posture**: Cross-tenant visibility enables you to view the overall security posture of all your tenants and where and how to best improve the [secure score](secure-score-security-controls.md) and [compliance posture](regulatory-compliance-dashboard.md) for each of them.
- **Remediate recommendations**: Monitor and remediate a [recommendation](review-security-recommendations.md) for many resources from various tenants at one time. You can then immediately tackle the vulnerabilities that present the highest risk across all tenants.
- **Manage Alerts**: Detect [alerts](alerts-overview.md) throughout the different tenants. Take action on resources that are out of compliance with actionable [remediation steps](managing-and-responding-alerts.yml).

- **Manage advanced cloud defense features and more**: Manage the various threat protection services, such as [just-in-time (JIT) VM access](just-in-time-access-usage.yml).

## Next steps

This article explains how cross-tenant management works in Defender for Cloud. To discover how Azure Lighthouse can simplify cross-tenant management within an enterprise that uses multiple Microsoft Entra tenants, see [Azure Lighthouse in enterprise scenarios](/azure/lighthouse/concepts/enterprise).
