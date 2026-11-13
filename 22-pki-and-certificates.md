# PKI and Certificates

[← Back to Index](README.md)

> Simple explanations with diagrams for how digital certificates work — the foundation of trusted communication on the internet.

---

## 1. What is PKI?

**Public Key Infrastructure (PKI)** is the system that lets computers trust each other online using **digital certificates** — like digital ID cards issued by a trusted authority.

> When you visit a bank website and see the padlock icon, that's PKI in action. Your browser checked the bank's certificate and confirmed: "Yes, this is really the bank."

---

## 2. X.509 Certificate Anatomy

A certificate is a digitally-signed document that says **"this public key belongs to this entity."**

> Like a passport: it has your name, photo, expiry date, and an official stamp that proves it wasn't forged.

```mermaid
flowchart TD
    subgraph CERT["📜 X.509 Certificate"]
        subgraph SUBJECT["Subject (Who owns this cert?)"]
            CN["CN = www.example.com\nCommon Name"]
            SAN["SAN = www.example.com, api.example.com\nSubject Alternative Names"]
            ORG["O = Example Corp\nOrganisation"]
        end
        subgraph ISSUER["Issuer (Who signed it?)"]
            ICA["Issued by: DigiCert CA"]
            VALID["Valid From: 2024-01-01\nValid To: 2025-01-01"]
        end
        subgraph PUBKEY["Public Key"]
            KEY["RSA 2048-bit public key\nor EC P-256"]
        end
        subgraph EXT["Extensions"]
            KU["Key Usage: Digital Signature, Key Encipherment"]
            EKU["Extended Key Usage: TLS Server Authentication"]
            BC["Basic Constraints: Not a CA"]
        end
        subgraph SIG["Signature"]
            SIGVAL["CA's digital signature over all the above"]
        end
    end
```

**Key fields explained:**

| Field | What it means |
|---|---|
| **CN** | The main name (usually domain or username) |
| **SAN** | All hostnames this cert is valid for |
| **Valid From/To** | The certificate's active window |
| **Key Usage** | What the cert can be used for (signing, encryption) |
| **Signature** | Proves the CA approved this cert |

---

## 3. Certificate Authority (CA) Hierarchy

No single CA signs everything. Trust is organised in a **chain** — like a government's chain of authority.

> Root CA = the Prime Minister. Never makes direct decisions, stays offline and secure. Intermediate CA = a minister. Issues authority to departments. Issuing CA = a local office. Issues actual ID cards to citizens.

```mermaid
flowchart TD
    ROOT["🏛️ Root CA\nSelf-signed, offline, air-gapped\nTrusted by operating systems / browsers"]
    ROOT -->|Signs| INT1["🏢 Intermediate CA 1\nOnline, issues to sub-CAs"]
    ROOT -->|Signs| INT2["🏢 Intermediate CA 2\nOnline, issues to sub-CAs"]
    INT1 -->|Signs| ISSUE1["🏬 Issuing CA — TLS Certs\nIssues server certificates"]
    INT1 -->|Signs| ISSUE2["🏬 Issuing CA — Client Certs\nIssues user/device certificates"]
    INT2 -->|Signs| ISSUE3["🏬 Issuing CA — Code Signing\nIssues developer certificates"]
    ISSUE1 -->|Signs| CERT1["📜 www.example.com cert"]
    ISSUE2 -->|Signs| CERT2["📜 alice@example.com client cert"]
    ISSUE3 -->|Signs| CERT3["📜 app-installer.exe signed cert"]

    style ROOT fill:#c0392b,color:#fff
    style INT1 fill:#2980b9,color:#fff
    style INT2 fill:#2980b9,color:#fff
    style ISSUE1 fill:#27ae60,color:#fff
    style ISSUE2 fill:#27ae60,color:#fff
    style ISSUE3 fill:#27ae60,color:#fff
```

**Why this hierarchy?**
- Root CA stays offline — extremely hard to compromise
- If an Intermediate CA is compromised, only revoke that CA — Root and other branches stay safe
- Separates responsibilities (TLS vs client certs vs code signing)

---

## 4. Certificate Issuance

How does a website or service get a certificate?

> Like applying for a passport: you fill in a form (CSR), submit it to the passport office (CA), they verify you are who you say you are, then issue the passport (certificate).

```mermaid
flowchart TD
    A([Server / Application]) -->|1. Generate key pair\nkeep private key secret| B[Create Certificate\nSigning Request - CSR\nContains: public key + subject info]
    B -->|2. Submit CSR| CA[Certificate Authority]
    CA -->|3. Validate identity\nDomain validation / org validation / extended validation| VALID{Valid?}
    VALID -->|No| REJECT([❌ Rejected])
    VALID -->|Yes| SIGN[4. CA signs CSR\nwith its private key]
    SIGN -->|5. Returns signed certificate| A
    A -->|6. Install cert on server| SERVE[Server serves cert\nduring TLS handshake]
```

---

## 5. Certificate Validation

When your browser connects to a site, it checks the certificate is genuine.

> Like checking a passport: is the signature real? Is it still valid? Has it been reported stolen (revoked)?

```mermaid
flowchart TD
    TLS([TLS Connection starts]) --> C1{1. Is the cert\nsigned by a trusted CA?}
    C1 -->|No| FAIL([❌ Untrusted])
    C1 -->|Yes| C2{2. Is today within\nthe validity period?}
    C2 -->|No — expired| FAIL
    C2 -->|Yes| C3{3. Is the cert\nrevoked?\nCRL or OCSP check}
    C3 -->|Revoked| FAIL
    C3 -->|Not revoked| C4{4. Does the cert's CN/SAN\nmatch the hostname?}
    C4 -->|No| FAIL
    C4 -->|Yes| SUCCESS([✅ Certificate trusted\nTLS connection proceeds])
```

---

## 6. Revocation — CRL and OCSP

Sometimes a certificate must be cancelled before it expires (private key stolen, employee left, etc.).

| Method | How it works | Drawback |
|---|---|---|
| **CRL** (Certificate Revocation List) | CA publishes a list of revoked certs | List can be large; may be stale |
| **OCSP** (Online Certificate Status Protocol) | Real-time query: "Is cert X revoked?" | Privacy concern; CA must be online |
| **OCSP Stapling** | Server pre-fetches OCSP response and includes it in TLS handshake | Best of both — fast, private |

```mermaid
sequenceDiagram
    participant Server
    participant CA_OCSP as CA OCSP Responder
    participant Browser

    Note over Server: Periodically (e.g. every hour)
    Server->>CA_OCSP: Is my cert still valid?
    CA_OCSP-->>Server: Signed OCSP response: GOOD ✅

    Browser->>Server: TLS ClientHello
    Server-->>Browser: TLS ServerHello\n+ Certificate\n+ Stapled OCSP Response

    Note over Browser: Browser checks stapled\nOCSP response — no extra\nnetwork call needed ✅
    Browser-->>Server: TLS Handshake complete
```

---

## 7. Mutual TLS (mTLS)

Standard TLS: browser verifies the server.
mTLS: **both** the client and server verify each other.

> Like two spies meeting — they both show their agency badges before exchanging information.

```mermaid
sequenceDiagram
    participant C as Client
    participant S as Server

    C->>S: ClientHello
    S-->>C: ServerHello + Server Certificate
    C->>C: Verify server cert chain ✅
    S-->>C: Request: Send your certificate too
    C->>S: Client Certificate
    S->>S: Verify client cert chain ✅
    S-->>C: Handshake complete — both verified

    Note over C,S: Encrypted, mutually-authenticated channel
    C->>S: Secure request
    S-->>C: Secure response
```

**Where mTLS is used:**
- Service-to-service communication (microservices, service mesh)
- API access from specific authorised clients
- Zero Trust network architecture

---

## 8. Certificate Expiry Risk

An expired certificate immediately breaks services — websites show scary warnings, APIs stop working.

```mermaid
flowchart TD
    subgraph Bad["❌ No Monitoring — Certificate Expires"]
        B1[Certificate issued\n1 year validity] --> B2[Team forgets\nabout it]
        B2 --> B3[Certificate expires\nat 2am Sunday]
        B3 --> B4[Website breaks\nfor all users]
        B4 --> B5[Emergency fix\nat 3am 😱]
    end

    subgraph Good["✅ With Monitoring — Proactive Renewal"]
        G1[Certificate issued\n1 year validity] --> G2[Monitoring tool\ntracks expiry dates]
        G2 --> G3[Alert at 60 days:\nRenew soon]
        G3 --> G4[Alert at 30 days:\nRenew now]
        G4 --> G5[Cert renewed\nbefore expiry ✅]
        G5 --> G2
    end
```

**Tools:** Let's Encrypt + Certbot (auto-renew), Venafi, cert-manager (Kubernetes)

---

## 9. Code Signing

Code signing proves that software came from a known publisher and **hasn't been tampered with** since it was signed.

> Like a wax seal on a medieval letter — if the seal is intact, the message is genuine and unaltered.

```mermaid
flowchart LR
    subgraph Dev["Developer Side"]
        CODE[Write application code] --> BUILD[Build binary / installer]
        BUILD --> SIGN["Sign with\ndeveloper's private key\n(code signing cert)"]
        SIGN --> PUBLISH[Publish to download page]
    end

    subgraph User["User Side"]
        DOWNLOAD[User downloads installer] --> VERIFY{OS verifies\ndigital signature}
        VERIFY -->|Signature invalid\nor cert untrusted| WARN([⚠️ Warning: Unknown publisher\nRun anyway?])
        VERIFY -->|Signature valid\ncert trusted| INSTALL([✅ Verified publisher\nSafe to install])
    end

    Dev --> User
```

---

## Quick Reference Cheat Sheet

| Concept | Simple Meaning | Key Point |
|---|---|---|
| **X.509 Certificate** | Digital ID card for a server/user/service | Contains public key + identity + CA signature |
| **CA Hierarchy** | Chain of trust: Root → Intermediate → Issuing | Root stays offline; compromise contained per branch |
| **CSR** | Application form for a certificate | Contains public key; keep private key secret |
| **CRL / OCSP** | Ways to check if a cert was cancelled | OCSP Stapling is the modern best practice |
| **mTLS** | Both sides show certificates | Use for service-to-service auth |
| **Code Signing** | Proves software came from known publisher | Required for Windows driver/app distribution |
| **Cert Expiry Monitoring** | Track all certs and alert before expiry | Automate renewal where possible |
