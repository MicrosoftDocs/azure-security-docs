---
title: SQL information protection policy
description: Learn how to customize information protection policies in Microsoft Defender for Cloud to secure your data effectively and meet compliance requirements.
ms.topic: how-to
ms.custom: devx-track-azurepowershell
author: dcurwin
ms.author: dacurwin
ms.date: 07/15/2025
#customer intent: As a user, I want to learn how to customize information protection policies in Microsoft Defender for Cloud so that I can secure my data effectively.
---

# SQL information protection policy in Microsoft Defender for Cloud

SQL information protection's [data discovery and classification mechanism](/azure/azure-sql/database/data-discovery-and-classification-overview) provides advanced capabilities for discovering, classifying, labeling, and reporting the sensitive data in your databases. It's built into [Azure SQL Database](/azure/azure-sql/database/sql-database-paas-overview), [Azure SQL Managed Instance](/azure/azure-sql/managed-instance/sql-managed-instance-paas-overview), and [Azure Synapse Analytics](/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is).

The classification mechanism is based on the following two elements:

- **Labels** – The main classification attributes, used to define the *sensitivity level of the data* stored in the column.
- **Information Types** – Provides additional granularity into the *type of data* stored in the column.

The information protection policy options within Defender for Cloud provide a predefined set of labels and information types that serve as the defaults for the classification engine. You can customize the policy, according to your organization's needs, as described below.

:::image type="content" source="./media/sql-information-protection-policy/sql-information-protection-policy-page.png" alt-text="The page showing your SQL information protection policy." lightbox="media/sql-information-protection-policy/sql-information-protection-policy-page.png":::

## How do I access the SQL information protection policy?

There are three ways to access the information protection policy:

- **(Recommended)** From the **Environment settings** page of Defender for Cloud
- From the security recommendation *Sensitive data in your SQL databases should be classified*
- From the Azure SQL DB data discovery page

Each of these is shown in the relevant tab below.

### [**From Defender for Cloud's settings**](#tab/sqlip-tenant)

<a name="sqlip-tenant"></a>

### Access the policy from Defender for Cloud's environment settings page

From Defender for Cloud's **Environment settings** page, select **SQL information protection**.

> [!NOTE]
> This option only appears for users with tenant-level permissions. [Grant tenant-wide permissions to yourself](tenant-wide-permissions-management.md#grant-tenant-wide-permissions-to-yourself).

:::image type="content" source="./media/sql-information-protection-policy/environment-settings-link-to-information-protection.png" alt-text="Accessing the SQL Information Protection policy from the environment settings page of Microsoft Defender for Cloud." lightbox="media/sql-information-protection-policy/environment-settings-link-to-information-protection.png":::

### [**From Defender for Cloud's recommendation**](#tab/sqlip-db)

<a name="sqlip-db"></a>

### Access the policy from the Defender for Cloud recommendation

Use Defender for Cloud's recommendation, *Sensitive data in your SQL databases should be classified*, to view the data discovery and classification page for your database. There, you'll also see the columns discovered to contain information that we recommend you classify.

1. From Defender for Cloud's **Recommendations** page, search for the recommendation **Sensitive data in your SQL databases should be classified**.

    :::image type="content" source="./media/sql-information-protection-policy/sql-sensitive-data-recommendation.png" alt-text="Finding the recommendation that provides access to the SQL information protection policies." lightbox="media/sql-information-protection-policy/sql-sensitive-data-recommendation.png":::

1. From the recommendation details page, select a database from the **healthy** or **unhealthy** tabs.

1. The **Data Discovery & Classification** page opens. Select **Configure**.

    :::image type="content" source="./media/sql-information-protection-policy/access-policy-from-security-center-recommendation.png" alt-text="Opening the SQL information protection policy from the relevant recommendation in Microsoft Defender for Cloud's" lightbox="media/sql-information-protection-policy/access-policy-from-security-center-recommendation.png":::

### [**From Azure SQL**](#tab/sqlip-azuresql)

<a name="sqlip-azuresql"></a>

### Access the policy from Azure SQL

1. From the Azure portal, open Azure SQL.

    :::image type="content" source="./media/sql-information-protection-policy/open-azure-sql.png" alt-text="Opening Azure SQL from the Azure portal." lightbox="media/sql-information-protection-policy/open-azure-sql.png":::

1. Select any database.

1. From the **Security** area of the menu, open the **Data Discovery & Classification** page (1) and select **Configure** (2).

    :::image type="content" source="./media/sql-information-protection-policy/access-policy-from-azure-sql.png" alt-text="Opening the SQL information protection policy from Azure SQL." lightbox="media/sql-information-protection-policy/access-policy-from-azure-sql.png":::

---

## Customize your information types

To manage and customize information types:

1. Select **Manage information types**.

    :::image type="content" source="./media/sql-information-protection-policy/manage-types.png" alt-text="Manage information types for your information protection policy." lightbox="media/sql-information-protection-policy/manage-types.png":::

1. To add a new type, select **Create information type**. You can configure a name, description, and search pattern strings for the information type. Search pattern strings can optionally use keywords with wildcard characters (using the character '%'), which the automated discovery engine uses to identify sensitive data in your databases, based on the columns' metadata.

    :::image type="content" source="./media/sql-information-protection-policy/configure-new-type.png" alt-text="Configure a new information type for your information protection policy." lightbox="media/sql-information-protection-policy/configure-new-type.png":::

1. You can also modify the built-in types by adding additional search pattern strings, disabling some of the existing strings, or by changing the description.

    > [!TIP]
    > You can't delete built-in types or change their names.

1. **Information types** are listed in order of ascending discovery ranking, meaning that the types higher in the list attempt to match first. To change the ranking between information types, drag the types to the right spot in the table, or use the **Move up** and **Move down** buttons to change the order.

1. Select **OK** when you're done.

1. After you completed managing your information types, be sure to associate the relevant types with the relevant labels, by selecting **Configure** for a particular label, and adding or deleting information types as appropriate.

1. To apply your changes, select **Save** in the main **Labels** page.

## Export and import a policy

You can download a JSON file with your defined labels and information types, edit the file in the editor of your choice, and then import the updated file.

:::image type="content" source="./media/sql-information-protection-policy/export-import.png" alt-text="Exporting and importing your information protection policy." lightbox="media/sql-information-protection-policy/export-import.png":::

> [!NOTE]
> You need tenant level permissions to import a policy file.

## Permissions

To customize the information protection policy for your Azure tenant, you need the following actions on the tenant's root management group:

- Microsoft.Security/informationProtectionPolicies/read
- Microsoft.Security/informationProtectionPolicies/write

Learn more in [Grant and request tenant-wide visibility](tenant-wide-permissions-management.md).

## Manage SQL information protection using Azure PowerShell

- [Get-AzSqlInformationProtectionPolicy](/powershell/module/az.security/get-azsqlinformationprotectionpolicy): Retrieves the effective tenant SQL information protection policy.
- [Set-AzSqlInformationProtectionPolicy](/powershell/module/az.security/set-azsqlinformationprotectionpolicy): Sets the effective tenant SQL information protection policy.

## Related articles

- [Azure SQL Database Data Discovery and Classification](/azure/azure-sql/database/data-discovery-and-classification-overview)

- [Microsoft Defender for Cloud data security](data-security.md)

## Next step

> [!div class="nextstepaction"]
> [Setting security policies in Microsoft Defender for Cloud](tutorial-security-policy.md)
