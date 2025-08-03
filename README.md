# Age 18+ Credential Specification

This repository defines the structure, semantics, and usage of the **Age 18+ Credential** (`AgeOver18Credential`), issued by [18plus.app](https://18plus.app). It enables privacy-preserving, verifiable age checks in compliance with the EU Digital Identity (EUDI) framework and the W3C Verifiable Credentials standard.

---

## 🧾 Overview

**AgeOver18Credential** is a digital credential asserting that the subject is over 18 years old. It is:

- **Minimally disclosive** (no birthdate, no exact age)
- **Machine-verifiable** (signed JWT format)
- **EU compliant** (based on MDAV and EUDI Wallet specifications)
- **Extensible** (adds transparent metadata like method and assurance)
- **Double-blind by design** (issuer and verifier never learn each other’s or the user’s identity)

---

## 📌 Credential Format (VC-JWT)

```json
{
  "iss": "did:web:18plus.app",
  "sub": "did:key:z6Mkw...",
  "nbf": 1752537600,
  "exp": 1794073600,
  "jti": "urn:uuid:b8d17e21-5d2e-41cf-9e92-6f3e35412dfc",
  "vc": {
    "@context": [
      "https://www.w3.org/ns/credentials/v2",
      "https://ec.europa.eu/credentials/contexts/age-over-18-v1.jsonld",
      "https://18plus.app/credentials/contexts/age-v1.jsonld"
    ],
    "type": ["VerifiableCredential", "AgeOver18Credential"],
    "credentialSubject": {
      "id": "did:key:z6Mkw...",
      "type": "Person",
      "isOver18": true,
      "evidenceProvider": "did:web:verifier.example",
      "verificationMethod": "passport",
      "assuranceLevel": "high",
      "session": "123e4567-e89b-12d3-a456-426614174000"
    }
  }
}
```

---

## 📚 Field Breakdown

### Top-level JWT fields

| Field | Description |
|-------|-------------|
| `iss` | DID of the credential issuer. 18plus.app uses `did:web:18plus.app`. |
| `sub` | DID of the credential subject (typically a wallet-held DID like `did:key`). |
| `nbf` | Unix timestamp for when the credential becomes valid (Not Before). |
| `exp` | Unix timestamp for expiration of the credential. |
| `jti` | A unique identifier for the JWT credential (UUID). |

### `vc` object (Verifiable Credential)

The `vc` object follows the [W3C Verifiable Credentials Data Model v2](https://www.w3.org/TR/vc-data-model-2.0/) and includes standard and custom context extensions.

| Field | Description |
|-------|-------------|
| `@context` | List of JSON-LD contexts used to define terms in the credential. Includes W3C, EU, and 18plus extensions. |
| `type` | Always includes `"VerifiableCredential"` and `"AgeOver18Credential"`. |
| `credentialSubject` | Contains the actual subject claims, including the age assertion and extended metadata (see below). |

### 🔍 `credentialSubject` Fields

| Field              | Type             | Required | Description |
|-------------------|------------------|----------|-------------|
| `id`              | string           | ✅       | DID of the subject (e.g. `did:key:...`) |
| `type`            | string           | ✅       | Always `"Person"` |
| `isOver18`        | boolean          | ✅       | Always `true`. This is the core assertion. |
| `evidenceProvider`| string (DID)     | ✅       | Who originally verified the user’s age. |
| `verificationMethod` | string        | ✅       | The method used to perform the verification (`passport`, `eID`, `face_scan`, `credit_card`, etc.) |
| `assuranceLevel`  | string           | ✅       | 18plus-assigned trust level (`low`, `medium`, or `high`) |
| `session`         | string (UUID/nonce) | ✅    | Session identifier for replay protection or binding issuance to an authentication flow |


## 📦 Custom JSON-LD Context

Defined at: https://18plus.app/credentials/contexts/age-v1.jsonld

```json
{
  "@context": {
    "@version": 1.1,
    "evidenceProvider": {
      "@id": "https://schema.18plus.app/evidenceProvider",
      "@type": "@id"
    },
    "verificationMethod": {
      "@id": "https://schema.18plus.app/verificationMethod",
      "@type": "https://www.w3.org/2001/XMLSchema#string"
    },
    "assuranceLevel": {
      "@id": "https://schema.18plus.app/assuranceLevel",
      "@type": "https://www.w3.org/2001/XMLSchema#string"
    },
    "session": {
      "@id": "https://schema.18plus.app/session",
      "@type": "https://www.w3.org/2001/XMLSchema#string"
    }
  }
}
```

This context defines additional fields needed to interpret metadata provided by 18plus.app while remaining fully interoperable with the EU’s context.

---
## 📚Motivation

This credential format is built for real-world, legally sensitive scenarios such as:
- Adult content access (pornography, gambling)
- Alcohol or tobacco purchase
- KYC-lite user onboarding
- Compliance with eIDAS 2.0, EUDI Wallet, and MDAV

It ensures:
- Minimal disclosure of data
- Strong cryptographic verifiability
- Transparent provenance and method tracking
- Assurance-level signaling for relying parties

---

## 🔐 Verification & Trust

- The credential is signed using `ES256` and can be verified using the public key listed in [https://18plus.app/.well-known/did.json](https://18plus.app/.well-known/did.json)

- The proof key is:

```json
    "verificationMethod": "did:web:18plus.app#key-1"
```

- Verifiers can resolve the did:web:18plus.app DID and validate the JWS signature from the JWT.

---

## 🧪 Example Usage

### Minimal Disclosure

```json
{
  "isOver18": true
}
```

### Full Disclosure (Optional Metadata)

```json
{
  "isOver18": true,
  "evidenceProvider": "did:web:verifier.example",
  "verificationMethod": "passport",
  "assuranceLevel": "high"
}
```
---

## ✅ EU Compliance Summary

| Requirement                          | Status |
|--------------------------------------|--------|
| EUDI Wallet compatible               | ✅     |
| AgeOver18Credential profile          | ✅     |
| MDAV-compliant (minimal disclosure)  | ✅     |
| Cryptographically verifiable         | ✅     |
| EU-compliant JSON-LD contexts        | ✅     |
| Selective disclosure support         | ✅     |
| Legal provenance tracking            | ✅     |

---

### 🇫🇷 French Double Anonymity Requirement

French regulator **ARCOM** requires age verification systems to support **double anonymity** — a strict separation ensuring:

1. The **service provider** cannot identify the **issuer** or **user**
2. The **issuer** cannot identify which **relying party** the user accesses
3. Verifiers cannot link multiple credentials to the same user or source

Your **AgeOver18Credential** implementation is *double‑blind by design*:
- The **issuer (18plus.app)** never learns which site the user visits
- The **verifier site** only sees the assertion (`isOver18 : true`), not the user's identity or SID
- Provenance metadata (`evidenceProvider`, `verificationMethod`, `assuranceLevel`, `session`) is self-contained and unlinkable

---

## 📁 Supporting Files

| File               | Purpose                                |
|--------------------|----------------------------------------|
| [`did.json`](https://18plus.app/.well-known/did.json) | Public DID document                     |
| [`age-v1.jsonld`](https://18plus.app/credentials/contexts/age-v1.jsonld) | Custom JSON-LD context                  |
| [`age-over-18.html`](https://18plus.app/credentials/age-over-18.html) | Human-readable spec                     |
| This `README.md`   | Markdown version for developers and auditors |


## License

This specification is published under the MIT License. You may use, modify, and extend it with attribution to 18plus.app.

## 🔗 Links

- [W3C VC Data Model v2](https://www.w3.org/TR/vc-data-model-2.0/)
- [EUDI Wallet Reference Framework](https://ec.europa.eu/digital-building-blocks/wikis/display/EUDI)
- [JSON Web Token (JWT)](https://datatracker.ietf.org/doc/html/rfc7519)
