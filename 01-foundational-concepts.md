# IAM Foundational Concepts

> Simple explanations with diagrams for the core building blocks of Identity and Access Management.

---

## 1. What is IAM?

**Identity and Access Management (IAM)** answers two fundamental questions:

- **Who are you?** — Identity & Authentication
- **What can you do?** — Authorization

Think of it like a hotel:
- Your **ID card** proves who you are (Identity)
- The receptionist **checks your booking** (Authentication)
- Your **room key** only opens your room, not others (Authorization)
- The hotel **logs every door entry** (Accountability)

```mermaid
flowchart LR
    A([👤 User]) --> B{Who are you?\nIdentity}
    B --> C{Prove it!\nAuthentication}
    C -->|Valid| D{What can\nyou access?\nAuthorization}
    C -->|Invalid| E([❌ Access Denied])
    D -->|Permitted| F([✅ Access Granted])
    D -->|Not permitted| E
    F --> G[(📋 Audit Log\nAccountability)]
    E --> G
```

### The 4 A's of IAM

| Concept | Simple Meaning | Example |
|---|---|---|
| **Identity** | Who you are | Username, employee ID |
| **Authentication** | Prove who you are | Password + OTP |
| **Authorization** | What you're allowed to do | Read-only on finance folder |
| **Accountability** | Track what you did | Login logs, audit trails |

---

## 2. Principle of Least Privilege (PoLP)

**Simple rule:** Give people only the access they need to do their job — nothing more.

> "Need to know, need to do."

A finance employee should be able to view invoices but **not** delete them.
An intern should read reports but **not** modify production databases.

```mermaid
flowchart TD
    subgraph Wrong["❌ Too Much Access (Bad)"]
        U1([Finance Analyst]) --> P1[Read Invoices]
        U1 --> P2[Edit Invoices]
        U1 --> P3[Delete Invoices]
        U1 --> P4[Access HR Data]
        U1 --> P5[Access IT Systems]
    end

    subgraph Right["✅ Least Privilege (Good)"]
        U2([Finance Analyst]) --> P6[Read Invoices]
        U2 --> P7[Create Invoices]
    end
```

### Why it matters
- Limits **blast radius** if an account is compromised
- Reduces **insider threat** risk
- Makes **audits** cleaner and simpler

---

## 3. Separation of Duties (SoD)

**Simple rule:** No single person should control an entire sensitive process end-to-end.

> "The person who requests money shouldn't also approve it."

This prevents fraud and errors by requiring **two or more people** to complete critical tasks.

```mermaid
flowchart LR
    subgraph SoD["Payment Process — Separated Duties"]
        A([Employee A\nRaises Purchase Order]) -->|Submits for approval| B([Employee B\nApproves Purchase Order])
        B -->|Sends to finance| C([Employee C\nProcesses Payment])
        C -->|Logs transaction| D[(Audit Log)]
    end

    subgraph Bad["❌ No SoD — Single Person Controls All"]
        X([Employee X]) -->|Raises| Y[Purchase Order]
        X -->|Approves| Y
        X -->|Pays| Y
    end
```

### Common SoD examples

| Process | Duty 1 | Duty 2 |
|---|---|---|
| Payments | Request payment | Approve payment |
| User accounts | Create account | Approve account |
| Code deployment | Write code | Deploy to production |
| Access requests | Request access | Grant access |

---

## 4. Defense in Depth

**Simple idea:** Use multiple layers of security so that if one layer fails, others still protect you.

> Like an onion — many layers, not just one wall.

```mermaid
flowchart TD
    ATK([🏴‍☠️ Attacker]) --> L1

    subgraph L1["Layer 1 — Perimeter\n(Firewall, DDoS protection)"]
        F1[Network Firewall]
    end

    L1 --> L2

    subgraph L2["Layer 2 — Identity\n(MFA, SSO, Strong Passwords)"]
        F2[Authentication Gate]
    end

    L2 --> L3

    subgraph L3["Layer 3 — Device\n(Endpoint protection, patch mgmt)"]
        F3[Device Trust Check]
    end

    L3 --> L4

    subgraph L4["Layer 4 — Application\n(RBAC, session controls)"]
        F4[App Authorization]
    end

    L4 --> L5

    subgraph L5["Layer 5 — Data\n(Encryption, DLP)"]
        F5[Data Controls]
    end

    L5 --> ASSET[(🏆 Protected Asset)]
```

Each layer **slows down or stops** an attacker. Even if one layer is bypassed, the next one takes over.

---

## 5. Zero Trust Architecture (ZTA)

**Old thinking:** "If you're inside the network, you're trusted."
**Zero Trust thinking:** "Never trust anyone — always verify, every time."

> "Trust nothing. Verify everything. Every time."

This matters because:
- Employees work from home (no clear perimeter)
- Cloud apps are outside the corporate network
- Stolen credentials can bypass perimeter defenses

```mermaid
flowchart TD
    U([👤 User / Device]) --> REQ[Access Request]

    REQ --> V1{Is the\nidentity valid?\nMFA check}
    V1 -->|No| DENY([❌ Denied])
    V1 -->|Yes| V2{Is the\ndevice healthy?\nEndpoint check}

    V2 -->|No| DENY
    V2 -->|Yes| V3{Is this\nbehavior normal?\nRisk score}

    V3 -->|High risk| DENY
    V3 -->|Low risk| V4{Does policy\nallow this action?\nAuthorization}

    V4 -->|No| DENY
    V4 -->|Yes| ACCESS([✅ Access Granted\nwith monitoring])

    ACCESS --> LOG[(📋 Continuous\nAudit Log)]
```

### ZTA Core Tenets (NIST SP 800-207)

```mermaid
mindmap
  root((Zero Trust))
    Verify Explicitly
      Always authenticate
      Always authorize
      Use all available signals
    Use Least Privilege
      Limit access scope
      Just-in-time access
      Time-bound sessions
    Assume Breach
      Minimise blast radius
      Encrypt everything
      Segment networks
      Monitor continuously
```

---

## 6. Identity Lifecycle

**Simple idea:** An identity is created, changes over time, and must be removed when no longer needed.

The three events that drive identity changes:

| Event | Meaning | Example |
|---|---|---|
| **Joiner** | Someone new arrives | New employee joins the company |
| **Mover** | Someone changes role | Employee gets promoted or transfers team |
| **Leaver** | Someone departs | Employee resigns or is terminated |

```mermaid
stateDiagram-v2
    [*] --> Provisioning : 🟢 Joiner\nNew hire / contractor

    Provisioning --> Active : Account created\nAccess granted

    Active --> Active : Mover — Role Change\nAccess updated

    Active --> Suspended : Temporary leave\nor investigation

    Suspended --> Active : Return from leave

    Suspended --> Deprovisioning : 🔴 Leaver\nResignation / termination

    Active --> Deprovisioning : 🔴 Leaver\nResignation / termination

    Deprovisioning --> Archived : Access revoked\nData retained for audit

    Archived --> [*] : Retention period\nexpired
```

### Why lifecycle management matters

```mermaid
flowchart LR
    A[Employee Leaves] -->|No offboarding| B[Account stays active]
    B --> C[Ex-employee still has access]
    C --> D[Data breach / insider threat risk]

    A2[Employee Leaves] -->|Proper offboarding| B2[Account disabled immediately]
    B2 --> C2[Access revoked]
    C2 --> D2[Risk eliminated ✅]
```

---

## 7. Trust Models

**Simple idea:** Trust defines which systems or organisations accept each other's identities.

### Explicit Trust
You explicitly configure which parties you trust.

```mermaid
flowchart LR
    A[Company A\nIdentity Provider] -->|Explicitly trusts| B[Company B\nService Provider]
    B -->|Does NOT automatically trust| C[Company C]
    A -->|Does NOT automatically trust| C
```

### Transitive Trust
If A trusts B and B trusts C, then A also trusts C (automatically).

```mermaid
flowchart LR
    A[Domain A] -->|Trusts| B[Domain B]
    B -->|Trusts| C[Domain C]
    A -.->|Transitively trusts| C

    style A fill:#4a90d9,color:#fff
    style B fill:#7b68ee,color:#fff
    style C fill:#50c878,color:#fff
```

> **Risk:** Transitive trust can create unintended access paths. Always audit trust chains.

### Implicit Trust
Trust is assumed without explicit configuration — usually inside a network perimeter.

```mermaid
flowchart TD
    subgraph Corp["Corporate Network (Old Model)"]
        PERIM[🔒 Firewall Perimeter]
        A[Laptop A] -.->|Implicitly trusted| B[Server B]
        A -.->|Implicitly trusted| C[Database C]
        PERIM -->|Once inside...| A
    end

    ATK([🏴‍☠️ Attacker who bypassed perimeter]) -.->|Now also implicitly trusted ❌| B
```

> **Zero Trust specifically eliminates implicit trust** — every request is verified regardless of network location.

### Trust Models Compared

```mermaid
quadrantChart
    title Trust Models — Risk vs Convenience
    x-axis Low Convenience --> High Convenience
    y-axis Low Risk --> High Risk
    quadrant-1 High Risk, High Convenience
    quadrant-2 High Risk, Low Convenience
    quadrant-3 Low Risk, Low Convenience
    quadrant-4 Low Risk, High Convenience
    Implicit Trust: [0.85, 0.85]
    Transitive Trust: [0.65, 0.60]
    Explicit Trust: [0.35, 0.25]
    Zero Trust: [0.20, 0.15]
```

---

## Putting It All Together

Here is how all foundational concepts connect in a real-world IAM system:

```mermaid
flowchart TD
    LC[Identity Lifecycle\nJoiner / Mover / Leaver] -->|Creates & manages| ID[(Identity Store)]

    ID --> AUTH[Authentication\nProve who you are]
    AUTH -->|Using| ZT[Zero Trust\nVerify every request]

    ZT --> AUTHZ[Authorization\nWhat can you do?]
    AUTHZ -->|Governed by| POLP[Least Privilege\nMinimum needed access]
    AUTHZ -->|Enforced with| SOD[Separation of Duties\nNo single person controls all]

    POLP --> DID[Defense in Depth\nMultiple security layers]
    SOD --> DID

    DID --> ACCESS([Protected Resource])

    TRUST[Trust Models\nExplicit / Transitive / Implicit] --> AUTH
    TRUST --> ZT

    ACCESS --> AUDIT[(Accountability\nAudit Logs)]
```

---

## Quick Reference Cheat Sheet

| Concept | One-liner | Key Rule |
|---|---|---|
| **IAM** | Who are you and what can you do? | Identity + Authentication + Authorization + Accountability |
| **Least Privilege** | Give only what's needed | No excess permissions, ever |
| **Separation of Duties** | Split critical tasks | No single person controls end-to-end |
| **Defense in Depth** | Many layers of security | One failure doesn't mean total compromise |
| **Zero Trust** | Never trust, always verify | Verify every request, every time |
| **Identity Lifecycle** | Manage from birth to death | Provision on join, update on move, revoke on leave |
| **Trust Models** | How systems trust each other | Explicit is safest; implicit is riskiest |
