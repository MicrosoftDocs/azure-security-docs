---
title: EKM Proxy API reference for Azure Key Vault Managed HSM external key management
description: Reference documentation for the EKM Proxy API used between Managed HSM and a customer-operated EKM Proxy in external key management
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: reference
ms.date: 07/02/2026
ms.author: mbaldwin
ai-usage: ai-assisted
---

# EKM Proxy API reference for Managed HSM external key management

This article documents the EKM Proxy API — the interface between Managed HSM and a customer-operated EKM Proxy in an [external key management](external-key-management-overview.md) configuration. The API version covered is `0.1-preview`.

## Overview

Managed HSM external key management lets customers store Key Encryption Keys (KEKs) in a customer-owned, customer-operated HSM outside Microsoft infrastructure. When Azure services need to wrap or unwrap a Data Encryption Key, Managed HSM delegates the cryptographic operation to a customer-run EKM Proxy using this API.

The Proxy API does not provide functionality for creating, deleting, or modifying keys within the external key management system. Keys must exist in the external HSM before you associate them with a Managed HSM key reference. The following operations are supported:

| Operation | Description |
|---|---|
| [Get proxy information](#get-proxy-information) | Returns proxy details and the highest supported API version. Also used to verify and monitor proxy reachability. |
| [Get key metadata](#get-key-metadata) | Returns key type, size, and supported operations for an external key. |
| [Wrap key](#wrap-key) | Wraps a cryptographic key using the specified algorithm. |
| [Unwrap key](#unwrap-key) | Unwraps a previously wrapped key. |

## Transport and content type

All requests and responses must use JSON over HTTPS with `Content-Type: application/json`. The proxy must support HTTP 1.1 and TLS 1.3 with the following cipher suites:

- `TLS_AES_256_GCM_SHA384`
- `TLS_AES_128_GCM_SHA256`

## Authentication

Mutual TLS (mTLS) authenticates both parties:

- **Managed HSM → Proxy:** Managed HSM presents an X.509 client certificate. The proxy must enforce mTLS by rejecting requests without a valid client certificate. Customers must be able to configure both the subject common name (CN) and the root CA used to authenticate Managed HSM requests.
- **Proxy → Managed HSM:** The proxy's server certificate must be issued by the CA configured in Managed HSM during EKM setup. The server certificate CN must match either the FQDN of the proxy endpoint or a wildcard for that domain. For example, if the endpoint is `server.ekmproxy.example.com`, the CN must be `server.ekmproxy.example.com` or `*.ekmproxy.example.com`.

## URI format

All API calls use the following URI pattern:

```
https://{server}/[path-prefix]/{api-specific-paths}?api-version={client-api-version}
```

The `path-prefix` is optional and set in Managed HSM during EKM connection configuration. It enables multi-customer use or isolation of multiple Managed HSM pools sharing the same proxy.

**Constraints:**
- Path prefix: maximum 64 characters; letters (a–z, A–Z), numbers (0–9), slashes (`/`), and hyphens (`-`) only.
- External key identifier: maximum 64 characters; letters (a–z, A–Z), numbers (0–9), and hyphens (`-`) only.

## Proxy sharing

A single EKM Proxy can serve more than one Managed HSM pool. Use the path prefix to route requests from different pools to isolated namespaces on the same proxy.

## Response time

The proxy must respond to all API calls within 250 milliseconds. Managed HSM times out requests that exceed this threshold.

## Request context

Every request body includes a `request_context` object that correlates Managed HSM logs with EKM Proxy logs. Record this information in proxy logs on every request.

| Name | Type | Description |
|---|---|---|
| `request_id` | string | Request ID assigned by the originating Managed HSM client. Optional. |
| `correlation_id` | string | Correlation ID assigned by Managed HSM, used to link logs and audit records across systems. |
| `pool_name` | string | Managed HSM pool name for log correlation. |

**Example:**

```json
{
    "request_id": "6476b291-7c15-4d0b-aef9-03688390cb8e",
    "correlation_id": "aa9a3cb4-82b4-11f0-afc6-5ffd3bdade85",
    "pool_name": "mhsm-pool-name"
}
```

## Error response

On failure, the proxy must return a standard HTTP error status code and the following JSON body.

### ProxyError

| Name | Type | Description |
|---|---|---|
| `code` | string | Error code or name. |
| `message` | string | Error message. |

**HTTP status codes:**

| Status code | Description |
|---|---|
| 400 | Invalid request parameter (for example, invalid ciphertext or unsupported algorithm). |
| 401 | Authentication failed (for example, missing client certificate or subject CN mismatch). |
| 403 | Access denied (for example, key is disabled, expired, or the operation isn't supported). |
| 404 | Key not found. |
| 429 | Too many requests. The proxy can't process the current request volume or the usage quota is exceeded. |
| 5xx | Server error. |

**Example:**

```json
{
    "code": "KeyDisabled",
    "message": "Operation WrapKey is not allowed on a disabled key"
}
```

## Key encoding rules

All integers and key material in this API use the following encoding:

1. Represent the value as an **unsigned big-endian binary byte sequence**.
1. **Base64url-encode** (RFC 7515) that byte sequence.

Textual forms must **not** be base64url-encoded. This includes decimal strings, hex strings, PEM, and JWK fields (`k`, `n`, `e`, and so on). Only raw bytes are valid input to the base64url encoder.

**Examples:**

- RSA exponent 65537 → bytes `01 00 01` → base64url `AQAB`
- A 256-bit AES key (raw 32 bytes) → the corresponding base64url string

## API reference

The following sections define each endpoint that a conformant EKM Proxy must implement.

### Get proxy information

Returns proxy details and the highest API version the proxy supports. Managed HSM calls this endpoint during EKM connection setup to verify reachability and authentication. Managed HSM also uses this endpoint as a health monitor at approximately 3 calls per minute.

```
POST https://{server}/[path-prefix]/info?api-version=0.1-preview
```

#### URI parameters

| Name | In | Required | Description |
|---|---|---|---|
| `path-prefix` | path | No | Optional path prefix configured during EKM connection setup. |
| `api-version` | query string | Yes | Client API version. |

#### Request body

| Name | Required | Type | Description |
|---|---|---|---|
| `request_context` | Yes | RequestContext | See [Request context](#request-context). |

#### Responses

| Status code | Type | Description |
|---|---|---|
| 200 OK | ProxyInfo | Proxy details. |
| Other | ProxyError | Error response. |

#### ProxyInfo

| Name | Type | Description |
|---|---|---|
| `api_version` | string | Highest API version supported by the proxy. |
| `proxy_vendor` | string | EKM Proxy vendor name. |
| `proxy_name` | string | EKM Proxy product name and version. |
| `ekm_vendor` | string | External key management system vendor name. |
| `ekm_product` | string | External key management system product name and version. |

#### Example

Request:

```
POST https://ekmproxy.example.com/path-prefix/info?api-version=0.1-preview
```

```json
{
    "request_context": {
        "request_id": "6476b291-7c15-4d0b-aef9-03688390cb8e",
        "correlation_id": "aa9a3cb4-82b4-11f0-afc6-5ffd3bdade85",
        "pool_name": "mhsm-pool-name"
    }
}
```

Response:

```json
{
    "api_version": "1.0",
    "proxy_vendor": "EKM Proxy Vendor",
    "proxy_name": "EKM Proxy Service v1.0",
    "ekm_vendor": "SoftHSM",
    "ekm_product": "SoftHSM v2.5.0"
}
```

---

### Get key metadata

Returns the key type, size, and supported operations for an external key.

```
POST https://{server}/[path-prefix]/{key-name}/metadata?api-version=0.1-preview
```

#### URI parameters

| Name | In | Required | Description |
|---|---|---|---|
| `path-prefix` | path | No | Optional path prefix configured during EKM connection setup. |
| `key-name` | path | Yes | External key identifier. |
| `api-version` | query string | Yes | Client API version. |

#### Request body

| Name | Required | Type | Description |
|---|---|---|---|
| `request_context` | Yes | RequestContext | See [Request context](#request-context). |

#### Responses

| Status code | Type | Description |
|---|---|---|
| 200 OK | KeyMetadata | External key metadata. |
| Other | ProxyError | Error response. |

#### KeyMetadata

| Name | Type | Description |
|---|---|---|
| `key_type` | string | Key type. See [Key types and sizes](#key-types-and-sizes). |
| `key_size` | integer (int32) | Key size in bits. See [Key types and sizes](#key-types-and-sizes). |
| `key_ops` | string[] | Must be `["wrapKey", "unwrapKey"]`. |
| `n` | string (base64url) | RSA modulus. Convert the integer to its unsigned big-endian binary representation, then base64url-encode. Textual forms (decimal, hex) must not be encoded. Required for RSA keys. |
| `e` | string (base64url) | RSA public exponent. Apply the same encoding as `n`. For exponent 65537: bytes `01 00 01` → `AQAB`. Required for RSA keys. |

#### Key types and sizes

| Key type | Description | Supported key sizes (bits) |
|---|---|---|
| `oct` | AES key | 256 |
| `RSA` | RSA key | 2048, 3072, 4096 |

#### Example

Request:

```
POST https://ekmproxy.example.com/path-prefix/aes-key-1/metadata?api-version=0.1-preview
```

```json
{
    "request_context": {
        "request_id": "6476b291-7c15-4d0b-aef9-03688390cb8e",
        "correlation_id": "aa9a3cb4-82b4-11f0-afc6-5ffd3bdade85",
        "pool_name": "mhsm-pool-name"
    }
}
```

Response:

```json
{
    "key_type": "oct",
    "key_size": 256,
    "key_ops": ["wrapKey", "unwrapKey"]
}
```

---

### Wrap key

Wraps a cryptographic key.

```
POST https://{server}/[path-prefix]/{key-name}/wrapkey?api-version=0.1-preview
```

#### URI parameters

| Name | In | Required | Description |
|---|---|---|---|
| `path-prefix` | path | No | Optional path prefix configured during EKM connection setup. |
| `key-name` | path | Yes | External key identifier. |
| `api-version` | query string | Yes | Client API version. |

#### Request body

| Name | Required | Type | Description |
|---|---|---|---|
| `request_context` | Yes | RequestContext | See [Request context](#request-context). |
| `alg` | Yes | string | Wrapping algorithm. See [Wrapping algorithms](#wrapping-algorithms). |
| `value` | Yes | string (base64url) | The key to wrap. Convert the key's raw byte sequence to its unsigned big-endian binary representation, then base64url-encode. Textual forms (hex, decimal, PEM, JWK) must not be encoded. |

#### Wrapping algorithms

| Key type | Supported algorithms |
|---|---|
| `oct` | `A256KW` — AES Key Wrap using a 256-bit key<br>`A256KWP` — AES Key Wrap with Padding using a 256-bit key |
| `RSA` | `RSA-OAEP-256` — RSAES OAEP using SHA-256 and MGF1 with SHA-256<br>`RSA-OAEP` — RSAES OAEP using SHA-1 and MGF1 with SHA-1 |

#### Responses

| Status code | Type | Description |
|---|---|---|
| 200 OK | WrapKeyOperationResult | The wrapped key. |
| Other | ProxyError | Error response. |

#### WrapKeyOperationResult

| Name | Type | Description |
|---|---|---|
| `value` | string (base64url) | The wrapped key. The raw byte sequence produced by the wrapping operation must be treated as unsigned big-endian, then base64url-encoded. Textual forms must not be encoded. |

#### Example

Request:

```
POST https://ekmproxy.example.com/path-prefix/aes-key-1/wrapkey?api-version=0.1-preview
```

```json
{
    "request_context": {
        "request_id": "6476b291-7c15-4d0b-aef9-03688390cb8e",
        "correlation_id": "aa9a3cb4-82b4-11f0-afc6-5ffd3bdade85",
        "pool_name": "mhsm-pool-name"
    },
    "alg": "A256KWP",
    "value": "MTIzNDU2NzgxMjM0NTY3ODEyMzQ1Njc4MTIzNDU2Nzg"
}
```

Response:

```json
{
    "value": "MTIzNDU2NzgxMjM0NTY3ODEyMzQ1Njc4MTIzNDU2NzgxMjM0NTY3OA"
}
```

---

### Unwrap key

Unwraps a previously wrapped key.

```
POST https://{server}/[path-prefix]/{key-name}/unwrapkey?api-version=0.1-preview
```

#### URI parameters

| Name | In | Required | Description |
|---|---|---|---|
| `path-prefix` | path | No | Optional path prefix configured during EKM connection setup. |
| `key-name` | path | Yes | External key identifier. |
| `api-version` | query string | Yes | Client API version. |

#### Request body

| Name | Required | Type | Description |
|---|---|---|---|
| `request_context` | Yes | RequestContext | See [Request context](#request-context). |
| `alg` | Yes | string | Wrapping algorithm. See [Wrapping algorithms](#wrapping-algorithms). |
| `value` | Yes | string (base64url) | The wrapped key to unwrap. Must be a base64url-encoded representation of the raw wrapped-key byte sequence, treated as unsigned big-endian. Textual forms must not be encoded. |

#### Responses

| Status code | Type | Description |
|---|---|---|
| 200 OK | UnwrapKeyOperationResult | The unwrapped key. |
| Other | ProxyError | Error response. |

#### UnwrapKeyOperationResult

| Name | Type | Description |
|---|---|---|
| `value` | string (base64url) | The unwrapped key. The raw byte sequence must be represented as unsigned big-endian, then base64url-encoded. Textual forms must not be encoded. |

#### Example

Request:

```
POST https://ekmproxy.example.com/path-prefix/aes-key-1/unwrapkey?api-version=0.1-preview
```

```json
{
    "request_context": {
        "request_id": "6476b291-7c15-4d0b-aef9-03688390cb8e",
        "correlation_id": "aa9a3cb4-82b4-11f0-afc6-5ffd3bdade85",
        "pool_name": "mhsm-pool-name"
    },
    "alg": "A256KWP",
    "value": "MTIzNDU2NzgxMjM0NTY3ODEyMzQ1Njc4MTIzNDU2NzgxMjM0NTY3OA"
}
```

Response:

```json
{
    "value": "MTIzNDU2NzgxMjM0NTY3ODEyMzQ1Njc4MTIzNDU2Nzg"
}
```

## Next steps

- [What is Managed HSM external key management?](external-key-management-overview.md)
- [Managed HSM external key management architecture](external-key-management-architecture.md)
- [Quickstart: Create your first external key using the Azure CLI](external-key-management-quickstart-cli.md)
- [Quickstart: Create your first external key using the Azure portal](external-key-management-quickstart-portal.md)
- [Configure networking and mTLS for Managed HSM external key management](external-key-management-networking.md)
- [Troubleshoot Managed HSM external key management](external-key-management-troubleshooting.md)
