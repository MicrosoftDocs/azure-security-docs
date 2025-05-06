---
title: Connect your AWS account
description: Defend your AWS resources with Microsoft Defender for Cloud, a guide to set up and configure Defender for Cloud to protect your workloads in AWS.
ms.topic: install-set-up-deploy
ms.date: 05/04/2025
---

# Connect AWS accounts to Microsoft Defender for Cloud

Workloads often span multiple cloud platforms, so cloud security services must do the same. Microsoft Defender for Cloud helps protect workloads in Amazon Web Services (AWS), but you need to set up the connection between them and Defender for Cloud.

The following screenshot shows AWS accounts displayed in the Defender for Cloud [overview dashboard](overview-page.md).

:::image type="content" source="./media/quickstart-onboard-aws/aws-account-in-overview.png" alt-text="Screenshot that shows four AWS projects listed on the overview dashboard in Defender for Cloud." lightbox="./media/quickstart-onboard-aws/aws-account-in-overview.png":::

You can learn more by watching the [New AWS connector in Defender for Cloud](episode-one.md) video from the *Defender for Cloud in the Field* video series.

## AWS authentication process

Defender for Cloud and AWS use federated authentication. All resources related to authentication are created as part of the CloudFormation template deployment, including:

- An identity provider (OpenID Connect)
- Identity and Access Management (IAM) roles with a federated principal (connected to the identity providers)

The architecture of the authentication process across clouds includes:

:::image type="content" source="media/quickstart-onboard-aws/architecture-authentication-across-clouds.png" alt-text="Diagram showing architecture of authentication process across clouds.":::

Defender for Cloud CSPM service acquires an Entra token with a validity lifetime of 1 hour, signed by Entra ID using the RS256 algorithm.

The Entra token is exchanged for AWS short-living credentials, and Defender for Cloud's CSPM service assumes the CSPM IAM role (assumed with web identity).

Since the principal of the role is a federated identity defined in a trust relationship policy, the AWS identity provider validates the Entra token against Entra ID through a process that includes:

- audience validation
- token digital signature validation
- certificate thumbprint

The Defender for Cloud CSPM role is assumed only after the validation conditions defined at the trust relationship have been met. The conditions defined for the role level are used for validation within AWS and allows only the Microsoft Defender for Cloud CSPM application (validated audience) access to the specific role (and not any other Microsoft token).

After the Entra token is validated by the AWS identity provider, the AWS STS exchanges the token with AWS short-living credentials which the CSPM service uses to scan the AWS account.

## Prerequisites

To complete the procedures in this article, you need:

- A Microsoft Azure subscription. If you don't have an Azure subscription, you can [sign up for a free one](https://azure.microsoft.com/pricing/free-trial/).

- [Microsoft Defender for Cloud](get-started.md#enable-defender-for-cloud-on-your-azure-subscription) set up on your Azure subscription.

- Access to an AWS account.

- Contributor level permission for the relevant Azure subscription.
  
- If CIEM is enabled as part of Defender for CSPM the user enabling the connector will also need [Security Admin role and Application.ReadWrite.All permission](enable-permissions-management.md#before-you-start) for your tenant.

> [!NOTE]
> The AWS connector isn't available on the national government clouds (Azure Government, Microsoft Azure operated by 21Vianet).

## Native connector plan requirements

Each plan has its own requirements for the native connector.

### [Defender for Containers](#tab/Defender-for-Containers)

If you choose the Microsoft Defender for Containers plan, you need:

- At least one Amazon EKS cluster with permission to access to the EKS Kubernetes API server. If you need to create a new EKS cluster, follow the instructions in [Getting started with Amazon EKS – eksctl](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html).
- The resource capacity to create a new Amazon SQS queue, ```Kinesis Data Firehose``` delivery stream, and Amazon S3 bucket in the cluster's region.

### [Defender for SQL](#tab/Defender-for-SQL)

If you choose the Microsoft Defender for SQL plan, you need:

- Microsoft Defender for SQL enabled on your subscription. [Learn how to protect your databases](tutorial-enable-databases-plan.md).
- An active AWS account, with EC2 instances running SQL Server or RDS Custom for SQL Server.
- Azure Arc for servers installed on your EC2 instances or RDS Custom for SQL Server.

We recommend that you use the autoprovisioning process to install Azure Arc on all of your existing and future EC2 instances. To enable the Azure Arc autoprovisioning, you need **Owner** permission on the relevant Azure subscription.

AWS Systems Manager (SSM) uses the SSM Agent to handle automatic provisioning. Some Amazon Machine Images already have the [SSM Agent preinstalled](https://docs.aws.amazon.com/systems-manager/latest/userguide/ami-preinstalled-agent.html). If your EC2 instances don't have the SSM Agent, install it by using these instructions from Amazon: [Install SSM Agent for a hybrid and multicloud environment (Windows)](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-install-managed-win.html).

Ensure that your SSM Agent has the managed policy [AmazonSSMManagedInstanceCore](https://docs.aws.amazon.com/aws-managed-policy/latest/reference/AmazonSSMManagedInstanceCore.html). It enables core functionality for the AWS Systems Manager service.

Enable these other extensions on the Azure Arc-connected machines:

- Microsoft Defender for Endpoint
- A vulnerability assessment solution (TVM or Qualys)
- The Log Analytics agent on Azure Arc-connected machines or the Azure Monitor agent

Make sure the selected Log Analytics workspace has a security solution installed. The Log Analytics agent and the Azure Monitor agent are currently configured at the *subscription* level. All of your AWS accounts and Google Cloud Platform (GCP) projects under the same subscription inherit the subscription settings for the Log Analytics agent and the Azure Monitor agent.

[Learn more about monitoring components](monitoring-components.md) for Defender for Cloud.

### [Defender for open-source databases (Preview)](#tab/Defender-for-open-source-databases-Preview)

If you choose the Defender for open-source relational databases plan, you need:

- You need a Microsoft Azure subscription. If you don't have an Azure subscription, you can [sign up for a free subscription](https://azure.microsoft.com/pricing/free-trial/).

- You must [enable Microsoft Defender for Cloud](get-started.md#enable-defender-for-cloud-on-your-azure-subscription) on your Azure subscription.

- Connect your [Azure account](connect-azure-subscription.md) or AWS account.

Region availability: All public AWS regions (excluding Tel Aviv, Milan, Jakarta, Spain, and Bahrain).

### [Defender for Servers](#tab/Defender-for-Servers)

If you choose the Microsoft Defender for Servers plan, you need:

- Microsoft Defender for Servers enabled on your subscription. Learn how to enable plans in [Enable enhanced security features](enable-enhanced-security.md).
- An active AWS account, with EC2 instances.
- Azure Arc for servers installed on your EC2 instances.

We recommend that you use the autoprovisioning process to install Azure Arc on all of your existing and future EC2 instances. To enable the Azure Arc autoprovisioning, you need **Owner** permission on the relevant Azure subscription.

AWS Systems Manager automatically provisions using the SSM Agent. Some Amazon Machine Images already have the [SSM Agent preinstalled](https://docs.aws.amazon.com/systems-manager/latest/userguide/ami-preinstalled-agent.html). If your EC2 instances don't have the SSM Agent, install it by using either of the following instructions from Amazon:

- [Install SSM Agent for a hybrid and multicloud environment (Windows)](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-install-managed-win.html)
- [Install SSM Agent for a hybrid and multicloud environment (Linux)](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-install-managed-linux.html)

Ensure that your SSM Agent has the managed policy [AmazonSSMManagedInstanceCore](https://docs.aws.amazon.com/aws-managed-policy/latest/reference/AmazonSSMManagedInstanceCore.html), which enables core functionality for the AWS Systems Manager service.

**You must have the SSM Agent for auto provisioning Arc agent on EC2 machines. If the SSM doesn't exist, or is removed from the EC2, the Arc provisioning won't be able to proceed.**

> [!NOTE]
> As part of the CloudFormation template that is run during the onboarding process, an automation process is created and triggered every 30 days, over all the EC2s that existed during the initial run of the CloudFormation. The goal of this scheduled scan is to ensure that all the relevant EC2s have an IAM profile with the required IAM policy that allows Defender for Cloud to access, manage, and provide the relevant security features (including the Arc agent provisioning). The scan doesn't apply to EC2s that were created after the run of the CloudFormation.

If you want to manually install Azure Arc on your existing and future EC2 instances, use the [EC2 instances should be connected to Azure Arc](https://portal.azure.com/#blade/Microsoft_Azure_Security/RecommendationsBlade/assessmentKey/231dee23-84db-44d2-bd9d-c32fbcfb42a3) recommendation to identify instances that don't have Azure Arc installed.

Enable these other extensions on the Azure Arc-connected machines:

- Microsoft Defender for Endpoint
- A vulnerability assessment solution (TVM or Qualys)
- The Log Analytics agent on Azure Arc-connected machines or the Azure Monitor agent

Make sure the selected Log Analytics workspace has a security solution installed. The Log Analytics agent and the Azure Monitor agent are currently configured at the *subscription* level. All of your AWS accounts and GCP projects under the same subscription inherit the subscription settings for the Log Analytics agent and the Azure Monitor agent.

[Learn more about monitoring components](monitoring-components.md) for Defender for Cloud.

> [!NOTE]
> As the Log Analytics agent (also known as MMA) is set to retire in [August 2024](https://azure.microsoft.com/updates/were-retiring-the-log-analytics-agent-in-azure-monitor-on-31-august-2024/), all Defender for Servers features and security capabilities that currently depend on it, including those described on this page, will be available through either [Microsoft Defender for Endpoint integration](integration-defender-for-endpoint.md) or [agentless scanning](concept-agentless-data-collection.md), before the retirement date. For more information about the roadmap for each of the features that are currently rely on Log Analytics Agent, see [this announcement](upcoming-changes.md#defender-for-cloud-plan-and-strategy-for-the-log-analytics-agent-deprecation).

Defender for Servers assigns tags to your Azure ARC resources on top of your EC2 instances to manage the autoprovisioning process. You must have these tags properly assigned to your resources so that Defender for Cloud can manage them: `AccountId`, `Cloud`, `InstanceId`, and `MDFCSecurityConnector`.

### [Defender CSPM](#tab/Defender-for-CSPM)

If you choose the Microsoft Defender CSPM plan, you need:

- a Microsoft Azure subscription. If you don't have an Azure subscription, you can [sign up for a free subscription](https://azure.microsoft.com/pricing/free-trial/).
- You must [enable Microsoft Defender for Cloud](get-started.md#enable-defender-for-cloud-on-your-azure-subscription) on your Azure subscription.
- Connect your [non-Azure machines](quickstart-onboard-machines.md) and AWS accounts.
- In order to gain access to all of the features available from the CSPM plan, the plan must be enabled by the **Subscription Owner**.
- To enable CIEM (Cloud Infrastructure Entitlement Management) capabilities, the Entra ID account used for the onboarding process must have either the Application Administrator or Cloud Application Administrator directory role for your tenant (or equivalent administrator rights to create app registrations). This requirement is only necessary during the onboarding process.

Learn more about how to [enable Defender CSPM](tutorial-enable-cspm-plan.md).

---

## Connect your AWS account

> [!IMPORTANT]
> If your AWS account is already connected to Microsoft Sentinel, you cannot connect it to Defender for Cloud. To ensure the connector works correctly, follow the instructions on [Connect a Sentinel connected AWS account to Defender for Cloud](sentinel-connected-aws.md).

To connect your AWS to Defender for Cloud by using a native connector:

1. Sign in to the [Azure portal](https://portal.azure.com).

1. Go to **Defender for Cloud** > **Environment settings**.

1. Select **Add environment** > **Amazon Web Services**.

    :::image type="content" source="media/quickstart-onboard-aws/add-aws-account-environment-settings.png" alt-text="Screenshot that shows connecting an AWS account to an Azure subscription." lightbox="media/quickstart-onboard-aws/add-aws-account-environment-settings.png":::

1. Enter the details of the AWS account, including the location where you store the connector resource.

    :::image type="content" source="media/quickstart-onboard-aws/add-aws-account-details.png" alt-text="Screenshot that shows the tab for entering account details for an AWS account." lightbox="media/quickstart-onboard-aws/add-aws-account-details.png":::

    The **AWS regions** dropdown allows you to select the regions to which Defender for Cloud makes API calls. Every region that is deselected from the dropdown, implies that Defender for Cloud won’t make API calls to those regions.

1. Select an interval to scan the AWS environment every 4, 6, 12, or 24 hours.

    Some data collectors run with fixed scan intervals and aren't affected by custom interval configurations. The following table shows the fixed scan intervals for each excluded data collector:

    | Data collector name | Scan interval |
    |--|--|
    | EC2Instance <br> ECRImage  <br> ECRRepository  <br> RDSDBInstance <br> S3Bucket  <br> S3BucketTags <br> S3Region <br> EKSCluster <br> EKSClusterName <br> EKSNodegroup <br> EKSNodegroupName <br> AutoScalingAutoScalingGroup  | 1 hour  |
    | EcsClusterArn <br> EcsService <br> EcsServiceArn <br> EcsTaskDefinition <br> EcsTaskDefinitionArn <br> EcsTaskDefinitionTags <br> AwsPolicyVersion <br> LocalPolicyVersion <br> AwsEntitiesForPolicy <br> LocalEntitiesForPolicy <br> BucketEncryption <br> BucketPolicy <br> S3PublicAccessBlockConfiguration <br> BucketVersioning <br> S3LifecycleConfiguration <br> BucketPolicyStatus <br> S3ReplicationConfiguration <br> S3AccessControlList <br> S3BucketLoggingConfig <br> PublicAccessBlockConfiguration | 12 hours |

    > [!NOTE]
    > (Optional) Select **Management account** to create a connector to a management account. Connectors are then created for each member account discovered under the provided management account. Autoprovisioning is also enabled for all of the newly onboarded accounts.
    >
    > (Optional) Use the AWS regions dropdown menu to select specific AWS regions to be scanned. All regions are selected by default.

Next, review and select the Defender for Cloud plans to enable for this AWS account.

## Select Defender plans

In this section of the wizard, you select the Defender for Cloud plans that you want to enable.

1. Select **Next: Select plans**.

    The **Select plans** tab is where you choose which Defender for Cloud capabilities to enable for this AWS account. Each plan has its own [requirements for permissions](#native-connector-plan-requirements) and might incur [charges](https://azure.microsoft.com/pricing/details/defender-for-cloud/?v=17.23h).

    :::image type="content" source="media/quickstart-onboard-aws/add-aws-account-plans-selection.png" alt-text="Screenshot that shows the tab for selecting plans for an AWS account." lightbox="media/quickstart-onboard-aws/add-aws-account-plans-selection.png":::

    > [!IMPORTANT]
    > To present the current status of your recommendations, the Microsoft Defender Cloud Security Posture Management plan queries the AWS resource APIs several times a day. These read-only API calls incur no charges, but they're registered in CloudTrail if you enable a trail for read events.
    >
    > [AWS's documentation](https://aws.amazon.com/cloudtrail/pricing/) explains that there are no extra charges for keeping one trail. If you're exporting the data out of AWS (for example, to an external SIEM system), this increased volume of calls might also increase ingestion costs. In such cases, we recommend filtering out the read-only calls from the Defender for Cloud user or ARN role: `arn:aws:iam::[accountId]:role/CspmMonitorAws`. (This is the default role name. Confirm the role name configured on your account.)

1. By default, the **Servers** plan is set to **On**. This setting is necessary to extend the coverage of Defender for Servers to AWS EC2. Ensure that you fulfilled the [network requirements for Azure Arc](/azure/azure-arc/servers/network-requirements?tabs=azure-cloud).

    Optionally, select **Configure** to edit the configuration as required.

    > [!NOTE]
    > The respective Azure Arc servers for EC2 instances or GCP virtual machines that no longer exist (and the respective Azure Arc servers with a status of [Disconnected or Expired](/azure/azure-arc/servers/overview)) are removed after seven days. This process removes irrelevant Azure Arc entities to ensure that only Azure Arc servers related to existing instances are displayed.

1. By default, the **Containers** plan is set to **On**. This setting is necessary to have Defender for Containers protect your AWS EKS clusters. Ensure that you fulfilled the [network requirements](./defender-for-containers-enable.md?pivots=defender-for-container-eks&source=docs&tabs=aks-deploy-portal%2ck8s-deploy-asc%2ck8s-verify-asc%2ck8s-remove-arc%2caks-removeprofile-api#network-requirements) for the Defender for Containers plan.

    > [!NOTE]
    > Azure Arc-enabled Kubernetes, the Azure Arc extensions for Defender sensor, and Azure Policy for Kubernetes should be installed. Use the dedicated Defender for Cloud recommendations to deploy the extensions (and Azure Arc, if necessary), as explained in [Protect Amazon Elastic Kubernetes Service clusters](defender-for-containers-enable.md?tabs=defender-for-container-eks).

    Optionally, select **Configure** to edit the configuration as required. If you choose to turn off this configuration, the **Threat detection (control plane)** feature is also disabled. [Learn more about feature availability](supported-machines-endpoint-solutions-clouds-containers.md).

1. By default, the **Databases** plan is set to **On**. This setting is necessary to extend coverage of Defender for SQL to AWS EC2 and RDS Custom for SQL Server and open-source relational databases on RDS.

    (Optional) Select **Configure** to edit the configuration as required. We recommend that you leave it set to the default configuration.

1. Select **Configure access** and select the following:

    a. Select a deployment type:

    - **Default access**: Allows Defender for Cloud to scan your resources and automatically include future capabilities.
    - **Least privilege access**: Grants Defender for Cloud access only to the current permissions needed for the selected plans. If you select the least privileged permissions, you receive notifications on any new roles and permissions that are required to get full functionality for connector health.

    b. Select a deployment method: **AWS CloudFormation** or **Terraform**.

    :::image type="content" source="media/quickstart-onboard-aws/add-aws-account-configure-access.png" alt-text="Screenshot that shows deployment options and instructions for configuring access." lightbox="media/quickstart-onboard-aws/add-aws-account-configure-access.png":::

    > [!NOTE]
    > If you select **Management account** to create a connector to a management account, then the tab to onboard with Terraform isn't visible in the UI, but you can still onboard using Terraform, similar to what's covered at [Onboarding your AWS/GCP environment to Microsoft Defender for Cloud with Terraform - Microsoft Community Hub](https://techcommunity.microsoft.com/t5/microsoft-defender-for-cloud/onboarding-your-aws-gcp-environment-to-microsoft-defender-for/ba-p/3798664).

1. Follow the on-screen instructions for the selected deployment method to complete the required dependencies on AWS. If you're onboarding a management account, you need to run the CloudFormation template both as Stack and as StackSet. Connectors are created for the member accounts up to 24 hours after the onboarding.

1. Select **Next: Review and generate**.

1. Select **Create**.

Defender for Cloud immediately starts scanning your AWS resources. Security recommendations appear within a few hours.

## Deploy a CloudFormation template to your AWS account

As part of connecting an AWS account to Microsoft Defender for Cloud, you deploy a CloudFormation template to the AWS account. This template creates all of the required resources for the connection.

Deploy the CloudFormation template by using Stack (or StackSet if you have a management account). When you're deploying the template, the Stack creation wizard offers the following options.

:::image type="content" source="media/quickstart-onboard-aws/cloudformation-template.png" alt-text="Screenshot that shows the Stack creation wizard with options for template sources." lightbox="media/quickstart-onboard-aws/cloudformation-template.png":::

- **Amazon S3 URL**: Upload the downloaded CloudFormation template to your own S3 bucket with your own security configurations. Enter the URL to the S3 bucket in the AWS deployment wizard.

- **Upload a template file**: AWS automatically creates an S3 bucket that the CloudFormation template is saved to. The automation for the S3 bucket has a security misconfiguration that causes the `S3 buckets should require requests to use Secure Socket Layer` recommendation to appear. You can remediate this recommendation by applying the following policy:

    ```bash
    { 
      "Id": "ExamplePolicy", 
      "Version": "2012-10-17", 
      "Statement": [ 
        { 
          "Sid": "AllowSSLRequestsOnly", 
          "Action": "s3:*", 
          "Effect": "Deny", 
          "Resource": [ 
            "<S3_Bucket ARN>", 
            "<S3_Bucket ARN>/*" 
          ], 
          "Condition": { 
            "Bool": { 
              "aws:SecureTransport": "false" 
            } 
          }, 
          "Principal": "*" 
        } 
      ] 
    } 
    ```

    > [!NOTE]
    > When running the CloudFormation StackSets when onboarding an AWS management account, you might encounter the following error message:
    > `You must enable organizations access to operate a service managed stack set`
    >
    > This error indicates that you haven't enabled [the trusted access for AWS Organizations](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacksets-orgs-activate-trusted-access.html).
    >
    > To remediate this error message, your CloudFormation StackSets page has a prompt with a button that you can select to enable trusted access. After trusted access is enabled, the CloudFormation Stack must be run again.

## Monitor your AWS resources

The security recommendations page in Defender for Cloud displays your AWS resources. You can use the environments filter to enjoy multicloud capabilities in Defender for Cloud.

To view all the active recommendations for your resources by resource type, use the asset inventory page in Defender for Cloud and filter to the AWS resource type that you're interested in.

:::image type="content" source="./media/quickstart-onboard-aws/aws-resource-types-in-inventory.png" alt-text="Screenshot of AWS options in the asset inventory page's resource type filter." lightbox="media/quickstart-onboard-aws/aws-resource-types-in-inventory.png":::

## Integrate with Microsoft Defender XDR

When you enable Defender for Cloud, its security alerts are automatically integrated into the Microsoft Defender Portal.

The integration between Microsoft Defender for Cloud and Microsoft Defender XDR brings your cloud environments into Microsoft Defender XDR. With Defender for Cloud's alerts and cloud correlations integrated into Microsoft Defender XDR, SOC teams can now access all security information from a single interface.

Learn more about Defender for Cloud's [alerts in Microsoft Defender XDR](concept-integration-365.md).

## Learn more

Check out the following blogs:

- [Ignite 2021: Microsoft Defender for Cloud news](https://techcommunity.microsoft.com/t5/microsoft-defender-for-cloud/ignite-2021-microsoft-defender-for-cloud-news/ba-p/2882807)
- [Security posture management and server protection for AWS and GCP](https://techcommunity.microsoft.com/t5/microsoft-defender-for-cloud/security-posture-management-and-server-protection-for-aws-and/ba-p/3271388)

## Clean up resources

There's no need to clean up any resources for this article.

## Next steps

Connecting your AWS account is part of the multicloud experience available in Microsoft Defender for Cloud:

- [Assign access to workload owners](assign-access-to-workload.md).
- [Protect all of your resources with Defender for Cloud](enable-all-plans.md).
- Set up your [on-premises machines](quickstart-onboard-machines.md) and [GCP projects](quickstart-onboard-gcp.md).
- Get answers to [common questions](faq-general.yml) about onboarding your AWS account.
- [Troubleshoot your multicloud connectors](troubleshooting-guide.md#troubleshoot-connectors).
