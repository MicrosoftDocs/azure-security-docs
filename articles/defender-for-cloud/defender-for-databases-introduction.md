---
title: Overview of Defender for Open-Source Relational Databases
description: Learn about the benefits and features of Microsoft Defender for Open-Source Relational Databases such as PostgreSQL, MySQL, and MariaDB.
ms.date: 05/13/2025
ms.topic: overview
ms.author: dacurwin
author: dcurwin
ms.custom: sfi-image-nochange
#customer intent: As a reader, I want to understand the purpose and features of Microsoft Defender for Open-Source Relational Databases so that I can make informed decisions about its usage.
---

# Overview of Microsoft Defender for Open-Source Relational Databases

In Microsoft Defender for Cloud, the *Defender for Open-Source Relational Databases* plan within Defender for Databases detects anomalous activities that indicate unusual and potentially harmful attempts to access or exploit databases. With this plan, you can address potential threats to databases without the need to be a security expert or manage advanced security-monitoring systems.

## Availability

For pricing information about Defender for Open-Source Relational Databases, see the [Defender for Cloud pricing page](https://azure.microsoft.com/pricing/details/defender-for-cloud/). You can also [estimate costs with the Defender for Cloud cost calculator](cost-calculator.md).

Defender for Open-Source Relational Databases is supported on platform as a service (PaaS) environments for Azure and Amazon Web Services (AWS). It isn't supported on Azure Arc-enabled machines. For more information about availability, see [Defender for Cloud support matrices for Azure commercial/other clouds](support-matrix-cloud-environment.md#cloud-support).

This plan brings threat protections for the following open-source relational databases on Azure.

### Azure Database for PostgreSQL

Protected versions of [Azure Database for PostgreSQL](/azure/postgresql/) include:

- Single Server: General Purpose and Memory Optimized pricing tiers. Learn more in [Pricing tiers in Azure Database for PostgreSQL - Single Server](/azure/postgresql/concepts-pricing-tiers).
- Flexible Server: All pricing tiers.

### Azure Database for MySQL

Protected versions of [Azure Database for MySQL](/azure/mysql/) include:

- Single Server: General Purpose and Memory Optimized pricing tiers.
- Flexible Server: All pricing tiers.

### Azure Database for MariaDB

Protected versions of [Azure Database for MariaDB](/azure/mariadb/) include:

- General Purpose and Memory Optimized pricing tiers. Learn more in [Azure Database for MariaDB pricing tiers](/azure/mariadb/concepts-pricing-tiers).

### Amazon RDS

Amazon Relational Database Service (RDS) instances on AWS (preview) support:

- Aurora PostgreSQL
- Aurora MySQL
- PostgreSQL
- MySQL
- MariaDB

## Benefits

Defender for Cloud provides multicloud alerts on anomalous activities so that you can detect potential threats and respond to them as they occur.

When you enable this plan, Defender for Cloud provides alerts when it detects anomalous database access and query patterns, along with suspicious database activities. The alerts include:

- Details of the suspicious activity that triggered them.
- The associated MITRE ATT&CK tactic.
- Recommended actions for how to investigate and mitigate the threat.
- Options for continuing your investigations by using Microsoft Sentinel.

:::image type="content" source="media/defender-for-databases-introduction/defender-alerts.png" alt-text="Screenshot that shows example multicloud alerts for databases where Microsoft Defender for Open-Source Relational Databases is enabled." lightbox="./media/defender-for-databases-introduction/defender-alerts.png":::

## Alert types

Activities that trigger multicloud alerts enriched with threat intelligence include:

- **Anomalous database access and query patterns**: For example, an abnormally high number of failed sign-in attempts with different credentials (a brute force attack). The alerts can separate successful brute force attacks from unsuccessful ones.
- **Suspicious database activity**: For example, a legitimate user accessing a SQL server from a breached computer that communicated with a crypto-mining command and control (C&C) server.

View the full list of multicloud alerts for database servers in [Alerts for open-source relational databases](alerts-open-source-relational-databases.md).

## Related content

- [Respond to Defender open-source database alerts](defender-for-databases-usage.md)
- [Common questions about Defender for Databases](faq-defender-for-databases.yml)
