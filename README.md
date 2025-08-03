# Age 18+ Credential Specification

This repository defines the structure, semantics, and usage of the **Age 18+ Credential** (`AgeOver18Credential`), issued by [18plus.app](https://18plus.app). It enables privacy-preserving, verifiable age checks in compliance with the EU Digital Identity (EUDI) framework and the W3C Verifiable Credentials standard.

**18+ App** acts as an intermediate wallet provider, issuing age credentials on demand to third-party relying parties based on age assurance provided by a PiD Provider.


---

## 🧾 Overview

**AgeOver18Credential** is a digital credential asserting that the subject is over 18 years old. It is:

- **Minimally disclosive** (no birthdate, no exact age)
- **Machine-verifiable** (signed JWT format)
- **EU compliant** (based on MDAV and EUDI Wallet specifications)
- **Extensible** (adds transparent metadata like method and assurance)
- **Double-blind by design** (issuer and verifier never learn each other’s or the user’s identity)

## 🧩 Position in EUDI Wallet Architecture

The **AgeOver18Credential** aligns with the **EUDI Wallet ARF** roles:

- **PID Provider** = PID Provider (performs identity verification and issues age evidence)
- **18plus.app** = Intermediate Wallet Provider (holds evidence, issues AgeOver18Credential)
- **Relying Party** = Service provider verifying credentials presented via user wallet

This model ensures **privacy**, **provenance**, and **compliance** with EU ARF architecture requirements.



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
      "pidProvider": "did:web:verifier.example",
      "verificationMethod": "passport",
      "loa": "LoA3",
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

| Field                  | Type               | Required | Description |
|------------------------|--------------------|----------|-------------|
| `id`                   | string             | ✔        | DID of the subject (e.g. `did:key:...`) |
| `type`                 | string             | ✔        | Always `"Person"` |
| `isOver18`             | boolean            | ✔        | Always `true`. This is the core assertion. |
| `pidProvider`          | string (DID)       |          | The PID Provider who performed the original verification. Omitted when regulatory frameworks (e.g. France’s ARCOM) prohibit issuer identification by relying parties. |
| `verificationMethod`   | string             |          | The method used to perform the verification (`passport`, `eID`, `face_scan`, `credit_card`, etc.) |
| `loa`                  | string             |          | Level of Assurance, in line with the eIDAS Regulation (`LoA1`, `LoA2`, or `LoA3`) |
| `session`              | string (UUID/nonce)|          | Session identifier for replay protection or binding issuance to an authentication flow |

Defined at: https://18plus.app/credentials/contexts/age-v1.jsonld

---

## 📦 Custom JSON-LD Context

The `AgeOver18Credential` provides additional fields. These enable transparency of the underlying age credential supporting to the **Relying Party** to support regulatory compliance in certain jurisdictions.

- **`pidProvider`** — The DID of the original PID Provider that verified the user’s age. *Optional.* Omitted where regulatory frameworks (e.g. France’s ARCOM) prohibit.

- **`verificationMethod`** — A string describing the method of verification (e.g. `passport`, `eID`, `face_scan`, `credit_card`). *Optional.* 

- **`loa`** — The Level of Assurance (`LoA1`, `LoA2`, `LoA3`), aligned with [eIDAS](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX%3A32014R0910) standards. *Optional.* 

- **`session`** — An ephemeral UUID or nonce generated by the gateway (either the issuer’s or the verifier’s). Used for replay protection and session binding. It is opaque and unlinkable across credentials.  *Optional.* 




```json
{
  "@context": {
    "@version": 1.1,
    "pidProvider": {
      "@id": "https://18plus.app/credentials/schema/pidProvider",
      "@type": "@id"
    },
    "verificationMethod": {
      "@id": "https://18plus.app/credentials/schema/verificationMethod",
      "@type": "https://www.w3.org/2001/XMLSchema#string"
    },
    "loa": {
      "@id": "https://18plus.app/credentials/schema/loa",
      "@type": "https://www.w3.org/2001/XMLSchema#string"
    },
    "session": {
      "@id": "https://18plus.app/credentials/schema/session",
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
  "pidProvider": "did:web:verifier.example",
  "verificationMethod": "passport",
  "loa": "LoA3"
}
```
---
## ✅ Level of Assurance

LoA refers to how strongly a credential is backed by identity verification and fraud resistance. Under the eIDAS Regulation, there are three core levels:
- `LoA1` (Low): Weak identity proofing, possibly self-registration
- `LoA2` (Substantial): Stronger identity checks and multi-factor authentication
- `LoA3` (High): Credential issued based on strong evidence (e.g. passport) and secure proofing methods  ￼

These are widely used across European national eID schemes to indicate the trust level of issued credentials  ￼

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


1. The **service provider** (e.g. adult website) must not learn the **user's** identity
2. The **service provider** must not learn the identity of the **PID Provider** (who performed age verification)
3. The **PID Provider** must not learn which services the **user** accesses
4. No party should be able to correlate multiple verifications back to the same user

Your **AgeOver18Credential** implementation is *double‑blind by design*:
- The **issuer (18plus.app)** never learns which site the user visits
- The **verifier site** only sees the assertion (`isOver18 : true`), not the user's identity or SID
- Provenance metadata (`pidProvider`, `verificationMethod`, `loa`, `session`) is self-contained and unlinkable

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
