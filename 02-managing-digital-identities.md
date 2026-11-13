# Managing Digital Identities

[← Back to Index](README.md)

> What a digital identity actually is, the different types, how they are structured, stored, and kept trustworthy across an organisation.

---

## 1. What is a Digital Identity?

A **digital identity** is a collection of attributes that uniquely represents a person, system, or thing in a digital environment.

Think of it like a passport — it contains attributes (name, date of birth, nationality) that distinguish you from every other person, and it is issued and trusted by an authority.

```mermaid
flowchart LR
    subgraph Identity["Digital Identity"]
        A1[Identifier\ne.g. username, email, employee ID]
        A2[Attributes\ne.g. name, department, role, location]
        A3[Credentials\ne.g. password, certificate, token]
        A4[Entitlements\ne.g. groups, permissions, roles]
    end

    SUBJECT([Person / System / Device]) --> Identity
    Identity --> ACCESS([Access to Resources])
```

---

## 2. Types of Digital Identities

Not all identities belong to humans. Modern organisations manage many types:

```mermaid
mindmap
  root((Digital\nIdentities))
    Human Identities
      Employees
      Contractors
      Partners / B2B
      Customers / Consumers
    Machine Identities
      Servers and VMs
      Containers and Pods
      IoT Devices
    Service Identities
      Applications
      APIs
      Microservices
      CI/CD Pipelines
    Privileged Identities
      Admin accounts
      Service accounts
      Break-glass accounts
```

```mermaid
flowchart TD
    subgraph Human["👤 Human Identities"]
        H1[Employee\nFull lifecycle, managed by HR]
        H2[Contractor\nTime-limited, restricted access]
        H3[Partner\nFederated from external org]
        H4[Customer\nManaged via CIAM]
    end

    subgraph Machine["🖥️ Machine Identities"]
        M1[Server / VM\nCertificate or managed identity]
        M2[Container / Pod\nSPIFFE SVID / workload cert]
        M3[IoT Device\nDevice certificate]
    end

    subgraph Service["⚙️ Service Identities"]
        S1[Application\nService account / client ID]
        S2[API\nOAuth client credentials]
        S3[CI/CD Pipeline\nShort-lived token / OIDC]
    end
```

---

## 3. Identity Attributes

Attributes are the data points that describe an identity. They are used by policy engines to make access decisions.

```mermaid
flowchart LR
    subgraph Core["Core Attributes"]
        C1[Unique Identifier\nemployee ID, UPN, UUID]
        C2[Display Name]
        C3[Email Address]
        C4[Account Status\nactive / disabled / locked]
    end

    subgraph Work["Work Attributes"]
        W1[Department]
        W2[Job Title / Role]
        W3[Manager]
        W4[Location / Office]
        W5[Cost Centre]
    end

    subgraph Security["Security Attributes"]
        S1[MFA Registered]
        S2[Last Login]
        S3[Risk Score]
        S4[Clearance Level]
    end

    Core & Work & Security --> POLICY[Policy Engine\nuses attributes\nto grant access]
```

---

## 4. Identity Assurance Levels (NIST 800-63)

Not all identities are equally trustworthy. NIST defines three **Identity Assurance Levels (IALs)** based on how rigorously the identity was verified.

```mermaid
flowchart LR
    IAL1["IAL 1 — Low Assurance\nSelf-asserted identity\nNo real-world verification\nExample: sign up with email only"]
    IAL2["IAL 2 — Medium Assurance\nRemote identity proofing\nDocument check + photo match\nExample: upload passport via app"]
    IAL3["IAL 3 — High Assurance\nIn-person or supervised proofing\nBiometric binding\nExample: visit government office"]

    IAL1 -->|Step up proofing| IAL2 -->|Step up proofing| IAL3
```

| IAL | Verification | Typical Use Case |
|---|---|---|
| IAL 1 | Self-asserted | Consumer app signup |
| IAL 2 | Remote document check | Online banking, healthcare portal |
| IAL 3 | In-person biometric | Government services, high-security access |

---

## 5. Identity Stores and Repositories

Identities must be stored somewhere. The choice of identity store affects scale, performance, and integration.

```mermaid
flowchart TD
    subgraph Stores["Identity Store Options"]
        LDAP[LDAP Directory\nOpenLDAP, AD LDS\nHierarchical, fast reads]
        AD[Active Directory\nWindows-native, Kerberos\nEnterprise standard]
        CLOUD[Cloud Directory\nAzure Entra ID, AWS IAM\nCloud-native, API-driven]
        DB[Database\nCustom user tables\nFlexible but no standards]
        CIAM[CIAM Platform\nAuth0, Cognito\nConsumer-scale, social login]
    end

    APP([Application]) -->|Queries| LDAP & AD & CLOUD & DB & CIAM
```

---

## 6. Authoritative Source of Identity

An **authoritative source** is the single system of record that owns the definitive version of an identity. All other systems derive their data from it.

```mermaid
flowchart TD
    HR[(HR System\nWorkday / SAP\nAuthoritative Source)] -->|Joiner / Mover / Leaver events| IGA[IGA Platform]

    IGA -->|Creates & updates| AD[Active Directory]
    IGA -->|Syncs| AZURE[Azure Entra ID]
    IGA -->|Provisions| APPS[Business Applications]
    IGA -->|Feeds| PAM[PAM — Privileged Accounts]

    subgraph Problem["❌ No Authoritative Source"]
        SYS1[HR System] -.->|Different data| SYS2[IT System]
        SYS2 -.->|Different data| SYS3[App Database]
        SYS3 -.->|Stale data| SYS1
    end
```

> **Rule:** Every identity attribute should have exactly one authoritative source. When systems disagree, the authoritative source wins.

---

## 7. Identity Federation

**Federation** allows one organisation to trust and use identities managed by another — without duplicating them.

```mermaid
flowchart LR
    subgraph OrgA["Organisation A\n(Identity Provider)"]
        USER([Employee: Alice]) --> IDP[Org A IdP\nAzure Entra ID]
    end

    subgraph OrgB["Organisation B\n(Service Provider)"]
        APP[Partner App] --> SP[Org B SP]
    end

    IDP -->|SAML / OIDC assertion\n'Alice is authenticated,\nshe is a Finance Manager'| SP
    SP -->|Grants access\nbased on assertion| APP
    USER -->|Accesses| APP
```

**Without federation:** Organisation B creates a duplicate account for Alice, manages her password separately, and must deactivate it when she leaves Organisation A — which often never happens.

**With federation:** Alice's identity lives only in Organisation A. When she leaves, one deprovisioning action cuts all access everywhere.

---

## 8. Non-Person Entities (NPEs)

A growing portion of identities are **not human**. These are called Non-Person Entities (NPEs) and require their own lifecycle management.

```mermaid
flowchart TD
    subgraph NPE["Non-Person Entity Identities"]
        A[Application\nClient ID + Secret\nor Certificate]
        B[Service Account\nAD service account\nor managed identity]
        C[CI/CD Pipeline\nOIDC token from\nGitHub Actions / GitLab]
        D[IoT Device\nX.509 device certificate]
        E[Microservice\nSPIFFE SVID\nor mTLS cert]
    end

    subgraph Risks["Common NPE Risks"]
        R1[Static passwords\nthat never rotate]
        R2[Overly broad permissions\n'admin' service accounts]
        R3[No owner assigned\norphaned after project ends]
        R4[Credentials in\nsource code / config files]
    end

    NPE -->|If poorly managed| Risks
```

**NPE best practices:**

| Risk | Mitigation |
|---|---|
| Static credentials | Use managed identities or short-lived tokens |
| Broad permissions | Apply least privilege to every service account |
| No owner | Assign a human owner to every NPE at creation |
| Hardcoded secrets | Use a secrets manager (Vault, AWS Secrets Manager) |

---

## 9. Identity Lifecycle Management

Every identity — human or machine — goes through a lifecycle that must be managed.

```mermaid
stateDiagram-v2
    [*] --> Requested : Identity requested\n(hire, new service, new device)

    Requested --> Verified : Identity proofed\n(IAL check, HR record)

    Verified --> Provisioned : Accounts created\nAccess assigned

    Provisioned --> Active : Identity in use

    Active --> Modified : Attributes changed\n(promotion, rename, role change)
    Modified --> Active : Updates applied

    Active --> Suspended : Temporary hold\n(leave, investigation)
    Suspended --> Active : Reinstated

    Active --> Deprovisioned : Identity no longer needed\n(leaver, decommission, end of contract)
    Suspended --> Deprovisioned : Not reinstated

    Deprovisioned --> Archived : Access removed\nRecord retained for audit

    Archived --> [*] : Retention period ends\nRecord purged
```

---

## 10. Identity Governance — Keeping Identities Clean

Over time, identities accumulate stale attributes, excess permissions, and drift from their intended state. Governance keeps them accurate.

```mermaid
flowchart TD
    DRIFT[Access Drift\nUsers accumulate excess\npermissions over time]
    ORPHAN[Orphaned Identities\nAccounts for people\nwho have left]
    STALE[Stale Attributes\nOutdated department,\ntitle, or location]
    CONFLICT[SoD Conflicts\nUser holds\nincompatible roles]

    DRIFT & ORPHAN & STALE & CONFLICT --> PROBLEMS[Security and\nCompliance Risks]

    REVIEWS[Periodic Access Reviews] -->|Recertify or revoke| DRIFT
    RECON[Reconciliation] -->|Detect and disable| ORPHAN
    SYNC[HR Sync] -->|Keep attributes current| STALE
    SOD[SoD Engine] -->|Detect and block| CONFLICT

    REVIEWS & RECON & SYNC & SOD --> CLEAN[Clean, Accurate\nIdentity Population]
```

---

## Quick Reference

| Concept | Simple Meaning |
|---|---|
| **Digital Identity** | Attributes that uniquely represent a person, system, or device |
| **Identity Attributes** | Data points (name, role, dept) used to describe and govern an identity |
| **IAL** | How rigorously an identity was verified (1 = low, 3 = high) |
| **Identity Store** | Where identities are persisted (AD, LDAP, cloud directory, database) |
| **Authoritative Source** | The single system of record that owns the definitive identity data |
| **Federation** | Trusting identities from another organisation without duplicating them |
| **NPE** | Non-Person Entity — application, service, device, or pipeline identity |
| **Identity Lifecycle** | The managed journey from creation through to deletion |
| **Access Drift** | Accumulation of excess permissions that were never removed |
| **Orphaned Identity** | An active account whose owner has left — a security risk |
