---
title: Manage user data
description: Learn how to manage the user data in Microsoft Defender for Cloud. Managing user data includes the ability to access, delete, or export data.
ms.topic: concept-article
ms.date: 05/18/2025
---

# Manage user data in Microsoft Defender for Cloud

This article provides information about how you can manage the user data in Microsoft Defender for Cloud. Managing user data includes the ability to access, delete, or export data.

[!INCLUDE [gdpr-intro-sentence.md](~/reusable-content/ce-skilling/azure/includes/gdpr-intro-sentence.md)]

A Defender for Cloud user assigned the role of Reader, Owner, Contributor, or Account Administrator can access customer data within the tool. To learn more about the Account Administrator role, see [Built-in roles for Azure role-based access control](/azure/role-based-access-control/built-in-roles) to learn more about the Reader, Owner, and Contributor roles. See [Azure subscription administrators](/azure/cost-management-billing/manage/add-change-subscription-administrator).

## Search for and identify personal data

A Defender for Cloud user can view their personal data through the Azure portal. Defender for Cloud only stores security contact details such as email addresses and phone numbers. For more information, see [Provide security contact details in Microsoft Defender for Cloud](configure-email-notifications.md).

In the Azure portal, a user can view allowed IP configurations using Defender for Cloud's just-in-time VM access feature. For more information, see [Manage virtual machine access using just-in-time](enable-just-in-time-access.md
).

In the Azure portal, a user can view security alerts provided by Defender for Cloud including IP addresses and attacker details. For more information, see [Managing and responding to security alerts in Microsoft Defender for Cloud](managing-and-responding-alerts.yml).

## Classify personal data

You don't need to classify personal data found in Defender for Cloud's security contact feature. The data saved is an email address (or multiple email addresses) and a phone number. [Contact data](configure-email-notifications.md) is validated by Defender for Cloud.

You don't need to classify the IP addresses and port numbers saved by Defender for Cloud's [just-in-time](enable-just-in-time-access.md
) feature.

Only a user assigned the role of Administrator can classify personal data by [viewing alerts](managing-and-responding-alerts.yml) in Defender for Cloud.

## Secure and control access to personal data

A Defender for Cloud user assigned the role of Reader, Owner, Contributor, or Account Administrator can access [security contact data](configure-email-notifications.md).

A Defender for Cloud user assigned the role of Reader, Owner, Contributor, or Account Administrator can access their [just-in-time](enable-just-in-time-access.md
) policies.

A Defender for Cloud user assigned the role of Reader, Owner, Contributor, or Account Administrator can view their [alerts](managing-and-responding-alerts.yml).

## Update personal data

A Defender for Cloud user assigned the role of Owner, Contributor, or Account Administrator can update [security contact data](configure-email-notifications.md) via the Azure portal.

A Defender for Cloud user assigned the role of Owner, Contributor, or Account Administrator can update their [just-in-time policies](enable-just-in-time-access.md
).

An Account Administrator can't edit alert incidents. An [alert incident](managing-and-responding-alerts.yml) is considered security data and is read only.

## Delete personal data

A Defender for Cloud user assigned the role of Owner, Contributor, or Account Administrator can delete [security contact data](configure-email-notifications.md) via the Azure portal.

A Defender for Cloud user assigned the role of Owner, Contributor, or Account Administrator can delete the [just-in-time policies](enable-just-in-time-access.md
) via the Azure portal.

A Defender for Cloud user can't delete alert incidents. For security reasons, an [alert incident](managing-and-responding-alerts.yml) is considered read-only data.

## Export personal data

A Defender for Cloud user assigned the role of Reader, Owner, Contributor, or Account Administrator can export [security contact data](configure-email-notifications.md) by:

- Copying from the Azure portal
- Executing the Azure REST API call, GET HTTP:

  ```HTTP
  GET https://<endpoint>/subscriptions/{subscriptionId}/providers/Microsoft.Security/securityContacts?api-version={api-version}
  ```

A Defender for Cloud user assigned the role of Account Administrator can export the [just-in-time policies](enable-just-in-time-access.md
) containing the IP addresses by:

- Copying from the Azure portal
- Executing the Azure REST API call, GET HTTP:

  ```HTTP
  GET https://<endpoint>/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Security/locations/{location}/jitNetworkAccessPolicies/default?api-version={api-version}
  ```

An Account Administrator can export the alert details by:

- Copying from the Azure portal
- Executing the Azure REST API call, GET HTTP:

  ```HTTP
  GET https://<endpoint>/subscriptions/{subscriptionId}/providers/microsoft.Security/alerts?api-version={api-version}
  ```

For more information, see [Get Security Alerts (GET Collection)](/previous-versions/azure/reference/mt704050(v=azure.100)).

## Restrict the use of personal data for profiling or marketing without consent

A Defender for Cloud user can choose to opt out by deleting their [security contact data](configure-email-notifications.md).

[Just-in-time data](enable-just-in-time-access.md
) is considered non-identifiable data and is retained for 30 days.

[Alert data](managing-and-responding-alerts.yml) is considered security data and is retained for two years.

## Auditing and reporting

Audit logs of security contact, just-in-time, and alert updates are maintained in [Azure Activity Logs](/azure/azure-monitor/essentials/platform-logs-overview).

## Respond to data subject export requests for Defender for APIs

The right of data portability allows data subjects to request a copy of their personal data in a structured, common, electronic format that can be transmitted to another data controller.

### Manage export and view requests

You can manage requests to export customer or user data.

#### Export customer data (Tenant administrator only)

As a tenant administrator, you have the ability to export customer data.

**To export customer data**:

1. Send an email to `D4APIS_DSRRequests@microsoft.com` that specifies the customer’s email address in the request.
1. The Defender for APIs team will respond with an email to the registered tenant's administrator email address that will ask for confirmation to export the data.
1. Acknowledge the confirmation to export the data for the requested customer. The exported data will be sent to the tenant administrator's email address.

## Next steps

[What is Microsoft Defender for Cloud?](defender-for-cloud-introduction.md)
