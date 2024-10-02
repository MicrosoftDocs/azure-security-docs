---
title: Virtualization-based security (VBS) protocol for Azure Attestation
description: VBS attestation protocol 
services: attestation
author: msmbaldwin
ms.service: azure-attestation
ms.topic: reference
ms.date: 01/23/2023
ms.author: mbaldwin

---

# Trusted Platform Module (TPM) and Virtualization based Security(VBS) enclave attestation protocol

Microsoft Azure Attestation guarantees strong security by verifying a chain of trust, maintained from a root of trust (TPM) to the launch of the hypervisor and secure kernel. To do so, Azure Attestation must attest to the boot state of the machine before we can establish trust in the secure enclave. The operating system, hypervisor, and secure kernel binaries must be signed by the correct official Microsoft authorities and configured in a secure way. Once we bind trust between the Trusted Platform Module (TPM) and the health of the hypervisor, we can trust the Virtualization Based Security(VBS) enclave IDKs provided in the Measured Boot Log, which in turn allows us to validate that the enclave generated a key pair and mint an attestation report that binds trust in that key and contains other claims such as the security level and boot attestation properties.

VBS enclaves require a TPM to provide the measurement to validate the security foundation. VBS enclaves are attested by the TPM endpoint with an addition to the request object in the protocol. 

## Protocol messages

The protocol has two messages exchanges: 
* Init Message
* Request Message

### Init message
Message to establish context for the request message.

#### Direction

Client -> Azure Attestation

#### Payload

```
{ 
  "type": "aikcert" 
} 
```

"type" (ASCII string): represents the type of attestation requested. Currently, only “aikcert” is supported.

### Challenge message

#### Direction

Azure Attestation -> Client

#### Payload

```
{ 
  "challenge": "<BASE64URL(CHALLENGE)>", 
  "service_context": "<BASE64URL(SERVICECONTEXT)>" 
} 
```

**challenge** (BASE64URL(OCTETS)): Random value issued by the service.

**service_context** (BASE64URL(OCTETS)): Opaque context created by the service.

## Request Message
Payload containing the data that the attestation service is to attested.

Note: Support for IMA measurement logs and Keys was added to Request message, and can be found in the Request Message V2 section.

## Request message v1

#### Direction

Client -> Azure Attestation 

#### Payload

```
{
  "request": "<JWS>"
}
```

**request** (JWS): Request consists of a JSON Web Signature (JWS) structure. The JWS Protected Header and JWS Payload are shown. As in any JWS structure, the final value consists of:

```json
BASE64URL(UTF8(JWS Protected Header)) || '.' ||

BASE64URL(JWS Payload) || '.' ||

BASE64URL(JWS Signature)
```

##### JWS protected header

```json
{
  "alg": "PS256",
  "typ": "attReq"
  // no "kid" parameter as the key specified by attest_key MUST sign this JWS to prove possession.
}
```

##### JWS payload

JWS payload can be of type basic or VBS. Basic is used when attestation evidence does not include VBS data. 

TPM-only sample:

``` 
{ 
  "att_type": "basic", 
  "att_data": { 
    "rp_id": "<URL>", 
    "rp_data": "<BASE64URL(RPCUSTOMDATA)>", 
    "challenge": "<BASE64URL(CHALLENGE)>", 

    "tpm_att_data": { 
      "srtm_boot_log": "<BASE64URL(SRTMBOOTLOG)>", 
      "srtm_resume_log": "<BASE64URL(SRTMRESUMELOG)>", 
      "drtm_boot_log": "<BASE64URL(DRTMBOOTLOG)>", 
      "drtm_resume_log": "<BASE64URL(DRTMRESUMELOG)>", 
      "aik_cert": "<BASE64URL(AIKCERTIFICATE)>", 

      // aik_pub is represented as a JSON Web Key (JWK) object (RFC 7517). 

      "aik_pub": { 
        "kty": "RSA", 
        "n": "<Base64urlUInt(MODULUS)>", 
        "e": "<Base64urlUInt(EXPONENT)>" 
      }, 
      "current_claim": "<BASE64URL(CURRENTCLAIM)>", 
      "boot_claim": "<BASE64URL(BOOTCLAIM)>" 
    }, 

    // attest_key is represented as a JSON Web Key (JWK) object (RFC 7517). 

    "attest_key": { 
      "kty": "RSA", 
      "n": "<Base64urlUInt(MODULUS)>", 
      "e": "<Base64urlUInt(EXPONENT)>" 
    }, 
    "custom_claims": [ 
      { 
        "name": "<name>", 
        "value": "<value>", 
        "value_type": "<value_type>" 
      }, 
      { 
        "name": "<name>", 
        "value": "<value>", 
        "value_type": "<value_type>" 
      } 
    ], 
    "service_context": "<BASE64URL(SERVICECONTEXT)>" 
  } 
} 
```

TPM + VBS enclave sample: 

``` 
{ 
  "att_type": "vbs", 
  "att_data": { 
    "report_signed": { 
      "rp_id": "<URL>", 
      "rp_data": "<BASE64URL(RPCUSTOMDATA)>", 
      "challenge": "<BASE64URL(CHALLENGE)>", 
      "tpm_att_data": { 
        "srtm_boot_log": "<BASE64URL(SRTMBOOTLOG)>", 
        "srtm_resume_log": "<BASE64URL(SRTMRESUMELOG)>", 
        "drtm_boot_log": "<BASE64URL(DRTMBOOTLOG)>", 
        "drtm_resume_log": "<BASE64URL(DRTMRESUMELOG)>", 
        "aik_cert": "<BASE64URL(AIKCERTIFICATE)>", 

        // aik_pub is represented as a JSON Web Key (JWK) object (RFC 7517). 

        "aik_pub": { 
          "kty": "RSA", 
          "n": "<Base64urlUInt(MODULUS)>", 
          "e": "<Base64urlUInt(EXPONENT)>" 
        }, 
        "current_claim": "<BASE64URL(CURRENTCLAIM)>", 
        "boot_claim": "<BASE64URL(BOOTCLAIM)>" 
      }, 

      // attest_key is represented as a JSON Web Key (JWK) object (RFC 7517). 

      "attest_key": { 
        "kty": "RSA", 
        "n": "<Base64urlUInt(MODULUS)>", 
        "e": "<Base64urlUInt(EXPONENT)>" 
      }, 
      "custom_claims": [ 
        { 
          "name": "<name>", 
          "value": "<value>", 
          "value_type": "<value_type>" 
        }, 
        { 
          "name": "<name>", 
          "value": "<value>", 
          "value_type": "<value_type>" 
        } 
      ], 
      "service_context": "<BASE64URL(SERVICECONTEXT)>" 
    }, 
    "vsm_report": "<BASE64URL(REPORT)>" 
  } 
} 
``` 

**rp_id** (StringOrURI): Relying party identifier. Used by the service in the computation of the machine ID claim

**rp_data** (BASE64URL(OCTETS)): Opaque data passed by the relying party. This is normally used by the relying party as a nonce to guarantee freshness of the report

**challenge** (BASE64URL(OCTETS)): Random value issued by the service

**tpm_att_data**: TPM-related attestation data

- **srtm_boot_log (BASE64URL(OCTETS))**: SRTM boot logs as retrieved by function Tbsi_Get_TCG_Log_Ex with log type = TBS_TCGLOG_SRTM_BOOT

- **srtm_resume_log (BASE64URL(OCTETS))**: SRTM resumes log as retrieved by function Tbsi_Get_TCG_Log_Ex with log type = TBS_TCGLOG_SRTM_RESUME

- **drtm_boot_log (BASE64URL(OCTETS))**: DRTM boot logs as retrieved by function Tbsi_Get_TCG_Log_Ex with log type = TBS_TCGLOG_DRTM_BOOT

- **drtm_resume_log (BASE64URL(OCTETS))**: DRTM resumes log as retrieved by function Tbsi_Get_TCG_Log_Ex with log type = TBS_TCGLOG_DRTM_RESUME

- **aik_cert (BASE64URL(OCTETS))**: The X.509 certificate for the AIK as returned by function NCryptGetProperty with property = NCRYPT_CERTIFICATE_PROPERTY

- **aik_pub**: The public part of the AIK represented as a JSON Web Key (JWK) object (RFC 7517)

- **current_claim (BASE64URL(OCTETS))**: The attestation claim for the current PCR state as returned by function NCryptCreateClaim with dwClaimType = NCRYPT_CLAIM_PLATFORM and parameter NCRYPTBUFFER_TPM_PLATFORM_CLAIM_PCR_MASK set to include all PCRs. The challenge sent by the service should also be used in the computation of this claim

- **boot_claim (BASE64URL(OCTETS))**: The attestation claim for the PCR state at boot as returned by function NCryptCreateClaim with dwClaimType = NCRYPT_CLAIM_PLATFORM and parameter NCRYPTBUFFER_TPM_PLATFORM_CLAIM_PCR_MASK set to include all PCRs

**vsm_report** (BASE64URL(OCTETS)): The VBS enclave attestation report as returned by function EnclaveGetAttestationReport. The EnclaveData parameter must be the SHA-512 hash of the value of report_signed (including the opening and closing braces). The hash function input is UTF8(report_signed)

**attest_key**: The public part of the enclave key represented as a JSON Web Key (JWK) object (RFC 7517)

**custom_claims**: Array of custom enclave claims sent to the service that can be evaluated by the policy. The claim

- **name (String)**: Name of the claim. This name will be appended to a url determined by the Attestation Service (to avoid conflicts) and the concatenated string becomes the type of the claim that can be used in the policy

- **value (String)**: Value of the claim

- **value_type (String)**: Data type of the claim’s value

**service_context** (BASE64URL(OCTETS)): Opaque context created by the service.

### Report message

#### Direction

Azure Attestation -> Client

#### Payload

```
{
  "report": "<JWT>"
}
```

**report** (JWT): The attestation report in JSON Web Token (JWT) format (RFC 7519).

## Request message v2


```
{
  "request": "<JWS>"
}
```

**request** (JWS): Request consists of a JSON Web Signature (JWS) structure. The JWS Protected Header and JWS Payload are shown below. As in any JWS structure, the final value consists of:
BASE64URL(UTF8(JWS Protected Header)) || '.' ||
BASE64URL(JWS Payload) || '.' ||
BASE64URL(JWS Signature)

##### JWS protected header
```
{
  "alg": "PS256",
  "typ": "attReqV2"
  // no "kid" parameter as the key specified by request_key MUST sign this JWS to prove possession.
}
```

#### JWS payload

JWS payload can be of type basic or vsm. Basic is used when attestation evidence does not include VSM data.

Basic example:

```
{
  "att_type": "basic",
  "att_data": {
    "rp_id": "<URL>",
    "rp_data": "<BASE64URL(RPCUSTOMDATA)>",
    "challenge": "<BASE64URL(CHALLENGE)>",
    "tpm_att_data": {
      "current_attestation": {
        "logs": [
          {
            "type": "TCG",
            "log": "<BASE64URL(CURRENT_LOG1)>"
          },
          {
            "type": "TCG",
            "log": "<BASE64URL(CURRENT_LOG2)>"
          },
          {
            "type": "TCG",
            "log": "<BASE64URL(CURRENT_LOG3)>"
          }
        ],
        "aik_cert": "<BASE64URL(AIKCERTIFICATE)>",
        // aik_pub is represented as a JSON Web Key (JWK) object (RFC 7517).
        "aik_pub": {
          "kty": "RSA",
          "n": "<Base64urlUInt(MODULUS)>",
          "e": "<Base64urlUInt(EXPONENT)>"
        },
        "pcrs": [
          {
            "algorithm": 4, // TPM_ALG_SHA1
            "values": [
              {
                "index": 0,
                "digest": "<BASE64URL(DIGEST)>"
              },
              {
                "index": 5,
                "digest": "<BASE64URL(DIGEST)>"
              }
            ]
          },
          {
            "algorithm": 11, // TPM_ALG_SHA256
            "values": [
              {
                "index": 2,
                "digest": "<BASE64URL(DIGEST)>"
              },
              {
                "index": 1,
                "digest": "<BASE64URL(DIGEST)>"
              }
            ]
          }
        ],
        "quote": "<BASE64URL(TPMS_ATTEST)>",
        "signature": "<BASE64URL(TPMT_SIGNATURE)>"
      },
      "boot_attestation": {
        "logs": [
          {
            "type": "TCG",
            "log": "<BASE64URL(BOOT_LOG1)>"
          },
          {
            "type": "TCG",
            "log": "<BASE64URL(BOOT_LOG2)>"
          }
        ],
        "aik_cert": "<BASE64URL(AIKCERTIFICATE)>",
        // aik_pub is represented as a JSON Web Key (JWK) object (RFC 7517).
        "aik_pub": {
          "kty": "RSA",
          "n": "<Base64urlUInt(MODULUS)>",
          "e": "<Base64urlUInt(EXPONENT)>"
        },
        "pcrs": [
          {
            "algorithm": 4, // TPM_ALG_SHA1
            "values": [
              {
                "index": 0,
                "digest": "<BASE64URL(DIGEST)>"
              },
              {
                "index": 5,
                "digest": "<BASE64URL(DIGEST)>"
              }
            ]
          },
          {
            "algorithm": 11, // TPM_ALG_SHA256
            "values": [
              {
                "index": 2,
                "digest": "<BASE64URL(DIGEST)>"
              },
              {
                "index": 1,
                "digest": "<BASE64URL(DIGEST)>"
              }
            ]
          }
        ],
        "quote": "<BASE64URL(TPMS_ATTEST)>",
        "signature": "<BASE64URL(TPMT_SIGNATURE)>"
      }
    },
    "request_key": {
      "jwk": {
        "kty": "RSA",
        "n": "<Base64urlUInt(MODULUS)>",
        "e": "<Base64urlUInt(EXPONENT)>"
      },
      "info": {
        "tpm_quote": {
          "hash_alg": "sha-256"
        }
      }
    },
    "other_keys": [
      {
        "jwk": {
          "kty": "RSA",
          "n": "<Base64urlUInt(MODULUS)>",
          "e": "<Base64urlUInt(EXPONENT)>"
        },
        "info": {
          "tpm_certify": {
            "public": "<BASE64URL(TPMT_PUBLIC)>",
            "certification": "<BASE64URL(TPMS_ATTEST)>",
            "signature": "<BASE64URL(TPMT_SIGNATURE)>"
          }
        }
      },
      {
        "jwk": {
          "kty": "RSA",
          "n": "<Base64urlUInt(MODULUS)>",
          "e": "<Base64urlUInt(EXPONENT)>"
        }
      }
    ],
    "custom_claims": [
      {
        "name": "<name>",
        "value": "<value>",
        "value_type": "<value_type>"
      },
      {
        "name": "<name>",
        "value": "<value>",
        "value_type": "<value_type>"
      }
    ],
    "service_context": "<BASE64URL(SERVICECONTEXT)>"
  }
}
```

TPM + VBS enclave example:
```
{
  "att_type": "vbs",
  "att_data": {
    "report_signed": {
      "rp_id": "<URL>",
      "rp_data": "<BASE64URL(RPCUSTOMDATA)>",
      "challenge": "<BASE64URL(CHALLENGE)>",
      "tpm_att_data": {
        "current_attestation": {
        "logs": [
            {
              "type": "TCG",
              "log": "<BASE64URL(CURRENT_LOG1)>"
            },
            {
              "type": "TCG",
              "log": "<BASE64URL(CURRENT_LOG2)>"
            },
            {
              "type": "TCG",
              "log": "<BASE64URL(CURRENT_LOG3)>"
            }
          ],
          "aik_cert": "<BASE64URL(AIKCERTIFICATE)>",
          // aik_pub is represented as a JSON Web Key (JWK) object (RFC 7517).
          "aik_pub": {
            "kty": "RSA",
            "n": "<Base64urlUInt(MODULUS)>",
            "e": "<Base64urlUInt(EXPONENT)>"
          },
          "pcrs": [
            {
              "algorithm": 4, // TPM_ALG_SHA1
              "values": [
                {
                  "index": 0,
                  "digest": "<BASE64URL(DIGEST)>"
                },
                {
                  "index": 5,
                  "digest": "<BASE64URL(DIGEST)>"
                }
              ]
            },
            {
              "algorithm": 11, // TPM_ALG_SHA256
              "values": [
                {
                  "index": 2,
                  "digest": "<BASE64URL(DIGEST)>"
                },
                {
                  "index": 1,
                  "digest": "<BASE64URL(DIGEST)>"
                }
              ]
            }
          ],
          "quote": "<BASE64URL(TPMS_ATTEST)>",
          "signature": "<BASE64URL(TPMT_SIGNATURE)>"
        },
        "boot_attestation": {
          "logs": [
            {
              "type": "TCG",
              "log": "<BASE64URL(BOOT_LOG1)>"
            },
            {
              "type": "TCG",
              "log": "<BASE64URL(BOOT_LOG2)>"
            }
          ],
          "aik_cert": "<BASE64URL(AIKCERTIFICATE)>",
          // aik_pub is represented as a JSON Web Key (JWK) object (RFC 7517).
          "aik_pub": {
            "kty": "RSA",
            "n": "<Base64urlUInt(MODULUS)>",
            "e": "<Base64urlUInt(EXPONENT)>"
          },
          "pcrs": [
            {
              "algorithm": 4, // TPM_ALG_SHA1
              "values": [
                {
                  "index": 0,
                  "digest": "<BASE64URL(DIGEST)>"
                },
                {
                  "index": 5,
                  "digest": "<BASE64URL(DIGEST)>"
                }
              ]
            },
            {
              "algorithm": 11, // TPM_ALG_SHA256
              "values": [
                {
                  "index": 2,
                  "digest": "<BASE64URL(DIGEST)>"
                },
                {
                  "index": 1,
                  "digest": "<BASE64URL(DIGEST)>"
                }
              ]
            }
          ],
          "quote": "<BASE64URL(TPMS_ATTEST)>",
          "signature": "<BASE64URL(TPMT_SIGNATURE)>"
        }
      },
      "request_key": {
        "jwk": {
          "kty": "RSA",
          "n": "<Base64urlUInt(MODULUS)>",
          "e": "<Base64urlUInt(EXPONENT)>"
        },
        "info": {
          "tpm_quote": {
            "hash_alg": "sha-256"
          }
        }
      },
      "other_keys": [
        {
          "jwk": {
            "kty": "RSA",
            "n": "<Base64urlUInt(MODULUS)>",
            "e": "<Base64urlUInt(EXPONENT)>"
          },
          "info": {
            "tpm_certify": {
              "public": "<BASE64URL(TPMT_PUBLIC)>",
              "certification": "<BASE64URL(TPMS_ATTEST)>",
              "signature": "<BASE64URL(TPMT_SIGNATURE)>"
            }
          }
        },
        {
          "jwk": {
            "kty": "RSA",
            "n": "<Base64urlUInt(MODULUS)>",
            "e": "<Base64urlUInt(EXPONENT)>"
          }
        }
      ],
      "custom_claims": [
        {
          "name": "<name>",
          "value": "<value>",
          "value_type": "<value_type>"
        },
        {
          "name": "<name>",
          "value": "<value>",
          "value_type": "<value_type>"
        }
      ],
      "service_context": "<BASE64URL(SERVICECONTEXT)>"
    },
    "vsm_report": {
      "enclave": {
        "report": "<BASE64URL(REPORT)>"
      }
    }
  }
}
```

**rp_id** (StringOrURI): Relying party identifier. Used by the service in the computation of the machine ID claim.

**rp_data** (BASE64URL(OCTETS)): Opaque data passed by the relying party. This is normally used by the relying party as a nonce to guarantee freshness of the report.

**challenge** (BASE64URL(OCTETS)): Random value issued by the service.

- ***current_attestation*** (Object): Contains logs and TPM quote for the current state of the system (either boot or resume). The nonce received from the service must be passed to the TPM2_Quote command in the 'qualifyingData' parameter.

- ***boot_attestation*** (Object): This is optional and contains logs and the TPM quote saved before the system hibernated and resumed. boot_attestation info must be associated with the same cold boot cycle (that is, the system was only hibernated and resumed between them).

- ***logs*** (Array(Object)): Array of logs. Each element of the array contains a log and the array must be in the order used for measurements.

- - ***type*** (String): Type of the log: “TCG” or “IMA”.

- - ***log*** (BASE64URL(OCTETS)): The log encoded as a BASE64URL string.

- ***aik_cert*** (BASE64URL(OCTETS)): The X.509 certificate representing the AIK.

- ***aik_pub*** (JWK): The public part of the AIK represented as a JSON Web Key (JWK) object (RFC 7517).

- ***pcrs*** (Array(Object)): Contains the set quoted. Each element of the array represents a PCR bank and the array must be in the order used to create the quote. A PCR bank is defined by its algorithm and its values (only the values quoted should be in the list).

- - ***algorithm*** (Integer): UINT16 value representing a hash algorithm defined by the TPM_ALG_ID constants.

- - ***values*** (Array(Object)): Array of digest values quoted. Each element of the array represents a digest value in the PCR bank.

- - - ***index*** (Integer): Index of the PCR digest value.

- - - ***digest*** (BASE64URL(OCTETS)): Digest value.

- - ***quote*** (BASE64URL(OCTETS)): TPMS_ATTEST returned by the TPM2_Quote command. If this field is in the “current_attestation” object, the challenge received from the service must be passed to the TPM2_Quote command in the 'qualifyingData' parameter. The specific ‘qualifyingData’ parameter value depends on the “request_key” binding method and is described in the KEY OBJECT section.

- - ***signature*** (BASE64URL(OCTETS)): TPMT_SIGNATURE returned by the TPM2_Quote command. If this field is in the “current_attestation” object, the challenge received from the service must be passed to the TPM2_Quote command in the 'qualifyingData' parameter. The specific ‘qualifyingData’ parameter value depends on the “request_key” binding method and is described in the KEY OBJECT section.

**vsm_report** (VSM Report Object): The VSM attestation report. See the VSM REPORT OBJECT section.

**request_key** (Key object): Key used to sign the request. If a TPM is present (request contains TPM quote), request_key must either be bound to the TPM via quote or be resident in the TPM (see KEY OBJECT).

**other_keys** (Array(Key object)): Array of keys to be sent to the service. Maximum of two keys.

**custom_claims** (Array(Object)): Array of custom enclave claims sent to the service that can be evaluated by the policy.

- ***name*** (String): Name of the claim. This name will be appended to a url determined by the Attestation Service (to avoid conflicts) and the concatenated string becomes the type of the claim that can be used in the policy.

- ***value*** (String): Value of the claim.

- ***value_type*** (String): Data type of the claim’s value.

**service_context** (BASE64URL(OCTETS)): Opaque, encrypted context created by the service that includes, among others, the challenge and an expiration time for that challenge.

## Key object

**jwk** (Object): The public part of the key represented as a JSON Web Key (JWK) object (RFC 7517).

**info** (Object): Extra information about the key.

No extra information:(Info object can be empty or missing from request)

•	Key bound to the TPM via quote:
- ***tpm_quote*** (Object): Data for the TPM quote binding method.
- ***hash_alg*** (String): The algorithm used to create the hash passed to the TPM2_Quote command in the 'qualifyingData' parameter. The hash is computed by HASH[UTF8(jwk) || 0x00 || <OCTETS(service challenge)>]. Note: UTF8(jwk) must be the exact string that will be sent on the wire as the service will compute the hash using the exact string received in the request without modifications.

>> Note: This binding method cannot be used for keys in the other_keys array. 

•	Key certified to be resident in the TPM:

- ***tpm_certify*** (Object): Data for the TPM certification binding method.
“public” (BASE64URL(OCTETS)): TPMT_PUBLIC structure representing the public area of the key in the TPM.

- ***certification*** (BASE64URL(OCTETS)): TPMS_ATTEST returned by the TPM2_Certify command. The challenge received from the service must be passed to the TPM2_Certify command in the 'qualifyingData' parameter. The AIK provided in the request must be used to certify the key.

- ***signature*** (BASE64URL(OCTETS)): TPMT_SIGNATURE returned by the TPM2_Certify command. The challenge received from the service must be passed to the TPM2_Certify command in the 'qualifyingData' parameter. The AIK provided in the request must be used to certify the key.

>> Note: When this binding method is used for the request_key, the 'qualifyingData' parameter value passed to the TPM2_Quote command is simply the challenge received from the service.

Examples:

Key not bound to the TPM:
```
{
  "jwk": {
    "kty": "RSA",
    "n": "<Base64urlUInt(MODULUS)>",
    "e": "<Base64urlUInt(EXPONENT)>"
  }
}
```

Key bound to the TPM via quote (either resident in a VBS enclave or not):
```
{
  "jwk": {
    "kty": "RSA",
    "n": "<Base64urlUInt(MODULUS)>",
    "e": "<Base64urlUInt(EXPONENT)>"
  },
  "info": {
    "tpm_quote":
      "hash_alg": "sha-256"
    }
  }
}
```

Key certified to be resident in the TPM:
```
{
  "jwk": {
    "kty": "RSA",
    "n": "<Base64urlUInt(MODULUS)>",
    "e": "<Base64urlUInt(EXPONENT)>"
  },
  "info": {
    "tpm_certify": {
      "public": "<BASE64URL(TPMT_PUBLIC)>",
      "certification": "<BASE64URL(TPMS_ATTEST)>",
      "signature": "<BASE64URL(TPMT_SIGNATURE)>"
    }
  }
}
```

## Policy key object

The policy key object is the version of the key object used as input claims in the policy. It is processed by the service in order to make it more readable and easier to evaluate by policy rules.

•	Key not bound to the TPM:
Same as the respective key object.
Example:
```
{
  "jwk": {
    "kty": "RSA",
    "n": "<Base64urlUInt(MODULUS)>",
    "e": "<Base64urlUInt(EXPONENT)>"
  }
}
```
•	Key bound to the TPM via quote (either resident in a VBS enclave or not):
Same as the respective key object.
Example:
```
{
  "jwk": {
    "kty": "RSA",
    "n": "<Base64urlUInt(MODULUS)>",
    "e": "<Base64urlUInt(EXPONENT)>"
  },
  "info": {
    "tpm_quote":
      "hash_alg": "sha-256"
    }
  }
}
```

•	Key certified to be resident in the TPM:

***jwk*** (Object): Same as the respective key object.
***info.tpm_certify*** (Object):
- ***name_alg*** (Integer): UINT16 value representing a hash algorithm defined by the TPM_ALG_ID constants.
- ***obj_attr*** (Integer): UINT32 value representing the attributes of the key object defined by TPMA_OBJECT 
- ***auth_policy*** (BASE64URL(OCTETS)): Optional policy for using this key object.

Example:
```
{
  "jwk": {
    "kty": "RSA",
    "n": "<Base64urlUInt(MODULUS)>",
    "e": "<Base64urlUInt(EXPONENT)>"
  },
  "info": {
    "tpm_certify": {
      "name_alg": 11, // 0xB (TPM_ALG_SHA256)
      "obj_attr": 50, // 0x32 (fixedTPM | fixedParent | sensitiveDataOrigin)
      "auth_policy": "<BASE64URL(AUTH_POLICY)>"
    }
  }
}
```

## VBS report object

###	Enclave attestation:
***enclave*** (Object): Data for VSM enclave attestation.
- ***report*** (BASE64URL(OCTETS)): The VSM enclave attestation report as returned by function EnclaveGetAttestationReport. The EnclaveData parameter must be the SHA-512 hash of the value of report_signed (including the opening and closing braces). The hash function input is UTF8(report_signed).

Examples:

Enclave attestation:
```
{
  "enclave": {
    "report": "<BASE64URL(REPORT)>"
  }
}
```

## Report message

Direction
Attestation Service -> Client

Payload
```
{
  "report": "<JWT>"
}
```

***report*** (JWT): The attestation report in JSON Web Token (JWT) format (RFC 7519).


## Next steps

- [Azure Attestation workflow](workflow.md)
