# Authorization

[← Back to Index](README.md)

> Authorization answers: **"What are you allowed to do?"** — after you've already proven who you are.

Think of it like a hotel keycard:
- The front desk **authenticated** you (checked your ID)
- Your keycard **authorizes** you — it opens your room (301) but not the penthouse, the pool after hours, or the staff areas

---

## 3.1 Access Control Models

Different models answer the question "who decides what access is allowed?" in different ways.

---

### DAC — Discretionary Access Control

The **owner** of a resource decides who can access it. Like sharing a Google Doc — you choose who can view or edit.

```mermaid
flowchart LR
    OWNER([👤 Resource Owner]) -->|Grants access to| U1([Alice — Read])
    OWNER -->|Grants access to| U2([Bob — Read + Edit])
    OWNER -->|Denies| U3([Charlie — No Access])
    OWNER --> RES[(📄 Document)]

    U1 -->|Can view| RES
    U2 -->|Can view + edit| RES
    U3 -->|❌ Blocked| RES
```

> **Risk:** Owners can accidentally over-share. No central control.

---

### MAC — Mandatory Access Control

The **system (not the owner)** enforces access based on security labels. Used in military/government contexts.

```mermaid
flowchart TD
    subgraph Labels["Security Classification Labels"]
        L1[🔴 Top Secret]
        L2[🟠 Secret]
        L3[🟡 Confidential]
        L4[🟢 Unclassified]
    end

    U1([General — Top Secret clearance]) -->|Can access| L1
    U1 -->|Can access| L2
    U1 -->|Can access| L3
    U1 -->|Can access| L4

    U2([Officer — Secret clearance]) -->|❌ Cannot access| L1
    U2 -->|Can access| L2
    U2 -->|Can access| L3

    U3([Civilian — Unclassified]) -->|❌ Cannot access| L1
    U3 -->|❌ Cannot access| L2
    U3 -->|❌ Cannot access| L3
    U3 -->|Can access| L4
```

> **Rule:** You can only read at or below your clearance level. The system enforces this — not the document owner.

---

### RBAC — Role-Based Access Control

Users are assigned **roles**, roles have **permissions**. This is the most widely used model in enterprise software.

> Like job titles — a "Manager" can approve timesheets; a "Staff" member cannot.

```mermaid
flowchart LR
    subgraph Users
        U1([Alice])
        U2([Bob])
        U3([Carol])
    end

    subgraph Roles
        R1[👔 Finance Manager]
        R2[📊 Finance Analyst]
        R3[👁️ Auditor]
    end

    subgraph Permissions
        P1[View Reports]
        P2[Edit Budgets]
        P3[Approve Payments]
        P4[Export Audit Log]
    end

    subgraph Resources
        RES1[(Finance System)]
        RES2[(Audit Database)]
    end

    U1 --> R1
    U2 --> R2
    U3 --> R3

    R1 --> P1
    R1 --> P2
    R1 --> P3

    R2 --> P1
    R2 --> P2

    R3 --> P1
    R3 --> P4

    P1 --> RES1
    P2 --> RES1
    P3 --> RES1
    P4 --> RES2
```

---

### ABAC — Attribute-Based Access Control

Access is decided by evaluating **policy rules** against attributes of the user, resource, and environment. Much more flexible than RBAC.

> "A doctor can read a patient's record — but only during their shift, and only if the patient is assigned to their department."

```mermaid
flowchart TD
    REQ[Access Request] --> PE[⚖️ Policy Engine]

    subgraph Inputs["Policy Inputs"]
        UA["👤 User Attributes
        role=doctor
        department=cardiology
        clearance=level2"]

        RA["📄 Resource Attributes
        type=patient-record
        department=cardiology
        sensitivity=high"]

        EA["🌍 Environment
        time=09:00–17:00
        location=hospital-network"]
    end

    UA --> PE
    RA --> PE
    EA --> PE

    PE -->|All conditions met| ALLOW([✅ Allow])
    PE -->|Condition failed| DENY([❌ Deny])
```

---

### ReBAC — Relationship-Based Access Control

Access is based on the **relationship** between the user and the resource in a graph. Used by Google (Zanzibar), GitHub (org/repo/team structure).

```mermaid
flowchart LR
    U([👤 Alice]) -->|member of| T[Team: Engineering]
    T -->|owns| P[Project: Backend API]
    P -->|contains| R[(Repository: api-service)]

    U2([👤 Bob]) -->|member of| T2[Team: Design]
    T2 -->|no relation to| P

    U -->|can access via relationship chain| R
    U2 -->|❌ no path to| R
```

---

### Access Control Models Compared

```mermaid
flowchart TD
    Q{Who decides\nwhat access\nis granted?} --> A1[The resource owner]
    Q --> A2[The system\nbased on labels]
    Q --> A3[Roles assigned\nto users]
    Q --> A4[Policy rules on\nattributes]
    Q --> A5[Graph of\nrelationships]

    A1 --> DAC[DAC\nDiscretionary]
    A2 --> MAC[MAC\nMandatory]
    A3 --> RBAC[RBAC\nRole-Based]
    A4 --> ABAC[ABAC\nAttribute-Based]
    A5 --> ReBAC[ReBAC\nRelationship-Based]
```

---

## 3.2 Access Control Concepts

### Subjects, Objects, Permissions

```mermaid
flowchart LR
    S([👤 Subject\nwho is acting]) -->|performs| ACT[Action / Permission\nread, write, delete, execute]
    ACT -->|on| O[(Object / Resource\nfile, API, database, service)]

    EXAMPLE["Example:\nAlice READ the invoice PDF"]
    style EXAMPLE fill:#f9f,color:#333
```

---

### Role Hierarchies

Roles can inherit permissions from lower-level roles — like job levels where a senior role includes everything the junior role can do.

```mermaid
flowchart BT
    R1[View Reports] --> R2[Finance Analyst]
    R2 --> R3[Finance Manager]
    R3 --> R4[Finance Director]

    R2_P[Edit Budgets] --> R3
    R3_P[Approve Payments] --> R4
    R4_P[Full Financial Access] --> R4

    style R4 fill:#4a90d9,color:#fff
    style R3 fill:#7b68ee,color:#fff
    style R2 fill:#50c878,color:#fff
```

---

### Role Explosion Problem

As an organisation grows, RBAC can generate thousands of roles — each a unique combination of permissions. This becomes unmanageable.

```mermaid
flowchart TD
    subgraph Small["Small Org — Manageable"]
        SR1[Admin]
        SR2[Editor]
        SR3[Viewer]
    end

    subgraph Large["Large Org — Role Explosion ❌"]
        LR1[US-East-Finance-Manager-Level3]
        LR2[EU-West-HR-Analyst-Read-Only]
        LR3[APAC-IT-Admin-NoDelete]
        LR4[US-West-Finance-Analyst-Level2-Restricted]
        LR5[... 847 more roles ...]
    end

    Small -->|Org grows + more regions + more apps| Large
```

> **Solution:** Move to ABAC or ReBAC for complex environments, or use role mining tools to consolidate.

---

### Just-in-Time (JIT) and Just-Enough-Access (JEA)

**JIT:** Access is granted only when needed, for a limited time — then automatically removed.
**JEA:** Access is scoped to only what's needed for that specific task.

> Like a visitor badge at a secure facility — valid today only, for the meeting room only.

```mermaid
flowchart TD
    U([👤 Engineer]) -->|Needs to debug production| REQ[Submit Access Request]
    REQ --> APPR{Manager\nApproval?}
    APPR -->|Denied| DEN([❌ No Access])
    APPR -->|Approved| GRANT[Temporary Access Granted\nRead-only, production DB, 2 hours]
    GRANT --> USE[Engineer investigates issue]
    USE --> TIMER[⏱️ Timer expires after 2 hours]
    TIMER --> REVOKE[Access auto-revoked]
    REVOKE --> LOG[(Audit log: who, what, when, why)]
```

---

### Implicit Deny

**Golden rule of authorization:** Everything is denied by default. Access is only granted when there is an **explicit allow**.

```mermaid
flowchart TD
    REQ[Access Request] --> SCAN{Does an explicit\nALLOW rule exist?}
    SCAN -->|Yes| CHECK{Does an explicit\nDENY rule also exist?}
    CHECK -->|Yes — deny wins| DENY([❌ Access Denied])
    CHECK -->|No| ALLOW([✅ Access Granted])
    SCAN -->|No allow rule found| IMPLICIT([❌ Implicitly Denied\nno action needed])
```

---

### Policy Evaluation Order and Conflict Resolution

When multiple rules apply, the system must decide which wins. Most systems follow: **explicit deny overrides explicit allow**.

```mermaid
flowchart TD
    REQ[User requests access] --> COLLECT[Collect all applicable policies]

    COLLECT --> D1{Any explicit\nDENY rule?}
    D1 -->|Yes| DENY([❌ Deny — DENY always wins])
    D1 -->|No| A1{Any explicit\nALLOW rule?}
    A1 -->|Yes| ALLOW([✅ Allow])
    A1 -->|No| IMPLICIT([❌ Implicit Deny])

    subgraph Example["Real Example"]
        E1[Policy A: Allow Finance team to read invoices]
        E2[Policy B: Deny all access during system maintenance window]
        E3[Result: Deny — maintenance policy wins ❌]
    end
```

---

## Quick Reference Cheat Sheet

| Concept | One-liner | Key Rule |
|---|---|---|
| **DAC** | Owner controls access | Flexible but risky — owner can over-share |
| **MAC** | System enforces labels/clearances | Used in military/government; very rigid |
| **RBAC** | Roles bundle permissions for users | Most common in enterprise; watch for role explosion |
| **ABAC** | Policy rules on attributes | Most flexible; good for complex, dynamic environments |
| **ReBAC** | Access via relationship graph | Used in GitHub, Google Drive |
| **JIT Access** | Grant access only when needed | Auto-revoke after time window |
| **JEA** | Scope access to minimum task | No "just in case" permissions |
| **Implicit Deny** | Default is always deny | Access requires an explicit allow |
| **Deny overrides** | Explicit deny beats explicit allow | Used in AWS IAM, Azure RBAC |
| **Role explosion** | Too many fine-grained roles | Sign that RBAC needs ABAC or consolidation |
