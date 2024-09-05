---
title: Support and prerequisites for data security posture management
description: Learn about the requirements for data security posture management in Microsoft Defender for Cloud.
author: dcurwin
ms.author: dacurwin
ms.service: defender-for-cloud
ms.topic: conceptual
ms.date: 09/03/2024
ms.custom: references_regions
---

# Support and prerequisites for data security posture management

Review the requirements on this page before setting up [data security posture management](concept-data-security-posture.md) in Microsoft Defender for Cloud.

## Enabling sensitive data discovery

Sensitive data discovery is available in the Defender CSPM, Defender for Storage and Defender for Databases plans.

- When you enable one of the plans, the sensitive data discovery extension is turned on as part of the plan.
- If you have existing plans running, the extension is available, but turned off by default.
- Existing plan status shows as “Partial” rather than “Full” if one or more extensions aren't turned on.
- The feature is turned on at the subscription level.
- If sensitive data discovery is turned on, but Defender CSPM isn't enabled, only storage resources will be scanned.
- If a subscription is enabled with Defender CSPM and in parallel you scanned the same resources with Purview, Purview's scan result is ignored and defaults to displaying the Microsoft Defender for Cloud's scanning results for the supported resource type.

## What's supported

The table summarizes availability and supported scenarios for sensitive data discovery.

|**Support** | **Details**|
|--- | ---|
|What Azure data resources can I discover? | **Object storage:**<br /><br />[Block blob](/azure/storage/blobs/storage-blobs-introduction) storage accounts in Azure Storage v1/v2<br/><br/> Azure Data Lake Storage Gen2<br/><br/>Storage accounts behind private networks are supported.<br/><br/>  Storage accounts encrypted with a customer-managed server-side key are supported.<br/><br/> Accounts aren't supported if any of these settings are enabled: Storage account is defined as [Azure DNS Zone](https://techcommunity.microsoft.com/t5/azure-storage-blog/public-preview-create-additional-5000-azure-storage-accounts/ba-p/3465466); The storage account endpoint has a [custom domain mapped to it](/azure/storage/blobs/storage-custom-domain-name).<br /><br /><br />**Databases**<br /><br />Azure SQL Databases </br> <br> Azure SQL Database encrypted with [Transparent data encryption](/azure/azure-sql/database/secure-database-tutorial#transparent-data-encryption)  |
|What AWS data resources can I discover? | **Object storage:**<br /><br />AWS S3 buckets<br/><br/> Defender for Cloud can discover KMS-encrypted data, but not data encrypted with a customer-managed key.<br /><br />**Databases**<br /><br />- Amazon Aurora<br />- Amazon RDS for PostgreSQL<br />- Amazon RDS for MySQL<br />- Amazon RDS for MariaDB<br />- Amazon RDS for SQL Server (noncustom)<br />- Amazon RDS for Oracle Database (noncustom, SE2 Edition only) <br /><br />Prerequisites and limitations: <br />- Automated backups need to be enabled. <br />- The IAM role created for the scanning purposes (DefenderForCloud-DataSecurityPostureDB by default) needs to have permissions to the KMS key used for the encryption of the RDS instance. <br />- You can't share a DB snapshot that uses an option group with permanent or persistent options, except for Oracle DB instances that have the **Timezone** or **OLS** option (or both). [Learn more](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ShareSnapshot.html) |
|What GCP data resources can I discover? | GCP storage buckets<br/> Standard Class<br/> Geo: region, dual region, multi region |
|What permissions do I need for discovery? | Storage account: Subscription Owner<br/> **or**<br/> `Microsoft.Authorization/roleAssignments/*` (read, write, delete) **and** `Microsoft.Security/pricings/*` (read, write, delete) **and** `Microsoft.Security/pricings/SecurityOperators` (read, write)<br/><br/> Amazon S3 buckets  and RDS instances: AWS account permission to run Cloud Formation (to create a role). <br/><br/>GCP storage buckets: Google account permission to run script (to create a role). |
|What file types are supported for sensitive data discovery? | Supported file types (you can't select a subset) - .doc, .docm, .docx, .dot, .gz, .odp, .ods, .odt, .pdf, .pot, .pps, .ppsx, .ppt, .pptm, .pptx, .xlc, .xls, .xlsb, .xlsm, .xlsx, .xlt, .csv, .json, .psv, .ssv, .tsv, .txt., xml, .parquet, .avro, .orc.|
|What Azure regions are supported? | You can discover Azure storage accounts in:<br/><br/> Asia East; Asia South East; Australia Central; Australia Central 2; Australia East; Australia South East; Brazil South; Brazil Southeast; Canada Central; Canada East; Europe North; Europe West; France Central; France South; Germany North; Germany West Central; India Central; India South; Japan East; Japan West; Jio India West; Korea Central; Korea South; Norway East; Norway West; South Africa North; South Africa West; Sweden Central; Switzerland North; Switzerland West; UAE North; UK South; UK West; US Central; US East; US East 2; US North Central; US South Central; US West; US West 2; US West 3; US West Central; <br/><br/> You can discover Azure SQL Databases in any region where Defender CSPM and Azure SQL Databases are supported. |
|What AWS regions are supported? | S3:<br /><br />Asia Pacific (Mumbai); Asia Pacific (Singapore); Asia Pacific (Sydney); Asia Pacific (Tokyo); Canada (Montreal); Europe (Frankfurt); Europe (Ireland); Europe (London); Europe (Paris); Europe (Stockholm); South America (São Paulo); US East (Ohio); US East (N. Virginia); US West (N. California): US West (Oregon).<br/><br/><br />RDS:<br /><br/>Africa (Cape Town); Asia Pacific (Hong Kong SAR); Asia Pacific (Hyderabad); Asia Pacific (Melbourne); Asia Pacific (Mumbai); Asia Pacific (Osaka); Asia Pacific (Seoul); Asia Pacific (Singapore); Asia Pacific (Sydney); Asia Pacific (Tokyo); Canada (Central); Europe (Frankfurt); Europe (Ireland); Europe (London); Europe (Paris); Europe (Stockholm); Europe (Zurich); Middle East (UAE); South America (São Paulo); US East (Ohio); US East (N. Virginia); US West (N. California): US West (Oregon).<br /><br /> Discovery is done locally within the region. |
|What GCP regions are supported? | europe-west1, us-east1, us-west1, us-central1, us-east4, asia-south1, northamerica-northeast1|
|Do I need to install an agent? | No, discovery requires no agent installation. |
|What's the cost? | The feature is included with the Defender CSPM and Defender for Storage plans, and doesn’t incur extra costs except for the respective plan costs. |
|What permissions do I need to view/edit data sensitivity settings? | You need one of these Microsoft Entra roles:<br> <li>Compliance Data Administrator, Compliance Administrator, or higher <br> <li>Security Operator, Security Administrator, or higher|
| What permissions do I need to perform onboarding? | You need one of these [Azure role-based access control (Azure RBAC) roles](/azure/role-based-access-control/role-assignments-portal): Security Admin, Contributor, Owner on the subscription level (where the GCP project/s reside). For consuming the security findings: Security Reader, Security Admin, Reader, Contributor, Owner on the subscription level (where the GCP project/s reside). |

## Configuring data sensitivity settings

The main steps for configuring data sensitivity settings include:

- [Import custom sensitivity info types/labels from Microsoft Purview compliance portal](data-sensitivity-settings.md#import-custom-sensitivity-info-typeslabels)
- [Customize sensitive data categories/types](data-sensitivity-settings.md#customize-sensitive-data-categoriestypes)
- [Set the threshold for sensitivity labels](data-sensitivity-settings.md#set-the-threshold-for-sensitive-data-labels)

[Learn more](/microsoft-365/compliance/create-sensitivity-labels) about sensitivity labels in Microsoft Purview.

## Discovery

Defender for Cloud starts discovering data immediately after enabling a plan, or after turning on the feature in plans that are already running.

For object storage:

- It takes up to 24 hours to see the results for a first-time discovery.
- After files are updated in the discovered resources, data is refreshed within eight days.
- A new Azure storage account that's added to an already discovered subscription is discovered within 24 hours or less.
- A new AWS S3 bucket or GCP storage bucket that's added to an already discovered AWS account or Google account is discovered within 48 hours or less.
- Sensitive data discovery for storage is conducted locally within your region. This ensures that your data doesn't leave your region. Only resource metadata, such as files, blobs, buckets names, detected sensitivity labels, and the names of identified Sensitive Information Types (SITs), are transferred to Defender for Cloud.

For databases:

- Databases are scanned on a weekly basis.
- For newly enabled subscriptions, results appear within 24 hours.

### Discovering AWS S3 buckets

In order to protect AWS resources in Defender for Cloud, you set up an AWS connector, using a CloudFormation template to onboard the AWS account.

- To discover AWS data resources, Defender for Cloud updates the CloudFormation template.
- The CloudFormation template creates a new role in AWS IAM, to allow permission for the Defender for Cloud scanner to access data in the S3 buckets.
- To connect AWS accounts, you need Administrator permissions on the account.
- The role allows these permissions: S3 read only; KMS decrypt.

### Discovering AWS RDS instances

To protect AWS resources in Defender for Cloud, set up an AWS connector using a CloudFormation template to onboard the AWS account.

- To discover AWS RDS instances, Defender for Cloud updates the CloudFormation template.
- The CloudFormation template creates a new role in AWS IAM, to allow permission for the Defender for Cloud scanner to take the last available automated snapshot of your instance and bring it online in an isolated scanning environment within the same AWS region.
- To connect AWS accounts, you need Administrator permissions on the account.
- Automated snapshots need to be enabled on the relevant RDS Instances/Clusters.
- The role allows these permissions (review the CloudFormation template for exact definitions):
  - List all RDS DBs/clusters
  - Copy all DB/cluster snapshots
  - Delete/update DB/cluster snapshot with prefix *defenderfordatabases*
  - List all KMS keys
  - Use all KMS keys only for RDS on source account
  - Create & full control on all KMS keys with tag prefix *DefenderForDatabases*
  - Create alias for KMS keys
- KMS keys are created once for each region that contains RDS instances. The creation of a KMS key might incur a minimal extra cost, according to AWS KMS pricing.

### Discovering GCP storage buckets

In order to protect GCP resources in Defender for Cloud, you can set up a Google connector using a script template to onboard the GCP account.

- To discover GCP storage buckets, Defender for Cloud updates the script template.
- The script template creates a new role in the Google account to allow permission for the Defender for Cloud scanner to access data in the GCP storage buckets.
- To connect Google accounts, you need Administrator permissions on the account.

## Exposed to the internet/allows public access

Defender CSPM attack paths and cloud security graph insights include information about storage resources that are exposed to the internet and allow public access. The following table provides more details.

| **State** | **Azure storage accounts** | **AWS S3 Buckets** | **GCP Storage Buckets** |
|--- | --- | --- | --- |
|**Exposed to the internet** | An Azure storage account is considered exposed to the internet if either of these settings enabled:<br/><br/> Storage_account_name > **Networking** > **Public network access** > **Enabled from all networks**<br/><br/> or<br/><br/>  Storage_account_name > **Networking** > **Public network access** > **Enable from selected virtual networks and IP addresses**. | An AWS S3 bucket is considered exposed to the internet if the AWS account/AWS S3 bucket policies don't have a condition set for IP addresses. | All GCP storage buckets are exposed to the internet by default. |
|**Allows public access** | An Azure storage account container is considered as allowing public access if these settings are enabled on the storage account:<br/><br/> Storage_account_name > **Configuration** >**Allow Blob Anonymous access** > **Enabled**.<br/><br/>and **either** of these settings:<br/><br/> Storage_account_name > **Containers** > container_name > **Public access level** set to **Blob (anonymous read access for blobs only)**<br/><br/> Or, storage_account_name > **Containers** > container_name > **Public access level** set to **Container (anonymous read access for containers and blobs)**. | An AWS S3 bucket is considered to allow public access if both the AWS account and the AWS S3 bucket have **Block all public access** set to **Off**, and **either** of these settings is set:<br/><br/> In the policy, **RestrictPublicBuckets** isn't enabled, and the **Principal** setting is set to * and **Effect** is set to **Allow**.<br/><br/> Or, in the access control list, **IgnorePublicAcl** isn't enabled, and permission is allowed for **Everyone**, or for **Authenticated users**. | A GCP storage bucket is considered to allow public access if: it has an IAM (Identity and Access Management) role that meets these criteria: <br/><br/> The role is granted to the principal **allUsers** or **allAuthenticatedUsers**. <br/><br/>The role has at least one storage permission that *isn't* **storage.buckets.create** or **storage.buckets.list**. Public access in GCP is called “Public to internet“.|

Database resources don't allow public access but can still be exposed to the internet.

Internet exposure insights are available for the following resources:

Azure:

- Azure SQL server
- Azure Cosmos DB
- Azure SQL Managed Instance
- Azure MySQL Single Server
- Azure MySQL Flexible Server
- Azure PostgreSQL Single Server
- Azure PostgreSQL Flexible Server
- Azure MariaDB Single Server
- Synapse Workspace

AWS:

- RDS instance

> [!NOTE]
>
> - Exposure rules that include 0.0.0.0/0 are considered “excessively exposed”, meaning that they can be accessed from any public IP.
> - Azure resources with the exposure rule “0.0.0.0” are accessible from any resource in Azure (regardless of tenant or subscription).

## Next step

[Enable](data-security-posture-enable.md) data security posture management.
