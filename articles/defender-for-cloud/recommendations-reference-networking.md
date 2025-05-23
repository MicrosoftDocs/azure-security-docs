---
title: Reference table for all networking security recommendations 
description: This article lists all Microsoft Defender for Cloud networking security recommendations that help you harden and protect your resources.
author: dcurwin
ms.service: defender-for-cloud
ms.topic: reference
ms.date: 05/18/2025
ms.author: dacurwin
ms.custom: generated
ai-usage: ai-assisted
---

# Networking security recommendations

This article lists all the networking security recommendations you might see in Microsoft Defender for Cloud.

The recommendations that appear in your environment are based on the resources that you're protecting and on your customized configuration. You can [see the recommendations in the portal](https://portal.azure.com/#view/Microsoft_Azure_Security/SecurityMenuBlade/~/5) that apply to your resources.

To learn about actions that you can take in response to these recommendations, see [Remediate recommendations in Defender for Cloud](implement-security-recommendations.md).

> [!TIP]
> If a recommendation description says *No related policy*, usually it's because that recommendation is dependent on a different recommendation.
>
> For example, the recommendation *Endpoint protection health failures should be remediated* relies on the recommendation that checks whether an endpoint protection solution is installed (*Endpoint protection solution should be installed*). The underlying recommendation *does* have a policy.
> Limiting policies to only foundational recommendations simplifies policy management.



## Azure networking recommendations

### Access to storage accounts with firewall and virtual network configurations should be restricted

**Description**: Review the settings of network access in your storage account firewall settings. We recommended configuring network rules so that only applications from allowed networks can access the storage account. To allow connections from specific internet or on-premises clients, access can be granted to traffic from specific Azure virtual networks or to public internet IP address ranges.
(Related policy: [Storage accounts should restrict network access](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f34c877ad-507e-4c82-993e-3452a6e0ad3c)).

**Severity**: Low

### Adaptive network hardening recommendations should be applied on internet facing virtual machines

**Description**: Defender for Cloud has analyzed the internet traffic communication patterns of the virtual machines listed below, and determined that the existing rules in the NSGs associated to them are overly permissive, resulting in an increased potential attack surface.
This typically occurs when this IP address doesn't communicate regularly with this resource. Alternatively, the IP address has been flagged as malicious by Defender for Cloud's threat intelligence sources.
(Related policy: [Adaptive network hardening recommendations should be applied on internet facing virtual machines](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f08e6af2d-db70-460a-bfe9-d5bd474ba9d6)).

**Severity**: High

### All network ports should be restricted on network security groups associated to your virtual machine

**Description**: Defender for Cloud has identified some of your network security groups' inbound rules to be too permissive. Inbound rules should not allow access from 'Any' or 'Internet' ranges. This can potentially enable attackers to target your resources.
(Related policy: [All network ports should be restricted on network security groups associated to your virtual machine](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f9daedab3-fb2d-461e-b861-71790eead4f6)).

**Severity**: High

### Azure DDoS Protection Standard should be enabled

**Description**: Defender for Cloud has discovered virtual networks with Application Gateway resources unprotected by the DDoS protection service. These resources contain public IPs. Enable mitigation of network volumetric and protocol attacks.
(Related policy: [Azure DDoS Protection Standard should be enabled](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2fa7aca53f-2ed4-4466-a25e-0b45ade68efd)).

**Severity**: Medium

### Internet-facing virtual machines should be protected with network security groups

**Description**: Protect your VM from potential threats by restricting access to it with a network security group (NSG). NSGs contain a list of Access Control List (ACL) rules that allow or deny network traffic to your VM from other instances, in or outside the same subnet.
To keep your machine as secure as possible, the VM access to the internet must be restricted and an NSG should be enabled on the subnet.
VMs with 'High' severity are internet-facing VMs.
(Related policy: [Internet-facing virtual machines should be protected with network security groups](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2ff6de0be7-9a8a-4b8a-b349-43cf02d22f7c)).

**Severity**: High

### IP forwarding on your virtual machine should be disabled

**Description**: Defender for Cloud has discovered that IP forwarding is enabled on some of your virtual machines. Enabling IP forwarding on a virtual machine's NIC allows the machine to receive traffic addressed to other destinations. IP forwarding is rarely required (e.g., when using the VM as a network virtual appliance), and therefore, this should be reviewed by the network security team.
(Related policy: [IP Forwarding on your virtual machine should be disabled](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2fbd352bd5-2853-4985-bf0d-73806b4a5744)).

**Severity**: Medium

### Machines should have ports closed that might expose attack vectors

**Description**: [Azure's terms of use](https://www.microsoft.com/legal/terms-of-use) prohibit the use of Azure services in ways that could damage, disable, overburden, or impair any Microsoft server or the network. This recommendation lists exposed ports that need to be closed for your continued security. It also illustrates the potential threat to each port.
(No related policy)

**Severity**: High

### Management ports of virtual machines should be protected with just-in-time network access control

**Description**: Defender for Cloud has identified some overly permissive inbound rules for management ports in your Network Security Group. Enable just-in-time access control to protect your VM from internet-based brute-force attacks. Learn more in [Understanding just-in-time (JIT) VM access](just-in-time-access-overview.md).
(Related policy: [Management ports of virtual machines should be protected with just-in-time network access control](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2fb0f33259-77d7-4c9e-aac6-3aabcfae693c)).

**Severity**: High

### Management ports should be closed on your virtual machines

**Description**: Open remote management ports are exposing your VM to a high level of risk from Internet-based attacks. These attacks attempt to brute force credentials to gain admin access to the machine.
(Related policy: [Management ports should be closed on your virtual machines](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f22730e10-96f6-4aac-ad84-9383d35b5917)).

**Severity**: Medium

### Non-internet-facing virtual machines should be protected with network security groups

**Description**: Protect your non-internet-facing virtual machine from potential threats by restricting access to it with a network security group (NSG). NSGs contain a list of Access Control List (ACL) rules that allow or deny network traffic to your VM from other instances, whether or not they're on the same subnet.
Note that to keep your machine as secure as possible, the VM's access to the internet must be restricted and an NSG should be enabled on the subnet.
(Related policy: [Non-internet-facing virtual machines should be protected with network security groups](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2fbb91dfba-c30d-4263-9add-9c2384e659a6)).

**Severity**: Low

### Secure transfer to storage accounts should be enabled

**Description**: Secure transfer is an option that forces your storage account to accept requests only from secure connections (HTTPS). Use of HTTPS ensures authentication between the server and the service and protects data in transit from network layer attacks such as man-in-the-middle, eavesdropping, and session-hijacking.
(Related policy: [Secure transfer to storage accounts should be enabled](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f404c3081-a854-4457-ae30-26a93ef643f9)).

**Severity**: High

### (Enable if required) Subnets should be associated with a network security group

**Description**: Protect your subnet from potential threats by restricting access to it with a network security group (NSG). NSGs contain a list of Access Control List (ACL) rules that allow or deny network traffic to your subnet. When an NSG is associated with a subnet, the ACL rules apply to all the VM instances and integrated services in that subnet, but don't apply to internal traffic inside the subnet. To secure resources in the same subnet from one another, enable NSG directly on the resources as well.
Note that the following subnet types will be listed as not applicable: GatewaySubnet, AzureFirewallSubnet, AzureBastionSubnet.

To enable this recommendation, navigate to your Security Policy for the applicable scope, and update the *Effect* parameter for the corresponding policy to audit. Learn more in [Manage security policies](tutorial-security-policy.md).
(Related policy: [Subnets should be associated with a Network Security Group](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2fe71308d3-144b-4262-b144-efdc3cc90517)).

**Severity**: Low

### Virtual networks should be protected by Azure Firewall

**Description**: Some of your virtual networks aren't protected with a firewall. Use [Azure Firewall](https://azure.microsoft.com/pricing/details/azure-firewall) to restrict access to your virtual networks and prevent potential threats.
(Related policy: [All Internet traffic should be routed via your deployed Azure Firewall](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2ffc5e4038-4584-4632-8c85-c0448d374b2c)).



## AWS networking recommendations

### Amazon EC2 should be configured to use VPC endpoints

**Description**: This control checks whether a service endpoint for Amazon EC2 is created for each VPC. The control fails if a VPC doesn't have a VPC endpoint created for the Amazon EC2 service.
 To improve the security posture of your VPC, you can configure Amazon EC2 to use an interface VPC endpoint. Interface endpoints are powered by AWS PrivateLink, a technology that enables you to access Amazon EC2 API operations privately. It restricts all network traffic between your VPC and Amazon EC2 to the Amazon network. Because endpoints are supported within the same Region only, you can't create an endpoint between a VPC and a service in a different Region. This prevents unintended Amazon EC2 API calls to other Regions.
To learn more about creating VPC endpoints for Amazon EC2, see [Amazon EC2 and interface VPC endpoints](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/interface-vpc-endpoints.html) in the Amazon EC2 User Guide for Linux Instances.

**Severity**: Medium

### Amazon ECS services should not have public IP addresses assigned to them automatically

**Description**: A public IP address is an IP address that is reachable from the internet.
 If you launch your Amazon ECS instances with a public IP address, then your Amazon ECS instances are reachable from the internet.
 Amazon ECS services shouldn't be publicly accessible, as this might allow unintended access to your container application servers.

**Severity**: High

### Amazon EMR cluster master nodes should not have public IP addresses

**Description**: This control checks whether master nodes on Amazon EMR clusters have public IP addresses.
The control fails if the master node has public IP addresses that are associated with any of its instances. Public IP addresses are designated in the PublicIp field of the NetworkInterfaces configuration for the instance.
 This control only checks Amazon EMR clusters that are in a RUNNING or WAITING state.

**Severity**: High

### Amazon Redshift clusters should use enhanced VPC routing

**Description**: This control checks whether an Amazon Redshift cluster has EnhancedVpcRouting enabled.
Enhanced VPC routing forces all COPY and UNLOAD traffic between the cluster and data repositories to go through your VPC. You can then use VPC features such as security groups and network access control lists to secure network traffic. You can also use VPC Flow Logs to monitor network traffic.

**Severity**: High

### Application Load Balancer should be configured to redirect all HTTP requests to HTTPS

**Description**: To enforce encryption in transit, you should use redirect actions with Application Load Balancers to redirect client HTTP requests to an HTTPS request on port 443.

**Severity**: Medium

### Application load balancers should be configured to drop HTTP headers

**Description**: This control evaluates AWS Application Load Balancers (ALB) to ensure they're configured to drop invalid HTTP headers. The control fails if the value of routing.http.drop_invalid_header_fields.enabled is set to false.
By default, ALBs aren't configured to drop invalid HTTP header values. Removing these header values prevents HTTP desync attacks.

**Severity**: Medium

### Configure Lambda functions to a VPC

**Description**: This control checks whether a Lambda function is in a VPC. It doesn't evaluate the VPC subnet routing configuration to determine public reachability.
 Note that if Lambda@Edge is found in the account, then this control generates failed findings. To prevent these findings, you can disable this control.

**Severity**: Low

### EC2 instances should not have a public IP address

**Description**: This control checks whether EC2 instances have a public IP address. The control fails if the "publicIp" field is present in the EC2 instance configuration item. This control applies to IPv4 addresses only.
 A public IPv4 address is an IP address that is reachable from the internet. If you launch your instance with a public IP address, then your EC2 instance is reachable from the internet. A private IPv4 address is an IP address that isn't reachable from the internet. You can use private IPv4 addresses for communication between EC2 instances in the same VPC or in your connected private network.
IPv6 addresses are globally unique, and therefore are reachable from the internet. However, by default all subnets have the IPv6 addressing attribute set to false. For more information about IPv6, see [IP addressing in your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-ip-addressing.html) in the Amazon VPC User Guide.
If you have a legitimate use case to maintain EC2 instances with public IP addresses, then you can suppress the findings from this control. For more information about front-end architecture options, see the [AWS Architecture Blog](http://aws.amazon.com/blogs/architecture/) or the [This Is My Architecture series](http://aws.amazon.com/blogs/architecture/).

**Severity**: High

### EC2 instances should not use multiple ENIs

**Description**: This control checks whether an EC2 instance uses multiple Elastic Network Interfaces (ENIs) or Elastic Fabric Adapters (EFAs). This control passes if a single network adapter is used. The control includes an optional parameter list to identify the allowed ENIs.
Multiple ENIs can cause dual-homed instances, meaning instances that have multiple subnets. This can add network security complexity and introduce unintended network paths and access.

**Severity**: Low

### EC2 instances should use IMDSv2

**Description**: This control checks whether your EC2 instance metadata version is configured with Instance Metadata Service Version 2 (IMDSv2). The control passes if "HttpTokens" is set to "required" for IMDSv2. The control fails if "HttpTokens" is set to "optional".
You use instance metadata to configure or manage the running instance. The IMDS provides access to temporary, frequently rotated credentials. These credentials remove the need to hard code or distribute sensitive credentials to instances manually or programmatically. The IMDS is attached locally to every EC2 instance. It runs on a special 'link local' IP address of 169.254.169.254. This IP address is only accessible by software that runs on the instance.
Version 2 of the IMDS adds new protections for the following types of vulnerabilities. These vulnerabilities could be used to try to access the IMDS.

- Open website application firewalls
- Open reverse proxies
- Server-side request forgery (SSRF) vulnerabilities
- Open Layer 3 firewalls and network address translation (NAT)
Security Hub recommends that you configure your EC2 instances with IMDSv2.

**Severity**: High

### EC2 subnets should not automatically assign public IP addresses

**Description**: This control checks whether the assignment of public IPs in Amazon Virtual Private Cloud (Amazon VPC) subnets have "MapPublicIpOnLaunch" set to "FALSE". The control passes if the flag is set to "FALSE".
 All subnets have an attribute that determines whether a network interface created in the subnet automatically receives a public IPv4 address. Instances that are launched into subnets that have this attribute enabled have a public IP address assigned to their primary network interface.

**Severity**: Medium

### Ensure a log metric filter and alarm exist for AWS Config configuration changes

**Description**: Real-time monitoring of API calls can be achieved by directing CloudTrail Logs to CloudWatch Logs and establishing corresponding metric filters and alarms.
 It's recommended that a metric filter and alarm be established for detecting changes to CloudTrail's configurations.
Monitoring changes to AWS Config configuration helps ensure sustained visibility of configuration items within the AWS account.

**Severity**: Low

### Ensure a log metric filter and alarm exist for AWS Management Console authentication failures

**Description**: Real-time monitoring of API calls can be achieved by directing CloudTrail Logs to CloudWatch Logs and establishing corresponding metric filters and alarms.
 It's recommended that a metric filter and alarm be established for failed console authentication attempts.
 Monitoring failed console logins might decrease lead time to detect an attempt to brute force a credential, which might provide an indicator, such as source IP, that can be used in other event correlation.

**Severity**: Low

### Ensure a log metric filter and alarm exist for changes to Network Access Control Lists (NACL)

**Description**: Real-time monitoring of API calls can be achieved by directing CloudTrail Logs to CloudWatch Logs and establishing corresponding metric filters and alarms. NACLs are used as a stateless packet filter to control ingress and egress traffic for subnets within a VPC.
 It is recommended that a metric filter and alarm be established for changes made to NACLs.
Monitoring changes to NACLs helps ensure that AWS resources and services aren't unintentionally exposed.

**Severity**: Low

### Ensure a log metric filter and alarm exist for changes to network gateways

**Description**: Real-time monitoring of API calls can be achieved by directing CloudTrail Logs to CloudWatch Logs and establishing corresponding metric filters and alarms. Network gateways are required to send/receive traffic to a destination outside of a VPC.
 It's recommended that a metric filter and alarm be established for changes to network gateways.
Monitoring changes to network gateways helps ensure that all ingress/egress traffic traverses the VPC border via a controlled path.

**Severity**: Low

### Ensure a log metric filter and alarm exist for CloudTrail configuration changes

**Description**: Real-time monitoring of API calls can be achieved by directing CloudTrail Logs to CloudWatch Logs and establishing corresponding metric filters and alarms.
 It's recommended that a metric filter and alarm be established for detecting changes to CloudTrail's configurations.

 Monitoring changes to CloudTrail's configuration helps ensure sustained visibility to activities performed in the AWS account.

**Severity**: Low

### Ensure a log metric filter and alarm exist for disabling or scheduled deletion of customer created CMKs

**Description**: Real-time monitoring of API calls can be achieved by directing CloudTrail Logs to CloudWatch Logs and establishing corresponding metric filters and alarms.
 It's recommended that a metric filter and alarm be established for customer created CMKs, which have changed state to disabled or scheduled deletion.
 Data encrypted with disabled or deleted keys will no longer be accessible.

**Severity**: Low

### Ensure a log metric filter and alarm exist for IAM policy changes

**Description**: Real-time monitoring of API calls can be achieved by directing CloudTrail Logs to CloudWatch Logs and establishing corresponding metric filters and alarms.
 It's recommended that a metric filter and alarm be established changes made to Identity and Access Management (IAM) policies.
 Monitoring changes to IAM policies helps ensure authentication and authorization controls remain intact.

**Severity**: Low

### Ensure a log metric filter and alarm exist for Management Console sign-in without MFA

**Description**: Real-time monitoring of API calls can be achieved by directing CloudTrail Logs to CloudWatch Logs and establishing corresponding metric filters and alarms.
 It's recommended that a metric filter and alarm be established for console logins that aren't protected by multifactor authentication (MFA).
Monitoring for single-factor console logins increases visibility into accounts that aren't protected by MFA.

**Severity**: Low

### Ensure a log metric filter and alarm exist for route table changes

**Description**: Real-time monitoring of API calls can be achieved by directing CloudTrail Logs to CloudWatch Logs and establishing corresponding metric filters and alarms. Routing tables are used to route network traffic between subnets and to network gateways.
 It's recommended that a metric filter and alarm be established for changes to route tables.
Monitoring changes to route tables helps ensure that all VPC traffic flows through an expected path.

**Severity**: Low

### Ensure a log metric filter and alarm exist for S3 bucket policy changes

**Description**: Real-time monitoring of API calls can be achieved by directing CloudTrail Logs to CloudWatch Logs and establishing corresponding metric filters and alarms.
 It is recommended that a metric filter and alarm be established for changes to S3 bucket policies.
Monitoring changes to S3 bucket policies might reduce time to detect and correct permissive policies on sensitive S3 buckets.

**Severity**: Low

### Ensure a log metric filter and alarm exist for security group changes

**Description**: Real-time monitoring of API calls can be achieved by directing CloudTrail Logs to CloudWatch Logs and establishing corresponding metric filters and alarms. Security Groups are a stateful packet filter that controls ingress and egress traffic within a VPC.
 It's recommended that a metric filter and alarm be established changes to Security Groups.
Monitoring changes to security group helps ensure that resources and services aren't unintentionally exposed.

**Severity**: Low

### Ensure a log metric filter and alarm exist for unauthorized API calls

**Description**: Real-time monitoring of API calls can be achieved by directing CloudTrail Logs to CloudWatch Logs and establishing corresponding metric filters and alarms.
 It's recommended that a metric filter and alarm be established for unauthorized API calls.
 Monitoring unauthorized API calls helps reveal application errors and might reduce time to detect malicious activity.

**Severity**: Low

### Ensure a log metric filter and alarm exist for usage of 'root' account

**Description**: Real-time monitoring of API calls can be achieved by directing CloudTrail Logs to CloudWatch Logs and establishing corresponding metric filters and alarms.
 It's recommended that a metric filter and alarm be established for root login attempts.

 Monitoring for root account logins provides visibility into the use of a fully privileged account and an opportunity to reduce the use of it.

**Severity**: Low

### Ensure a log metric filter and alarm exist for VPC changes

**Description**: Real-time monitoring of API calls can be achieved by directing CloudTrail Logs to CloudWatch Logs and establishing corresponding metric filters and alarms.
 It's possible to have more than one VPC within an account, in addition it's also possible to create a peer connection between 2 VPCs enabling network traffic to route between VPCs. It's recommended that a metric filter and alarm be established for changes made to VPCs.
Monitoring changes to IAM policies helps ensure authentication and authorization controls remain intact.

**Severity**: Low

### Ensure no security groups allow ingress from 0.0.0.0/0 to port 3389

**Description**: Security groups provide stateful filtering of ingress/egress network traffic to AWS resources. It's recommended that no security group allows unrestricted ingress access to port 3389.
 When you remove unfettered connectivity to remote console services, such as RDP, it reduces a server's exposure to risk.

**Severity**: High

### RDS databases and clusters should not use a database engine default port

**Description**: This control checks whether the RDS cluster or instance uses a port other than the default port of the database engine.
If you use a known port to deploy an RDS cluster or instance, an attacker can guess information about the cluster or instance.
 The attacker can use this information in conjunction with other information to connect to an RDS cluster or instance or gain additional information about your application.
When you change the port, you must also update the existing connection strings that were used to connect to the old port.
 You should also check the security group of the DB instance to ensure that it includes an ingress rule that allows connectivity on the new port.

**Severity**: Low

### RDS instances should be deployed in a VPC

**Description**: VPCs provide a number of network controls to secure access to RDS resources.
 These controls include VPC Endpoints, network ACLs, and security groups.
 To take advantage of these controls, we recommend that you move EC2-Classic RDS instances to EC2-VPC.

**Severity**: Low

### S3 buckets should require requests to use Secure Socket Layer

**Description**: We recommend requiring requests to use Secure Socket Layer (SSL) on all Amazon S3 bucket.
 S3 buckets should have policies that require all requests ('Action: S3:*') to only accept transmission of data over HTTPS in the S3 resource policy, indicated by the condition key 'aws:SecureTransport'.

**Severity**: Medium

### Security groups should not allow ingress from 0.0.0.0/0 to port 22

**Description**: To reduce the server's exposure, it's recommended not to allow unrestricted ingress access to port '22'.

**Severity**: High

### Security groups should not allow unrestricted access to ports with high risk

**Description**: This control checks whether unrestricted incoming traffic for the security groups is accessible to the specified ports that have the highest risk. This control passes when none of the rules in a security group allow ingress traffic from 0.0.0.0/0 for those ports.
Unrestricted access (0.0.0.0/0) increases opportunities for malicious activity, such as hacking, denial-of-service attacks, and loss of data.
Security groups provide stateful filtering of ingress and egress network traffic to AWS resources. No security group should allow unrestricted ingress access to the following ports:

- 3389 (RDP)
- 20, 21 (FTP)
- 22 (SSH)
- 23 (Telnet)
- 110 (POP3)
- 143 (IMAP)
- 3306 (MySQL)
- 8080 (proxy)
- 1433, 1434 (MSSQL)
- 9200 or 9300 (Elasticsearch)
- 5601 (Kibana)
- 25 (SMTP)
- 445 (CIFS)
- 135 (RPC)
- 4333 (ahsp)
- 5432 (postgresql)
- 5500 (fcp-addr-srvr1)

**Severity**: Medium

### Security groups should only allow unrestricted incoming traffic for authorized ports

**Description**: This control checks whether the security groups that are in use allow unrestricted incoming traffic. Optionally the rule checks whether the port numbers are listed in the "authorizedTcpPorts" parameter.

- If the security group rule port number allows unrestricted incoming traffic, but the port number is specified in "authorizedTcpPorts", then the control passes. The default value for "authorizedTcpPorts" is **80, 443**.
- If the security group rule port number allows unrestricted incoming traffic, but the port number isn't specified in authorizedTcpPorts input parameter, then the control fails.
- If the parameter isn't used, then the control fails for any security group that has an unrestricted inbound rule.
Security groups provide stateful filtering of ingress and egress network traffic to AWS. Security group rules should follow the principle of least privileged access. Unrestricted access (IP address with a /0 suffix) increases the opportunity for malicious activity such as hacking, denial-of-service attacks, and loss of data.
Unless a port is specifically allowed, the port should deny unrestricted access.

**Severity**: High

### Unused EC2 EIPs should be removed

**Description**: Elastic IP addresses that are allocated to a VPC should be attached to Amazon EC2 instances or in-use elastic network interfaces (ENIs).

**Severity**: Low

### Unused network access control lists should be removed

**Description**: This control checks whether there are any unused network access control lists (ACLs).
 The control checks the item configuration of the resource "AWS::EC2::NetworkAcl" and determines the relationships of the network ACL.
 If the only relationship is the VPC of the network ACL, then the control fails.
If other relationships are listed, then the control passes.

**Severity**: Low

### VPC's default security group should restricts all traffic

**Description**: Security group should restrict all traffic to reduce resource exposure.

**Severity**: Low


## GCP networking recommendations

### Cluster hosts should be configured to use only private, internal IP addresses to access Google APIs

**Description**: This recommendation evaluates whether the privateIpGoogleAccess property of a subnetwork is set to false.

**Severity**: High

### Compute instances should use a load balancer that is configured to use a target HTTPS proxy

**Description**: This recommendation evaluates if the selfLink property of the targetHttpProxy resource matches the target attribute in the forwarding rule, and if the forwarding rule contains a loadBalancingScheme field set to External.

**Severity**: Medium

### Control Plane Authorized Networks should be enabled on GKE clusters

**Description**: This recommendation evaluates the masterAuthorizedNetworksConfig property of a cluster for the key-value pair, 'enabled': false.

**Severity**: High

### Egress deny rule should be set on a firewall to block unwanted outbound traffic

**Description**: This recommendation evaluates whether the destinationRanges property in the firewall is set to 0.0.0.0/0 and the denied property contains the key-value pair, ```'IPProtocol': 'all.'```

**Severity**: Low

### Ensure Firewall Rules for instances behind Identity Aware Proxy (IAP) only allow the traffic from Google Cloud Loadbalancer (GCLB) Health Check and Proxy Addresses

**Description**: Access to VMs should be restricted by firewall rules that allow only IAP traffic by ensuring only connections proxied by the IAP are allowed.
To ensure that load balancing works correctly health checks should also be allowed.
IAP ensures that access to VMs is controlled by authenticating incoming requests.
 However if the VM is still accessible from IP addresses other than the IAP it might still be possible to send unauthenticated requests to the instance.
 Care must be taken to ensure that loadblancer health checks aren't blocked as this would stop the load balancer from correctly knowing the health of the VM and load balancing correctly.

**Severity**: Medium

### Ensure legacy networks do not exist for a project

**Description**: In order to prevent use of legacy networks, a project shouldn't have a legacy network configured.
 Legacy networks have a single network IPv4 prefix range and a single gateway IP address for the whole network. The network is global in scope and spans all cloud regions.
 Subnetworks can't be created in a legacy network and are unable to switch from legacy to auto or custom subnet networks. Legacy networks can have an impact for high network traffic projects and are subject to a single point of contention or failure.

**Severity**: Medium

### Ensure 'log_hostname' database flag for Cloud SQL PostgreSQL instance is set appropriately

**Description**: PostgreSQL logs only the IP address of the connecting hosts.
 The "log_hostname" flag controls the logging of "hostnames" in addition to the IP addresses logged.
 The performance hit is dependent on the configuration of the environment and the host name resolution setup.
 This parameter can only be set in the "postgresql.conf" file or on the server command line.
 Logging hostnames can incur overhead on server performance as for each statement logged, DNS resolution will be required to convert IP address to hostname.
 Depending on the setup, this might be non-negligible.
 Additionally, the IP addresses that are logged can be resolved to their DNS names later when reviewing the logs excluding the cases where dynamic hostnames are used.
 This recommendation is applicable to PostgreSQL database instances.

**Severity**: Low

### Ensure no HTTPS or SSL proxy load balancers permit SSL policies with weak cipher suites

**Description**: Secure Sockets Layer (SSL) policies determine what port Transport Layer Security (TLS) features clients are permitted to use when connecting to load balancers.
To prevent usage of insecure features, SSL policies should use (a) at least TLS 1.2 with the MODERN profile;
 or (b) the RESTRICTED profile, because it effectively requires clients to use TLS 1.2 regardless of the chosen minimum TLS version;
 or (3) a CUSTOM profile that doesn't support any of the following features:
TLS_RSA_WITH_AES_128_GCM_SHA256
TLS_RSA_WITH_AES_256_GCM_SHA384
TLS_RSA_WITH_AES_128_CBC_SHA
TLS_RSA_WITH_AES_256_CBC_SHA
TLS_RSA_WITH_3DES_EDE_CBC_SHA

Load balancers are used to efficiently distribute traffic across multiple servers.
 Both SSL proxy and HTTPS load balancers are external load balancers, meaning they distribute traffic from the Internet to a GCP network.
 GCP customers can configure load balancer SSL policies with a minimum TLS version (1.0, 1.1, or 1.2) that clients can use to establish a connection, along with a profile (Compatible, Modern, Restricted, or Custom) that specifies permissible cipher suites.
 To comply with users using outdated protocols, GCP load balancers can be configured to permit insecure cipher suites.
 In fact, the GCP default SSL policy uses a minimum TLS version of 1.0 and a Compatible profile, which allows the widest range of insecure cipher suites.
 As a result, it's easy for customers to configure a load balancer without even knowing that they're permitting outdated cipher suites.

**Severity**: Medium

### Ensure that Cloud DNS logging is enabled for all VPC networks

**Description**: Cloud DNS logging records the queries from the name servers within your VPC to Stackdriver.
 Logged queries can come from Compute Engine VMs, GKE containers, or other GCP resources provisioned within the VPC.
Security monitoring and forensics can't depend solely on IP addresses from VPC flow logs, especially when considering the dynamic IP usage of cloud resources, HTTP virtual host routing,
and other technology that can obscure the DNS name used by a client from the IP address.
Monitoring of Cloud DNS logs provides visibility to DNS names requested by the clients within the VPC.
These logs can be monitored for anomalous domain names, evaluated against threat intelligence, and

For full capture of DNS, firewall must block egress UDP/53 (DNS)
and TCP/443 (DNS over HTTPS) to prevent client from using external DNS name server for resolution.

**Severity**: High

### Ensure that DNSSEC is enabled for Cloud DNS

**Description**: Cloud Domain Name System (DNS) is a fast, reliable, and cost-effective domain name system that powers millions of domains on the internet.
 Domain Name System Security Extensions (DNSSEC) in Cloud DNS enables domain owners to take easy steps to protect their domains against DNS hijacking and man-in-the-middle and other attacks.
 Domain Name System Security Extensions (DNSSEC) adds security to the DNS protocol by enabling DNS responses to be validated.
 Having a trustworthy DNS that translates a domain name like `www.example.com` into its associated IP address is an increasingly important building block of today's web-based applications.
 Attackers can hijack this process of domain/IP lookup and redirect users to a malicious site through DNS hijacking and man-in-the-middle attacks.
 DNSSEC helps mitigate the risk of such attacks by cryptographically signing DNS records.
 As a result, it prevents attackers from issuing fake DNS responses that might misdirect browsers to nefarious websites.

**Severity**: Medium

### Ensure that RDP access is restricted from the Internet

**Description**: GCP Firewall Rules are specific to a VPC Network. Each rule either allows or denies traffic when its conditions are met. Its conditions allow users to specify the type of traffic, such as ports and protocols, and the source or destination of the traffic, including IP addresses, subnets, and instances.
Firewall rules are defined at the VPC network level and are specific to the network in which they're defined. The rules themselves can't be shared among networks. Firewall rules only support IPv4 traffic.
When you specify a source for an ingress rule or a destination for an egress rule by address, an IPv4 address or IPv4 block in CIDR notation can be used. Generic (0.0.0.0/0) incoming traffic from the Internet to a VPC or VM instance using RDP on Port 3389 can be avoided.
 GCP Firewall Rules within a VPC Network. These rules apply to outgoing (egress) traffic from instances and incoming (ingress) traffic to instances in the network.
 Egress and ingress traffic flows are controlled even if the traffic stays within the network (for example, instance-to-instance communication). For an instance to have outgoing Internet access, the network must have a valid Internet gateway route or custom route whose destination IP is specified.
 This route simply defines the path to the Internet, to avoid the most general (0.0.0.0/0) destination IP Range specified from the Internet through RDP with the default Port 3389. Generic access from the Internet to a specific IP Range should be restricted.

**Severity**: High

### Ensure that RSASHA1 is not used for the key-signing key in Cloud DNS DNSSEC

**Description**: DNSSEC algorithm numbers in this registry might be used in CERT RRs.
 Zone signing (DNSSEC) and transaction security mechanisms (SIG(0) and TSIG) make use of particular subsets of these algorithms.
 The algorithm used for key signing should be a recommended one and it should be strong.
 Domain Name System Security Extensions (DNSSEC) algorithm numbers in this registry might be used in CERT RRs.
 Zone signing (DNSSEC) and transaction security mechanisms (SIG(0) and TSIG) make use of particular subsets of these algorithms.
 The algorithm used for key signing should be a recommended one and it should be strong.
 When you enable DNSSEC for a managed zone, or creat a managed zone with DNSSEC, the user can select the DNSSEC signing algorithms and the denial-of-existence type.
 Changing the DNSSEC settings is only effective for a managed zone if DNSSEC isn't already enabled.
 If there's a need to change the settings for a managed zone where it has been enabled, turn off DNSSEC and then re-enable it with different settings.

**Severity**: Medium

### Ensure that RSASHA1 is not used for the zone-signing key in Cloud DNS DNSSEC

**Description**: DNSSEC algorithm numbers in this registry might be used in CERT RRs.
 Zone signing (DNSSEC) and transaction security mechanisms (SIG(0) and TSIG) make use of particular subsets of these algorithms.
 The algorithm used for key signing should be a recommended one and it should be strong.
 DNSSEC algorithm numbers in this registry might be used in CERT RRs.
 Zone signing (DNSSEC) and transaction security mechanisms (SIG(0) and TSIG) make use of particular subsets of these algorithms.
 The algorithm used for key signing should be a recommended one and it should be strong.
 When you enable DNSSEC for a managed zone, or create a managed zone with DNSSEC, the DNSSEC signing algorithms, and the denial-of-existence type can be selected.
 Changing the DNSSEC settings is only effective for a managed zone if DNSSEC isn't already enabled.
 If the need exists to change the settings for a managed zone where it has been enabled, turn off DNSSEC and then re-enable it with different settings.

**Severity**: Medium

### Ensure that SSH access is restricted from the internet

**Description**: GCP Firewall Rules are specific to a VPC Network. Each rule either allows or denies traffic when its conditions are met. Its conditions allow the user to specify the type of traffic, such as ports and protocols, and the source or destination of the traffic, including IP addresses, subnets, and instances.
Firewall rules are defined at the VPC network level and are specific to the network in which they're defined. The rules themselves can't be shared among networks. Firewall rules only support IPv4 traffic.
When you specify a source for an ingress rule or a destination for an egress rule by address, only an IPv4 address or IPv4 block in CIDR notation can be used. Generic (0.0.0.0/0) incoming traffic from the internet to VPC or VM instance using SSH on Port 22 can be avoided.
 GCP Firewall Rules within a VPC Network apply to outgoing (egress) traffic from instances and incoming (ingress) traffic to instances in the network.
Egress and ingress traffic flows are controlled even if the traffic stays within the network (for example, instance-to-instance communication).
For an instance to have outgoing Internet access, the network must have a valid Internet gateway route or custom route whose destination IP is specified.
This route simply defines the path to the Internet, to avoid the most general (0.0.0.0/0) destination IP Range specified from the Internet through SSH with the default Port '22.'
 Generic access from the Internet to a specific IP Range needs to be restricted.

**Severity**: High

### Ensure that the default network does not exist in a project

**Description**: To prevent use of "default" network, a project shouldn't have a "default" network.
 The default network has a preconfigured network configuration and automatically generates the following insecure firewall rules:

- default-allow-internal: Allows ingress connections for all protocols and ports among instances in the network.
- default-allow-ssh: Allows ingress connections on TCP port 22(SSH) from any source to any instance in the network.
- default-allow-rdp: Allows ingress connections on TCP port 3389(RDP) from any source to any instance in the network.
- default-allow-icmp: Allows ingress ICMP traffic from any source to any instance in the network.

These automatically created firewall rules don't get audit logged and can't be configured to enable firewall rule logging.
Furthermore, the default network is an auto mode network, which means that its subnets use the same predefined range of IP addresses, and as a result, it's not possible to use Cloud VPN or VPC Network Peering with the default network.
Based on organization security and networking requirements, the organization should create a new network and delete the default network.

**Severity**: Medium

### Ensure that the log metric filter and alerts exist for VPC network changes

**Description**: It's recommended that a metric filter and alarm be established for Virtual Private Cloud (VPC) network changes.
It's possible to have more than one VPC within a project. In addition, it's also possible to create a peer connection between two VPCs enabling network traffic to route between VPCs.
Monitoring changes to a VPC will help ensure VPC traffic flow isn't getting impacted.

**Severity**: Low

### Ensure that the log metric filter and alerts exist for VPC Network Firewall rule changes

**Description**: It's recommended that a metric filter and alarm be established for Virtual Private Cloud (VPC) Network Firewall rule changes.
Monitoring for Create or Update Firewall rule events gives insight to network access changes and might reduce the time it takes to detect suspicious activity.

**Severity**: Low

### Ensure that the log metric filter and alerts exist for VPC network route changes

**Description**: It's recommended that a metric filter and alarm be established for Virtual Private Cloud (VPC) network route changes.
Google Cloud Platform (GCP) routes define the paths network traffic takes from a VM instance to another destination. The other destination can be inside the organization VPC network (such as another VM) or outside of it. Every route consists of a destination and a next hop. Traffic whose destination IP is within the destination range is sent to the next hop for delivery.
Monitoring changes to route tables will help ensure that all VPC traffic flows through an expected path.

**Severity**: Low

### Ensure that the 'log_connections' database flag for Cloud SQL PostgreSQL instance is set to 'on'

**Description**: Enabling the log_connections setting causes each attempted connection to the server to be logged, along with successful completion of client authentication. This parameter can't be changed after the session starts.
PostgreSQL doesn't log attempted connections by default. Enabling the log_connections setting will create log entries for each attempted connection as well as successful completion of client authentication, which can be useful in troubleshooting issues and to determine any unusual connection attempts to the server.
 This recommendation is applicable to PostgreSQL database instances.

**Severity**: Medium

### Ensure that the 'log_disconnections' database flag for Cloud SQL PostgreSQL instance is set to 'on'

**Description**: Enabling the log_disconnections setting logs the end of each session, including the session duration.
PostgreSQL doesn't log session details such as duration and session end by default. Enabling the log_disconnections setting will create log entries at the end of each session, which can be useful in troubleshooting issues and determine any unusual activity across a time period.
The log_disconnections and log_connections work hand in hand and generally, the pair would be enabled/disabled together. This recommendation is applicable to PostgreSQL database instances.

**Severity**: Medium

### Ensure that VPC Flow Logs is enabled for every subnet in a VPC Network

**Description**: Flow Logs is a feature that enables users to capture information about the IP traffic going to and from network interfaces in the organization's VPC Subnets. Once a flow log is created, the user can view and retrieve its data in Stackdriver Logging.
 It's recommended that Flow Logs be enabled for every business-critical VPC subnet.
VPC networks and subnetworks provide logically isolated and secure network partitions where GCP resources can be launched. When Flow Logs is enabled for a subnet, VMs within that subnet start reporting on all Transmission Control Protocol (TCP) and User Datagram Protocol (UDP) flows.
 Each VM samples the TCP and UDP flows it sees, inbound and outbound, whether the flow is to or from another VM, a host in the on-premises datacenter, a Google service, or a host on the Internet. If two GCP VMs are communicating, and both are in subnets that have VPC Flow Logs enabled, both VMs report the flows.
Flow Logs supports the following use cases: 1. Network monitoring. 2. Understanding network usage and optimizing network traffic expenses. 3. Network forensics. 4. Real-time security analysis
Flow Logs provide visibility into network traffic for each VM inside the subnet and can be used to detect anomalous traffic or insight during security workflows.

**Severity**: Low

### Firewall rule logging should be enabled

**Description**: This recommendation evaluates the logConfig property in firewall metadata to see if it's empty or contains the key-value pair 'enable': false.

**Severity**: Medium

### Firewall should not be configured to be open to public access

**Description**: This recommendation evaluates the sourceRanges and allowed properties for one of two configurations:

 The sourceRanges property contains 0.0.0.0/0 and the allowed property contains a combination of rules that includes any protocol or protocol:port, except the following:
- icmp
- tcp: 22
- tcp: 443
- tcp: 3389
- udp: 3389
- sctp: 22

 The sourceRanges property contains a combination of IP ranges that includes any nonprivate IP address and the allowed property contains a combination of rules that permit either all tcp ports or all udp ports.

**Severity**: High

### Firewall should not be configured to have an open CASSANDRA port that allows generic access

**Description**: This recommendation evaluates the allowed property in firewall metadata for the following protocols and ports: TCP: 7000-7001, 7199, 8888, 9042, 9160, 61620-61621.

**Severity**: Low

### Firewall should not be configured to have an open CISCOSECURE_WEBSM port that allows generic access

**Description**: This recommendation evaluates the allowed property in firewall metadata for the following protocol and port: TCP: 9090.

**Severity**: Low

### Firewall should not be configured to have an open DIRECTORY_SERVICES port that allows generic access

**Description**: This recommendation evaluates the allowed property in firewall metadata for the following protocols and ports: TCP: 445 and UDP: 445.

**Severity**: Low

### Firewall should not be configured to have an open DNS port that allows generic access

**Description**: This recommendation evaluates the allowed property in firewall metadata for the following protocols and ports: TCP: 53 and UDP: 53.

**Severity**: Low

### Firewall should not be configured to have an open ELASTICSEARCH port that allows generic access

**Description**: This recommendation evaluates the allowed property in firewall metadata for the following protocols and ports: TCP: 9200, 9300.

**Severity**: Low

### Firewall should not be configured to have an open FTP port that allows generic access

**Description**: This recommendation evaluates the allowed property in firewall metadata for the following protocol and port: TCP: 21.

**Severity**: Low

### Firewall should not be configured to have an open HTTP port that allows generic access

**Description**: This recommendation evaluates the allowed property in firewall metadata for the following protocols and ports: TCP: 80.

**Severity**: Low

### Firewall should not be configured to have an open LDAP port that allows generic access

**Description**: This recommendation evaluates the allowed property in firewall metadata for the following protocols and ports: TCP: 389, 636 and UDP: 389.

**Severity**: Low

### Firewall should not be configured to have an open MEMCACHED port that allows generic access

**Description**: This recommendation evaluates the allowed property in firewall metadata for the following protocols and ports: TCP: 11211, 11214-11215 and UDP: 11211, 11214-11215.

**Severity**: Low

### Firewall should not be configured to have an open MONGODB port that allows generic access

**Description**: This recommendation evaluates the allowed property in firewall metadata for the following protocols and ports: TCP: 27017-27019.

**Severity**: Low

### Firewall should not be configured to have an open MYSQL port that allows generic access

**Description**: This recommendation evaluates the allowed property in firewall metadata for the following protocol and port: TCP: 3306.

**Severity**: Low

### Firewall should not be configured to have an open NETBIOS port that allows generic access

**Description**: This recommendation evaluates the allowed property in firewall metadata for the following protocols and ports: TCP: 137-139 and UDP: 137-139.

**Severity**: Low

### Firewall should not be configured to have an open ORACLEDB port that allows generic access

**Description**: This recommendation evaluates the allowed property in firewall metadata for the following protocols and ports: TCP: 1521, 2483-2484 and UDP: 2483-2484.

**Severity**: Low

### Firewall should not be configured to have an open POP3 port that allows generic access

**Description**: This recommendation evaluates the allowed property in firewall metadata for the following protocol and port: TCP: 110.

**Severity**: Low

### Firewall should not be configured to have an open PostgreSQL port that allows generic access

**Description**: This recommendation evaluates the allowed property in firewall metadata for the following protocols and ports: TCP: 5432 and UDP: 5432.

**Severity**: Low

### Firewall should not be configured to have an open REDIS port that allows generic access

**Description**: This recommendation evaluates whether the allowed property in firewall metadata contains the following protocol and port: TCP: 6379.

**Severity**: Low

### Firewall should not be configured to have an open SMTP port that allows generic access

**Description**: This recommendation evaluates whether the allowed property in firewall metadata contains the following protocol and port: TCP: 25.

**Severity**: Low

### Firewall should not be configured to have an open SSH port that allows generic access

**Description**: This recommendation evaluates whether the allowed property in firewall metadata contains the following protocols and ports: TCP: 22 and SCTP: 22.

**Severity**: Low

### Firewall should not be configured to have an open TELNET port that allows generic access

**Description**: This recommendation evaluates whether the allowed property in firewall metadata contains the following protocol and port: TCP: 23.

**Severity**: Low

### GKE clusters should have alias IP ranges enabled

**Description**: This recommendation evaluates whether the useIPAliases field of the ipAllocationPolicy in a cluster is set to false.

**Severity**: Low

### GKE clusters should have Private clusters enabled

**Description**: This recommendation evaluates whether the enablePrivateNodes field of the privateClusterConfig property is set to false.

**Severity**: High

### Network policy should be enabled on GKE clusters

**Description**: This recommendation evaluates the networkPolicy field of the addonsConfig property for the key-value pair, 'disabled': true.

**Severity**: Medium

## Related content

- [What are security policies, initiatives, and recommendations?](security-policy-concept.md)
- [Review your security recommendations](review-security-recommendations.md)
