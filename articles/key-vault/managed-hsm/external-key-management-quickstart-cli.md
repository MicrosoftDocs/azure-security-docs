---
title: "Quickstart: Create your first external key in Azure Key Vault Managed HSM using the Azure CLI (preview)"
description: In this quickstart, you create your first external key in Managed HSM external key management by registering an EKM Proxy connection and binding it to a key in your customer-operated HSM
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: quickstart
ms.date: 07/06/2026
ms.author: mbaldwin
ai-usage: ai-assisted
ms.custom: devx-track-azurecli, mode-api
#Customer intent: As a security or platform engineer with an existing customer-operated HSM and an EKM Proxy, I want a step-by-step CLI walkthrough so that I can create my first external key in Managed HSM and verify wrap/unwrap.
---

# Quickstart: Create your first external key using the Azure CLI (preview)

> [!IMPORTANT]
> Managed HSM external key management is in **preview**. Preview features are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature might change before general availability.

In this quickstart, you register an EKM Proxy connection to your Managed HSM, then create a Managed HSM key reference that points at a key in your customer-operated HSM. At the end, you run a wrap/unwrap round-trip to confirm the integration works.

If you prefer to use the Azure portal, see [Quickstart: Create your first external key using the Azure portal](external-key-management-quickstart-portal.md).

## Prerequisites

Before you begin, you need:

- An existing Managed HSM deployed in any Azure public region, with external key management enabled on the subscription by your Microsoft account team. Contact your account team to request enablement. To create a Managed HSM, see [Quickstart: Provision and activate a Managed HSM using Azure CLI](quick-create-cli.md).
- An operational EKM Proxy reachable from Azure. The proxy must implement the EKM Proxy API specification. For supported vendors, see [What is Managed HSM external key management?](external-key-management-overview.md).
- A key created in your external HSM with a known external key identifier. The external key identifier is the identifier your proxy uses to look up the key.
- The root CA certificate (PEM format) that signed your proxy's TLS server certificate.
- Azure CLI version 2.x with the latest `keyvault` extension installed. Run `az extension add --name keyvault` or `az extension update --name keyvault` to get the latest version.
- The `Managed HSM EKM Administrator` role to manage external key management connections, and the `Managed HSM Crypto User` role to create and use keys. For role assignment steps, see [Managed HSM access control](access-control.md).

## Sign in to Azure

```azurecli
az login
```

Set your subscription if you have more than one:

```azurecli
az account set --subscription "<subscription-id>"
```

## Show the Managed HSM client certificate

Managed HSM presents an X.509 client certificate to your EKM Proxy on every inbound mutual TLS (mTLS) connection. Before you create the external key management connection, retrieve the certificate subject common name and the root CA certificate and allow-list it at your proxy. This is how the proxy validates that the connection is coming from your Managed HSM — not an arbitrary caller.

```azurecli
az keyvault ekm-connection certificate show --hsm-name <Managed HSM Name>
```

The output looks similar to this:

```json
{
  "caCertificates": [
    "MIIDj...<truncated>...MrY="
  ],
  "subjectCommonName": "contoso.managedhsmclient.azure.net"
}
```

Configure your proxy to trust connections that present certificate with the `subjectCommonName` and rooted to one of the certificates in `caCertificates` list above. The exact steps depend on your proxy vendor.

> [!NOTE]
> For rotation steps and certificate lifecycle details, see [Configure networking and mTLS for Managed HSM external key management](external-key-management-networking.md).

## Create the external key management connection

The external key management connection binds your Managed HSM to a single EKM Proxy. It contains the proxy address, the server CA trust anchor, and an optional path prefix.

```azurecli
az keyvault ekm-connection create \
    --hsm-name <Managed HSM Name> \
    --host <EKMProxy Host> \
    --server-ca-certificate <Root cert> \
    [--path-prefix <prefix>]
```

Parameter reference:

- `--hsm-name`: The name of your Managed HSM.
- `--host`: The fully qualified hostname of your EKM Proxy. For example, `proxy.contoso.com`. The proxy must listen on TCP port 443; a nonstandard port isn't supported in preview.
- `--server-ca-certificate`: Specifies the path to the root CA certificate file (PEM or DER format) used to verify the proxy's TLS server certificate.
- `--path-prefix` (optional): A URL path prefix if your proxy multiplexes multiple customers or pools by path. For example, `/contoso/prod`.

> [!NOTE]
> Pass the **root CA certificate** to `--server-ca-certificate`, not the proxy's leaf certificate. Managed HSM uses this CA to validate the entire certificate chain presented by your proxy during the mTLS handshake. Passing the leaf certificate instead causes the handshake to fail for any certificate renewal.

## Verify the connection

After creating the connection, verify that Managed HSM can reach your proxy:

```azurecli
az keyvault ekm-connection check --hsm-name <Managed HSM Name>
```

This command calls the proxy's `/info` endpoint through the configured connection. Successful output looks similar to this:

```json
{
  "apiVersion": "1.0",
  "ekmProduct": "Contoso HSM v1.0.0",
  "ekmVendor": "Contoso HSM",
  "proxyName": "Contoso Proxy Service",
  "proxyVendor": "Contoso Proxy"
}
```

Common failure causes include firewall rules blocking the proxy port, an incorrect `--host` value, or the proxy rejecting the Managed HSM client certificate. For remediation steps, see [Troubleshoot Managed HSM external key management](external-key-management-troubleshooting.md).

## Create the external key

Create a Managed HSM key reference that points at the external key in your HSM. No key material is generated inside Managed HSM — this command registers a reference to an existing key identified by its external key identifier.

```azurecli
az keyvault key create \
    --external-key-id <external-key-identifier> \
    --hsm-name <Managed HSM Name> \
    --name <key-ref-name>
```

Parameter reference:

- `--external-key-id`: The external key identifier registered at your proxy. This is the identifier the proxy uses to look up the correct key in your external HSM.
- `--hsm-name`: The name of your Managed HSM.
- `--name`: The name you want to give the Managed HSM key reference. This becomes part of the key URI that Azure services use.

> [!IMPORTANT]
> The external key identifier is **immutable** for the lifetime of a key version. You can't change it after creation. To rotate the key, create a new key version with a new external key identifier using `az keyvault key create` with the same `--name`. The key URI (HSM URI + key name) stays stable across versions; only the version segment changes.

The command returns the key URI in the form `https://<hsm-name>.managedhsm.azure.net/keys/<key-ref-name>/<version>`. Use this URI when configuring customer-managed key (CMK) settings in Azure services.

## Verify wrap/unwrap

Confirm the integration end-to-end with a wrap/unwrap round-trip. The actual wrap and unwrap operations happen at your proxy and external HSM — Managed HSM routes the request and returns the result.

> [!IMPORTANT]
> Don't use `az keyvault key encrypt` to test an external key. That command issues an `encrypt` operation, which external key management doesn't support — external key management serves `wrapKey` and `unwrapKey` only. Call the `wrapkey` and `unwrapkey` data-plane endpoints directly with `az rest` instead.

1. Create a request body for the wrap operation. Set `value` to the base64url-encoded key material you want to wrap.

   ```json
   {
     "alg": "RSA-OAEP-256",
     "value": "<base64url-encoded-plaintext-key>"
   }
   ```

   Save it as `wrapkey.json`. For AES external keys, use `"alg": "A256KW"` instead of `RSA-OAEP-256`.

1. Call the `wrapkey` endpoint on your key reference:

   ```azurecli
   az rest --method POST \
       --uri "https://<Managed HSM Name>.managedhsm.azure.net/keys/<key-ref-name>/wrapkey?api-version=7.5" \
       --resource "https://managedhsm.azure.net" \
       --headers "Content-Type=application/json" \
       --body @wrapkey.json
   ```

   The response returns the key version (`kid`) and a `value` field containing the base64url-encoded wrapped key. Copy the `value` for the next step.

1. Create a request body for the unwrap operation, using the wrapped `value` from the previous step:

   ```json
   {
     "alg": "RSA-OAEP-256",
     "value": "<wrapped-value-from-previous-step>"
   }
   ```

   Save it as `unwrapkey.json`.

1. Call the `unwrapkey` endpoint to confirm round-trip correctness:

   ```azurecli
   az rest --method POST \
       --uri "https://<Managed HSM Name>.managedhsm.azure.net/keys/<key-ref-name>/unwrapkey?api-version=7.5" \
       --resource "https://managedhsm.azure.net" \
       --headers "Content-Type=application/json" \
       --body @unwrapkey.json
   ```

   A successful response returns the original plaintext `value` you wrapped in the first step. If the unwrap fails, see [Troubleshoot Managed HSM external key management](external-key-management-troubleshooting.md) for proxy error codes and remediation steps.

## View audit logs

Every EKM Proxy call produces an `EkmProxyOperation` entry in your Managed HSM diagnostic logs. To query these in Log Analytics:

```kusto
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.KEYVAULT"
| where OperationName contains "Ekm"
| project TimeGenerated, Resource, OperationName, requestUri_s, ResultType, ResultDescription
```

The log includes the operation type (wrap or unwrap), the external key identifier, the HTTP status code returned by the proxy, and the round-trip latency. For the full logging and monitoring guide, including alert configuration and proxy-side log correlation, see [Logging and monitoring for Managed HSM external key management](external-key-management-logging-monitoring.md).

## Clean up resources

To remove the resources created in this quickstart:

1. Delete the Managed HSM key reference:

   ```azurecli
   az keyvault key delete \
       --hsm-name <Managed HSM Name> \
       --name <key-ref-name>
   ```

1. Delete the external key management connection:

   ```azurecli
   az keyvault ekm-connection delete \
       --hsm-name <Managed HSM Name>
   ```

Deleting the Managed HSM key reference doesn't affect the key material in your external HSM. That key remains in your HSM until you remove it there.

## Next steps

- [Quickstart: Create your first external key using the Azure portal](external-key-management-quickstart-portal.md) — complete the same flow in the Azure portal
- [External key lifecycle in Managed HSM external key management](external-key-management-key-lifecycle.md) — key rotation, versioning, and decommissioning external keys
- [Configure networking and mTLS for Managed HSM external key management](external-key-management-networking.md) — public internet connectivity and certificate rotation
- [Logging and monitoring for Managed HSM external key management](external-key-management-logging-monitoring.md) — alert rules, Kusto Query Language (KQL) queries, and proxy log correlation
- [Troubleshoot Managed HSM external key management](external-key-management-troubleshooting.md) — error codes, proxy diagnostics, and remediation steps
