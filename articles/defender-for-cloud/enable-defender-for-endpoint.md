---
title: Enable the Defender for Endpoint integration
description: Learn how to deploy the Microsoft Defender for Endpoint integration in Defender for Cloud to protect Azure, hybrid, and multicloud machines.
author: dcurwin
ms.author: dacurwin
ms.topic: how-to
ms.date: 03/13/2024
#customer intent: As a user, I want to learn how to enable the Defender for Endpoint integration in Defender for Cloud so that I can protect my Azure, hybrid, and multicloud machines.
---

# Enable the Defender for Endpoint integration

Microsoft Defender for Cloud's integration with Microsoft Defender for Endpoint, provides a cloud-based endpoint security solution that offers a wide range of features. The integration provides risk-based vulnerability management and assessment, which helps identify and prioritize vulnerabilities that need to be addressed. The solution also includes attack surface reduction, which helps minimize the attack surface of endpoints, as well as behavioral-based and cloud-powered protection to detect and respond to threats. Additionally, Microsoft Defender for Endpoint offers endpoint detection and response (EDR), automatic investigation and remediation, and managed hunting services to help organizations quickly detect, investigate, and respond to security incidents.

## Prerequisites

Before you can enable the Microsoft Defender for Endpoint integration with Defender for Cloud, you must confirm that your machine meets the necessary requirements for Defender for Endpoint:

- Ensure the machine is connected to Azure and the internet as required:

  - **Azure virtual machines (Windows or Linux)** - Configure the network settings described in configure device proxy and internet connectivity settings: [Windows](/microsoft-365/security/defender-endpoint/configure-proxy-internet) or [Linux](/microsoft-365/security/defender-endpoint/linux-static-proxy-configuration).

  - **On-premises machines** - Connect your target machines to Azure Arc as explained in [Connect hybrid machines with Azure Arc-enabled servers](/azure/azure-arc/servers/learn/quick-enable-hybrid-vm).

- Enable **Microsoft Defender for Servers**. See [Quickstart: Enable Defender for Cloud's enhanced security features](enable-enhanced-security.md).

    > [!IMPORTANT]
    > Defender for Cloud's integration with Microsoft Defender for Endpoint is enabled by default. So when you enable enhanced security features, you give consent for Microsoft Defender for Servers to access the Microsoft Defender for Endpoint data related to vulnerabilities, installed software, and alerts for your endpoints.

- For Windows servers, make sure that your servers meet the requirements for [onboarding Microsoft Defender for Endpoint](/microsoft-365/security/defender-endpoint/configure-server-endpoints#windows-server-2012-r2-and-windows-server-2016).

- For Linux servers, you must have Python installed. Python 3 is recommended for all distros, but is required for RHEL 8.x and Ubuntu 20.04 or higher. If needed, see Step-by-step Instructions for Installing Python on Linux.

- If you've moved your subscription between Azure tenants, some manual preparatory steps are also required. For details, [contact Microsoft support](https://portal.azure.com/#view/Microsoft_Azure_Support/HelpAndSupportBlade/~/overview).

## Enable the integration

- On [Windows](#windows)

- On [Linux](#linux)

### Windows

[The Defender for Endpoint unified solution](/microsoft-365/security/defender-endpoint/configure-server-endpoints#new-windows-server-2012-r2-and-2016-functionality-in-the-modern-unified-solution) doesn't use or require installation of the Log Analytics agent. The unified solution is automatically deployed for Azure Windows 2012 R2 and 2016 servers, Windows servers connected through Azure Arc, and Windows multicloud servers connected through the multicloud connectors.

You'll deploy Defender for Endpoint to your Windows machines in one of two ways - depending on whether you've already deployed it to your Windows machines:

- [Users with Defender for Servers enabled and Microsoft Defender for Endpoint deployed](#users-with-defender-for-servers-enabled-and-microsoft-defender-for-endpoint-deployed)
- [Users who never enabled the integration with Microsoft Defender for Endpoint](#users-who-never-enabled-the-integration-with-microsoft-defender-for-endpoint-for-windows)

#### Users with Defender for Servers enabled and Microsoft Defender for Endpoint deployed

If you've already enabled the integration with **Defender for Endpoint**, you have complete control over when and whether to deploy the Defender for Endpoint unified solution to your **Windows** machines.

To deploy the Defender for Endpoint unified solution, you need to use the [REST API call](#enable-the-microsoft-defender-for-endpoint-unified-solution-at-scale) or the Azure portal:

1. From Defender for Cloud's menu, select **Environment settings** and select the subscription with the Windows machines that you want to receive Defender for Endpoint.

1. In the Monitoring coverage column of the Defender for Servers plan, select **Settings**.

    The status of the Endpoint protections component is **Partial**, meaning that not all parts of the component are enabled.

    > [!NOTE]
    > If the status is **Off**, use the instructions in [Users who've never enabled the integration with Microsoft Defender for Endpoint for Windows](#users-who-never-enabled-the-integration-with-microsoft-defender-for-endpoint-for-windows).

1. Select **Fix** to see the components that aren't enabled.

    :::image type="content" source="./media/integration-defender-for-endpoint/fix-defender-for-endpoint.png" alt-text="Screenshot of Fix button that enables Microsoft Defender for Endpoint support.":::

1. To enable the Unified solution for Windows Server 2012 R2 and 2016 machines, select **Enable**.

    :::image type="content" source="./media/integration-defender-for-endpoint/enable-defender-for-endpoint-unified.png" alt-text="Screenshot of enabling the use of the Defender for Endpoint unified solution for Windows Server 2012 R2 and 2016 machines.":::

1. To save the changes, select **Save** at the top of the page and then select **Continue** in the Settings and monitoring page.

Microsoft Defender for Cloud will:

- Stop the existing Defender for Endpoint process in the Log Analytics agent that collects data for Defender for Servers.
- Install the Defender for Endpoint unified solution for all existing and new Windows Server 2012 R2 and 2016 machines.

Microsoft Defender for Cloud will automatically onboard your machines to Microsoft Defender for Endpoint. Onboarding might take up to 12 hours. For new machines created after the integration has been enabled, onboarding takes up to an hour.

> [!NOTE]
> If you choose not to deploy the Defender for Endpoint unified solution to your Windows 2012 R2 and 2016 servers in Defender for Servers Plan 2 and then downgrade Defender for Servers to Plan 1, the Defender for Endpoint unified solution is not deployed to those servers so that your existing deployment is not changed without your explicit consent.

#### Users who never enabled the integration with Microsoft Defender for Endpoint for Windows

If you've never enabled the integration for Windows, Endpoint protection enables Defender for Cloud to deploy Defender for Endpoint to *both* your Windows and Linux machines.

To deploy the Defender for Endpoint unified solution, you'll need to use the [REST API call](#enable-the-microsoft-defender-for-endpoint-unified-solution-at-scale) or the Azure portal:

1. From Defender for Cloud's menu, select **Environment settings** and select the subscription with the machines that you want to receive Defender for Endpoint.

1. In the status of the Endpoint protection component, select **On** to enable the integration with Microsoft Defender for Endpoint.

    :::image type="content" source="media/integration-defender-for-endpoint/enable-defender-for-endpoint.png" alt-text="Screenshot of Status toggle that enables Microsoft Defender for Endpoint." lightbox="media/integration-defender-for-endpoint/enable-defender-for-endpoint.png":::

The Defender for Endpoint agent unified solution is deployed to all of the machines in the selected subscription.

### Linux

You'll deploy Defender for Endpoint to your Linux machines in one of these ways, depending on whether you've already deployed it to your Windows machines:

- Enable for a specific subscription in the Azure portal environment settings
  - [Existing users with Defender for Cloud's enhanced security features enabled and Microsoft Defender for Endpoint for Windows](#existing-users-with-defender-for-clouds-enhanced-security-features-enabled-and-microsoft-defender-for-endpoint-for-windows)
  - [New users who never enabled the integration with Microsoft Defender for Endpoint for Windows](#new-users-who-never-enabled-the-integration-with-microsoft-defender-for-endpoint-for-windows)
- [Enable for multiple subscriptions in the Azure portal dashboard](#enable-on-multiple-subscriptions-in-the-azure-portal-dashboard)
- Enable for multiple subscriptions with a PowerShell script

> [!NOTE]
> When you enable automatic deployment, Defender for Endpoint for Linux installation will abort on machines with pre-existing running services using [fanotify](/microsoft-365/security/defender-endpoint/microsoft-defender-endpoint-linux#system-requirements) and other services that can also cause Defender for Endpoint to malfunction or might be affected by Defender for Endpoint, such as security services.
> After you validate potential compatibility issues, we recommend that you manually install Defender for Endpoint on these servers.

#### Existing users with Defender for Cloud's enhanced security features enabled and Microsoft Defender for Endpoint for Windows

If you've already enabled the integration with **Defender for Endpoint for Windows**, you have complete control over when and whether to deploy Defender for Endpoint to your **Linux** machines.

1. From Defender for Cloud's menu, select **Environment settings** and select the subscription with the Linux machines that you want to receive Defender for Endpoint.

1. In the Monitoring coverage column of the Defender for Server plan, select **Settings**.

    The status of the Endpoint protections component is **Partial**, meaning that not all parts of the component are enabled.

    > [!NOTE]
    > If the status is **Off** isn't selected, use the instructions in [Users who've never enabled the integration with Microsoft Defender for Endpoint for Windows](#users-who-never-enabled-the-integration-with-microsoft-defender-for-endpoint-for-windows).

1. Select **Fix** to see the components that aren't enabled.

    :::image type="content" source="./media/integration-defender-for-endpoint/fix-defender-for-endpoint.png" alt-text="Screenshot of Fix button that enables Microsoft Defender for Endpoint support.":::

1. To enable deployment to Linux machines, select **Enable**.

    :::image type="content" source="./media/integration-defender-for-endpoint/enable-defender-for-endpoint-linux.png" alt-text="Screenshot of enabling the integration between Defender for Cloud and Microsoft's EDR solution, Microsoft Defender for Endpoint for Linux.":::

1. To save the changes, select **Save** at the top of the page and then select **Continue** in the Settings and monitoring page.

    Microsoft Defender for Cloud will:

    - Automatically onboard your Linux machines to Defender for Endpoint
    - Detect any previous installations of Defender for Endpoint and reconfigure them to integrate with Defender for Cloud

    Microsoft Defender for Cloud will automatically onboard your machines to Microsoft Defender for Endpoint. Onboarding might take up to 12 hours. For new machines created after the integration has been enabled, onboarding takes up to an hour.

    > [!NOTE]
    > The next time you return to this page of the Azure portal, the **Enable for Linux machines** button won't be shown. To disable the integration for Linux, you'll need to disable it for Windows too by turning the toggle **off** in **Endpoint Protection**, and selecting **Continue**.

1. To verify installation of Defender for Endpoint on a Linux machine, run the following shell command on your machines:

    `mdatp health`

    If Microsoft Defender for Endpoint is installed, you'll see its health status:

    `healthy : true`

    `licensed: true`

    Also, in the Azure portal you'll see a new Azure extension on your machines called `MDE.Linux`.

#### New users who never enabled the integration with Microsoft Defender for Endpoint for Windows

If you've never enabled the integration for Windows, endpoint protection enables Defender for Cloud to deploy Defender for Endpoint to *both* your Windows and Linux machines.

1. From Defender for Cloud's menu, select **Environment settings** and select the subscription with the Linux machines that you want to receive Defender for Endpoint.

1. In the Monitoring coverage column of the Defender for Server plan, select **Settings**.

1. In the status of the Endpoint protection component, select **On** to enable the integration with Microsoft Defender for Endpoint.

    :::image type="content" source="media/integration-defender-for-endpoint/enable-defender-for-endpoint.png" alt-text="Screenshot of Status toggle that enables Microsoft Defender for Endpoint." lightbox="media/integration-defender-for-endpoint/enable-defender-for-endpoint.png":::

    Microsoft Defender for Cloud will:

    - Automatically onboard your Windows and Linux machines to Defender for Endpoint
    - Detect any previous installations of Defender for Endpoint and reconfigure them to integrate with Defender for Cloud

    Onboarding might take up to 1 hour.

1. Select **Continue** and **Save** to save your settings.

1. To verify installation of Defender for Endpoint on a Linux machine, run the following shell command on your machines:

    `mdatp health`

    If Microsoft Defender for Endpoint is installed, you'll see its health status:

    `healthy : true`

    `licensed: true`

    In addition, in the Azure portal you'll see a new Azure extension on your machines called `MDE.Linux`.

#### Enable on multiple subscriptions in the Azure portal dashboard

If one or more of your subscriptions don't have Endpoint protections enabled for Linux machines, you'll see an insight panel in the Defender for Cloud dashboard. The insight panel tells you about subscriptions that have Defender for Endpoint integration enabled for Windows machines, but not for Linux machines. You can use the insight panel to see the affected subscriptions with the number of affected resources in each subscription. Subscriptions that don't have Linux machines show no affected resources. You can then select the subscriptions to enable endpoint protection for Linux integration.

After you select **Enable** in the insight panel, Defender for Cloud:

- Automatically onboards your Linux machines to Defender for Endpoint in the selected subscriptions.
- Detects any previous installations of Defender for Endpoint and reconfigure them to integrate with Defender for Cloud.

Use the [Defender for Endpoint status workbook](https://github.com/Azure/Microsoft-Defender-for-Cloud/tree/main/Workbooks/Defender%20for%20Servers%20Deployment%20Status) to verify installation and deployment status of Defender for Endpoint on a Linux machine.

#### Enable on multiple subscriptions with a PowerShell script

Use our [PowerShell script](https://github.com/Azure/Microsoft-Defender-for-Cloud/tree/main/Powershell%20scripts/MDE%20Integration/Enable%20MDE%20Integration%20for%20Linux) from the Defender for Cloud GitHub repository to enable endpoint protection on Linux machines that are in multiple subscriptions.

#### Manage automatic updates configuration for Linux

In Windows, Defender for Endpoint version updates are provided via continuous knowledge base updates; in Linux you need to update the Defender for Endpoint package. When you use Defender for Servers with the `MDE.Linux` extension, automatic updates for Microsoft Defender for Endpoint are enabled by default. If you wish to manage the Defender for Endpoint version updates manually, you can disable automatic updates on your machines. To do so, add the following tag for machines onboarded with the `MDE.Linux` extension.

- Tag name: 'ExcludeMdeAutoUpdate'
- Tag value:  'true'

This configuration is supported for Azure VMs and Azure Arc machines, where the `MDE.Linux` extension initiates auto-update.

## Enable the Microsoft Defender for Endpoint unified solution at scale

You can also enable the Defender for Endpoint unified solution at scale through the supplied REST API version 2022-05-01. For full details, see the [API documentation](/rest/api/defenderforcloud-composite/settings/update?view=rest-defenderforcloud-composite-latest&tabs=HTTP&preserve-view=true).

Here's an example request body for the PUT request to enable the Defender for Endpoint unified solution:

URI: `https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.Security/settings/WDATP?api-version=2022-05-01`

```json
{
    "name": "WDATP",
    "type": "Microsoft.Security/settings",
    "kind": "DataExportSettings",
    "properties": {
        "enabled": true
    }
}
```

## Track MDE deployment status

You can use the [Defender for Endpoint deployment status workbook](https://github.com/Azure/Microsoft-Defender-for-Cloud/tree/main/Workbooks/Defender%20for%20Servers%20Deployment%20Status) to track the Defender for Endpoint deployment status on your Azure VMs and non-Azure machines that are connected via Azure Arc. The interactive workbook provides an overview of machines in your environment showing their Microsoft Defender for Endpoint extension deployment status.

## Access the Microsoft Defender for Endpoint portal

1. Ensure the user account has the necessary permissions. Learn more in [Assign user access to Microsoft Defender Security Center](/microsoft-365/security/defender-endpoint/assign-portal-access).

1. Check whether you have a proxy or firewall that is blocking anonymous traffic. The Defender for Endpoint sensor connects from the system context, so anonymous traffic must be permitted. To ensure unhindered access to the Defender for Endpoint portal, follow the instructions in [Enable access to service URLs in the proxy server](/microsoft-365/security/defender-endpoint/configure-proxy-internet#enable-access-to-microsoft-defender-for-endpoint-service-urls-in-the-proxy-server).

1. Open the [Microsoft Defender Portal](https://security.microsoft.com/). Learn about [Microsoft Defender for Endpoint in Microsoft Defender XDR](/microsoft-365/security/defender/microsoft-365-security-center-mde).

## Send a test alert

To generate a benign test alert from Defender for Endpoint, select the tab for the relevant operating system of your endpoint:

- [Test on Windows](#test-on-windows)

- [Test on Linux](#test-on-linux)

### Test on Windows

For endpoints running Windows:

1. Create a folder 'C:\test-MDATP-test'.
1. Use Remote Desktop to access your machine.
1. Open a command-line window.
1. At the prompt, copy and run the following command. The command prompt window will close automatically.

    ```powershell
    powershell.exe -NoExit -ExecutionPolicy Bypass -WindowStyle Hidden (New-Object System.Net.WebClient).DownloadFile('http://127.0.0.1/1.exe', 'C:\\test-MDATP-test\\invoice.exe'); Start-Process 'C:\\test-MDATP-test\\invoice.exe'
    ```

    :::image type="content" source="./media/integration-defender-for-endpoint/generate-edr-alert.png" alt-text="A command prompt window with the command to generate a test alert.":::

    If the command is successful, you'll see a new alert on the workload protection dashboard and the Microsoft Defender for Endpoint portal. This alert might take a few minutes to appear.
1. To review the alert in Defender for Cloud, go to **Security alerts** > **Suspicious PowerShell CommandLine**.
1. From the investigation window, select the link to go to the Microsoft Defender for Endpoint portal.

    > [!TIP]
    > The alert is triggered with **Informational** severity.

### Test on Linux

For endpoints running Linux:

1. Download the test alert tool from: <https://aka.ms/LinuxDIY>
1. Extract the contents of the zip file and execute this shell script:

    `./mde_linux_edr_diy`

    If the command is successful, you'll see a new alert on the workload protection dashboard and the Microsoft Defender for Endpoint portal. This alert might take a few minutes to appear.

1. To review the alert in Defender for Cloud, go to **Security alerts** > **Enumeration of files with sensitive data**.
1. From the investigation window, select the link to go to the Microsoft Defender for Endpoint portal.

    > [!TIP]
    > The alert is triggered with **Low** severity.

## Remove Defender for Endpoint from a machine

To remove the Defender for Endpoint solution from your machines:

1. Disable the integration:
    1. From Defender for Cloud's menu, select **Environment settings** and select the subscription with the relevant machines.
    1. In the Defender plans page, select **Settings & Monitoring**.
    1. In the status of the Endpoint protection component, select **Off** to disable the integration with Microsoft Defender for Endpoint.
    1. Select **Continue** and **Save** to save your settings.

1. Remove the MDE.Windows/MDE.Linux extension from the machine.

1. Follow the steps in [Offboard devices from the Microsoft Defender for Endpoint service](/microsoft-365/security/defender-endpoint/offboard-machines) from the Defender for Endpoint documentation.

## Related content

- [Platforms and features supported by Microsoft Defender for Cloud](security-center-os-coverage.md)
- [Learn how recommendations help you protect your Azure resources](review-security-recommendations.md)
- View common question about the [Defender for Cloud integration with Microsoft Defender for Endpoint](faq-defender-for-servers.yml)
