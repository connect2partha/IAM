# Identity Governance and Administration (IGA)

[← Back to Index](README.md)

> IGA is the control tower of IAM — it ensures the right people have the right access, for the right reasons, at the right time, and that someone is checking regularly.

---

## 1. What is IGA?

**Simple idea:** IGA is the system that manages *who has access to what*, automates giving and removing access, and makes sure someone regularly checks that access is still appropriate.

Think of a building's key card system:
- Security creates cards when staff join (provisioning)
- Updates them when staff change roles (mover)
- Deactivates them when staff leave (deprovisioning)
- Sends managers a list to review every quarter (access certification)

```mermaid
flowchart TD
    HR[(👥 HR System\nSource of Truth)] -->|Identity events| IGA[🏛️ IGA Platform]

    IGA -->|Creates accounts| AD[(Active Directory)]
    IGA -->|Creates accounts| EMAIL[(Email / M365)]
    IGA -->|Creates accounts| CRM[(CRM / Salesforce)]
    IGA -->|Creates accounts| CLOUD[(Cloud Apps)]
    IGA -->|Creates accounts| ERP[(ERP / SAP)]

    IGA --> GOV[Governance Layer]
    GOV --> CERT[Access Certifications]
    GOV --> SOD[SoD Enforcement]
    GOV --> AUDIT[Audit Reports]
```

---

## 2. Automated Provisioning

**Simple idea:** When someone joins the company, IGA automatically creates all their accounts — no IT helpdesk ticket needed.

> Like a hotel check-in: you arrive, give your name, and your key card, room assignment, breakfast voucher and gym access are all ready automatically.

```mermaid
flowchart TD
    HR[HR System\nNew employee record created] -->|Joiner event detected| IGA[IGA Platform]

    IGA --> PROFILE[Build Identity Profile\nname, role, department, location]

    PROFILE --> MATCH[Match to Role Catalogue\nWhich access does this role need?]

    MATCH --> PROV[Automated Provisioning]

    PROV --> AD[✅ Create AD account]
    PROV --> EMAIL[✅ Create email mailbox]
    PROV --> APPS[✅ Grant app access per role]
    PROV --> BADGE[✅ Request building badge]

    AD & EMAIL & APPS & BADGE --> NOTIFY[📧 Welcome email sent to new employee]
    NOTIFY --> LOG[(Audit Log — all actions recorded)]
```

### Deprovisioning — the critical part

```mermaid
flowchart TD
    HR[HR System\nEmployee termination recorded] -->|Leaver event| IGA[IGA Platform]

    IGA --> DISABLE[Immediately disable AD account]
    IGA --> REVOKE[Revoke all app access]
    IGA --> EXPIRE[Expire active sessions & tokens]
    IGA --> FORWARD[Set email forwarding / archive]
    IGA --> BADGE2[Deactivate building badge]
    IGA --> VAULT[Transfer files to manager]

    DISABLE & REVOKE & EXPIRE & FORWARD & BADGE2 & VAULT --> LOG[(Audit Log — timestamped)]

    style DISABLE fill:#e74c3c,color:#fff
    style REVOKE fill:#e74c3c,color:#fff
    style EXPIRE fill:#e74c3c,color:#fff
```

> **Why deprovisioning matters:** Orphaned accounts (active accounts for ex-employees) are one of the most common sources of data breaches.

---

## 3. Access Request and Approval Workflow

**Simple idea:** When an employee needs access to something new, they submit a request. The right people approve it. IGA fulfils it and records everything.

> Like requesting annual leave: you submit, your manager approves, HR records it.

```mermaid
flowchart TD
    USER([👤 Employee]) -->|Submits access request via portal| REQ[Access Request Created\nWhat resource? Why needed? How long?]

    REQ --> OWNER{Resource Owner\nor Manager}
    OWNER -->|Approves| FULFIL[IGA Provisions Access]
    OWNER -->|Rejects| DENY[❌ Request Denied\nNotification sent]
    OWNER -->|No response in 48h| ESC[Escalated to\nsecond approver]
    ESC -->|Approves| FULFIL
    ESC -->|No response| AUTO_DENY[Auto-denied]

    FULFIL --> ACCESS[✅ Access Granted]
    FULFIL --> LOG[(Audit Log)]
    DENY --> LOG
    AUTO_DENY --> LOG

    ACCESS --> EXPIRY[⏱️ Access Expiry set\nif time-limited]
```

---

## 4. Access Certifications (Access Reviews)

**Simple idea:** Periodically, managers are asked "Does your team still need all this access?" They review and approve or revoke. This catches access that was granted long ago but is no longer needed.

> Like an annual property inspection — the landlord checks everything is still appropriate.

```mermaid
flowchart TD
    SCHEDULE[⏰ Certification Campaign Triggered\nquarterly / annually / on-demand] --> GENERATE[IGA generates review tasks\nfor each manager]

    GENERATE --> MGR([👔 Manager receives review list])

    MGR -->|Reviews each user+access pair| DECISION{Decision}

    DECISION -->|Certify — still needed| CERT[✅ Access Retained\nLogged with justification]
    DECISION -->|Revoke — no longer needed| REVOKE[❌ Access Removed\nIGA deprovisions immediately]
    DECISION -->|No response by deadline| AUTO[🔴 Auto-Revoke\nor escalation per policy]

    CERT & REVOKE & AUTO --> REPORT[Compliance Report Generated]
    REPORT --> AUDIT[(Audit Trail\nfor SOX / ISO 27001 etc.)]
```

---

## 5. Role Management and Role Mining

**Simple idea:** Instead of assigning individual permissions to each person, group common permissions into **roles** (e.g., "Finance Analyst" role). Assign the role, not 50 individual permissions.

**Role mining** is the process of analysing who has what access and discovering natural role groupings.

```mermaid
flowchart LR
    subgraph Before["❌ Before Role Mining — Permission Chaos"]
        U1([Alice]) --> P1[Read Invoices]
        U1 --> P2[Create Invoices]
        U1 --> P3[View Reports]
        U2([Bob]) --> P1
        U2 --> P2
        U2 --> P3
        U3([Carol]) --> P1
        U3 --> P2
        U3 --> P3
    end

    subgraph After["✅ After Role Mining — Clean Roles"]
        ROLE[Finance Analyst Role] --> P4[Read Invoices]
        ROLE --> P5[Create Invoices]
        ROLE --> P6[View Reports]
        U4([Alice]) --> ROLE
        U5([Bob]) --> ROLE
        U6([Carol]) --> ROLE
    end
```

```mermaid
flowchart TD
    MINE[Role Mining Process] --> COLLECT[Collect current access data\nfrom all target systems]
    COLLECT --> ANALYSE[Analyse patterns\nWho has similar access?]
    ANALYSE --> DISCOVER[Discover candidate roles\nbased on actual usage]
    DISCOVER --> REVIEW[Role owner reviews\nand approves role definitions]
    REVIEW --> PUBLISH[Publish roles to role catalogue]
    PUBLISH --> ASSIGN[Assign roles to users\nRemove individual entitlements]
    ASSIGN --> CLEAN[🎉 Cleaner, auditable access model]
```

---

## 6. Reconciliation — Finding Orphan Accounts

**Simple idea:** IGA regularly checks target systems for accounts that shouldn't exist — accounts for people who have left, or accounts created outside IGA's control.

> Like a stocktake — comparing what you think you have vs what's actually on the shelf.

```mermaid
flowchart TD
    IGA[IGA Platform] -->|Scheduled reconciliation job| SCAN[Scan all target systems\nAD, email, apps, cloud]

    SCAN --> COMPARE{Compare accounts found\nvs identities in IGA}

    COMPARE -->|Account has matching active identity| OK[✅ Account is authorised\nNo action needed]
    COMPARE -->|Account has no matching identity| ORPHAN[🔴 Orphan Account Found]
    COMPARE -->|Identity exists but account missing| MISSING[⚠️ Missing Account\nShould be provisioned]

    ORPHAN --> ACTION{Remediation}
    ACTION -->|Auto-disable| DISABLE[Account disabled]
    ACTION -->|Raise ticket| TICKET[Manual review ticket]

    MISSING --> REPROV[Re-provision account]
```

---

## 7. Separation of Duties (SoD) in IGA

**Simple idea:** IGA detects when a single person is being granted conflicting permissions that together would let them commit fraud or errors without detection.

```mermaid
flowchart TD
    REQ[Access Request:\nUser requests 'Approve Payments' role] --> CHECK[IGA runs SoD check]

    CHECK --> COMPARE{Does user already have\n'Create Payments' role?}

    COMPARE -->|No conflict| ALLOW[✅ Request proceeds\nto approval workflow]
    COMPARE -->|⚠️ Conflict detected| BLOCK[SoD Violation Flagged]

    BLOCK --> NOTIFY[Notify requester and approver]
    BLOCK --> OPTS{Options}

    OPTS -->|Risk accepted with justification| OVERRIDE[Approved with compensating control\nExtra monitoring applied]
    OPTS -->|Not accepted| DENY[❌ Request Denied]
```

### Common SoD conflict pairs

| Conflicting Permission A | Conflicting Permission B | Risk |
|---|---|---|
| Create purchase order | Approve purchase order | Fraudulent POs |
| Create payroll entry | Approve payroll entry | Payroll fraud |
| Create user account | Approve user account | Unauthorised access |
| Submit expenses | Approve expenses | Expense fraud |
| Write code | Deploy to production | Untested code in prod |

---

## 8. Audit Logging and Compliance Reporting

**Simple idea:** Every action IGA takes — granting access, revoking it, certifications — is logged with who did what, when, and why. This creates an evidence trail for auditors.

```mermaid
flowchart LR
    EVENTS[IGA Events] --> LOG[(Immutable Audit Log)]

    EVENTS --> E1[Account provisioned]
    EVENTS --> E2[Access request approved]
    EVENTS --> E3[Certification completed]
    EVENTS --> E4[SoD violation detected]
    EVENTS --> E5[Account deprovisioned]

    LOG --> REPORT[Compliance Reports]

    REPORT --> SOX[SOX — Financial access controls]
    REPORT --> ISO[ISO 27001 — Access control evidence]
    REPORT --> SOC[SOC 2 — Logical access review]
    REPORT --> GDPR[GDPR — Data access justification]
```

---

## 9. IGA Tools at a Glance

| Tool | Strength | Best For |
|---|---|---|
| **SailPoint IdentityNow** | Comprehensive IGA, cloud-native | Large enterprises |
| **Saviynt** | Strong cloud app connectors, CIAM | Cloud-first orgs |
| **One Identity** | Deep AD integration | AD-heavy environments |
| **Microsoft Identity Governance** | Native Azure/M365 integration | Microsoft shops |

---

## Quick Reference Cheat Sheet

| Concept | One-liner | Key Outcome |
|---|---|---|
| **Provisioning** | Automatically create accounts on join | No manual IT tickets; instant access |
| **Deprovisioning** | Remove all access on departure | No orphan accounts |
| **Access Request** | Formal workflow to request new access | Approval trail for auditors |
| **Access Certification** | Periodic review: is this access still needed? | Remove stale access |
| **Role Mining** | Discover natural role groupings from real access | Reduce permission sprawl |
| **Reconciliation** | Find accounts that exist but shouldn't | Catch shadow accounts |
| **SoD Enforcement** | Block conflicting permissions | Prevent fraud |
| **Audit Logging** | Record every IGA action immutably | Compliance evidence |
