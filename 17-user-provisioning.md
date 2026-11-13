# User Provisioning

[← Back to Index](README.md)

> How user accounts are created, updated, and removed across systems — and how to do it at scale without manual effort.

---

## 1. What is User Provisioning?

**User provisioning** is the process of creating, managing, and removing user accounts and their access rights across all the systems an organisation uses.

Think of it like a new employee's first day:
- HR creates their record in the HR system
- IT creates their laptop account, email, Slack, Jira, Salesforce...
- When they leave, all of those accounts must be disabled

**Without automation**, this means IT gets a flood of tickets every time someone joins, moves, or leaves. With provisioning automation, it happens instantly and consistently.

```mermaid
flowchart LR
    subgraph Manual["❌ Manual Provisioning"]
        HR1([HR creates record]) -->|Emails IT| IT1[IT opens tickets]
        IT1 --> A1[Create AD account]
        IT1 --> A2[Create email]
        IT1 --> A3[Add to Slack]
        IT1 --> A4[Add to Jira]
        A1 & A2 & A3 & A4 --> DONE1([Days later... done])
    end

    subgraph Auto["✅ Automated Provisioning"]
        HR2([HR creates record]) -->|Triggers event| IGA[IGA / Provisioning Engine]
        IGA --> B1[AD account ✅]
        IGA --> B2[Email ✅]
        IGA --> B3[Slack ✅]
        IGA --> B4[Jira ✅]
        B1 & B2 & B3 & B4 --> DONE2([Minutes later... done])
    end
```

---

## 2. The Provisioning Lifecycle

User provisioning covers three key events — **Joiner, Mover, Leaver** — and one ongoing process: **reconciliation**.

```mermaid
stateDiagram-v2
    [*] --> Provisioning : Joiner\nNew hire detected in HR

    Provisioning --> Active : Accounts created\nAccess assigned by role

    Active --> Modified : Mover\nPromotion / team change\nAccess updated

    Modified --> Active : Changes applied

    Active --> Suspended : Temporary leave\nParental / sabbatical

    Suspended --> Active : Return from leave

    Active --> Deprovisioning : Leaver\nResignation / termination

    Suspended --> Deprovisioning : Leaver\nwhile on leave

    Deprovisioning --> Archived : All access revoked\nAccount disabled\nData retained

    Archived --> [*] : Retention period ends
```

---

## 3. Provisioning Methods

There are four common ways provisioning can be triggered:

### 3.1 Role-Based Provisioning
Access is assigned automatically based on the user's **role or department** in the HR system.

```mermaid
flowchart TD
    HR[(HR System\nRole: Finance Analyst\nDept: Finance\nLocation: London)] -->|Sends joiner event| IGA[IGA Engine]

    IGA -->|Looks up role profile| RP[Role: Finance Analyst\n---\n✅ Active Directory account\n✅ Email / M365\n✅ Finance reporting app\n✅ Expense system\n❌ HR admin system\n❌ Engineering tools]

    RP -->|Provisions exactly these| APPS[Target Systems]
```

### 3.2 Request-Based Provisioning
User or manager **manually requests** additional access beyond the base role profile.

```mermaid
flowchart LR
    U([User]) -->|Submits access request| PORTAL[Self-Service Portal]
    PORTAL --> MGR{Manager\nApproval}
    MGR -->|Approved| OWN{Resource Owner\nApproval}
    MGR -->|Rejected| DENY([❌ Denied])
    OWN -->|Approved| IGA[IGA Provisions Access]
    OWN -->|Rejected| DENY
    IGA --> APP[(Target App)]
    IGA --> LOG[(Audit Log)]
```

### 3.3 Event-Driven (JIT) Provisioning
Account is created **on-demand at first login** — no pre-provisioning needed.

```mermaid
sequenceDiagram
    actor User
    participant App
    participant IdP
    participant IGA

    User->>App: Attempts to log in
    App->>IdP: Redirect to IdP for auth
    IdP->>User: Authenticate (MFA etc.)
    User->>IdP: Credentials verified

    IdP->>App: SAML/OIDC token with user attributes

    App->>App: Does account exist?
    Note over App: Account does NOT exist yet

    App->>IGA: Trigger JIT provisioning\n(name, email, role from token)
    IGA->>App: Account created ✅

    App->>User: Access granted
```

### 3.4 HR-Driven (Automated) Provisioning
HR system is the **authoritative source of truth**. Changes in HR automatically cascade to all downstream systems.

```mermaid
flowchart TD
    HR[(HR System\nWorkday / SAP / BambooHR)] -->|New hire / change / termination| SCIM[SCIM 2.0 / API]

    SCIM --> IGA[IGA Platform\nSailPoint / Saviynt]

    IGA --> AD[Active Directory]
    IGA --> AZURE[Azure Entra ID]
    IGA --> EMAIL[Email / M365]
    IGA --> SLACK[Slack]
    IGA --> JIRA[Jira]
    IGA --> CRM[CRM / Salesforce]
    IGA --> CUSTOM[Other Business Apps]

    AD & AZURE & EMAIL & SLACK & JIRA & CRM & CUSTOM --> LOG[(Audit Log)]
```

---

## 4. SCIM — The Provisioning Standard

**SCIM (System for Cross-domain Identity Management)** is the industry standard API for automating provisioning. It gives every app a consistent way to receive user create/update/delete events.

```mermaid
sequenceDiagram
    participant HR as HR System\n(Workday)
    participant IGA as IGA / IdP\n(Okta / SailPoint)
    participant APP as Target App\n(Salesforce)

    HR->>IGA: New employee: Jane Smith\nRole: Sales, Dept: EMEA

    IGA->>APP: POST /scim/v2/Users\n{ "userName": "jane.smith",\n  "displayName": "Jane Smith",\n  "emails": [...],\n  "roles": ["sales-rep"] }
    APP-->>IGA: 201 Created ✅

    Note over HR,APP: Six months later — Jane promoted to Sales Manager

    HR->>IGA: Jane's role updated: Sales Manager

    IGA->>APP: PATCH /scim/v2/Users/jane.smith\n{ "roles": ["sales-manager"] }
    APP-->>IGA: 200 OK ✅

    Note over HR,APP: Jane leaves the company

    HR->>IGA: Jane terminated

    IGA->>APP: DELETE /scim/v2/Users/jane.smith
    APP-->>IGA: 204 No Content ✅
```

**SCIM operations mapped to lifecycle events:**

| SCIM Operation | HTTP Method | Lifecycle Event |
|---|---|---|
| Create user | `POST /Users` | Joiner |
| Update user | `PATCH /Users/{id}` | Mover (role/dept change) |
| Deactivate user | `PATCH /Users/{id}` `active: false` | Leaver (soft delete) |
| Delete user | `DELETE /Users/{id}` | Leaver (hard delete) |
| List users | `GET /Users` | Reconciliation |

---

## 5. Deprovisioning — The Most Critical Part

Deprovisioning (removing access when someone leaves) is **more important than provisioning** from a security perspective. Orphaned accounts — active accounts belonging to ex-employees — are one of the most common sources of data breaches.

```mermaid
flowchart TD
    LEAVE([Employee Submits Resignation]) --> HR[HR marks employee\nas terminated in HR system]

    HR --> IGA[IGA detects leaver event]

    IGA --> S1[Disable AD account\nimmediately]
    IGA --> S2[Revoke SSO sessions\nforce logout]
    IGA --> S3[Disable email\nset out-of-office]
    IGA --> S4[Remove from all\napp accounts]
    IGA --> S5[Revoke MFA devices]
    IGA --> S6[Remove from groups\nand distribution lists]
    IGA --> S7[Rotate shared\ncredentials / secrets]

    S1 & S2 & S3 & S4 & S5 & S6 & S7 --> ARCHIVE[Archive account\nRetain data per policy]

    ARCHIVE --> AUDIT[(Full audit trail\nretained)]
```

### What happens without proper deprovisioning?

```mermaid
flowchart LR
    LEAVE([Employee Leaves]) -->|No offboarding| A[Account stays active]
    A --> B[Ex-employee retains access]
    B --> C{Malicious\nor accidental?}
    C -->|Malicious| D[Data theft / sabotage]
    C -->|Accidental| E[Accesses systems\nthey shouldn't]
    D & E --> F[Security incident\nCompliance violation]
```

---

## 6. Reconciliation

Reconciliation is the process of **comparing what IGA thinks exists** with **what actually exists** in target systems — and fixing the gaps.

```mermaid
flowchart TD
    IGA[(IGA — expected state\nList of active users\nand their access)] -->|Compare| REC{Reconciliation\nEngine}
    TARGET[(Target System\nActual accounts\nthat exist)] -->|Compare| REC

    REC -->|Account in IGA, missing in target| MISSING[⚠️ Missing Account\nRe-provision or investigate]
    REC -->|Account in target, not in IGA| ORPHAN[🚨 Orphaned Account\nDisable immediately]
    REC -->|Access differs from role profile| DRIFT[⚠️ Access Drift\nRevoke excess permissions]
    REC -->|All matches| OK[✅ In sync]

    ORPHAN & MISSING & DRIFT --> REMEDIATE[Auto-remediate\nor raise alert]
```

---

## 7. Provisioning Governance

Provisioning without governance creates **access sprawl** — users accumulate permissions over time and never lose them. Governance controls keep it clean.

```mermaid
flowchart LR
    subgraph Controls["Provisioning Governance Controls"]
        A[Role Profiles\nDefine what each role gets] --> P[Provisioning]
        B[Approval Workflows\nExtra access requires sign-off] --> P
        C[Access Reviews\nPeriodic recertification] --> P
        D[SoD Rules\nBlock conflicting access] --> P
        E[Expiry Policies\nTemp access auto-revokes] --> P
    end

    P --> CLEAN[Clean, minimal\naccess across all users]
```

---

## 8. Common Provisioning Patterns by Organisation Size

```mermaid
flowchart TD
    SIZE{Organisation\nSize} -->|Small\n< 200 users| SMALL[Manual + SCIM\nIdP with SCIM connectors\neg. Okta / Azure Entra ID]

    SIZE -->|Medium\n200–2000 users| MED[IdP-driven SCIM\n+ lightweight IGA\neg. Okta Lifecycle Mgmt\nor Microsoft Identity Governance]

    SIZE -->|Large\n2000+ users| LARGE[Full IGA Platform\nSailPoint / Saviynt\n+ HR integration\n+ role management\n+ access reviews]

    SIZE -->|Enterprise\n10000+ users| ENT[IGA + PAM + CIAM\nFull lifecycle governance\nMultiple HR sources\nGlobal compliance]
```

---

## 9. Quick Reference

| Concept | Simple Meaning |
|---|---|
| **Provisioning** | Creating accounts and assigning access |
| **Deprovisioning** | Removing accounts and revoking access |
| **Role-based provisioning** | Access assigned automatically from role in HR |
| **Request-based provisioning** | Extra access requested and approved manually |
| **JIT provisioning** | Account created on first login, not in advance |
| **SCIM** | Standard API for automating provisioning across apps |
| **Orphaned account** | Active account for a user who has left — a security risk |
| **Reconciliation** | Comparing expected vs actual accounts and fixing gaps |
| **Access drift** | Accumulation of excess permissions over time |
| **IGA** | Platform that orchestrates the entire provisioning lifecycle |
