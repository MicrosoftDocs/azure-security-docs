---
title: Enable just-in-time access
description: Learn how just-in-time VM access (JIT) in Microsoft Defender for Cloud helps you control access to your Azure virtual machines.
author: dcurwin
ms.author: dacurwin
ms.date: 02/19/2025
ms.topic: how-to
ms.custom:
  - ge-structured-content-pilot
---

# Enable just-in-time access

[Defender for Servers](defender-for-servers-overview.md) in Microsoft Defender for Cloud provides a just-in-time machine access feature.

You can use Microsoft Defender for Cloud's just-in-time access to protect your Azure VMs from unauthorized network access. Many times firewalls contain allow rules that leave your VMs vulnerable to attack. JIT lets you allow access to your VMs only when the access is needed, on the ports needed, and for the period of time needed.

In this article, you learn how to set up and use just-in-time access, including how to:

-   Enable just-in-time on VMs from the Azure portal or programmatically
-   Request access to a VM that has just-in-time access enabled from the Azure portal or programmatically
-   [Audit just-in-time access activity](#audit-jit-access-activity-in-defender-for-cloud) to make sure your VMs are secured appropriately

## Prerequisites

-   [Microsoft Defender for Servers Plan 2](defender-for-servers-overview.md) must be enabled on the subscription.

-   Supported VMs: VMs deployed through Azure Resource Manager, VMs protected by Azure Firewalls on the same VNET as the VM, AWS EC2 instances (Preview).

-   Unsupported VMs: VMs deployed with [classic deployment models](/azure/azure-resource-manager/management/deployment-models), VMs protected by Azure Firewalls controlled by [Azure Firewall Manager](/azure/firewall-manager/overview).

-   To set up just-in-time access on your AWS VMs, you need to [connect your AWS account](quickstart-onboard-aws.md) to Microsoft Defender for Cloud.

-   To JIT policy, the policy name, together with the targeted VM name, must not exceed a total of 56 characters.

-   You need **Reader** and **SecurityReader** permissions, or a custom role can view the JIT status and parameters.

-   For a custom role, assign the permissions summarized in the table. To create a least-privileged role for users that only need to request JIT access to a VM, use the [Set-JitLeastPrivilegedRole script](https://github.com/Azure/Microsoft-Defender-for-Cloud/tree/main/Powershell%20scripts/JIT%20Scripts/JIT%20Custom%20Role).

| User action                               | Permissions to set                                                                                                                                                                                                                                                                                                                                    |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Configure or edit a JIT policy for a VM   | *Assign these actions to the role:* <ul><li>On the scope of a subscription (or resource group when using API or PowerShell only) that is associated with the VM:<br/> `Microsoft.Security/locations/jitNetworkAccessPolicies/write` </li><li> On the scope of a subscription (or resource group when using API or PowerShell only) of VM: <br/>`Microsoft.Compute/virtualMachines/write`</li></ul> |
| Request JIT access to a VM                | *Assign these actions to the user:* <ul><li> `Microsoft.Security/locations/jitNetworkAccessPolicies/initiate/action` </li><li> `Microsoft.Security/locations/jitNetworkAccessPolicies/*/read` </li><li> `Microsoft.Compute/virtualMachines/read` </li><li> `Microsoft.Network/networkInterfaces/*/read` </li> <li> `Microsoft.Network/publicIPAddresses/read` </li></ul> |
| Read JIT policies                         | *Assign these actions to the user:* <ul><li>`Microsoft.Security/locations/jitNetworkAccessPolicies/read`</li><li>`Microsoft.Security/locations/jitNetworkAccessPolicies/initiate/action`</li><li>`Microsoft.Security/policies/read`</li><li>`Microsoft.Security/pricings/read`</li><li>`Microsoft.Compute/virtualMachines/read`</li><li>`Microsoft.Network/*/read`</li>|

> [!NOTE]
> Only the `Microsoft.Security` permissions are relevant for AWS.
> To create a least-privileged role for users that only need to request JIT access to a VM, use the Set-JitLeastPrivilegedRole script.

## Work with JIT VM access using Microsoft Defender for Cloud

You can use Defender for Cloud or you can programmatically enable JIT VM access with your own custom options, or you can enable JIT with default, hard-coded parameters from Azure virtual machines.

**Just-in-time VM access** shows your VMs grouped into:

-   **Configured** - VMs configured to support just-in-time VM access, and shows:
    -   the number of approved JIT requests in the last seven days
    -   the last access date and time
    -   the connection details configured
    -   the last user
-   **Not configured** - VMs without JIT enabled, but that can support JIT. We recommend that you enable JIT for these VMs.
-   **Unsupported** - VMs that don't support JIT because:
    -   Missing network security group (NSG) or Azure Firewall - JIT requires an NSG to be configured or a Firewall configuration (or both)
    -   Classic VM - JIT supports VMs that are deployed through Azure Resource Manager.
    -   Other - The JIT solution is disabled in the security policy of the subscription or the resource group.

### Enable JIT on your VMs from Microsoft Defender for Cloud

:::image type="content" source="./media/just-in-time-access-usage/configure-just-in-time-access.gif" alt-text="Screenshot showing configuring JIT VM access in Microsoft Defender for Cloud." lightbox="media/just-in-time-access-usage/configure-just-in-time-access.gif":::

From Defender for Cloud, you can enable and configure the JIT VM access.

1.  Open **Workload protections** and, in the advanced protections, select **Just-in-time VM access**.

1.  In the **Not configured** virtual machines tab, mark the VMs to protect with JIT and select **Enable JIT on VMs**.

    The JIT VM access page opens listing the ports that Defender for Cloud recommends protecting:

    -   22 - SSH
    -   3389 - RDP
    -   5985 - WinRM
    -   5986 - WinRM

    To customize the JIT access:

    1.  Select **Add**.

    1.  Select one of the ports in the list to edit it or enter other ports. For each port, you can set the:
        -   **Protocol**
        -   **Allowed source IPs**
        -   **Maximum request time**

    1.  Select **OK**.

1.  To save the port configuration, select **Save**.

### Edit the JIT configuration on a JIT-enabled VM using Defender for Cloud

You can modify a VM's just-in-time configuration by adding and configuring a new port to protect for that VM, or by changing any other setting related to an already protected port.

To edit the existing JIT rules for a VM:

1.  Open **Workload protections** and, in the advanced protections, select **Just-in-time VM access**.

1.  In the **Configured** virtual machines tab, right-click on a VM and select **Edit**.

1.  In the **JIT VM access configuration**, you can either edit the list of port or select Add a new custom port.

1.  When you finish editing the ports, select **Save**.

### Request access to a JIT-enabled VM from Microsoft Defender for Cloud

When a VM has JIT enabled, you have to request access to connect to it. You can request access in any of the supported ways, regardless of how you enabled JIT.

1.  From the **Just-in-time VM access** page, select the **Configured** tab.

1.  Select the VMs you want to access.

    -   The icon in the **Connection Details** column indicates whether JIT is enabled on the network security group or firewall. If it's enabled on both, only the firewall icon appears.

    -   The **Connection Details** column shows the user and ports that can access the VM.

1.  Select **Request access**. The **Request access** window opens.

1.  Under **Request access**, select the ports that you want to open for each VM, the source IP addresses that you want the port opened on, and the time window to open the ports.

1.  Select **Open ports**.

    > [!NOTE]
    > If a user who is requesting access is behind a proxy, you can enter the IP address range of the proxy.

## Other ways to work with JIT VM access

### Azure virtual machines

#### Enable JIT on your VMs from Azure virtual machines

You can enable JIT on a VM from the Azure virtual machines pages of the Azure portal.

> [!TIP]
> If a VM already has JIT enabled, the VM configuration page shows that JIT is enabled. You can use the link to open the JIT VM access page in Defender for Cloud to view and change the settings.

1.  From the [Azure portal](https://portal.azure.com), search for and select **Virtual machines**.

1.  Select the virtual machine you want to protect with JIT.

1.  In the menu, select **Configuration**.

1.  Under **Just-in-time access**, select **Enable just-in-time**.

    By default, just-in-time access for the VM uses these settings:

    -   Windows machines:
        -   RDP port: 3389
        -   Maximum allowed access: 3 hours
        -   Allowed source IP addresses: Any
    -   Linux machines:
        -   SSH port: 22
        -   Maximum allowed access: 3 hours
        -   Allowed source IP addresses: Any

1.  To edit any of these values or add more ports to your JIT configuration, use Microsoft Defender for Cloud's just-in-time page:
    1.  From Defender for Cloud's menu, select **Just-in-time VM access**.

    1.  From the **Configured** tab, right-click on the VM to which you want to add a port, and select **Edit**.
        
        ![Editing a JIT VM access configuration in Microsoft Defender for Cloud.](./media/just-in-time-access-usage/jit-policy-edit-security-center.png)

    1.  Under **JIT VM access configuration**, you can either edit the existing settings of an already protected port or add a new custom port.

    1.  When you've finished editing the ports, select **Save**.

#### Request access to a JIT-enabled VM from the Azure virtual machine's connect page

When a VM has a JIT enabled, you have to request access to connect to it. You can request access in any of the supported ways, regardless of how you enabled JIT.

![Screenshot showing jit just-in-time request.](./media/just-in-time-access-usage/jit-request-vm.png)

To request access from Azure virtual machines:

1.  In the Azure portal, open the virtual machines pages.

1.  Select the VM to which you want to connect, and open the **Connect** page.
    
    Azure checks to see if JIT is enabled on that VM.

    -   If JIT isn't enabled for the VM, you're prompted to enable it.
    -   If JIT is enabled, select **Request access** to pass an access request with the requesting IP, time range, and ports that were configured for that VM.

> [!NOTE]
> After a request is approved for a VM protected by Azure Firewall, Defender for Cloud provides the user with the proper connection details (the port mapping from the DNAT table) to use to connect to the VM.

### PowerShell

#### Enable JIT on your VMs using PowerShell

To enable just-in-time VM access from PowerShell, use the official Microsoft Defender for Cloud PowerShell cmdlet `Set-AzJitNetworkAccessPolicy`.

**Example** - Enable just-in-time VM access on a specific VM with the following rules:

-   Close ports 22 and 3389
-   Set a maximum time window of 3 hours for each so they can be opened per approved request
-   Allow the user who is requesting access to control the source IP addresses
-   Allow the user who is requesting access to establish a successful session upon an approved just-in-time access request

The following PowerShell commands create this JIT configuration:

1.  Assign a variable that holds the just-in-time VM access rules for a VM:

    ```azurepowershell
    $JitPolicy = (@{
        id="/subscriptions/SUBSCRIPTIONID/resourceGroups/RESOURCEGROUP/providers/Microsoft.Compute/virtualMachines/VMNAME";
        ports=(@{
            number=22;
            protocol="*";
            allowedSourceAddressPrefix=@("*");
            maxRequestAccessDuration="PT3H"},
            @{
            number=3389;
            protocol="*";
            allowedSourceAddressPrefix=@("*");
            maxRequestAccessDuration="PT3H"})})
    ```

1.  Insert the VM just-in-time VM access rules into an array:

    ```azurepowershell
    $JitPolicyArr=@($JitPolicy)
    ```

1.  Configure the just-in-time VM access rules on the selected VM:

    ```azurepowershell
    Set-AzJitNetworkAccessPolicy -Kind "Basic" -Location "LOCATION" -Name "default" -ResourceGroupName "RESOURCEGROUP" -VirtualMachine $JitPolicyArr
    ```

    Use the -Name parameter to specify a VM. For example, to establish the JIT configuration for two different VMs, VM1 and VM2, use: `Set-AzJitNetworkAccessPolicy -Name VM1` and `Set-AzJitNetworkAccessPolicy -Name VM2`.

#### Request access to a JIT-enabled VM using PowerShell

In the following example, you can see a just-in-time VM access request to a specific VM for port 22, for a specific IP address, and for a specific amount of time:

Run the following commands in PowerShell:

1.  Configure the VM request access properties:

    ```azurepowershell
    $JitPolicyVm1 = (@{
        id="/subscriptions/SUBSCRIPTIONID/resourceGroups/RESOURCEGROUP/providers/Microsoft.Compute/virtualMachines/VMNAME";
        ports=(@{
            number=22;
            endTimeUtc="2020-07-15T17:00:00.3658798Z";
            allowedSourceAddressPrefix=@("IPV4ADDRESS")})})
    ```

1.  Insert the VM access request parameters in an array:

    ```azurepowershell
    $JitPolicyArr=@($JitPolicyVm1)
    ```

1.  Send the request access (use the resource ID from step 1)

    ```azurepowershell
    Start-AzJitNetworkAccessPolicy -ResourceId "/subscriptions/SUBSCRIPTIONID/resourceGroups/RESOURCEGROUP/providers/Microsoft.Security/locations/LOCATION/jitNetworkAccessPolicies/default" -VirtualMachine $JitPolicyArr
    ```

Learn more in the [PowerShell cmdlet documentation](/powershell/scripting/developer/cmdlet/cmdlet-overview).

### REST API

#### Enable JIT on your VMs using the REST API

The just-in-time VM access feature can be used via the Microsoft Defender for Cloud API. Use this API to get information about configured VMs, add new ones, request access to a VM, and more.

Learn more at [JIT network access policies](/rest/api/defenderforcloud-composite/jit-network-access-policies?view=rest-defenderforcloud-composite-latest&preserve-view=true).

#### Request access to a JIT-enabled VM using the REST API

The just-in-time VM access feature can be used via the Microsoft Defender for Cloud API. Use this API to get information about configured VMs, add new ones, request access to a VM, and more.

Learn more at [JIT network access policies](/rest/api/defenderforcloud-composite/jit-network-access-policies?view=rest-defenderforcloud-composite-latest&preserve-view=true).

## Audit JIT access activity in Defender for Cloud

You can gain insights into VM activities using log search. To view the logs:

1.  From **Just-in-time VM access**, select the **Configured** tab.

1.  For the VM that you want to audit, open the ellipsis menu at the end of the row.

1.  Select **Activity Log** from the menu.

    ![Select just-in-time JIT activity log.](./media/just-in-time-access-usage/jit-select-activity-log.png)

    The activity log provides a filtered view of previous operations for that VM along with time, date, and subscription.

1.  To download the log information, select **Download as CSV**.

## Next Step

-   [Understand just-in-time VM access management](just-in-time-access-overview.md)