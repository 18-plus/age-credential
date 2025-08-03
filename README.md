# Age Credentials

This repository defines the structure, semantics, and usage of age verification credentials issued by [18plus.app](https://18plus.app). These credentials support privacy-preserving, machine-verifiable age checks and are compatible with both EU Digital Identity (EUDI) and broader international requirements.

**18plus.app** acts as an intermediate wallet provider, issuing age credentials on demand to third-party relying parties based on age assurance performed by a PID Provider.

---

## 🎯 Why Two Credential Types?

We provide **two credential profiles** to serve different regulatory and implementation needs:

### [`AgeOver18Credential`](./AgeOver18Credential/)

- Targeted for **EU use** and aligned with **EUDI Wallet** and **ARF** specifications.
- Optimized for **double-blind** age assurance (per French ARCOM requirements).
- Encodes a **boolean `isOver18: true`** assertion and optional provenance metadata.

### [`AgeCredential`](./AgeCredential/)

- More **general-purpose** and internationally useful (e.g., U.S., Canada).
- Encodes a **string-based `ageTier`** such as `"13+"`, `"16+"`, `"18+"`, `"21+"`.
- Can be extended to serve age-gated services requiring different thresholds.

Use the credential type that best fits your jurisdiction, use case, and regulatory obligations.

---

## 🧾 Credential Specifications

| Credential               | Spec File                                      | Purpose                            |
|--------------------------|-----------------------------------------------|------------------------------------|
| `AgeOver18Credential`    | [`/AgeOver18Credential/README.md`](./AgeOver18Credential/README.md) | Strictly EU-compliant, minimal disclosure |
| `AgeCredential`          | [`/AgeCredential/README.md`](./AgeCredential/README.md)           | Tiered assertion for broader age-gating  |

---

## 📁 Shared Resources

| File/Folder                                 | Purpose                              |
|---------------------------------------------|--------------------------------------|
| [`/schema`](./schema)                        | Custom JSON-LD terms used across credentials |
| [`/credentials/contexts`](./credentials/contexts) | Linked JSON-LD context documents     |

---

## 🛡️ Privacy & Compliance

All issued credentials are:
- **Cryptographically signed** (VC-JWT)
- **Minimally disclosive** by design
- **Double-anonymous** 
- **Self-contained and unlinkable**

---

## License

This repository and specification are published under the MIT License. Attribution required when reused or modified.