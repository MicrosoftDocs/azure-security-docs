---
title: Configure networking and mTLS for Azure Key Vault Managed HSM external key management (preview)
description: Public-internet connectivity, outbound traffic from Managed HSM, and mTLS certificate lifecycle for Azure Key Vault Managed HSM external key management.
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: how-to
ms.date: 07/06/2026
ms.author: mbaldwin
ai-usage: ai-assisted
#Customer intent: As a network or security engineer deploying Managed HSM external key management, I need to understand connectivity options to my EKM Proxy and how to manage the mTLS certificates that secure the connection.
---

# Configure networking and mTLS for Managed HSM external key management (preview)

> [!IMPORTANT]
> Managed HSM external key management is in **preview**. Preview features are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature might change before general availability.

This article covers the network and transport-layer security configuration required to connect Managed HSM to your EKM Proxy. It's aimed at network and security engineers who are setting up or maintaining an external key management deployment.

You'll learn how to choose a connectivity model, what outbound traffic Managed HSM generates, and how to provision and rotate the mutual TLS (mTLS) certificates that authenticate both sides of the connection.

For the end-to-end architecture and request flow, see [Managed HSM external key management architecture](external-key-management-architecture.md). For command-level setup steps, see [Quickstart: Create your first external key using the Azure CLI](external-key-management-quickstart-cli.md).

## Connectivity model

Managed HSM initiates an outbound mTLS connection to your EKM Proxy. In the initial preview release, the EKM Proxy must be reachable over the **public internet** on a publicly resolvable FQDN. This is the only supported connectivity model.

Security doesn't depend on network-layer restrictions. The connection is protected by mutual TLS (mTLS): both sides present and verify X.509 certificates before any payload is exchanged. Managed HSM source IPs aren't static and can't be allow-listed, so you can't restrict inbound traffic to only Managed HSM. Certificate-based identity is the supported security mechanism.

## Configure the public-internet model

In this model, the EKM Proxy is reachable over the public internet on a publicly resolvable FQDN.

1. Deploy the EKM Proxy on a host with a public IP and a DNS record.
1. Obtain a TLS server certificate from a certificate authority (CA) whose root certificate you'll register on the external key management connection.
1. Configure the proxy to listen on **TCP port 443** with TLS 1.3 enabled. Port 443 is required for preview.
1. Create the external key management connection with `--host` set to the public FQDN and `--server-ca-certificate` set to the root CA in PEM format.

For the full `az keyvault ekm-connection create` command, see [Quickstart: Create your first external key using the Azure CLI](external-key-management-quickstart-cli.md).

> [!NOTE]
> Even though the proxy is publicly reachable, the connection is still mutually authenticated. An attacker who reaches the proxy endpoint without a valid Managed HSM client certificate can't complete the handshake.

## mTLS overview

The connection between Managed HSM and the EKM Proxy uses mutual TLS (mTLS). Both sides present X.509 certificates and both sides verify the peer's certificate before exchanging any payload.

- **Server certificate** — the EKM Proxy presents this certificate to Managed HSM. Managed HSM validates it against the root CA you registered on the external key management connection.
- **Client certificate** — Managed HSM presents this certificate to the EKM Proxy. The proxy must validate it and reject connections from unknown issuers.

This bidirectional authentication is what establishes identity. Managed HSM source IPs aren't stable and aren't customer-pinnable, so IP-based allow-listing is insufficient. Certificate-based identity is the only supported mechanism.

**TLS requirements:**

- Minimum version: TLS 1.3.
- Required cipher suites: `TLS_AES_256_GCM_SHA384`, `TLS_AES_128_GCM_SHA256`.

## Server certificate (proxy → Managed HSM)

The proxy's server certificate is a standard PKIX certificate. You're responsible for provisioning, renewing, and revoking it.

**What Managed HSM validates:**

- The certificate chains to the root CA registered in the external key management connection's `--server-ca-certificate` field.
- The certificate is within its validity window.
- The subject alternative name (SAN) matches the proxy hostname configured on the external key management connection.

**What `--server-ca-certificate` accepts:**

A root CA certificate in PEM or DER format. Pass the file path to the certificate when running `az keyvault ekm-connection create` or `az keyvault ekm-connection update`. In the Azure portal upload flow, use PEM format with a `.cer` extension.

**Rotating the server certificate:**

- If the new certificate chains to the **same root CA** already registered on the external key management connection, deploy the new certificate to the proxy. No change to the external key management connection is required.
- If the new certificate uses a **different root CA**, update the external key management connection with the new CA before you deploy the new certificate:

  ```azurecli
  az keyvault ekm-connection update \
    --hsm-name <Managed HSM Name> \
    --server-ca-certificate oldandnewrootcas.pem
  ```

  Update the connection first to avoid a window where the proxy presents a certificate that Managed HSM can't validate.

## Client certificate (Managed HSM → proxy)

Managed HSM presents a pool-specific X.509 client certificate to the proxy on every connection. Microsoft issues and manages this certificate.

**Retrieve information about the client certificate Managed HSM uses to authenticate to the EKM Proxy:**

```azurecli
az keyvault ekm-connection certificate show --hsm-name <Managed HSM Name>
```

The command returns the client certificate's subject common name and an array of X.509 root certificates, where each certificate is encoded as Base64 DER.

**Configure the proxy to accept the certificate:**

The proxy must allow-list both:

- The client certificate's **subject common name** (`subjectCommonName`).
- The **root certificates** (`caCertificates`) returned by the command.

Because allow-listing is based on the subject common name and root CA rather than a specific certificate fingerprint, the proxy continues to accept connections after Managed HSM rotates the client certificate, as long as the subject common name and root CA are unchanged.

### mTLS client certificate rotation

Microsoft rotates the Managed HSM client certificate on a schedule. The new certificate is issued before the old one expires, and Managed HSM presents the new certificate automatically after rotation.

Because the proxy allow-lists by subject common name and root certificates rather than by a specific certificate, no action is required when the certificate rotates, as long as the subject common name and `caCertificates` are unchanged.

**Monitoring strategy:**

1. Run `az keyvault ekm-connection certificate show --hsm-name <Managed HSM Name>` on a schedule (for example, weekly by using a cron job or Azure Automation runbook).
1. Watch for changes to the returned `caCertificates` array.
1. If the `caCertificates` change, retrieve the new `caCertificates` array and allow-list them on your EKM Proxy side.

## Firewall and DNS requirements

Configure both firewall rules and DNS on the network where your EKM Proxy runs.

### Firewall rules

Configure the firewall on the network segment where the proxy runs to:

- Allow inbound **TCP 443** from the public internet. The preview requires port 443 specifically. Managed HSM source IPs aren't static and can't be allow-listed, so you can't restrict inbound traffic to only Managed HSM.
- Allow inbound TLS 1.3 with the required cipher suites.

> [!NOTE]
> No Azure service tags exist for Managed HSM EKM outbound traffic in preview. Because Managed HSM source IPs are not static, IP-based allow-listing is not supported. The proxy must accept TCP 443 from any source. mTLS client certificate validation is the sole supported access control mechanism.

Don't rely on IP-based allow-listing as a security control. mTLS client certificate validation is the authentication mechanism that protects the proxy.

### DNS requirements

- The proxy FQDN configured on the external key management connection must resolve to a publicly reachable address from Managed HSM's perspective.
- If you change the proxy address, update the external key management connection to reflect the new FQDN or IP. DNS resolution happens at Managed HSM on each connection.

## Troubleshooting common networking failures

For the full troubleshooting guide, see [Troubleshoot Managed HSM external key management](external-key-management-troubleshooting.md). The three most common networking-specific symptoms are:

**Certificate mismatch**

The mTLS handshake fails because the proxy's server certificate doesn't chain to the CA registered on the external key management connection, or the Managed HSM client certificate's subject name or root CA isn't in the proxy's allow-list. Check `az keyvault ekm-connection certificate show` and verify the proxy allow-list is current. Verify the external key management connection's `--server-ca-certificate` matches the root CA that issued the current proxy certificate.

**Name resolution failure**

Managed HSM can't resolve the proxy FQDN. This happens when the public DNS record for the proxy is missing or stale. Confirm that the FQDN in the external key management connection resolves to the correct public IP.

**Connection timeout**

The TCP connection to the proxy times out. A common cause is a firewall rule blocking inbound traffic on the proxy port. Check firewall logs and verify the proxy is listening on the expected port.

## Related content

- [What is Managed HSM external key management?](external-key-management-overview.md)
- [Managed HSM external key management architecture](external-key-management-architecture.md)
- [SLA and shared responsibility for Managed HSM external key management](external-key-management-shared-responsibility.md)
- [Quickstart: Create your first external key using the Azure CLI](external-key-management-quickstart-cli.md)
- [Quickstart: Create your first external key using the Azure portal](external-key-management-quickstart-portal.md)
- [Troubleshoot Managed HSM external key management](external-key-management-troubleshooting.md)
