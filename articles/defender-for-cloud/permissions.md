---
title: User roles and permissions
description: Learn how Microsoft Defender for Cloud uses role-based access control to assign permissions and identify permitted actions for each role.
ms.topic: limits-and-quotas
ms.date: 03/10/2025
---

# User roles and permissions

Microsoft Defender for Cloud uses [Azure role-based access control (Azure Role-Based Access Control)](/azure/role-based-access-control/role-assignments-portal) to provide [built-in roles](/azure/role-based-access-control/built-in-roles). Assign these roles to users, groups, and services in Azure to give them access to resources according to the role's defined access.

Defender for Cloud assesses resource configurations and identifies security issues and vulnerabilities. In Defender for Cloud, view resource information when assigned one of these roles for the subscription or resource group: Owner, Contributor, or Reader.

In addition to the built-in roles, there are two roles specific to Defender for Cloud:

- **Security Reader**: A user in this role has read-only access to Defender for Cloud. The user can view recommendations, alerts, security policies, and security states but can't make changes.
- **Security Admin**: A user in this role has the same access as the Security Reader and can also update security policies and dismiss alerts and recommendations.

Assign the least permissive role needed for users to complete their tasks.

For example, assign the Reader role to users who only need to view security health information of a resource without taking any action. Users with a Reader role can't apply recommendations or edit policies.

## Roles and allowed actions

The following table displays roles and allowed actions in Defender for Cloud.

| **Action**   | [Security Reader](/azure/role-based-access-control/built-in-roles#security-reader) /<br> [Reader](/azure/role-based-access-control/built-in-roles#reader) | [Security Admin](/azure/role-based-access-control/built-in-roles#security-admin) | [Contributor](/azure/role-based-access-control/built-in-roles#contributor) / [Owner](/azure/role-based-access-control/built-in-roles#owner) | [Contributor](/azure/role-based-access-control/built-in-roles#contributor) | [Owner](/azure/role-based-access-control/built-in-roles#owner) |
|:-|:-:|:-:|:-:|:-:|:-:|
|  |  |  | **(Resource group level)** | **(Subscription level)** | **(Subscription level)** |
| Add/assign initiatives (including regulatory compliance standards) | - | ✔ | - | - | ✔ |
| Edit security policy | - | ✔ | - | - | ✔ |
| Enable / disable Microsoft Defender plans | - | ✔ | - | ✔ | ✔ |
| Dismiss alerts | - | ✔ | - | ✔ | ✔ |
| Apply security recommendations for a resource</br> (Use [Fix](implement-security-recommendations.md)) | - | - | ✔ | ✔ | ✔ |
| View alerts and recommendations | ✔ | ✔ | ✔ | ✔ | ✔ |
| Exempt security recommendations | - |✔|-|-| ✔ |
| Configure email notifications | - | ✔ | ✔| ✔ | ✔ |

> [!NOTE]
> While the three roles mentioned are sufficient for enabling and disabling Defender for Cloud plans, the Owner role is required to enable all capabilities of a plan.

The specific role required to deploy monitoring components depends on the extension you deploy. Learn more about [monitoring components](monitoring-components.md).

## Roles used to automatically configure agents and extensions

To allow the Security Admin role to automatically configure agents and extensions used in Defender for Cloud plans, Defender for Cloud uses policy remediation similar to [Azure Policy](/azure/governance/policy/how-to/remediate-resources). To use remediation, Defender for Cloud needs to create service principals, also called managed identities, that assign roles at the subscription level. For example, the service principals for the Defender for Containers plan are:

| Service Principal                                            | Roles                                                        |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| Defender for Containers provisioning Azure Kubernetes Service (AKS) Security Profile | Kubernetes Extension Contributor<br>Contributor<br>Azure Kubernetes Service Contributor<br>Log Analytics Contributor |
| Defender for Containers provisioning Arc-enabled Kubernetes  | Azure Kubernetes Service Contributor<br>Kubernetes Extension Contributor<br>Contributor<br>Log Analytics Contributor |
| Defender for Containers provisioning Azure Policy for Kubernetes | Kubernetes Extension Contributor<br>Contributor<br>Azure Kubernetes Service Contributor |
| Defender for Containers provisioning Policy extension for Arc-enabled Kubernetes | Azure Kubernetes Service Contributor<br>Kubernetes Extension Contributor<br>Contributor |

## Permissions on AWS

When you onboard an Amazon Web Services (AWS) connector, Defender for Cloud creates roles and assigns permissions on your AWS account. The following table shows the roles and permissions assigned by each plan on your AWS account.

| Defender for Cloud plan                           | Role created                                          | Permissions assigned on AWS account                           |
| ------------------------------------------------- | ----------------------------------------------------- | ------------------------------------------------------------ |
| Defender Cloud Security Posture Management (CSPM) | CspmMonitorAws                                        | To discover AWS resources permissions,  read all resources except:<br> consolidatedbilling:<br> freetier: <br> invoicing:<br> payments: <br> billing:<br> tax: <br> cur: |
| Defender CSPM <br><br> Defender for Servers       | DefenderForCloud-AgentlessScanner                     | To create and clean up disk snapshots (scoped by tag) "CreatedBy": "Microsoft Defender for Cloud" Permissions:<br> ec2:DeleteSnapshot ec2:ModifySnapshotAttribute<br>  ec2:DeleteTags<br>  ec2:CreateTags<br>  ec2:CreateSnapshots<br>  ec2:CopySnapshot<br>  ec2:CreateSnapshot<br>  ec2:DescribeSnapshots<br>  ec2:DescribeInstanceStatus<br>  Permission to EncryptionKeyCreation kms:CreateKey<br>  kms:ListKeys<br>  Permissions to EncryptionKeyManagement kms:TagResource<br>  kms:GetKeyRotationStatus<br>  kms:PutKeyPolicy<br>  kms:GetKeyPolicy<br>  kms:CreateAlias<br>  kms:TagResource<br>  kms:ListResourceTags<br>  kms:GenerateDataKeyWithoutPlaintext<br>  kms:DescribeKey<br> kms:RetireGrant<br>  kms:CreateGrant<br>  kms:ReEncryptFrom |
| Defender CSPM <br><br> Defender for Storage       | SensitiveDataDiscovery                                | Permissions to discover S3 buckets in the AWS account, permission for the Defender for Cloud scanner to access data in the S3 buckets <br> S3 read only <br/><br/> KMS decrypt <br/> kms:Decrypt |
| CIEM                                              | DefenderForCloud-Ciem <br> DefenderForCloud-OidcCiem  | Permissions for Ciem Discovery<br> sts:AssumeRole<br>  sts:AssumeRoleWithSAML<br>  sts:GetAccessKeyInfo<br>  sts:GetCallerIdentity<br>  sts:GetFederationToken<br>  sts:GetServiceBearerToken<br>  sts:GetSessionToken<br>  sts:TagSession |
| Defender for Servers                              | DefenderForCloud-DefenderForServers                   | Permissions to configure JIT Network Access: <br>ec2:RevokeSecurityGroupIngress<br>  ec2:AuthorizeSecurityGroupIngress<br> ec2:DescribeInstances<br>  ec2:DescribeSecurityGroupRules<br>  ec2:DescribeVpcs<br>  ec2:CreateSecurityGroup<br>  ec2:DeleteSecurityGroup<br> ec2:ModifyNetworkInterfaceAttribute<br>  ec2:ModifySecurityGroupRules<br>  ec2:ModifyInstanceAttribute<br>  ec2:DescribeSubnets<br>  ec2:DescribeSecurityGroups |
| Defender for Containers                           |                 | See [Defender for Containers AWS permissions](containers-permissions.md) |
| Defender for Servers                              | DefenderForCloud-ArcAutoProvisioning                  | Permissions to install Azure Arc on all EC2 instances using SSM <br>ssm:CancelCommand<br>  ssm:DescribeInstanceInformation<br>  ssm:GetCommandInvocation<br>  ssm:UpdateServiceSetting<br>  ssm:GetServiceSetting<br>  ssm:GetAutomationExecution<br>  ec2:DescribeIamInstanceProfileAssociations<br>  ec2:DisassociateIamInstanceProfile<br>  ec2:DescribeInstances<br>  ssm:StartAutomationExecution<br> iam:GetInstanceProfile<br> iam:ListInstanceProfilesForRole<br>  ssm:GetAutomationExecution<br>  ec2:DescribeIamInstanceProfileAssociations<br>  ec2:DisassociateIamInstanceProfile<br>  ec2:DescribeInstances<br>  ssm:StartAutomationExecution<br>  iam:GetInstanceProfile<br>  iam:ListInstanceProfilesForRole |
| Defender CSPM                                     | DefenderForCloud-DataSecurityPostureDB                | Permission to Discover RDS instances in AWS account, create RDS instance snapshot, <br> - List all RDS DBs/clusters <br> - List all DB/Cluster snapshots <br> - Copy all DB/cluster snapshots <br> - Delete/update DB/cluster snapshot with prefix *defenderfordatabases* <br> - List all KMS keys <br> - Use all KMS keys only for RDS on source account <br> - List KMS keys with tag prefix *DefenderForDatabases* <br> - Create alias for KMS keys <br><br> Permissions required to discover, RDS instances<br> rds:DescribeDBInstances<br> rds:DescribeDBClusters<br> rds:DescribeDBClusterSnapshots<br>  rds:DescribeDBSnapshots<br> rds:CopyDBSnapshot<br> rds:CopyDBClusterSnapshot<br> rds:DeleteDBSnapshot<br>  rds:DeleteDBClusterSnapshot<br> rds:ModifyDBSnapshotAttribute<br> rds:ModifyDBClusterSnapshotAttribute rds:DescribeDBClusterParameters<br> rds:DescribeDBParameters<br>  rds:DescribeOptionGroups<br>  kms:CreateGrant<br> kms:ListAliases<br> kms:CreateKey<br> kms:TagResource<br> kms:ListGrants<br>  kms:DescribeKey<br> kms:PutKeyPolicy<br> kms:Encrypt<br> kms:CreateGrant<br> kms:EnableKey<br>  kms:CancelKeyDeletion<br> kms:DisableKey<br> kms:ScheduleKeyDeletion<br> kms:UpdateAlias<br>  kms:UpdateKeyDescription |

## Permissions on GCP

When you onboard a Google Cloud Platforms (GCP) connector, Defender for Cloud creates roles and assigns permissions on your GCP project. The following table shows the roles and permissions assigned by each plan on your GCP project.

| Defender for Cloud plan | Role created | Permission assigned on AWS account |
|--|--|--|
| Defender CSPM  | MDCCspmCustomRole  | These permissions allow the CSPM role to discover and scan resources within the organization:<br><br>Allows the role to view and organizations, projects, and folders:<br> resourcemanager.folders.get<br> resourcemanager.folders.list resourcemanager.folders.getIamPolicy<br> resourcemanager.organizations.get<br> resourcemanager.organizations.getIamPolicy<br> storage.buckets.getIamPolicy<br><br>Allows the autoprovisioning process of new projects and removal of deleted projects:<br> resourcemanager.projects.get<br> resourcemanager.projects.list<br><br>Allows the role to enable Google Cloud services used for the discovery of resources:<br> serviceusage.services.enable<br><br>Used to create and list IAM roles:<br> iam.roles.create<br> iam.roles.list<br><br>Allows the role to act as a service account and gain permission to resources:<br>iam.serviceAccounts.actAs<br><br>Allows the role to view project details and set common instance metadata:<br>compute.projects.get<br>compute.projects.setCommonInstanceMetadata <br><br>Used to discover and scan AI platform resources within the organization:<br>aiplatform.batchPredictionJobs.list<br>aiplatform.customJobs.list<br>aiplatform.datasets.list<br>aiplatform.datasets.get<br>aiplatform.endpoints.getIamPolicy<br>aiplatform.endpoints.list<br>aiplatform.indexEndpoints.list<br>aiplatform.indexes.list<br>aiplatform.models.list<br>aiplatform.models.get<br>aiplatform.pipelineJobs.list<br>aiplatform.schedules.list<br>aiplatform.tuningJobs.list<br>discoveryengine.dataStores.list<br>discoveryengine.documents.list<br>discoveryengine.engines.list<br>notebooks.instances.list|
| Defender for Servers | microsoft-defender-for-servers <br> azure-arc-for-servers-onboard | Read-only access to get and list Compute Engine resources: <br> compute.viewer<br> iam.serviceAccountTokenCreator<br> osconfig.osPolicyAssignmentAdmin<br> osconfig.osPolicyAssignmentReportViewer |
| Defender for Database | defender-for-databases-arc-ap | Permissions to Defender for databases ARC auto provisioning <br> compute.viewer <br> iam.workloadIdentityUser <br> iam.serviceAccountTokenCreator<br> osconfig.osPolicyAssignmentAdmin<br> osconfig.osPolicyAssignmentReportViewer |
| Defender CSPM <br><br> Defender for Storage | data-security-posture-storage | Permission for the Defender for Cloud scanner to discover GCP storage buckets, to access data in the GCP storage buckets <br> storage.objects.list<br> storage.objects.get<br> storage.buckets.get |
| Defender CSPM <br><br> Defender for Storage | data-security-posture-storage | Permission for the Defender for Cloud scanner to discover GCP storage buckets, to access data in the GCP storage buckets<br> storage.objects.list<br> storage.objects.get<br> storage.buckets.get |
| Defender CSPM | microsoft-defender-ciem  | Permissions to get details about the organization resource.<br> resourcemanager.folders.getIamPolicy<br> resourcemanager.folders.list<br> resourcemanager.organizations.get<br> resourcemanager.organizations.getIamPolicy<br> storage.buckets.getIamPolicy |
| Defender CSPM <br><br> Defender for Servers | MDCAgentlessScanningRole | Permissions for agentless disk scanning:<br> compute.disks.createSnapshot<br> compute.instances.get |
| Defender CSPM <br><br> Defender for servers | cloudkms.cryptoKeyEncrypterDecrypter | Permissions to an existing GCP KMS role are granted to support scanning disks that are encrypted with CMEK |
| Defender for Containers                           |                 | See [Defender for Containers GCP permissions](containers-permissions.md) |

## Next steps

This article explained how Defender for Cloud uses Azure  Role-Based Access Control to assign permissions to users and identified the allowed actions for each role. Now that you're familiar with the role assignments needed to monitor the security state of your subscription, edit security policies, and apply recommendations, learn how to:

- [Set security policies in Defender for Cloud](tutorial-security-policy.md)
- [Manage security recommendations in Defender for Cloud](review-security-recommendations.md)
- [Manage and respond to security alerts in Defender for Cloud](managing-and-responding-alerts.yml)
- [Monitor partner security solutions](./partner-integration.md)
