# 🔐 FINAL MASTER PROMPT — REST-ONLY EFPE SAAS (KEY RELEASE + C# DLL)

---

## Role & Context

You are a **principal security architect with 20+ years of experience** in **banking systems, cryptographic services, and regulated SaaS platforms**.

Design a **multi-tenant EFPE (Element / Field-Level Format-Preserving Encryption) SaaS platform** with the following strict constraints:

* The platform is **100% REST-based**
* **No UI**, **no React**, **no frontend ecosystem**
* The SaaS acts as a **Key Release & Policy Control Plane**
* **All encryption and decryption are performed locally** by a **C# DLL**
* The SaaS **never receives plaintext or ciphertext**
* The system must support **software-based HSM initially**, with a **clear migration path to hardware HSM**

---

## 1️⃣ High-Level Operating Model

### Responsibilities Split (Non-Negotiable)

| Component          | Responsibility                          |
| ------------------ | --------------------------------------- |
| EFPE SaaS (REST)   | Key lifecycle, policy, audit, telemetry |
| Tenant Application | Business logic                          |
| Tenant EFPE C# DLL | Encrypt / Decrypt locally               |
| HSM (Soft → Hard)  | Key generation & protection             |

---

## 2️⃣ EFPE SaaS — REST-Only Key Release Service

### Mandatory Characteristics

* Pure **REST APIs**
* Stateless
* JSON over HTTPS
* OAuth2 / mTLS authentication
* No UI
* No SDK logic inside SaaS
* No encryption operations inside SaaS

### SaaS MUST Provide

* Key issuance (wrapped / derived keys)
* Key disable / revoke / re-issue
* Policy distribution
* Key version resolution
* Audit log access
* Telemetry ingestion

### SaaS MUST NOT Provide

❌ Any UI
❌ Any frontend frameworks
❌ Any encryption / decryption
❌ Any data storage for customer data

---

## 3️⃣ Key Management Model (Critical)

### Key Types

* **Tenant Master Key (TMK)** – HSM-protected, never leaves SaaS
* **Data Encryption Keys (DEK)** – issued to tenants in **wrapped or derived form**
* **Key Versions** – mandatory

### Key States

| State       | Behavior          |
| ----------- | ----------------- |
| ACTIVE      | Encrypt + Decrypt |
| DISABLED    | Decrypt only      |
| REVOKED     | No usage          |
| EXPIRED     | Rotation required |
| COMPROMISED | Immediate kill    |

---

## 4️⃣ Software-Based HSM (Phase-1) with Hardware Path

### Phase-1 (Mandatory)

* Software-based HSM / KMS abstraction
* AES-256 keys
* Non-exportable logical model
* Centralized key lifecycle control

### Phase-2 (Planned)

* Drop-in replacement with:

  * Thales
  * Azure Managed HSM
  * AWS CloudHSM
* No API contract changes
* No tenant impact

> Design **KeyProvider abstraction** so that:
>
> * Software HSM = implementation
> * Hardware HSM = configuration change

---

## 5️⃣ Key Release API (REST Only)

Design REST APIs such as:

* `POST /keys/issue`
* `POST /keys/disable`
* `POST /keys/revoke`
* `POST /keys/reissue`
* `GET  /keys/{id}/status`
* `GET  /policies/{field}`
* `POST /telemetry`
* `GET  /audit`

All APIs must be:

* Tenant-scoped
* Fully auditable
* Rate-limited
* Idempotent where applicable

---

## 6️⃣ C# EFPE DLL (Tenant Side)

### DLL Responsibilities

* Authenticate with EFPE SaaS (REST)
* Fetch policy
* Fetch wrapped / derived DEKs
* Cache keys **in memory only**
* Perform FF1 / FF3-1 encryption
* Perform FF1 / FF3-1 decryption
* Enforce key state rules
* Emit telemetry events

### DLL Must Support

* Multiple SaaS endpoints
* Failover between servers
* Region-based endpoint selection
* Configuration-driven routing

Example:

```json
{
  "PrimaryEndpoint": "https://kms.region1.example.com",
  "SecondaryEndpoint": "https://kms.region2.example.com",
  "TimeoutMs": 3000,
  "RetryCount": 3
}
```

---

## 7️⃣ Cryptography (Unchanged, Mandatory)

* NIST SP 800-38G
* FF1 (primary)
* FF3-1 (optional)
* AES-256
* Deterministic + tweak-based
* No deprecated FF3

---

## 8️⃣ Telemetry (From C# DLL Only)

### Telemetry Must Capture

✔ Encrypt count
✔ Decrypt count
✔ Field identifier
✔ Key ID & version
✔ Policy ID
✔ Timestamp
✔ DLL version
✔ Endpoint used

### Telemetry Must NEVER Capture

❌ Plaintext
❌ Ciphertext
❌ Keys
❌ Tweaks

Telemetry is:

* Asynchronous
* Signed / tamper-evident
* Non-blocking

---

## 9️⃣ Audit Logging (SaaS Side)

Audit must track:

* Key issuance
* Key disable / revoke
* Key re-issuance
* Policy changes
* SDK authentication
* Telemetry ingestion
* Admin actions

Logs must be:

* Immutable
* Tenant-isolated
* Exportable (SIEM)

---

## 🔟 Key Disable & Re-Issuance (Explicit Requirement)

Design flows such that:

* Any issued DEK can be **disabled instantly**
* Disabled keys:

  * Cannot encrypt
  * May decrypt (policy-controlled)
* New DEKs can be issued at any time
* DLL automatically switches to newest key
* Old keys honored only per policy

---

## 1️⃣1️⃣ Compliance Positioning

Explain how this REST-only, DLL-based model:

* Meets RBI expectations
* Meets PCI DSS key management rules
* Avoids SaaS data processing liability
* Supports forensic investigation
* Enables tenant-controlled cryptography

---

## 1️⃣2️⃣ Deliverables Expected

Produce:

1. REST-only SaaS architecture
2. Key lifecycle state machine
3. Software-to-hardware HSM migration design
4. C# EFPE DLL architecture
5. Multi-endpoint configuration strategy
6. Telemetry schema
7. Audit schema
8. Threat model
9. Explicit security trade-offs

---

## ✅ Final Instruction

> **Design a REST-only, multi-tenant EFPE SaaS platform that releases and governs encryption keys using a software-based HSM (with hardware HSM roadmap), while all encryption and decryption are performed locally by a configurable C# DLL that supports multi-server connectivity, key revocation, audit logging, and secure telemetry — without any UI or frontend components.**

