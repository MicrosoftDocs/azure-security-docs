---
title: Enable Defender for open-source relational databases on Azure
description: Learn how to enable Microsoft Defender for open-source relational databases to detect potential security threats on Azure environments.
ms.date: 06/30/2025
ms.topic: how-to
ms.author: dacurwin
author: dcurwin
#customer intent: As a reader, I want to learn how to configure Microsoft Defender for open-source relational databases to enhance the security of my Azure databases.
---

# Enable Defender for open-source relational databases on Azure

Microsoft Defender for Cloud detects anomalous activities indicating unusual and potentially harmful attempts to access or exploit databases for the following services:

- [Azure Database for PostgreSQL](/azure/postgresql/)
- [Azure Database for MySQL](/azure/mysql/)
- [Azure Database for MariaDB](/azure/mariadb/)

To get alerts from the Microsoft Defender plan, you need to follow the instructions on this page to enable Defender for open-source relational databases Azure.

Learn more about this Microsoft Defender plan in [Overview of Microsoft Defender for open-source relational databases](defender-for-databases-introduction.md).

## Prerequisites

- You need a Microsoft Azure subscription. If you don't have an Azure subscription, you can [sign up for a free subscription](https://azure.microsoft.com/pricing/free-trial/).

- You must [enable Microsoft Defender for Cloud](get-started.md#enable-defender-for-cloud-on-your-azure-subscription) on your Azure subscription.

- (Optional) Connect your [non-Azure machines](quickstart-onboard-machines.md)

## Enable Defender for open-source relational databases on your Azure account

1. Sign in to [the Azure portal](https://portal.azure.com)

1. Search for and select **Azure Database for MySQL servers**.

1. Select the relevant database.

1. Expand the security menu.
 
1. Select **Microsoft Defender for Cloud**.

1. If Defender for open-source relational databases isn't enabled, the **Enable Microsoft Defender for [Database type]** (for example, "Microsoft Defender for MySQL") button will be present, select the button.

    :::image type="content" source="media/defender-for-databases-usage/enable-defender-for-mysql.png" alt-text="Screenshot that shows you where and what the Enable Microsoft Defender for MySQL button looks like and is located." lightbox="media/defender-for-databases-usage/enable-defender-for-mysql.png":::

    > [!TIP]
    > This page in the portal will be the same regardless of the database type (PostgreSQL, MySQL, or MariaDB).

1. Select **Save**

## Related content

- [Optional configurations after in-place migration from Azure Database for MySQL Single Server to Flexible Server](/azure/mysql/migrate/whats-happening-to-mysql-single-server#configure-microsoft-defender-for-cloud-properties-in-flexible-server).

## Next step

> [!div class="nextstepaction"]
> [Respond to Defender OSS alerts](defender-for-databases-usage.md)
