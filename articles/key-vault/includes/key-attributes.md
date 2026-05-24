---
author: msmbaldwin
ms.service: azure-key-vault
ms.topic: include
ms.date: 05/21/2026
ms.author: mbaldwin
---

In addition to the key material, you can specify the following attributes. In a JSON request, you must include the `attributes` keyword and braces (`{` `}`) even if you don't specify any attributes.

- *enabled*: boolean, optional, default is **true**. Specifies whether the key is enabled and usable for cryptographic operations. Use the *enabled* attribute with *nbf* and *exp*. When an operation occurs between *nbf* and *exp*, the operation is permitted only if *enabled* is set to **true**. Operations outside the *nbf* / *exp* window are automatically disallowed, except for decrypt, release, unwrap, and verify.
- *nbf*: IntDate, optional, default is now. The *nbf* (not before) attribute identifies the time before which the key MUST NOT be used for cryptographic operations, except for decrypt, release, unwrap, and verify. The processing of the *nbf* attribute requires that the current date/time MUST be after or equal to the not-before date/time listed in the *nbf* attribute. Some small leeway (normally no more than a few minutes) may be provided to account for clock skew. Its value MUST be a number containing an IntDate value.
- *exp*: IntDate, optional, default is "forever". The *exp* (expiration time) attribute identifies the expiration time on or after which the key MUST NOT be used for cryptographic operations, except for decrypt, release, unwrap, and verify. The processing of the *exp* attribute requires that the current date/time MUST be before the expiration date/time listed in the *exp* attribute. Some small leeway (typically no more than a few minutes) may be provided to account for clock skew. Its value MUST be a number containing an IntDate value.

The following read-only attributes are included in any response that includes key attributes:

- *created*: IntDate, optional. The *created* attribute indicates when this version of the key was created. The value is null for keys created before the addition of this attribute. Its value MUST be a number containing an IntDate value.
- *updated*: IntDate, optional. The *updated* attribute indicates when this version of the key was updated. The value is null for keys that were last updated before the addition of this attribute. Its value MUST be a number containing an IntDate value.

### Date-time controlled operations

Not-yet-valid and expired keys, outside the *nbf* / *exp* window, work for **decrypt**, **release**, **unwrap**, and **verify** operations (they don't return 403, Forbidden). The rationale for using the not-yet-valid state is to allow a key to be tested before production use. The rationale for using the expired state is to allow recovery operations on data that was created when the key was valid. You can also disable access to a key by updating the *enabled* attribute to **false**.

For more information about other possible attributes, see the [JSON Web Key (JWK)](https://tools.ietf.org/html/draft-ietf-jose-json-web-key-41) specification.
