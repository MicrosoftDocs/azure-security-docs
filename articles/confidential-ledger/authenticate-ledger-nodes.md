---
title: Authenticating Azure confidential ledger nodes
description: Learn how to authenticate Azure confidential ledger nodes to establish trust before exchanging data, including TLS certificate verification and Intel SGX attestation.
author: msmbaldwin
ms.author: mbaldwin
ms.date: 04/01/2026
ms.service: azure-confidential-ledger
ms.topic: how-to
ai-usage: ai-assisted
---

# Authenticating Azure confidential ledger nodes

Azure confidential ledger runs on hardware-backed secure enclaves that provide strong isolation and protection for your data. Before exchanging sensitive data with the ledger, you should verify the authenticity of the ledger nodes to ensure you're communicating with legitimate Azure confidential ledger instances and not a malicious actor.

This article explains the trust model behind Azure confidential ledger node authentication, walks through the service identity certificate retrieval process, and shows how to verify node authenticity using SDK code and Intel SGX attestation.

## Why authenticate ledger nodes?

Azure confidential ledger uses the TLS 1.3 protocol for client connections, with connections terminating inside hardware-backed security enclaves (Intel SGX enclaves). Because the TLS connection terminates inside the enclave, neither Azure administrators nor anyone else can access the data in transit — the hardware enforces this guarantee.

However, TLS alone doesn't prove that the remote endpoint is a genuine Azure confidential ledger node running trusted code. Authenticating ledger nodes provides additional assurance that:

- **The nodes are running within genuine Intel SGX enclaves.** The hardware attestation proves the execution environment is legitimate.
- **The nodes are running trusted, unmodified code.** The MRENCLAVE measurement confirms the exact code running inside the enclave.
- **Your data flows only to legitimate Azure confidential ledger instances.** Certificate verification prevents impersonation.
- **Man-in-the-middle attacks are prevented.** Combining TLS with node authentication closes the gap between transport encryption and endpoint trust.

## Understand the trust model

Azure confidential ledger is built on the [Confidential Consortium Framework (CCF)](https://www.microsoft.com/research/project/confidential-consortium-framework), which establishes trust through a chain of cryptographic evidence:

1. **Network identity certificate**: Each Azure confidential ledger instance has a unique network identity certificate (also called the service identity certificate). This certificate serves as the root of trust for TLS connections to any node in the ledger network.

1. **Node identity**: Each ledger node has its own identity key pair. The node's public key is endorsed by the network identity certificate, creating a chain of trust from individual nodes to the overall ledger instance.

1. **Intel SGX attestation**: Each node generates an Intel SGX quote — a hardware-signed attestation that includes the cryptographic hash of the node's identity public key and the MRENCLAVE value (a measurement of the enclave code). This quote proves the node is running within a genuine SGX enclave with trusted code.

1. **Consensus-based trust**: Azure confidential ledger runs across three or more identical instances, each in a dedicated hardware-backed enclave. The integrity of the ledger is maintained through consensus-based blockchain, so a single compromised node can't alter the ledger state.

## Retrieve the service identity certificate

The service identity certificate is a network identity public key certificate that serves as the root of trust (Certificate Authority) for TLS connections to the ledger. You must retrieve this certificate from the Azure confidential ledger identity service, which runs at a well-known endpoint separate from the ledger itself.

### Azure CLI

```azurecli
curl https://identity.confidential-ledger.core.azure.com/ledgerIdentity/<ledger-name> \
  --silent | jq '.ledgerTlsCertificate' | xargs echo -e > service_cert.pem
```

Replace `<ledger-name>` with the name of your confidential ledger instance.

### Python

```python
from azure.confidentialledger.certificate import ConfidentialLedgerCertificateClient

identity_url = "https://identity.confidential-ledger.core.azure.com"
ledger_name = "<ledger-name>"

identity_client = ConfidentialLedgerCertificateClient(identity_url)
network_identity = identity_client.get_ledger_identity(ledger_id=ledger_name)

ledger_tls_cert_file_name = "service_cert.pem"
with open(ledger_tls_cert_file_name, "w") as cert_file:
    cert_file.write(network_identity["ledgerTlsCertificate"])
```

### .NET

```csharp
using Azure.Security.ConfidentialLedger;

var identityClient = new ConfidentialLedgerCertificateClient(
    new Uri("https://identity.confidential-ledger.core.azure.com"));

string ledgerName = "<ledger-name>";
Azure.Response response = identityClient.GetLedgerIdentity(ledgerName);

var certificateResponse = System.Text.Json.JsonDocument.Parse(response.Content.ToString());
string serviceCert = certificateResponse.RootElement
    .GetProperty("ledgerTlsCertificate").GetString();

File.WriteAllText("service_cert.pem", serviceCert);
```

### Java

```java
import com.azure.security.confidentialledger.certificate.ConfidentialLedgerCertificateClient;
import com.azure.security.confidentialledger.certificate.ConfidentialLedgerCertificateClientBuilder;
import com.azure.core.http.HttpClient;
import com.azure.core.http.rest.Response;
import com.azure.core.util.BinaryData;
import com.azure.identity.DefaultAzureCredentialBuilder;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;

ConfidentialLedgerCertificateClient identityClient =
    new ConfidentialLedgerCertificateClientBuilder()
        .certificateEndpoint("https://identity.confidential-ledger.core.azure.com")
        .credential(new DefaultAzureCredentialBuilder().build())
        .httpClient(HttpClient.createDefault())
        .buildClient();

String ledgerName = "<ledger-name>";
Response<BinaryData> certResponse = identityClient
    .getLedgerIdentityWithResponse(ledgerName, null);

ObjectMapper mapper = new ObjectMapper();
JsonNode jsonNode = mapper.readTree(certResponse.getValue().toBytes());
String serviceCert = jsonNode.get("ledgerTlsCertificate").asText();
```

## Connect using the service identity certificate

After you retrieve the service identity certificate, use it as the trusted CA when creating a confidential ledger client. The SDKs use this certificate to verify the TLS connection to the ledger node.

### Python

```python
from azure.identity import DefaultAzureCredential
from azure.confidentialledger import ConfidentialLedgerClient

credential = DefaultAzureCredential()

ledger_client = ConfidentialLedgerClient(
    endpoint="https://<ledger-name>.confidential-ledger.azure.com",
    credential=credential,
    ledger_certificate_path="service_cert.pem"
)
```

### .NET

```csharp
using Azure.Identity;
using Azure.Security.ConfidentialLedger;

var ledgerClient = new ConfidentialLedgerClient(
    new Uri("https://<ledger-name>.confidential-ledger.azure.com"),
    new DefaultAzureCredential());
```

> [!NOTE]
> The .NET SDK handles certificate verification internally. You don't need to pass the service certificate file explicitly.

### Java

```java
import io.netty.handler.ssl.SslContext;
import io.netty.handler.ssl.SslContextBuilder;
import java.io.ByteArrayInputStream;
import java.nio.charset.StandardCharsets;

SslContext sslContext = SslContextBuilder.forClient()
    .trustManager(new ByteArrayInputStream(
        serviceCert.getBytes(StandardCharsets.UTF_8)))
    .build();

reactor.netty.http.client.HttpClient reactorClient =
    reactor.netty.http.client.HttpClient.create()
        .secure(sslContextSpec -> sslContextSpec.sslContext(sslContext));

HttpClient httpClient = new NettyAsyncHttpClientBuilder(reactorClient)
    .wiretap(true).build();

ConfidentialLedgerClient ledgerClient = new ConfidentialLedgerClientBuilder()
    .credential(new DefaultAzureCredentialBuilder().build())
    .httpClient(httpClient)
    .ledgerEndpoint("https://<ledger-name>.confidential-ledger.azure.com")
    .buildClient();
```

### TypeScript

```typescript
import ConfidentialLedger, { getLedgerIdentity }
    from "@azure-rest/confidential-ledger";
import { DefaultAzureCredential } from "@azure/identity";

const ledgerIdentity = await getLedgerIdentity("<ledger-name>");

const ledgerClient = ConfidentialLedger(
    "https://<ledger-name>.confidential-ledger.azure.com",
    ledgerIdentity.ledgerIdentityCertificate,
    new DefaultAzureCredential()
);
```

## Verify Intel SGX enclave attestation

TLS certificate verification confirms you're connecting to a legitimate ledger instance, but it doesn't prove that the node is running inside a genuine Intel SGX enclave. For the highest level of assurance, you can verify the Intel SGX quote presented by ledger nodes.

The SGX quote is a hardware-signed attestation report that contains:

- **The MRENCLAVE value**: A measurement of the code running inside the enclave. This value uniquely identifies the exact binary loaded into the enclave.
- **The node identity public key hash**: The cryptographic hash (DER-encoded) of the node's identity public key, embedded in the SGX report data field.
- **Platform attestation**: Intel's attestation infrastructure signs the quote, proving the enclave is running on genuine Intel SGX hardware.

When a node joins the Azure confidential ledger network, it must present a valid quote. The network accepts the node only if the quote is valid and the MRENCLAVE value matches one of the allowed values in the auditable governance configuration.

### Verify a node quote

You can verify node quotes using the `verify_quote.sh` script that ships with the [CCF Python package](https://pypi.org/project/ccf/) or the [CCF installation](https://microsoft.github.io/CCF/main/build_apps/install_bin.html). You can also use the `oeverify` tool from the [Open Enclave SDK](https://github.com/openenclave/openenclave/blob/master/tools/oeverify/README.md).

**Prerequisites:**

- Ubuntu 20.04-LTS 64-bit
- [CCF](https://microsoft.github.io/CCF/main/build_apps/install_bin.html) or the [CCF Python package](https://pypi.org/project/ccf/)
- [Open Enclave Host-verify SDK](https://github.com/openenclave/openenclave/blob/master/docs/GettingStartedDocs/install_host_verify_Ubuntu_20.04.md)
- [jq](https://jqlang.github.io/jq/download/)

**Steps:**

1. Download the service identity certificate if you haven't already:

    ```bash
    curl https://identity.confidential-ledger.core.azure.com/ledgerIdentity/<ledger-name> \
      --silent | jq '.ledgerTlsCertificate' | xargs echo -e > service_cert.pem
    ```

1. Run the verification script:

    ```bash
    /opt/ccf_virtual/bin/verify_quote.sh \
      https://<ledger-name>.confidential-ledger.azure.com:443 \
      --cacert service_cert.pem
    ```

The script performs two checks:

- It verifies that the cryptographic hash of the node's identity public key (DER-encoded) matches the SGX report data.
- It verifies that the MRENCLAVE value present in the quote is trusted.

### Verify against a specific MRENCLAVE value

You can download the list of trusted MRENCLAVE values from the ledger's governance endpoint:

```bash
curl https://<ledger-name>.confidential-ledger.azure.com/node/code \
  --cacert service_cert.pem --silent | jq
```

To verify that a node is running a specific version of the trusted code, supply the `mrenclave` parameter:

```bash
/opt/ccf_virtual/bin/verify_quote.sh \
  https://<ledger-name>.confidential-ledger.azure.com:443 \
  --cacert service_cert.pem \
  --mrenclave <expected-mrenclave-value>
```

If supplied, the MRENCLAVE value in the quote must match exactly.

For step-by-step details on the quote verification process, see [Establish trust on Azure confidential ledger](verify-node-quotes.md).

## Best practices

- **Always retrieve the service identity certificate from the identity service.** Don't hardcode or cache the certificate indefinitely. Retrieve it from `https://identity.confidential-ledger.core.azure.com` and refresh periodically, because service certificates can be rotated during disaster recovery or maintenance events.

- **Use SDK-provided verification.** The Azure confidential ledger SDKs handle TLS certificate verification automatically. Use the SDKs whenever possible to avoid implementing certificate pinning logic manually.

- **Implement SGX quote verification for high-security scenarios.** For workloads with the strictest security requirements, implement SGX quote verification to confirm that nodes are running trusted code in genuine enclaves.

- **Verify the service endorsement chain.** When verifying [transaction receipts](write-transaction-receipts.md), check the `serviceEndorsements` field. If the service identity was renewed (for example, after a disaster recovery), this field contains the chain of previous service certificates needed to trace trust back to the current certificate.

- **Understand certificate rotation implications.** Service identity certificates can change during disaster recovery or service updates. Your application should handle certificate rotation gracefully by re-fetching the service identity from the identity service endpoint when TLS connection failures occur.

## Next steps

- [Establish trust with SGX quote verification](verify-node-quotes.md)
- [Azure confidential ledger authentication with Microsoft Entra ID](authentication-azure-ad.md)
- [Azure confidential ledger architecture](architecture.md)
- [Azure Confidential Ledger write transaction receipts](write-transaction-receipts.md)
- [Verify write transaction receipts](verify-write-transaction-receipts.md)
