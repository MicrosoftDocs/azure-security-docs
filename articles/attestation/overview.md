---
title: Azure Attestation overview
description: An overview of Microsoft Azure Attestation, a solution for attesting Trusted Execution Environments (TEEs)
services: attestation
author: msmbaldwin
ms.service: azure-attestation
ms.topic: overview
ms.date: 04/15/2025
ms.author: mbaldwin
ms.custom: references_regions

---
# Microsoft Azure Attestation

Microsoft Azure Attestation is a unified solution for remotely verifying the trustworthiness of a platform and integrity of the binaries running inside it. The service supports attestation of the platforms backed by Trusted Platform Modules (TPMs) alongside the ability to attest to the state of Trusted Execution Environments (TEEs) such as [Intel® Software Guard Extensions](https://www.intel.com/content/www/us/en/architecture-and-technology/software-guard-extensions.html) (SGX) enclaves, [Virtualization-based Security](/windows-hardware/design/device-experiences/oem-vbs) (VBS) enclaves, [Trusted Platform Modules (TPMs)](/windows/security/information-protection/tpm/trusted-platform-module-overview),  [Trusted launch for Azure VMs](/azure/virtual-machines/trusted-launch) and [Azure confidential VMs](/azure/confidential-computing/confidential-vm-overview).

Attestation is a process for demonstrating that software binaries were properly instantiated on a trusted platform. Remote relying parties can then gain confidence that only such intended software is running on trusted hardware. Azure Attestation is a unified customer-facing service and framework for attestation.

Azure Attestation enables cutting-edge security paradigms such as [Azure Confidential computing](/azure/confidential-computing/overview) and Intelligent Edge protection. The service receives evidence from compute entities, turns them into a set of claims, validates them against configurable policies, and produces cryptographic proofs for claims-based applications (for example, relying parties and auditing authorities).

Azure Attestation supports both platform- and guest-attestation of AMD SEV-SNP based Confidential VMs (CVMs). Azure Attestation-based platform attestation happens automatically during critical boot path of CVMs, with no customer action needed. For more information on guest attestation, see [Announcing general availability of guest attestation for confidential VMs](https://techcommunity.microsoft.com/t5/azure-confidential-computing/announcing-general-availability-of-guest-attestation-for/ba-p/3648228).

## Use cases

Azure Attestation provides comprehensive attestation services for multiple environments and distinctive use cases.

### AMD SEV-SNP attestation on Confidential VMs

Azure [Confidential VM](/azure/confidential-computing/confidential-vm-overview) (CVM) is based on [AMD processors with SEV-SNP technology](/azure/confidential-computing/virtual-machine-options). CVM offers VM OS disk encryption option with platform-managed keys or customer-managed keys and binds the disk encryption keys to the virtual machine's TPM. When a CVM boots up, SNP report containing the guest VM firmware measurements are sent to Azure Attestation. The service validates the measurements and issues an attestation token that is used to release keys from [Managed-HSM](../key-vault/managed-hsm/overview.md) or [Azure Key Vault](../key-vault/general/basic-concepts.md). These keys are used to decrypt the vTPM state of the guest VM, unlock the OS disk and start the CVM. The attestation and key release process is performed automatically on each CVM boot, and the process ensures the CVM boots up only upon successful attestation of the hardware.

### AMD SEV-SNP attestation on Confidential Containers

Azure [Confidential Containers](/azure/confidential-computing/confidential-containers) is based on [AMD processors with SEV-SNP technology](/azure/confidential-computing/virtual-machine-options). Confidential containers, hosted on [Azure Container Instances](/azure/container-instances/container-instances-confidential-overview) and on [Azure Kubernetes Service (in preview)](/azure/aks/deploy-confidential-containers-default-policy) offer the ability to run groups of containers in an SEV-SNP protected trusted execution environment which isolates that group of containers from the container management control plane and other running containers.  Attestation in confidential containers involves fetching the AMD hardware attestation report directly from the processor.  This can be accomplished with our [SKR sidecar container](https://github.com/microsoft/confidential-sidecar-containers/tree/main/cmd/skr) or compiled directly into your application logic.  The hardware report can then be exchanged with Azure Attestation and [managed-HSM](../key-vault/managed-hsm/overview.md) or Premium [Azure Key Vault (AKV)](../key-vault/general/basic-concepts.md) to retrieve secrets.  You can also provide the hardware report to your own key vault system as desired.

### Trusted Launch attestation

Azure customers can [prevent bootkit and rootkit infections](https://www.youtube.com/watch?v=CQqu_rTSi0Q) by enabling [trusted launch](/azure/virtual-machines/trusted-launch) for their virtual machines (VMs). When the VM is Secure Boot and vTPM enabled with guest attestation extension installed, vTPM measurements get submitted to Azure Attestation periodically for monitoring boot integrity. An attestation failure indicates potential malware, which is surfaced to customers via Microsoft Defender for Cloud, through Alerts and Recommendations.

### TPM attestation

[Trusted Platform Modules (TPM)](/windows/security/information-protection/tpm/trusted-platform-module-overview) based attestation is critical to provide proof of a platform's state. A TPM acts as the root of trust and the security coprocessor to provide cryptographic validity to the measurements (evidence). Devices with a TPM can rely on attestation to prove that boot integrity isn't compromised and use the claims to detect feature state enablement during boot. 

Client applications can be designed to take advantage of TPM attestation by delegating security-sensitive tasks to only take place after a platform has been validated to be secure. Such applications can then make use of Azure Attestation to routinely establish trust in the platform and its ability to access sensitive data.

### SGX enclave attestation

[Intel® Software Guard Extensions](https://www.intel.com/content/www/us/en/architecture-and-technology/software-guard-extensions.html) (SGX) refers to hardware-grade isolation, which is supported on certain Intel CPU models. SGX enables code to run in sanitized compartments known as SGX enclaves. Access and memory permissions are then managed by hardware to ensure a minimal attack surface with proper isolation.

Client applications can be designed to take advantage of SGX enclaves by delegating security-sensitive tasks to take place inside those enclaves. Such applications can then make use of Azure Attestation to routinely establish trust in the enclave and its ability to access sensitive data.

Intel® Xeon® Scalable processors only support [ECDSA-based attestation solutions](https://software.intel.com/content/www/us/en/develop/topics/software-guard-extensions/attestation-services.html#Elliptic%20Curve%20Digital%20Signature%20Algorithm%20(ECDSA)%20Attestation) for remotely attesting SGX enclaves. Utilizing ECDSA based attestation model, Azure Attestation supports validation of Intel® Xeon® E3 processors and Intel® Xeon® Scalable processor-based server platforms.

> [!NOTE]
> To perform attestation of Intel® Xeon® Scalable processor-based server platforms using Azure Attestation, users are expected to install [Azure DCAP version 1.10.0](https://github.com/microsoft/Azure-DCAP-Client) or higher.

### Open Enclave attestation
[Open Enclave](https://openenclave.io/sdk/) (OE) is a collection of libraries targeted at creating a single unified enclaving abstraction for developers to build TEE-based applications. It offers a universal secure app model that minimizes platform specificities. Microsoft views it as an essential stepping-stone toward democratizing hardware-based enclave technologies such as SGX and increasing their uptake on Azure.

OE standardizes specific requirements for verification of an enclave evidence. This qualifies OE as a highly fitting attestation consumer of Azure Attestation.

## Azure Attestation runs in a TEE

Azure Attestation is critical to Confidential Computing scenarios, as it performs the following actions:

- Verifies if the enclave evidence is valid.
- Evaluates the enclave evidence against a customer-defined policy.
- Manages and stores tenant-specific policies.
- Generates and signs a token that is used by relying parties to interact with the enclave.

To keep Microsoft operationally out of trusted computing base (TCB), critical operations of Azure Attestation like quote validation, token generation, policy evaluation and token signing are moved into an SGX enclave.

## Why use Azure Attestation

Azure Attestation is the preferred choice for attesting TEEs as it offers the following benefits:

- Unified framework for attesting multiple environments such as TPMs, SGX enclaves and VBS enclaves.
- Allows creation of custom attestation providers and configuration of policies to restrict token generation.
- Protects its data while-in use with implementation in an SGX enclave or Confidential Virtual Machine based on AMD SEV-SNP.
- Highly available service 

## How to establish trust with Azure Attestation

1. **Verify if attestation token is generated by Azure Attestation** - An attestation token generated by Azure Attestation is signed using a self-signed certificate. The signing certificates URL is exposed via an [OpenID metadata endpoint](/rest/api/attestation/metadata-configuration/get?tabs=HTTP#get-openid-metadata). A relying party can retrieve the signing certificate and perform signature verification of the attestation token. See [code samples](https://github.com/Azure-Samples/microsoft-azure-attestation/blob/master/maa.jwt.verifier.dotnet/Program.cs#L94) for more information.
1. **Verify if Azure Attestation is running inside an SGX enclave or SEV-SNP container** - The token signing certificates include information about the TEE inside which Azure Attestation runs. This TEE collateral might be an SGX quote or an SEV-SNP report. The relying party can check if Azure Attestation is running inside a valid TEE by locally validating the quote or the report. See code samples for more information – [SGX](https://github.com/Azure-Samples/microsoft-azure-attestation/blob/master/sgx.attest.sample.oe.sdk/validatequotes.net/MaaQuoteValidator.cs#L62-L65), [SEV-SNP](https://github.com/Azure-Samples/microsoft-azure-attestation/blob/master/maa.jwt.verifier.dotnet/Program.cs#L63) 
1. **Validate binding of Azure Attestation TEE report with the key that signed the attestation token** – The relying party can verify whether the hash of the public key that signed the attestation token matches the report data field of the Azure Attestation TEE report. Refer [here](https://github.com/Azure-Samples/microsoft-azure-attestation/blob/master/maa.jwt.verifier.dotnet/Program.cs#L426) for more details. 
1. **Validate if Azure Attestation code measurements match the Azure published values** - The TEE collateral embedded in the attestation token signing certificates includes TEE code measurements of Azure Attestation. A relying party can validate that the quote or report belongs to Azure Attestation by comparing specific values retrieved from the TEE collateral in the attestation token signing certificate with values provided by the Azure Attestation team. For SGX, MRSIGNER should be validated. For SEV-SNP, HOST_DATA should be validated. Refer [here](https://github.com/Azure-Samples/microsoft-azure-attestation/blob/master/maa.jwt.verifier.dotnet/Program.cs#L395) for more details. If you are interested in performing this validation, submit a request on the [Azure support page](https://azure.microsoft.com/support/options/). The Azure Attestation team reaches out to you when these specific values are scheduled for rotation. 

The values identifying valid Azure Attestation instances are expected to change when code-signing certificates are rotated or when security updates require new policy versions. The Azure Attestation team follows the rollout schedule below for every planned rotation: 

i.	The Azure Attestation team notifies consumers of the new values with a two-month grace period to implement relevant code changes

ii.	After the two-month grace period, Azure Attestation starts using the new values.

iii. Three months after the notification date, Azure Attestation stops using the old values. 

For unplanned rotations, including those required by security updates, the Azure Attestation team communicates new values with a one-month grace period. 

## Business Continuity and Disaster Recovery (BCDR) support

[Business Continuity and Disaster Recovery](/azure/reliability/cross-region-replication-azure) (BCDR) for Azure Attestation enables you to mitigate service disruptions resulting from significant availability issues or disaster events in a region.

Clusters deployed in two regions operate independently under normal circumstances. In the case of a fault or outage of one region, the following takes place:

- Azure Attestation BCDR provides seamless failover in which customers don't need to take any extra step to recover.
- The [Azure Traffic Manager](/azure/traffic-manager/) for the region will detect that the health probe is degraded and switches the endpoint to paired region.
- Existing connections won't work and will receive internal server error or timeout issues.
- All control plane operations will be blocked. Customers won't be able to create attestation providers in the primary region.
- All data plane operations, including attest calls and policy configuration, will be served by secondary region. Customers can continue to work on data plane operations with the original URI corresponding to primary region.

## Next steps
- Learn about [Azure Attestation basic concepts](basic-concepts.md)
- [How to author and sign an attestation policy](author-sign-policy.md)
- [Set up Azure Attestation using PowerShell](quickstart-powershell.md)
