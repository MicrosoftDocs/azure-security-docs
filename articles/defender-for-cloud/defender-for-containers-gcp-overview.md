---
title: Defender for Containers on GCP (GKE) - Overview
description: Learn about Microsoft Defender for Containers capabilities for Google Kubernetes Engine (GKE) clusters.
ms.topic: overview
ms.date: 01/08/2026
ai-usage: ai-assisted
---

# Defender for Containers on GCP (GKE) - overview

Microsoft Defender for Containers helps you protect Google Kubernetes Engine (GKE) clusters by providing vulnerability scanning, runtime threat detection, software supply chain capabilities, and security posture management.

For pricing information, see [Microsoft Defender for Cloud pricing](https://azure.microsoft.com/pricing/details/defender-for-cloud/).

## What is Defender for Containers on GCP?

Defender for Containers on GCP is a cloud-native security solution that protects your GKE clusters and containerized workloads running in Google Cloud Platform (GCP). When you connect your GCP projects to Microsoft Defender for Cloud, you gain visibility and protection across your entire container estate, whether running in Azure, GCP, AWS, or on-premises.

The solution integrates with GCP services such as Google Container Registry (GCR), Artifact Registry, and GKE native security features. It also uses Microsoft's threat intelligence and security capabilities. This integration enables organizations to maintain consistent security policies and threat detection across their multicloud container deployments.

## How it works with GCP

Defender for Containers uses a lightweight architecture that respects GCP's security model. When you create a GCP connector in Defender for Cloud, the solution automatically:

- Discovers all GKE clusters in your connected GCP projects
- Connects clusters to Azure Arc for centralized management and monitoring


This architecture is designed to minimize performance impact on your workloads while maintaining GCP security boundaries and compliance requirements.

## Key capabilities

Defender for Containers delivers comprehensive security across four critical areas:

| Capability | Description | Key Features |
|------------|-------------|--------------|
| **Vulnerability assessment** | Scans container images in GCR and Artifact Registry for known vulnerabilities | • Native integration with Google registries<br>• Support for private endpoints<br>• Automated scanning on push<br>• CVSS scoring with remediation guidance |
| **Runtime threat protection** | Monitors GKE clusters in real-time for malicious activities and anomalies | • GKE audit log analysis<br>• Workload behavior monitoring<br>• Network anomaly detection<br>• Integration with Binary Authorization |
| **Security posture management** | Evaluates cluster configurations against security benchmarks and best practices | • CIS GKE Benchmark assessment<br>• GKE-specific recommendations<br>• Workload Identity validation<br>• Pod Security Standards enforcement |
| **Gated deployment** | Prevents vulnerable or misconfigured workloads from reaching production | • Block deployments based on vulnerability severity<br>• Enforce security baselines for configurations<br>• Integration with Azure Policy and admission controllers<br>• DevOps pipeline gates |

## Architecture overview

The Defender for Containers architecture on GCP includes the following components:

**GCP Connector**: Establishes secure connectivity between your GCP projects and Microsoft Defender for Cloud by using service account authentication or Workload Identity Federation.

**Azure Arc for Kubernetes**: Provides a control plane for managing GKE clusters from Azure without requiring inbound connectivity.

**Defender Sensor**: Deployed as a DaemonSet on each GKE cluster, collects runtime telemetry and security events with minimal resource consumption.

**Registry Integration**: Native integration with GCR and Artifact Registry enables vulnerability scanning without moving images or exposing credentials.

**Cloud Logging Integration**: Leverages GKE audit logs and Cloud Logging for comprehensive security monitoring without duplicating data.

These components work together to provide end-to-end security while respecting GCP's shared responsibility model and your existing security controls.

## Deployment options

Defender for Containers on GCP supports the following deployment options:

- **[Azure portal deployment](defender-for-containers-gcp-enable-portal.md)** - Guided experience with automated setup scripts
- **[Programmatic deployment](defender-for-containers-gcp-enable-programmatically.md)** - Script-based deployment for automation scenarios

Choose the deployment method that meets your operational practices and security requirements.

## Google Cloud console prerequisites

Before you deploy Defender for Containers on Google Kubernetes Engine (GKE), make sure that the following prerequisites are met:

- An active Azure subscription with Defender for Cloud enabled
- A GCP project with Owner or Security Admin role
- One or more GKE clusters running version 1.19 or later
- The following GCP APIs enabled: Kubernetes Engine, Container Registry, and Cloud Asset
- Network connectivity from GKE to Azure endpoints


## View your current coverage

Defender for Cloud provides access to [workbooks](custom-dashboards-azure-workbooks.md) through [Azure workbooks](/azure/azure-monitor/visualize/workbooks-overview). Workbooks are customizable reports that provide insights into your security posture.

The [coverage workbook](custom-dashboards-azure-workbooks.md#coverage-workbook) helps you understand your current coverage by showing which plans are enabled on your subscriptions and resources.

## Next steps

To deploy Defender for Containers on GKE, choose one of the following methods:

- [Enable all components via portal](defender-for-containers-gcp-enable-portal.md) - Recommended for initial setup
- [Deploy programmatically](defender-for-containers-gcp-enable-programmatically.md) - For automation and scale
- [Verify deployment](defender-for-containers-gcp-verify.md) - Ensure components are working correctly
