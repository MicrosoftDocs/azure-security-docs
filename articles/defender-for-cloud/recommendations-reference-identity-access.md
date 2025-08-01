---
title: Reference table for all identity and access security recommendations in Microsoft Defender for cloud
description: This article lists all Microsoft Defender for Cloud identity and access security recommendations that help you harden and protect your resources.
author: dcurwin
ms.service: defender-for-cloud
ms.topic: reference
ms.date: 05/18/2025
ms.author: dacurwin
ms.custom: generated
ai-usage: ai-assisted
---

# Identity and access security recommendations

This article lists all the identity and access security recommendations you might see in Microsoft Defender for Cloud.

The recommendations that appear in your environment are based on the resources that you're protecting and on your customized configuration. You can [see the recommendations in the portal](https://portal.azure.com/#view/Microsoft_Azure_Security/SecurityMenuBlade/~/5) that apply to your resources.

To learn about actions that you can take in response to these recommendations, see [Remediate recommendations in Defender for Cloud](implement-security-recommendations.md).

> [!TIP]
> If a recommendation description says *No related policy*, usually it's because that recommendation is dependent on a different recommendation.
>
> For example, the recommendation *Endpoint protection health failures should be remediated* relies on the recommendation that checks whether an endpoint protection solution is installed (*Endpoint protection solution should be installed*). The underlying recommendation *does* have a policy.
> Limiting policies to only foundational recommendations simplifies policy management.

## Azure identity and access recommendations

### A maximum of 3 owners should be designated for subscriptions

**Description**: To reduce the potential for breaches by compromised owner accounts, we recommend limiting the number of owner accounts to a maximum of 3
(Related policy: [A maximum of 3 owners should be designated for your subscription](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f4f11b553-d42e-4e3a-89be-32ca364cad4c)).

**Severity**: High

### Azure Cosmos DB accounts should use Azure Active Directory as the only authentication method

**Description**: The best way to authenticate to Azure services is by using Role-Based Access Control (RBAC). RBAC allows you to maintain the minimum privilege principle and supports the ability to revoke permissions as an effective method of response when compromised. You can configure your Azure Cosmos DB account to enforce RBAC as the only authentication method. When the enforcement is configured, all other methods of access will be denied (primary/secondary keys and access tokens).
(No related policy)

**Severity**: Medium

### Blocked accounts with owner permissions on Azure resources should be removed

**Description**: Accounts that have been blocked from signing in on Active Directory, should be removed from your Azure resources. These accounts can be targets for attackers looking to find ways to access your data without being noticed.
(No related policy)

**Severity**: High

### Blocked accounts with read and write permissions on Azure resources should be remove

**Description**: Accounts that have been blocked from signing in on Active Directory, should be removed from your Azure resources. These accounts can be targets for attackers looking to find ways to access your data without being noticed.
(No related policy)

**Severity**: High

### Deprecated accounts should be removed from subscriptions

**Description**: User accounts that have been blocked from signing in, should be removed from your subscriptions.
These accounts can be targets for attackers looking to find ways to access your data without being noticed.
(Related policy: [Deprecated accounts should be removed from your subscription](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f6b1cbf55-e8b6-442f-ba4c-7246b6381474)).

**Severity**: High

### Deprecated accounts with owner permissions should be removed from subscriptions

**Description**: User accounts that have been blocked from signing in, should be removed from your subscriptions.
These accounts can be targets for attackers looking to find ways to access your data without being noticed.
(Related policy: [Deprecated accounts with owner permissions should be removed from your subscription](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2febb62a0c-3560-49e1-89ed-27e074e9f8ad)).

**Severity**: High

### Diagnostic logs in Key Vault should be enabled

**Description**: Enable logs and retain them for up to a year. This enables you to recreate activity trails for investigation purposes when a security incident occurs or your network is compromised.
(Related policy: [Diagnostic logs in Key Vault should be enabled](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2fcf820ca0-f99e-4f3e-84fb-66e913812d21)).

**Severity**: Low

### External accounts with owner permissions should be removed from subscriptions

**Description**: Accounts with owner permissions that have different domain names (external accounts), should be removed from your subscription. This prevents unmonitored access. These accounts can be targets for attackers looking to find ways to access your data without being noticed.
(Related policy: [External accounts with owner permissions should be removed from your subscription](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2ff8456c1c-aa66-4dfb-861a-25d127b775c9)).

**Severity**: High

### External accounts with read permissions should be removed from subscriptions

**Description**: Accounts with read permissions that have different domain names (external accounts), should be removed from your subscription. This prevents unmonitored access. These accounts can be targets for attackers looking to find ways to access your data without being noticed.
(Related policy: [External accounts with read permissions should be removed from your subscription](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f5f76cf89-fbf2-47fd-a3f4-b891fa780b60)).

**Severity**: High

### External accounts with write permissions should be removed from subscriptions

**Description**: Accounts with write permissions that have different domain names (external accounts), should be removed from your subscription. This prevents unmonitored access. These accounts can be targets for attackers looking to find ways to access your data without being noticed.
(Related policy: [External accounts with write permissions should be removed from your subscription](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f5c607a2e-c700-4744-8254-d77e7c9eb5e4)).

**Severity**: High

### Firewall should be enabled on Key Vault

**Description**: Key vault's firewall prevents unauthorized traffic from reaching your key vault and provides an additional layer of protection for your secrets. Enable the firewall to make sure that only traffic from allowed networks can access your key vault.
(Related policy: [Firewall should be enabled on Key Vault](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f55615ac9-af46-4a59-874e-391cc3dfb490)).

**Severity**: Medium

### Guest accounts with owner permissions on Azure resources should be removed

**Description**: Accounts with owner permissions that have been provisioned outside of the Azure Active Directory tenant (different domain names), should be removed from your Azure resources. Guest accounts aren't managed to the same standards as enterprise tenant identities. These accounts can be targets for attackers looking to find ways to access your data without being noticed.
(No related policy)

**Severity**: High

### Guest accounts with read permissions on Azure resources should be removed

**Description**: Accounts with read permissions that have been provisioned outside of the Azure Active Directory tenant (different domain names), should be removed from your Azure resources. Guest accounts aren't managed to the same standards as enterprise tenant identities. These accounts can be targets for attackers looking to find ways to access your data without being noticed.
(No related policy)

**Severity**: High

### Guest accounts with write permissions on Azure resources should be removed

**Description**: Accounts with write permissions that have been provisioned outside of the Azure Active Directory tenant (different domain names), should be removed from your Azure resources. Guest accounts aren't managed to the same standards as enterprise tenant identities. These accounts can be targets for attackers looking to find ways to access your data without being noticed.
(No related policy)

**Severity**: High

### Key Vault keys should have an expiration date

**Description**: Cryptographic keys should have a defined expiration date and not be permanent. Keys that are valid forever provide a potential attacker with more time to compromise the key. It's a recommended security practice to set expiration dates on cryptographic keys.
(Related policy: [Key Vault keys should have an expiration date](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f152b15f7-8e1f-4c1f-ab71-8c010ba5dbc0)).

**Severity**: High

### Key Vault secrets should have an expiration date

**Description**: Secrets should have a defined expiration date and not be permanent. Secrets that are valid forever provide a potential attacker with more time to compromise them. It's a recommended security practice to set expiration dates on secrets.
(Related policy: [Key Vault secrets should have an expiration date](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f98728c90-32c7-4049-8429-847dc0f4fe37)).

**Severity**: High

### Key vaults should have purge protection enabled

**Description**: Malicious deletion of a key vault can lead to permanent data loss. A malicious insider in your organization can potentially delete and purge key vaults. Purge protection protects you from insider attacks by enforcing a mandatory retention period for soft deleted key vaults. No one inside your organization or Microsoft will be able to purge your key vaults during the soft delete retention period.
(Related policy: [Key vaults should have purge protection enabled](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f0b60c0b2-2dc2-4e1c-b5c9-abbed971de53)).

**Severity**: Medium

### Key vaults should have soft delete enabled

**Description**: Deleting a key vault without soft delete enabled permanently deletes all secrets, keys, and certificates stored in the key vault. Accidental deletion of a key vault can lead to permanent data loss. Soft delete allows you to recover an accidentally deleted key vault for a configurable retention period.
(Related policy: [Key vaults should have soft delete enabled](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f1e66c121-a66a-4b1f-9b83-0fd99bf0fc2d)).

**Severity**: High

### Microsoft Defender for Key Vault should be enabled

**Description**: Microsoft Defender for Cloud includes Microsoft Defender for Key Vault, providing an additional layer of security intelligence.
Microsoft Defender for Key Vault detects unusual and potentially harmful attempts to access or exploit Key Vault accounts.

Protections from this plan are charged as shown on the **Defender plans** page. If you don't have any key vaults in this subscription, you won't be charged. If you later create key vaults on this subscription, they'll automatically be protected and charges will begin. Learn about the [pricing details per region](https://aka.ms/pricing-security-center).
Learn more in [Introduction to Microsoft Defender for Key Vault](defender-for-key-vault-introduction.md).
(Related policy: [Azure Defender for Key Vault should be enabled](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fmicrosoft.authorization%2fpolicyDefinitions%2f0e6763cc-5078-4e64-889d-ff4d9a839047)).

**Severity**: High

### Permissions of inactive identities in your Azure subscription should be revoked

**Description**: Microsoft Defender for Cloud discovered an identity that has not performed any action on any resource within your Azure subscription in the past 45 days. It is recommended to revoke permissions of inactive identities, in order to reduce the attack surface of your cloud environment.

**Severity**: Medium

### Private endpoint should be configured for Key Vault

**Description**: Private link provides a way to connect Key Vault to your Azure resources without sending traffic over the public internet. Private link provides defense in depth protection against data exfiltration.
(Related policy: [Private endpoint should be configured for Key Vault](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f5f0bc445-3935-4915-9981-011aa2b46147)).

**Severity**: Medium

### Storage account public access should be disallowed

**Description**: Anonymous public read access to containers and blobs in Azure Storage is a convenient way to share data, but might present security risks. To prevent data breaches caused by undesired anonymous access, Microsoft recommends preventing public access to a storage account unless your scenario requires it.
(Related policy: [Storage account public access should be disallowed](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fmicrosoft.authorization%2fpolicyDefinitions%2f4fa4b6c0-31ca-4c0d-b10d-24b96f62a751)).

**Severity**: Medium

### There should be more than one owner assigned to subscriptions

**Description**: Designate more than one subscription owner in order to have administrator access redundancy.
(Related policy: [There should be more than one owner assigned to your subscription](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f09024ccc-0c5f-475e-9457-b7c0d9ed487b)).

**Severity**: High

### Validity period of certificates stored in Azure Key Vault should not exceed 12 months

**Description**: Ensure your certificates do not have a validity period that exceeds 12 months.
(Related policy: [Certificates should have the specified maximum validity period](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f0a075868-4c26-42ef-914c-5bc007359560)).

**Severity**: Medium

### Azure overprovisioned identities should have only the necessary permissions

**Description**: Overprovisioned identities, or over permissioned identities, don't use many of their granted permissions. Regularly right-size permissions of these identities to reduce the risk of permissions misuse, either accidental or malicious. This action decreases the potential blast radius during a security incident.

**Severity**: Medium

### Privileged roles should not have permanent access at the subscription and resource group level

**Description:** Microsoft Defender for Cloud discovered an identity that has not performed any action on any resource within your Azure subscription in the past 45 days. It is recommended to revoke permissions of inactive identities, in order to reduce the attack surface of your cloud environment.

**Severity**: High

### Service Principals should not be assigned with administrative roles at the subscription and resource group level

**Description**: Defender for Cloud identified Service Principals that are assigned with privileged roles at the resource group or subscription level. Privileged admin roles are roles that can perform sensitive operations on the resource, such as, Owner, Contributor or User Access Administrator. Service principals play a crucial role in managing Azure resources efficiently and securely, eliminating the need for human intervention. It is important to follow the [principle of least privilege](https://en.wikipedia.org/wiki/Principle_of_least_privilege), grant only the minimum level of access necessary for a given service principal to perform their duties. Admins and privileged access are primary target of hackers. For best practices when using privileged administrator role assignments, see [Best practices for Azure RBAC](/azure/role-based-access-control/best-practices?WT.mc_id=Portal-Microsoft_Azure_Security). [Best practices for Azure RBAC](/azure/role-based-access-control/best-practices?WT.mc_id=Portal-Microsoft_Azure_Security). For a list of available roles in Azure RBAC, see [Azure's built-in roles](/azure/role-based-access-control/built-in-roles?WT.mc_id=Portal-Microsoft_Azure_Security).

**Severity**: High

## AWS identity and access recommendations

### Amazon Elasticsearch Service domains should be in a VPC

**Description**: VPC can't contain domains with a public endpoint.
This doesn't evaluate the VPC subnet routing configuration to determine public reachability.

**Severity**: High

### Amazon S3 permissions granted to other AWS accounts in bucket policies should be restricted

**Description**: Implementing least privilege access is fundamental to reducing security risk and the impact of errors or malicious intent. If an S3 bucket policy allows access from external accounts, it could result in data exfiltration by an insider threat or an attacker. The 'blacklistedactionpatterns' parameter allows for successful evaluation of the rule for S3 buckets. The parameter grants access to external accounts for action patterns that aren't included in the 'blacklistedactionpatterns' list.

**Severity**: High

### Avoid the use of the "root" account

**Description**: The "root" account has unrestricted access to all resources in the AWS account. It's highly recommended that the use of this account be avoided.
The "root" account is the most privileged AWS account. Minimizing the use of this account and adopting the principle of least privilege for access management will reduce the risk of accidental changes and unintended disclosure of highly privileged credentials.

**Severity**: High

### AWS KMS keys should not be unintentionally deleted

**Description**: This control checks whether KMS keys are scheduled for deletion. The control fails if a KMS key is scheduled for deletion.
KMS keys can't be recovered once deleted. Data encrypted under a KMS key is also permanently unrecoverable if the KMS key is deleted. If meaningful data has been encrypted under a KMS key scheduled for deletion, consider decrypting the data or re-encrypting the data under a new KMS key unless you're intentionally performing a cryptographic erasure.
When a KMS key is scheduled for deletion, a mandatory waiting period is enforced to allow time to reverse the deletion, if it was scheduled in error. The default waiting period is 30 days, but it can be reduced to as short as seven days when the KMS key is scheduled for deletion. During the waiting period, the scheduled deletion can be canceled and the KMS key won't be deleted.
For more information regarding deleting KMS keys, see [Deleting KMS keys](https://docs.aws.amazon.com/kms/latest/developerguide/deleting-keys.html) in the AWS Key Management Service Developer Guide.

**Severity**: High

### AWS overprovisioned identities should have only the necessary permissions

**Description**: An over-provisioned active identity is an identity that has access to privileges that they haven't used. Over-provisioned active identities, especially for non-human accounts that have defined actions and responsibilities, can increase the blast radius in the event of a user, key, or resource compromise. Remove unneeded permissions and establish review processes to achieve the least privileged permissions.

**Severity**: Medium

### AWS WAF Classic global web ACL logging should be enabled

**Description**: This control checks whether logging is enabled for an AWS WAF global Web ACL. This control fails if logging isn't enabled for the web ACL.
Logging is an important part of maintaining the reliability, availability, and performance of AWS WAF globally. It's a business and compliance requirement in many organizations, and allows you to troubleshoot application behavior. It also provides detailed information about the traffic that is analyzed by the web ACL that is attached to AWS WAF.

**Severity**: Medium

### CloudFront distributions should have a default root object configured

**Description**: This control checks whether an Amazon CloudFront distribution is configured to return a specific object that is the default root object. The control fails if the CloudFront distribution doesn't have a default root object configured.
A user might sometimes request the distributions root URL instead of an object in the distribution. When this happens, specifying a default root object can help you to avoid exposing the contents of your web distribution.

**Severity**: High

### CloudFront distributions should have origin access identity enabled

**Description**: This control checks whether an Amazon CloudFront distribution with Amazon S3 Origin type has Origin Access Identity (OAI) configured. The control fails if OAI isn't configured.
CloudFront OAI prevents users from accessing S3 bucket content directly. When users access an S3 bucket directly, they effectively bypass the CloudFront distribution and any permissions that are applied to the underlying S3 bucket content.

**Severity**: Medium

### CloudTrail log file validation should be enabled

**Description**: To ensure additional integrity checking of CloudTrail logs, we recommend enabling file validation on all CloudTrails.

**Severity**: Low

### CloudTrail should be enabled

**Description**: AWS CloudTrail is a web service that records AWS API calls for your account and delivers log files to you. Not all services enable logging by default for all APIs and events.
 You should implement any additional audit trails other than CloudTrail and review the documentation for each service in CloudTrail Supported Services and Integrations.

**Severity**: High

### CloudTrail trails should be integrated with CloudWatch Logs

**Description**: In addition to capturing CloudTrail logs within a specified S3 bucket for long term analysis, real-time analysis can be performed by configuring CloudTrail to send logs to CloudWatch Logs.
 For a trail that is enabled in all regions in an account, CloudTrail sends log files from all those regions to a CloudWatch Logs log group. We recommended that CloudTrail logs will be sent to CloudWatch Logs to ensure AWS account activity is being captured, monitored, and appropriately alarmed on.
Sending CloudTrail logs to CloudWatch Logs facilitates real-time and historic activity logging based on user, API, resource, and IP address, and provides opportunity to establish alarms and notifications for anomalous or sensitivity account activity.

**Severity**: Low

### Database logging should be enabled

**Description**: This control checks whether the following logs of Amazon RDS are enabled and sent to CloudWatch Logs:

- Oracle: (Alert, Audit, Trace, Listener)
- PostgreSQL: (Postgresql, Upgrade)
- MySQL: (Audit, Error, General, SlowQuery)
- MariaDB: (Audit, Error, General, SlowQuery)
- SQL Server: (Error, Agent)
- Aurora: (Audit, Error, General, SlowQuery)
- Aurora-MySQL: (Audit, Error, General, SlowQuery)
- Aurora-PostgreSQL: (Postgresql, Upgrade).
RDS databases should have relevant logs enabled. Database logging provides detailed records of requests made to RDS. Database logs can assist with security and access audits and can help to diagnose availability issues.

**Severity**: Medium

### Disable direct internet access for Amazon Sage Maker notebook instances

**Description**: Direct internet access should be disabled for a Sage Maker notebook instance.
 This checks whether the 'DirectInternetAccess' field is disabled for the notebook instance.
 Your instance should be configured with a VPC and the default setting should be Disable - Access the internet through a VPC.
 In order to enable internet access to train or host models from a notebook, make sure that your VPC has a NAT gateway and your security group allows outbound connections. Ensure access to your Sage Maker configuration is limited to only authorized users, and restrict users' IAM permissions to modify Sage Maker settings and resources.

**Severity**: High

### Do not set up access keys during initial user setup for all IAM users that have a console password

**Description**: AWS console defaults the checkbox for creating access keys to enabled. This results in many access keys being generated unnecessarily.
 In addition to unnecessary credentials, it also generates unnecessary management work in auditing and rotating these keys.
 Requiring that additional steps be taken by the user after their profile has been created will give a stronger indication of intent that access keys are [a] necessary for their work and [b] once the access key is established on an account that the keys might be in use somewhere in the organization.

**Severity**: Medium

### Ensure a support role has been created to manage incidents with AWS Support

**Description**: AWS provides a support center that can be used for incident notification and response, as well as technical support and customer services.
 Create an IAM Role to allow authorized users to manage incidents with AWS Support.
When you implement least privilege for access control, an IAM Role requires an appropriate IAM Policy to allow Support Center Access in order to manage Incidents with AWS Support.

**Severity**: Low

### Ensure access keys are rotated every 90 days or less

**Description**: Access keys consist of an access key ID and secret access key, which are used to sign programmatic requests that you make in AWS.
 AWS users need their own access keys to make programmatic calls to AWS from the AWS Command Line Interface (AWS CLI), Tools for Windows PowerShell, the AWS SDKs, or direct HTTP calls using the APIs for individual AWS services.
 It's recommended that all access keys be regularly rotated.
 Rotating access keys reduce the window of opportunity for an access key that is associated with a compromised or terminated account to be used.
 Access keys should be rotated to ensure that data can't be accessed with an old key, which might have been lost, cracked, or stolen.

**Severity**: Medium

### Ensure AWS Config is enabled in all regions

**Description**: AWS Config is a web service that performs configuration management of supported AWS resources within your account and delivers log files to you.
The recorded information includes the configuration item (AWS resource), relationships between configuration items (AWS resources), any configuration changes between resources.
It's recommended to enable AWS Config be enabled in all regions.

The AWS configuration item history captured by AWS Config enables security analysis, resource change tracking, and compliance auditing.

**Severity**: Medium

### Ensure CloudTrail is enabled in all regions

**Description**: AWS CloudTrail is a web service that records AWS API calls for your account and delivers log files to you.
The recorded information includes the identity of the API caller, the time of the API call, the source IP address of the API caller, the request parameters, and the response elements returned by the AWS service. CloudTrail provides a history of AWS API calls for an account, including API calls made via the Management Console, SDKs, command line tools, and higher-level AWS services (such as CloudFormation).
The AWS API call history produced by CloudTrail enables security analysis, resource change tracking, and compliance auditing. Additionally:

- Checking that a multi-regions trail exists ensures that unexpected activity occurring in otherwise unused regions is detected.
- Checking that a multi-regions trail exists ensures that "Global Service Logging" is enabled for a trail by default to capture recording of events generated on AWS global services.
- For a multi-regions trail, checking that management events are configured for all type of Read/Writes ensures recording of management operations that are performed on all resources in an AWS account.

**Severity**: High

### Ensure credentials unused for 90 days or greater are disabled

**Description**: AWS IAM users can access AWS resources using different types of credentials, such as passwords or access keys.
 It's recommended that all credentials that have been unused in 90 or greater days be removed or deactivated.
 Disabling or removing unnecessary credentials reduce the window of opportunity for credentials associated with a compromised or abandoned account to be used.

**Severity**: Medium

### Ensure IAM password policy expires passwords within 90 days or less

**Description**: IAM password policies can require passwords to be rotated or expired after a given number of days.
 It's recommended that the password policy expire passwords after 90 days or less.
 Reducing the password lifetime increases account resiliency against brute force login attempts. Additionally, requiring regular password changes help in the following scenarios:

- Passwords can be stolen or compromised sometimes without your knowledge. This can happen via a system compromise, software vulnerability, or internal threat.
- Certain corporate and government web filters or proxy servers have the ability to intercept and record traffic even if it's encrypted.
- Many people use the same password for many systems such as work, email, and personal.
- Compromised end user workstations might have a keystroke logger.

**Severity**: Low

### Ensure IAM password policy prevents password reuse

**Description**: IAM password policies can prevent the reuse of a given password by the same user.
It's recommended that the password policy prevent the reuse of passwords.
 Preventing password reuse increases account resiliency against brute force login attempts.

**Severity**: Low

### Ensure IAM password policy requires at least one lowercase letter

**Description**: Password policies are, in part, used to enforce password complexity requirements. IAM password policies can be used to ensure password are composed of different character sets.
 It's recommended that the password policy require at least one lowercase letter.
Setting a password complexity policy increases account resiliency against brute force login attempts.

**Severity**: Medium

### Ensure IAM password policy requires at least one number

**Description**: Password policies are, in part, used to enforce password complexity requirements. IAM password policies can be used to ensure password are composed of different character sets.
 It's recommended that the password policy require at least one number.
 Setting a password complexity policy increases account resiliency against brute force login attempts.

**Severity**: Medium

### Ensure IAM password policy requires at least one symbol

**Description**: Password policies are, in part, used to enforce password complexity requirements.
 IAM password policies can be used to ensure password are composed of different character sets.
 It's recommended that the password policy require at least one symbol.
 Setting a password complexity policy increases account resiliency against brute force login attempts.

**Severity**: Medium

### Ensure IAM password policy requires at least one uppercase letter

**Description**: Password policies are, in part, used to enforce password complexity requirements. IAM password policies can be used to ensure password are composed of different character sets.
 It's recommended that the password policy require at least one uppercase letter.
 Setting a password complexity policy increases account resiliency against brute force login attempts.

**Severity**: Medium

### Ensure IAM password policy requires minimum length of 14 or greater

**Description**: Password policies are, in part, used to enforce password complexity requirements. IAM password policies can be used to ensure password are at least a given length.
It's recommended that the password policy require a minimum password length '14'.
 Setting a password complexity policy increases account resiliency against brute force login attempts.

**Severity**: Medium

### Ensure multifactor authentication (MFA) is enabled for all IAM users that have a console password

**Description**: Multifactor Authentication (MFA) adds an extra layer of protection on top of a user name and password.
 With MFA enabled, when a user signs in to an AWS website, they'll be prompted for their user name and password as well as for an authentication code from their AWS MFA device.
 It's recommended that MFA be enabled for all accounts that have a console password.
Enabling MFA provides increased security for console access as it requires the authenticating principal to possess a device that emits a time-sensitive key and have knowledge of a credential.

**Severity**: Medium

### GuardDuty should be enabled

**Description**: To provide additional protection against intrusions, GuardDuty should be enabled on your AWS account and region.

GuardDuty might not be a complete solution for every environment.

**Severity**: Medium

### Hardware MFA should be enabled for the "root" account

**Description**: The root account is the most privileged user in an account. MFA adds an extra layer of protection on top of a user name and password. With MFA enabled, when a user signs in to an AWS website, they're prompted for their user name and password and for an authentication code from their AWS MFA device.
 For Level 2, it's recommended that you protect the root account with a hardware MFA. A hardware MFA has a smaller attack surface than a virtual MFA. For example, a hardware MFA doesn't suffer the attack surface introduced by the mobile smartphone that a virtual MFA resides on.
When you use hardware for MFA for many, many accounts it might create a logistical device management issue. If this occurs, consider implementing this Level 2 recommendation selectively to the highest security accounts. You can then apply the Level 1 recommendation to the remaining accounts.

**Severity**: Low

### IAM authentication should be configured for RDS clusters

**Description**: This control checks whether an RDS DB cluster has IAM database authentication enabled.
IAM database authentication allows for password-free authentication to database instances. The authentication uses an authentication token. Network traffic to and from the database is encrypted using SSL. For more information, see IAM database authentication in the Amazon Aurora User Guide.

**Severity**: Medium

### IAM authentication should be configured for RDS instances

**Description**: This control checks whether an RDS DB instance has IAM database authentication enabled.
IAM database authentication allows authentication to database instances with an authentication token instead of a password. Network traffic to and from the database is encrypted using SSL. For more information, see IAM database authentication in the Amazon Aurora User Guide.

**Severity**: Medium

### IAM customer managed policies should not allow decryption actions on all KMS keys

**Description**: Checks whether the default version of IAM customer managed policies allow principals to use the AWS KMS decryption actions on all resources. This control uses [Zelkova](http://aws.amazon.com/blogs/security/protect-sensitive-data-in-the-cloud-with-automated-reasoning-zelkova), an automated reasoning engine, to validate and warn you about policies that might grant broad access to your secrets across AWS accounts. This control fails if the "kms: Decrypt" or "kms: ReEncryptFrom" actions are allowed on all KMS keys. The control evaluates both attached and unattached customer managed policies. It doesn't check inline policies or AWS managed policies.
With AWS KMS, you control who can use your KMS keys and gain access to your encrypted data. IAM policies define which actions an identity (user, group, or role) can perform on which resources. Following security best practices, AWS recommends that you allow least privilege. In other words, you should grant to identities only the "kms:Decrypt" or "kms:ReEncryptFrom" permissions and only for the keys that are required to perform a task. Otherwise, the user might use keys that aren't appropriate for your data.
Instead of granting permissions for all keys, determine the minimum set of keys that users need to access encrypted data. Then design policies that allow users to use only those keys. For example, don't allow "kms: Decrypt" permission on all KMS keys. Instead, allow "kms: Decrypt" only on keys in a particular Region for your account. By adopting the principle of least privilege, you can reduce the risk of unintended disclosure of your data.

**Severity**: Medium

### IAM customer managed policies that you create should not allow wildcard actions for services

**Description**: This control checks whether the IAM identity-based policies that you create have Allow statements that use the \* wildcard to  grant permissions for all actions on any service. The control fails if any policy statement includes 'Effect': 'Allow' with 'Action': 'Service:*'.
 For example, the following statement in a policy results in a failed finding.

```json
'Statement': [
{
  'Sid': 'EC2-Wildcard',
  'Effect': 'Allow',
  'Action': 'ec2:*',
  'Resource': '*'
}
```

 The control also fails if you use 'Effect': 'Allow' with 'NotAction': 'service:*'. In that case, the NotAction element provides access to all of the actions in an AWS service, except for the actions specified in NotAction.
This control only applies to customer managed IAM policies. It doesn't apply to IAM policies that are managed by AWS.
 When you assign permissions to AWS services, it's important to scope the allowed IAM actions in your IAM policies. You should restrict IAM actions to only those actions that are needed. This helps you to provision least privilege permissions. Overly permissive policies might lead to privilege escalation if the policies are attached to an IAM principal that might not require the permission.
In some cases, you might want to allow IAM actions that have a similar prefix, such as DescribeFlowLogs and DescribeAvailabilityZones. In these authorized cases, you can add a suffixed wildcard to the common prefix. For example, ec2:Describe*.

This control passes if you use a prefixed IAM action with a suffixed wildcard. For example, the following statement in a policy results in a passed finding.

```json
 'Statement': [
{
  'Sid': 'EC2-Wildcard',
  'Effect': 'Allow',
  'Action': 'ec2:Describe*',
  'Resource': '*'
}
```

When you group related IAM actions in this way, you can also avoid exceeding the IAM policy size limits.

**Severity**: Low

### IAM policies should be attached only to groups or roles

**Description**: By default, IAM users, groups, and roles have no access to AWS resources. IAM policies are the means by which privileges are granted to users, groups, or roles.
 It's recommended that IAM policies be applied directly to groups and roles but not users.
Assigning privileges at the group or role level reduces the complexity of access management as the number of users grow.
 Reducing access management complexity might also reduce opportunity for a principal to inadvertently receive or retain excessive privileges.

**Severity**: Low

### IAM policies that allow full "*:*" administrative privileges should not be created

**Description**: IAM policies are the means by which privileges are granted to users, groups, or roles.
 It's recommended and considered a standard security advice to grant least privilege-that is, granting only the permissions required to perform a task.
 Determine what users need to do and then craft policies for them that let the users perform only those tasks, instead of allowing full administrative privileges.
 It's more secure to start with a minimum set of permissions and grant additional permissions as necessary, rather than starting with permissions that are too lenient and then trying to tighten them later.
 Providing full administrative privileges instead of restricting to the minimum set of permissions that the user is required to do exposes the resources to potentially unwanted actions.
 IAM policies that have a statement with "Effect": "Allow" with "Action": "*" over "Resource": "*" should be removed.

**Severity**: High

### IAM principals should not have IAM inline policies that allow decryption actions on all KMS keys

**Description**: Checks whether the inline policies that are embedded in your IAM identities (role, user, or group) allow the AWS KMS decryption actions on all KMS keys. This control uses [Zelkova](http://aws.amazon.com/blogs/security/protect-sensitive-data-in-the-cloud-with-automated-reasoning-zelkova), an automated reasoning engine, to validate and warn you about policies that might grant broad access to your secrets across AWS accounts.
This control fails if ```kms:Decrypt``` or ```kms:ReEncryptFrom``` actions are allowed on all KMS keys in an inline policy.
With AWS KMS, you control who can use your KMS keys and gain access to your encrypted data. IAM policies define which actions an identity (user, group, or role) can perform on which resources. Following security best practices, AWS recommends that you allow least privilege. In other words, you should grant to identities only the permissions they need and only for keys that are required to perform a task. Otherwise, the user might use keys that aren't appropriate for your data.
Instead of granting permission for all keys, determine the minimum set of keys that users need to access encrypted data. Then design policies that allow the users to use only those keys. For example, don't allow ```kms:Decrypt``` permission on all KMS keys. Instead, allow them only on keys in a particular Region for your account. By adopting the principle of least privilege, you can reduce the risk of unintended disclosure of your data.

**Severity**: Medium

### Lambda functions should restrict public access

**Description**: Lambda function resource-based policy should restrict public access. This recommendation doesn't check access by internal principals.
 Ensure access to the function is restricted to authorized principals only by using least privilege resource-based policies.

**Severity**: High

### MFA should be enabled for all IAM users

**Description**: All IAM users should have multifactor authentication (MFA) enabled.

**Severity**: Medium

### MFA should be enabled for the "root" account

**Description**: The root account is the most privileged user in an account. MFA adds an extra layer of protection on top of a user name and password. With MFA enabled, when a user signs in to an AWS website, they're prompted for their user name and password and for an authentication code from their AWS MFA device.
 When you use virtual MFA for root accounts, it's recommended that the device used isn't a personal device. Instead, use a dedicated mobile device (tablet or phone) that you manage to keep charged and secured independent of any individual personal devices.
 This lessens the risks of losing access to the MFA due to device loss, device trade-in, or if the individual owning the device is no longer employed at the company.

**Severity**: Low

### Password policies for IAM users should have strong configurations

**Description**: Checks whether the account password policy for IAM users uses the following minimum configurations.

- RequireUppercaseCharacters- Require at least one uppercase character in password. (Default = true)
- RequireLowercaseCharacters- Require at least one lowercase character in password. (Default = true)
- RequireNumbers- Require at least one number in password. (Default = true)
- MinimumPasswordLength- Password minimum length. (Default = 7 or longer)
- PasswordReusePrevention- Number of passwords before allowing reuse. (Default = 4)
- MaxPasswordAge- Number of days before password expiration. (Default = 90)

**Severity**: Medium

### Permissions of inactive identities in your AWS account should be revoked

**Description**: Microsoft Defender for Cloud discovered an identity that has not performed any action on any resource within your AWS account in the past 45 days. It is recommended to revoke permissions of inactive identities, in order to reduce the attack surface of your cloud environment.

**Severity**: Medium

### Root account access key shouldn't exist

**Description**: The root account is the most privileged user in an AWS account. AWS Access Keys provide programmatic access to a given AWS account.
 It's recommended that all access keys associated with the root account be removed.
 Removing access keys associated with the root account limits vectors by which the account can be compromised.
 Additionally, removing the root access keys encourages the creation and use of role based accounts that are least privileged.

**Severity**: High

### S3 Block Public Access setting should be enabled

**Description**: Enabling Block Public Access setting for your S3 bucket can help prevent sensitive data leaks and protect your bucket from malicious actions.

**Severity**: Medium

### S3 Block Public Access setting should be enabled at the bucket level

**Description**: This control checks whether S3 buckets have bucket-level public access blocks applied. This control fails if any of the following settings are set to false:

- ignorePublicAcls
- blockPublicPolicy
- blockPublicAcls
- restrictPublicBuckets
Block Public Access at the S3 bucket level provides controls to ensure that objects never have public access. Public access is granted to buckets and objects through access control lists (ACLs), bucket policies, or both.
Unless you intend to have your S3 buckets publicly accessible, you should configure the bucket level Amazon S3 Block Public Access feature.

**Severity**: High

### S3 buckets public read access should be removed

**Description**: Removing public read access to your S3 bucket can help protect your data and prevent a data breach.

**Severity**: High

### S3 buckets public write access should be removed

**Description**: Allowing public write access to your S3 bucket can leave you vulnerable to malicious actions such as storing data at your expense, encrypting your files for ransom, or using your bucket to operate malware.

**Severity**: High

### Secrets Manager secrets should have automatic rotation enabled

**Description**: This control checks whether a secret stored in AWS Secrets Manager is configured with automatic rotation.
Secrets Manager helps you improve the security posture of your organization. Secrets include database credentials, passwords, and third-party API keys. You can use Secrets Manager to store secrets centrally, encrypt secrets automatically, control access to secrets, and rotate secrets safely and automatically.
Secrets Manager can rotate secrets. You can use rotation to replace long-term secrets with short-term ones. Rotating your secrets limits how long an unauthorized user can use a compromised secret. For this reason, you should rotate your secrets frequently. To learn more about rotation, see [Rotating your AWS Secrets Manager secrets](https://docs.aws.amazon.com/secretsmanager/latest/userguide/rotating-secrets.html) in the AWS Secrets Manager User Guide.

**Severity**: Medium

### Stopped EC2 instances should be removed after a specified time period

**Description**: This control checks whether any EC2 instances have been stopped for more than the allowed number of days. An EC2 instance fails this check if it's stopped for longer than the maximum allowed time period, which by default is 30 days.
 A failed finding indicates that an EC2 instance has not run for a significant period of time. This creates a security risk because the EC2 instance isn't being actively maintained (analyzed, patched, updated). If it's later launched, the lack of proper maintenance could result in unexpected issues in your AWS environment. To safely maintain an EC2 instance over time in a nonrunning state, start it periodically for maintenance and then stop it after maintenance. Ideally this is an automated process.

**Severity**: Medium

### Unused identities in your AWS environment should be removed

**Description**: Inactive identities are human and non-human entities that haven't performed any action on any resource in the last 90 days. Inactive IAM identities with high-risk permissions in your AWS account can be prone to attack if left as is and leave organizations open to credential misuse or exploitation. Proactively detecting and responding to unused identities helps you prevent unauthorized entities from gaining access to your AWS resources.

**Severity**: Medium

## GCP identity and access recommendations

### Cryptographic keys should not have more than three users

**Description**: This recommendation evaluates IAM policies for key rings, projects, and organizations, and retrieves principals with roles that allow them to encrypt, decrypt, or sign data using Cloud KMS keys: roles/owner, roles/cloudkms.cryptoKeyEncrypterDecrypter, roles/cloudkms.cryptoKeyEncrypter, roles/cloudkms.cryptoKeyDecrypter, roles/cloudkms.signer, and roles/cloudkms.signerVerifier.

**Severity**: Medium

### Ensure API keys are not created for a project

**Description**: Keys are insecure because they can be viewed publicly, such as from within a browser, or they can be accessed on a device where the key resides. It's recommended to use standard authentication flow instead.

 Security risks involved in using API-Keys appear below:

- API keys are simple encrypted strings
- API keys don't identify the user or the application making the API request
- API keys are typically accessible to clients, making it easy to discover and steal an API key

 To avoid the security risk in using API keys, it's recommended to use standard authentication flow instead.

**Severity**: High

### Ensure API keys are restricted to only APIs that application needs access

**Description**: API keys are insecure because they can be viewed publicly, such as from within a browser, or they can be accessed on a device where the key resides. It's recommended to restrict API keys to use (call) only APIs required by an application.

 Security risks involved in using API-Keys are below:

- API keys are simple encrypted strings
- API keys don't identify the user or the application making the API request
- API keys are typically accessible to clients, making it easy to discover and steal an API key

In light of these potential risks, Google recommends using the standard authentication flow instead of API-Keys. However, there are limited cases where API keys are more appropriate. For example, if there's a mobile application that needs to use the Google Cloud Translation API, but doesn't otherwise need a backend server, API keys are the simplest way to authenticate to that API.

 In order to reduce attack surfaces by providing least privileges, API-Keys can be restricted to use (call) only APIs required by an application.

**Severity**: High

### Ensure API keys are restricted to use by only specified Hosts and Apps

**Description**: Unrestricted keys are insecure because they can be viewed publicly, such as from within a browser, or they can be accessed on a device where the key resides. It's recommended to restrict API key usage to trusted hosts, HTTP referrers, and apps.

 Security risks involved in using API-Keys appear below:

- API keys are simple encrypted strings
- API keys don't identify the user or the application making the API request
- API keys are typically accessible to clients, making it easy to discover and steal an API key

In light of these potential risks, Google recommends using the standard authentication flow instead of API keys. However, there are limited cases where API keys are more appropriate.
For example, if there's a mobile application that needs to use the Google Cloud Translation API, but doesn't otherwise need a backend server, API keys are the simplest way to authenticate to that API.

 In order to reduce attack vectors, API-Keys can be restricted only to trusted hosts, HTTP referrers, and applications.

**Severity**: High

### Ensure API keys are rotated every 90 days

**Description**: It's recommended to rotate API keys every 90 days.

 Security risks involved in using API-Keys are listed below:

- API keys are simple encrypted strings
- API keys don't identify the user or the application making the API request
- API keys are typically accessible to clients, making it easy to discover and steal an API key

Because of these potential risks, Google recommends using the standard authentication flow instead of API Keys. However, there are limited cases where API keys are more appropriate. For example, if there's a mobile application that needs to use the Google Cloud Translation API, but doesn't otherwise need a backend server, API keys are the simplest way to authenticate to that API.

 Once a key is stolen, it has no expiration, meaning it might be used indefinitely unless the project owner revokes or regenerates the key. Rotating API keys will reduce the window of opportunity for an access key that is associated with a compromised or terminated account to be used.

 API keys should be rotated to ensure that data can't be accessed with an old key that might have been lost, cracked, or stolen.

**Severity**: High

### Ensure KMS encryption keys are rotated within a period of 90 days

**Description**: Google Cloud Key Management Service stores cryptographic keys in a hierarchical structure designed for useful and elegant access control management.
 The format for the rotation schedule depends on the client library that is used.
 For the gcloud command-line tool, the next rotation time must be in "ISO" or "RFC3339" format, and the rotation period must be in the form "INTEGER[UNIT]," where units can be one of seconds (s), minutes (m), hours (h), or days (d).
 Set a key rotation period and starting time. A key can be created with a specified "rotation period," which is the time between when new key versions are generated automatically.
 A key can also be created with a specified next rotation time.
 A key is a named object representing a "cryptographic key" used for a specific purpose.
 The key material, the actual bits used for "encryption," can change over time as new key versions are created.
 A key is used to protect some "corpus of data." A collection of files could be encrypted with the same key and people with "decrypt" permissions on that key would be able to decrypt those files.
 Therefore, it's necessary to make sure the "rotation period" is set to a specific time.

**Severity**: Medium

### Ensure log metric filter and alerts exist for project ownership assignments/changes

**Description**: In order to prevent unnecessary project ownership assignments to users/service-accounts and further misuses of projects and resources, all "roles/Owner" assignments should be monitored.
 Members (users/Service-Accounts) with a role assignment to primitive role "roles/Owner" are project owners.
 The project owner has all the privileges on the project the role belongs to. These are summarized below:

- All viewer permissions on all GCP Services within the project
- Permissions for actions that modify the state of all GCP services within the project
- Manage roles and permissions for a project and all resources within the project
- Set up billing for a project
 Granting the owner role to a member (user/Service-Account) will allow that member to modify the Identity and Access Management (IAM) policy. Therefore, grant the owner role only if the member has a legitimate purpose to manage the IAM policy. This is because the project IAM policy contains sensitive access control data. Having a minimal set of users allowed to manage IAM policy will simplify any auditing that might be necessary.
Project ownership has the highest level of privileges on a project. To avoid misuse of project resources, the project ownership assignment/change actions mentioned above should be monitored and alerted to concerned recipients.
- Sending project ownership invites
- Acceptance/Rejection of project ownership invite by user
- Adding `role\Owner` to a user/service-account
- Removing a user/Service account from `role\Owner`

**Severity**: Low

### Ensure oslogin is enabled for a Project

**Description**: Enabling OS login binds SSH certificates to IAM users and facilitates effective SSH certificate management.
Enabling osLogin ensures that SSH keys used to connect to instances are mapped with IAM users. Revoking access to IAM user will revoke all the SSH keys associated with that particular user.
It facilitates centralized and automated SSH key pair management, which is useful in handling cases like response to compromised SSH key pairs and/or revocation of external/third-party/Vendor users.
To find out which instance causes the project to be unhealthy see recommendation "Ensure oslogin is enabled for all instances."

**Severity**: Medium

### Ensure oslogin is enabled for all instances

**Description**: Enabling OS login binds SSH certificates to IAM users and facilitates effective SSH certificate management.
Enabling osLogin ensures that SSH keys used to connect to instances are mapped with IAM users. Revoking access to IAM user will revoke all the SSH keys associated with that particular user.
It facilitates centralized and automated SSH key pair management, which is useful in handling cases like response to compromised SSH key pairs and/or revocation of external/third-party/Vendor users.

**Severity**: Medium

### Ensure that Cloud Audit Logging is configured properly across all services and all users from a project

**Description**: It's recommended that Cloud Audit Logging is configured to track all admin activities and read, write access to user data.

Cloud Audit Logging maintains two audit logs for each project, folder, and organization: Admin Activity and Data Access.

- Admin Activity logs contain log entries for API calls or other administrative actions that modify the configuration or metadata of resources.
- Admin Activity audit logs are enabled for all services and can't be configured.
- Data Access audit logs record API calls that create, modify, or read user-provided data. These are disabled by default and should be enabled.

There are three kinds of Data Access audit log information:

- Admin read: Records operations that read metadata or configuration information. Admin Activity audit logs record writes of metadata and configuration information that can't be disabled.
- Data read: Records operations that read user-provided data.
- Data write: Records operations that write user-provided data.

 It's recommended to have an effective default audit config configured in such a way that:

- Log type is set to DATA_READ (to log user activity tracking) and DATA_WRITES (to log changes/tampering to user data).
- Audit config is enabled for all the services supported by the Data Access audit logs feature.
- Logs should be captured for all users, that is, there are no exempted users in any of the audit config sections. This will ensure overriding the audit config will not contradict the requirement.

**Severity**: Medium

### Ensure that Cloud KMS crypto keys are not anonymously or publicly accessible

**Description**: It's recommended that the IAM policy on Cloud KMS crypto keys should restrict anonymous and/or public access.
 Granting permissions to "allUsers" or "allAuthenticatedUsers" allows anyone to access the dataset.
 Such access might not be desirable if sensitive data is stored at the location.
 In this case, ensure that anonymous and/or public access to a Cloud KMS crypto key isn't allowed.

**Severity**: High

### Ensure that corporate login credentials are used

**Description**: Use corporate login credentials instead of personal accounts, such as Gmail accounts.
 It's recommended fully managed corporate Google accounts be used for increased visibility, auditing, and controlling access to Cloud Platform resources.
 Gmail accounts based outside of the user's organization, such as personal accounts, shouldn't be used for business purposes.

**Severity**: High

### Ensure that IAM users are not assigned the Service Account User or Service Account Token Creator roles at project level

**Description**: It's recommended to assign the "Service Account User (iam.serviceAccountUser)" and "Service Account Token Creator (iam.serviceAccountTokenCreator)" roles to a user for a specific service account rather than assigning the role to a user at project level.
 A service account is a special Google account that belongs to an application or a virtual machine (VM), instead of to an individual end-user.
 Application/VM-Instance uses the service account to call the service's Google API so that users aren't directly involved.
 In addition to being an identity, a service account is a resource that has IAM policies attached to it. These policies determine who can use the service account.
 Users with IAM roles to update the App Engine and Compute Engine instances (such as App Engine Deployer or Compute Instance Admin) can effectively run code as the service accounts used to run these instances, and indirectly gain access to all the resources for which the service accounts have access.
 Similarly, SSH access to a Compute Engine instance might also provide the ability to execute code as that instance/Service account.
 Based on business needs, there could be multiple user-managed service accounts configured for a project.
 Granting the "iam.serviceAccountUser" or "iam.serviceAserviceAccountTokenCreatorccountUser" roles to a user for a project gives the user access to all service accounts in the project, including service accounts that might be created in the future.
 This can result in elevation of privileges by using service accounts and corresponding "Compute Engine instances."
 In order to implement "least privileges" best practices, IAM users shouldn't be assigned the "Service Account User" or "Service Account Token Creator" roles at the project level. Instead, these roles should be assigned to a user for a specific service account, giving that user access to the service account. The "Service Account User" allows a user to bind a service account to a long-running job service, whereas the "Service Account Token Creator" role allows a user to directly impersonate (or assert) the identity of a service account.

**Severity**: Medium

### Ensure that Separation of duties is enforced while assigning KMS related roles to users

**Description**: It's recommended that the principle of 'Separation of Duties' is enforced while assigning KMS related roles to users.
 The built-in/predefined IAM role "Cloud KMS Admin" allows the user/identity to create, delete, and manage service account(s).
 The built-in/predefined IAM role ```Cloud KMS CryptoKey Encrypter/Decrypter``` allows the user/identity (with adequate privileges on concerned resources) to encrypt and decrypt data at rest using an encryption key(s).
 The built-in/predefined IAM role ```Cloud KMS CryptoKey Encrypter``` allows the user/identity (with adequate privileges on concerned resources) to encrypt data at rest using an encryption key(s).
 The built-in/predefined IAM role ```Cloud KMS Crypto Key Decrypter``` allows the user/identity (with adequate privileges on concerned resources) to decrypt data at rest using an encryption key(s).
 Separation of duties is the concept of ensuring that one individual doesn't have all necessary permissions to be able to complete a malicious action.
 In Cloud KMS, this could be an action such as using a key to access and decrypt data a user shouldn't normally have access to.
 Separation of duties is a business control typically used in larger organizations, meant to help avoid security or privacy incidents and errors.
 It's considered best practice. No user(s) should have Cloud KMS Admin and any of the ```Cloud KMS CryptoKey Encrypter/Decrypter```, ```Cloud KMS CryptoKey Encrypter```, ```Cloud KMS CryptoKey Decrypter``` roles assigned at the same time.

**Severity**: High

### Ensure that Separation of duties is enforced while assigning service account related roles to users

**Description**: It's recommended that the principle of 'Separation of Duties' is enforced while assigning service-account related roles to users.
 The built-in/predefined IAM role "Service Account admin" allows the user/identity to create, delete, and manage service account(s).
 The built-in/predefined IAM role "Service Account User" allows the user/identity (with adequate privileges on Compute and App Engine) to assign service account(s) to Apps/Compute Instances.
 Separation of duties is the concept of ensuring that one individual doesn't have all necessary permissions to be able to complete a malicious action.
 In Cloud IAM - service accounts, this could be an action such as using a service account to access resources that user shouldn't normally have access to.
 Separation of duties is a business control typically used in larger organizations, meant to help avoid security or privacy incidents and errors. It's considered best practice.
 No user should have "Service Account Admin" and "Service Account User" roles assigned at the same time.

**Severity**: Medium

### Ensure that Service Account has no Admin privileges

**Description**: A service account is a special Google account that belongs to an application or a VM, instead of to an individual end-user.
 The application uses the service account to call the service's Google API so that users aren't directly involved.
 It's recommended not to use admin access for ServiceAccount.
 Service accounts represent service-level security of the Resources (application or a VM) which can be determined by the roles assigned to it.
 Enrolling ServiceAccount with Admin rights gives full access to an assigned application or a VM.
 A ServiceAccount Access holder can perform critical actions like delete, update change settings, etc.
 without user intervention.
 For this reason, it's recommended that service accounts not have Admin rights.

**Severity**: Medium

### Ensure that sinks are configured for all log entries

**Description**: It's recommended to create a sink that will export copies of all the log entries. This can help aggregate logs from multiple projects and export them to a Security Information and Event Management (SIEM).
 Log entries are held in Stackdriver Logging. To aggregate logs, export them to a SIEM. To keep them longer, it's recommended to set up a log sink. Exporting involves writing a filter that selects the log entries to export, and choosing a destination in Cloud Storage, BigQuery, or Cloud Pub/Sub.
 The filter and destination are held in an object called a sink. To ensure all log entries are exported to sinks, ensure that there's no filter configured for a sink. Sinks can be created in projects, organizations, folders, and billing accounts.

**Severity**: Low

### Ensure that the log metric filter and alerts exist for Audit Configuration changes

**Description**: Google Cloud Platform (GCP) services write audit log entries to Administrator Activity and Data Access logs. Entries help to answer the questions of, "who did what, where, and when?" within GCP projects.
Cloud audit logging records information includes the identity of the API caller, the time of the API call, the source IP address of the API caller, the request parameters, and the response elements returned by GCP services. Cloud audit logging provides a history of GCP API calls for an account, including API calls made via the console, SDKs, command-line tools, and other GCP services.
Administrator activity and data access logs produced by cloud audit logging enable security analysis, resource change tracking, and compliance auditing.
Configuring the metric filter and alerts for audit configuration changes ensures the recommended state of audit configuration is maintained so that all activities in the project are audit-able at any point in time.

**Severity**: Low

### Ensure that the log metric filter and alerts exist for Custom Role changes

**Description**: It's recommended that a metric filter and alarm be established for changes to Identity and Access Management (IAM) role creation, deletion, and updating activities.
Google Cloud IAM provides predefined roles that give granular access to specific Google Cloud Platform resources and prevent unwanted access to other resources. However, to cater to organization-specific needs, Cloud IAM also provides the ability to create custom roles. Project owners and administrators with the Organization Role Administrator role or the IAM Role Administrator role can create custom roles. Monitoring role creation, deletion and updating activities will help in identifying any over-privileged role at early stages.

**Severity**: Low

### Ensure user-managed/external keys for service accounts are rotated every 90 days or less

**Description**: Service Account keys consist of a key ID (Private_key_Id) and Private key, which are used to sign programmatic requests users make to Google cloud services accessible to that particular service account.
 It's recommended that all Service Account keys are regularly rotated.
 Rotating Service Account keys will reduce the window of opportunity for an access key that is associated with a compromised or terminated account to be used. Service Account keys should be rotated to ensure that data can't be accessed with an old key that might have been lost, cracked, or stolen.
 Each service account is associated with a key pair managed by Google Cloud Platform (GCP). It's used for service-to-service authentication within GCP. Google rotates the keys daily.
 GCP provides the option to create one or more user-managed (also called external key pairs) key pairs for use from outside GCP (for example, for use with Application Default Credentials). When a new key pair is created, the user is required to download the private key (which isn't retained by Google).

With external keys, users are responsible for keeping the private key secure and other management operations such as key rotation. External keys can be managed by the IAM API, gcloud command-line tool, or the Service Accounts page in the Google Cloud Platform Console.

GCP facilitates up to 10 external service account keys per service account to facilitate key rotation.

**Severity**: Medium

### GCP overprovisioned identities should have only the necessary permissions

**Description**: An over-provisioned active identity is an identity that has access to privileges that they haven't used. Over-provisioned active identities, especially for nonhuman accounts that have very defined actions and responsibilities, can increase the blast radius in the event of a user, key, or resource compromise The principle of least privilege states that a resource should only have access to the exact resources it needs in order to function. This principle was developed to address the risk of compromised identities granting an attacker access to a wide range of resources.

### GKE web dashboard should be disabled

**Description**: This recommendation evaluates the kubernetesDashboard field of the addonsConfig property for the key-value pair, 'disabled': false.

**Severity**: High

### Legacy Authorization should be disabled on GKE clusters

**Description**: This recommendation evaluates the legacyAbac property of a cluster for the key-value pair, 'enabled': true.

**Severity**: High

### Permissions of inactive identities in your GCP project should be revoked

**Description**: Microsoft Defender for Cloud discovered an identity that has not performed any action on any resource within your GCP project in the past 45 days. It is recommended to revoke permissions of inactive identities, in order to reduce the attack surface of your cloud environment.

**Severity**: Medium

### Redis IAM role should not be assigned at the organization or folder level

**Description**: This recommendation evaluates the IAM allow policy in resource metadata for principals assigned roles/redis.admin, roles/redis.editor, roles/redis.viewer at the organization or folder level.

**Severity**: High

### Service accounts should have restricted project access in a cluster

**Description**: This recommendation evaluates the config property of a node pool to check if no service account is specified or if the default service account is used.

**Severity**: High

### Users should have least privilege access with granular IAM roles

**Description**: This recommendation evaluates the IAM policy in resource metadata for any principals assigned roles/Owner, roles/Writer, or roles/Reader.

**Severity**: High

## Related content

- [Learn about security recommendations](security-policy-concept.md)
- [Review security recommendations](review-security-recommendations.md)
