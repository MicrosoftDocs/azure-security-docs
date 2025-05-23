---
title: Deploy Defender for Azure SQL Databases
description: Learn how to enable Defender for Azure SQL Databases as part of the Databases plan on your Azure subscription to enhance your database security.
ms.topic: how-to
ms.date: 01/26/2025
#customer intent: As a database administrator, I want to enable Defender for Azure SQL Databases so that I can enhance the security of my databases.
---

# Deploy Defender for Azure SQL Databases

Defender for Azure SQL Databases in Microsoft Defender for Cloud lets you protect your Azure SQL databases with attack detection and threat response. Defender for Cloud protects Azure SQL database engines and data types based on their attack surface and security risks.

## Prerequisites

- You need a Microsoft Azure subscription. If you don't have an Azure subscription, [sign up for a free subscription](https://azure.microsoft.com/pricing/free-trial/).

- You must [enable Microsoft Defender for Cloud](get-started.md#enable-defender-for-cloud-on-your-azure-subscription) on your Azure subscription.

- You must connect a [non-Azure machines](quickstart-onboard-machines.md), [Amazon Web Service (AWS) account](quickstart-onboard-aws.md), or [Google Cloud Platform (GCP)](quickstart-onboard-gcp.md).

- You must [enable the Defender for Databases plan](tutorial-enable-databases-plan.md) on your Defender for Cloud subscription.

## Enable Defender for Azure SQL Databases

Enabling the Defender for Azure SQL Databases plan activates protection for all Azure SQL databases in your subscription.

1. Sign in to the [Azure portal](https://portal.azure.com).

1. Search for and select **Microsoft Defender for Cloud**.

1. In the Defender for Cloud menu, select **Environment settings**.

1. Select the relevant Azure subscription, AWS account, or GCP project.

1. Locate the Databases plan and select **Select types**.

    :::image type="content" source="media//enable-sql-database-plan//select-type.png" alt-text="Screenshot of the environment settings page that shows you where the select types button is located." lightbox="media//enable-sql-database-plan//select-type.png":::

1. Switch Azure SQL Databases to **On**.

    :::image type="content" source="media//enable-sql-database-plan//toggle-on.png" alt-text="Screenshot that shows you where to select, to enable the Azure SQL Databases plan." lightbox="media//enable-sql-database-plan//toggle-on.png":::

1. Select **Continue**.

1. Select **Save**.

 ## Related content

- [Enable Microsoft Defender for SQL servers on machines](defender-for-sql-usage.md)
