---
title: Configure the Microsoft Security DevOps GitHub action
description: Learn how to configure the Microsoft Security DevOps GitHub action to enhance your project's security and DevOps processes.
ms.date: 05/18/2025
ms.topic: how-to
---

# Configure the Microsoft Security DevOps GitHub action

Microsoft Security DevOps is a command line application that integrates static analysis tools into the development lifecycle. Security DevOps installs, configures, and runs the latest versions of static analysis tools such as, SDL, security and compliance tools. Security DevOps is data-driven with portable configurations that enable deterministic execution across multiple environments.

Microsoft Security DevOps uses the following Open Source tools:

| Name | Language | License |
|--|--|--|
| [AntiMalware](https://www.microsoft.com/windows/comprehensive-security) | AntiMalware protection in Windows from Microsoft Defender for Endpoint, that scans for malware and breaks the build if malware has been found. This tool scans by default on windows-latest agent. | Not Open Source |
| [Bandit](https://github.com/PyCQA/bandit) | Python | [Apache License 2.0](https://github.com/PyCQA/bandit/blob/master/LICENSE) |
| [BinSkim](https://github.com/Microsoft/binskim) | Binary--Windows, ELF | [MIT License](https://github.com/microsoft/binskim/blob/main/LICENSE) |
| [Checkov](https://github.com/bridgecrewio/checkov) | Terraform, Terraform plan, CloudFormation, AWS SAM, Kubernetes, Helm charts, Kustomize, Dockerfile, Serverless, Bicep, OpenAPI, ARM | [Apache License 2.0](https://github.com/bridgecrewio/checkov/blob/main/LICENSE) |
| [ESlint](https://github.com/eslint/eslint) | JavaScript | [MIT License](https://github.com/eslint/eslint/blob/main/LICENSE) |
| [Template Analyzer](https://github.com/Azure/template-analyzer) | ARM Template, Bicep | [MIT License](https://github.com/Azure/template-analyzer/blob/main/LICENSE.txt) |
| [Terrascan](https://github.com/accurics/terrascan) | Terraform (HCL2), Kubernetes (JSON/YAML), Helm v3, Kustomize, Dockerfiles, CloudFormation | [Apache License 2.0](https://github.com/accurics/terrascan/blob/master/LICENSE) |
| [Trivy](https://github.com/aquasecurity/trivy) | container images, Infrastructure as Code (IaC) | [Apache License 2.0](https://github.com/aquasecurity/trivy/blob/main/LICENSE) |

## Prerequisites

- An Azure subscription If you don’t have an Azure subscription, create a [free account](https://azure.microsoft.com/free/) before you begin.

- [Connect your GitHub repositories](quickstart-onboard-github.md).

- Open the [Microsoft Security DevOps GitHub action](https://github.com/marketplace/actions/security-devops-action) in a new window.

- Ensure that [Workflow permissions are set to Read and Write](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/managing-github-actions-settings-for-a-repository#setting-the-permissions-of-the-github_token-for-your-repository) on the GitHub repository. This includes setting "id-token: write" permissions in the GitHub Workflow for federation with Defender for Cloud.

## Configure the Microsoft Security DevOps GitHub action

**To setup GitHub action**:

1. Sign in to [GitHub](https://www.github.com).

1. Select a repository you want to configure the GitHub action to.

1. Select **Actions**.

    :::image type="content" source="media/msdo-github-action/actions.png" alt-text="Screenshot that shows you where the Actions button is located.":::

1. Select **New workflow**.

1. On the Get started with GitHub Actions page, select **set up a workflow yourself**

    :::image type="content" source="media/msdo-github-action/new-workflow.png" alt-text="Screenshot showing where to select the new workflow button.":::

1. In the text box, enter a name for your workflow file. For example, `msdevopssec.yml`.

    :::image type="content" source="media/msdo-github-action/devops.png" alt-text="Screenshot that shows you where to enter a name for your new workflow.":::

1. Copy and paste the following [sample action workflow](https://github.com/microsoft/security-devops-action/blob/main/.github/workflows/sample-workflow.yml) into the Edit new file tab.

    ```yml
    name: MSDO
    on:
      push:
        branches:
          - main

    jobs:
      sample:
        name: Microsoft Security DevOps

        # Windows and Linux agents are supported
        runs-on: windows-latest
    
        permissions:
          contents: read
          id-token: write
          actions: read
          # Write access for security-events is only required for customers looking for MSDO results to appear in the codeQL security alerts tab on GitHub (Requires GHAS)
          security-events: write
    
        steps:

          # Checkout your code repository to scan
        - uses: actions/checkout@v3

          # Run analyzers
        - name: Run Microsoft Security DevOps
          uses: microsoft/security-devops-action@latest
          id: msdo
        # with:
          # config: string. Optional. A file path to an MSDO configuration file ('*.gdnconfig').
          # policy: 'GitHub' | 'microsoft' | 'none'. Optional. The name of a well-known Microsoft policy. If no configuration file or list of tools is provided, the policy may instruct MSDO which tools to run. Default: GitHub.
          # categories: string. Optional. A comma-separated list of analyzer categories to run. Values: 'code', 'artifacts', 'IaC', 'containers'. Example: 'IaC, containers'. Defaults to all.
          # languages: string. Optional. A comma-separated list of languages to analyze. Example: 'javascript,typescript'. Defaults to all.
          # tools: string. Optional. A comma-separated list of analyzer tools to run. Values: 'bandit', 'binskim', 'checkov', 'eslint', 'templateanalyzer', 'terrascan', 'trivy'.

          # Upload alerts to the Security tab - required for MSDO results to appear in the codeQL security alerts tab on GitHub (Requires GHAS)
        # - name: Upload alerts to Security tab
        #  uses: github/codeql-action/upload-sarif@v3
        #  with:
        #    sarif_file: ${{ steps.msdo.outputs.sarifFile }}

          # Upload alerts file as a workflow artifact - required for MSDO results to appear in the codeQL security alerts tab on GitHub (Requires GHAS)
        # - name: Upload alerts file as a workflow artifact
        #  uses: actions/upload-artifact@v3
        #  with:  
        #    name: alerts
        #    path: ${{ steps.msdo.outputs.sarifFile }}
    ```
   > [!NOTE]
   >  **For additional tool configuration options and instructions, see [the Microsoft Security DevOps wiki](https://github.com/microsoft/security-devops-action/wiki)**

1. Select **Start commit**

   :::image type="content" source="media/msdo-github-action/start-commit.png" alt-text="Screenshot showing you where to select start commit.":::
   
1. Select **Commit new file**. Note that the process can take up to one minute to complete.

   :::image type="content" source="media/msdo-github-action/commit-new.png" alt-text="Screenshot showing you how to commit a new file.":::

1. Select **Actions** and  verify the new action is running.

   :::image type="content" source="media/msdo-github-action/verify-actions.png" alt-text="Screenshot showing you where to navigate to, to see that your new action is running." lightbox="media/msdo-github-action/verify-actions.png":::

## View Scan Results

**To view your scan results**:

1. Sign in to Azure.

1. Navigate to Defender for Cloud > DevOps Security.

1. From the DevOps security blade, you should begin seeing the same MSDO security results developers see in their CI logs within minutes for the associated repository. Customers with GitHub Advanced Security will see the findings ingested from these tools as well.

## Learn more

- Learn about [GitHub actions for Azure](/azure/developer/github/github-actions).

- Learn how to [deploy apps from GitHub to Azure](/azure/developer/github/deploy-to-azure).

## Next steps

Learn more about [DevOps security in Defender for Cloud](defender-for-devops-introduction.md).

Learn how to [connect your GitHub Organizations](quickstart-onboard-github.md) to Defender for Cloud.
