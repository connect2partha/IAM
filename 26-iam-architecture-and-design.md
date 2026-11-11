# IAM Architecture and Design Patterns

> Simple explanations of how IAM systems are structured — from the choices you make at the drawing board to how you keep them running when things go wrong.

---

## Why Architecture Matters

IAM is infrastructure. Getting the architecture right means every application, cloud, and user experience works smoothly. Getting it wrong means fragile integrations, security gaps, and painful migrations later.

Think of IAM architecture like **city planning for identity** — the roads (trust relationships), buildings (identity providers), and traffic rules (policies) all need to work together.

---

## 1. Centralised vs Federated Identity

### Centralised Identity
One identity provider. All applications trust it. Simple to manage, single point of failure.

**Analogy:** One master key to the building made by one locksmith — simple, but if the locksmith disappears, all locks need replacing.

```mermaid
flowchart TD
    IDP[🏛️ Central Identity Provider\ne.g. Okta / Azure Entra ID]

    IDP --> APP1[App 1\nHR System]
    IDP --> APP2[App 2\nFinance]
    IDP --> APP3[App 3\nCRM]
    IDP --> APP4[App 4\nEmail]
    IDP --> APP5[App 5\nCloud Console]

    USER([👤 User]) --> IDP

    style IDP fill:#4a90d9,color:#fff
```

**Pros:** Simple, single policy enforcement point, easy audit
**Cons:** Single point of failure, vendor lock-in, may not work across org boundaries

---

### Federated Identity
Multiple identity providers. Trust is established between them. Each organisation keeps its own IdP.

**Analogy:** Multiple countries agreeing to honour each other's passports — your UK passport gets you into France without needing a French ID card.

```mermaid
flowchart TD
    subgraph OrgA["Organisation A"]
        IDP_A[🏛️ IdP A\nActive Directory]
        USERS_A([👥 Employees A])
    end

    subgraph OrgB["Organisation B"]
        IDP_B[🏛️ IdP B\nOkta]
        USERS_B([👥 Employees B])
    end

    subgraph Broker["Identity Broker / Hub"]
        BROKER[🔄 Identity Broker\ne.g. Ping / ADFS]
    end

    IDP_A -->|SAML / OIDC trust| BROKER
    IDP_B -->|SAML / OIDC trust| BROKER

    BROKER --> APP1[Shared App 1]
    BROKER --> APP2[Shared App 2]
    BROKER --> APP3[Partner Portal]

    USERS_A --> IDP_A
    USERS_B --> IDP_B
```

**Pros:** Works across organisational boundaries, each org controls own identity, flexible
**Cons:** More complex, trust relationships must be maintained, debugging is harder

---

## 2. HR as the Authoritative Source of Identity

**Simple idea:** The HR system is the single source of truth for who exists in the organisation. Everything else follows from it.

**Analogy:** The birth certificate is the authoritative source for a person's legal identity — everything else (passport, driving licence) derives from it.

```mermaid
flowchart TD
    HR[(🗂️ HR System\nWorkday / SAP SuccessFactors\nSingle Source of Truth)]

    HR -->|New hire created| IGA[IGA Platform\ne.g. SailPoint]

    IGA -->|Creates identity| DIR[👤 Active Directory\nor Entra ID]
    IGA -->|Provisions account| EMAIL[📧 Email / M365]
    IGA -->|Grants access| APP1[Business Apps\nSalesforce, SAP, etc.]
    IGA -->|Creates vault account| PAM[🔐 PAM System\nfor privileged users]

    HR -->|Role change| IGA
    IGA -->|Updates access| DIR
    IGA -->|Removes old access| APP1
    IGA -->|Grants new access| APP2[New Role Apps]

    HR -->|Termination event| IGA
    IGA -->|Disables account| DIR
    IGA -->|Revokes all access| EMAIL
    IGA -->|Revokes all access| APP1
    IGA -->|Revokes all access| PAM

    style HR fill:#e8623a,color:#fff
    style IGA fill:#4a90d9,color:#fff
```

**Why this matters:**
- No manual "please create an account" emails
- No forgotten access when someone leaves
- Access always matches job role
- One place to update when things change

---

## 3. Identity Hub / Broker Pattern

**Problem:** Without a broker, every identity provider needs a direct integration with every service provider. With 10 IdPs and 20 apps, that's 200 integrations to build and maintain.

**Solution:** An identity broker sits in the middle — IdPs connect to the broker, apps connect to the broker. 10 + 20 = 30 integrations instead of 200.

**Analogy:** An airport hub — airlines don't fly direct between every pair of small cities. Instead they all fly to a central hub, and passengers connect from there.

```mermaid
flowchart TD
    subgraph WithoutBroker["❌ Without Broker — N × M Integrations"]
        I1[IdP 1] --> S1A[App A]
        I1 --> S1B[App B]
        I1 --> S1C[App C]
        I2[IdP 2] --> S2A[App A]
        I2 --> S2B[App B]
        I2 --> S2C[App C]
        I3[IdP 3] --> S3A[App A]
        I3 --> S3B[App B]
        I3 --> S3C[App C]
    end

    subgraph WithBroker["✅ With Broker — N + M Integrations"]
        P1[IdP 1] --> HUB[🔄 Identity\nBroker / Hub]
        P2[IdP 2] --> HUB
        P3[IdP 3] --> HUB
        HUB --> Q1[App A]
        HUB --> Q2[App B]
        HUB --> Q3[App C]
    end
```

**What an identity broker does:**
- Translates between protocols (SAML ↔ OIDC ↔ LDAP)
- Applies additional policies (step-up auth, attribute enrichment)
- Provides a single audit point for all authentications
- Enables B2B federation (partner access to internal apps)

---

## 4. Multi-Tenant IAM Design

**Simple idea:** One platform serves multiple separate customers ("tenants"), but their data, users, and policies are completely isolated from each other.

**Analogy:** An apartment building — shared walls, roof, and entrance, but each flat is fully private with its own lock.

```mermaid
flowchart TD
    subgraph PLATFORM["Shared IAM Platform Infrastructure"]
        subgraph T1["🔵 Tenant A — Acme Corp"]
            T1_DIR[User Directory A]
            T1_POL[Policy Engine A]
            T1_APP[Apps A]
        end

        subgraph T2["🟢 Tenant B — GlobalBank"]
            T2_DIR[User Directory B]
            T2_POL[Policy Engine B]
            T2_APP[Apps B]
        end

        subgraph T3["🟠 Tenant C — HealthCo"]
            T3_DIR[User Directory C]
            T3_POL[Policy Engine C]
            T3_APP[Apps C]
        end

        SHARED_INFRA[Shared Infrastructure\nAuth engine, DB, TLS, Monitoring]
    end

    T1_DIR --- SHARED_INFRA
    T2_DIR --- SHARED_INFRA
    T3_DIR --- SHARED_INFRA
```

**Key isolation requirements:**
- Separate encryption keys per tenant
- Tenant-scoped API tokens (cannot cross tenant boundaries)
- Independent policy configuration per tenant
- Separate audit logs per tenant
- Data residency controls (tenant data stays in specified region)

---

## 5. Cross-Domain Trust and Federation Design

**Simple idea:** When Company A merges with Company B, or a contractor from Agency C needs access to Company A's systems, we need a formal trust relationship.

```mermaid
sequenceDiagram
    participant U as 👤 User (Company B)
    participant IDP_B as 🏛️ IdP B (Okta)
    participant IDP_A as 🏛️ IdP A (Azure AD)
    participant APP as 🖥️ Application (Company A)

    U->>APP: Access request
    APP->>IDP_A: Not authenticated — redirect to login
    IDP_A->>U: Which organisation are you from?
    U->>IDP_A: I'm from Company B
    IDP_A->>IDP_B: Redirect to Company B's IdP (SAML/OIDC)
    IDP_B->>U: Authenticate with your Company B credentials
    U->>IDP_B: Username + MFA
    IDP_B->>IDP_A: SAML Assertion — user verified, here are attributes
    IDP_A->>IDP_A: Map Company B attributes → local roles
    IDP_A->>APP: Authorisation token — user has role X
    APP->>U: ✅ Access granted
```

**Types of cross-domain trust:**

| Type | Description | Use Case |
|---|---|---|
| B2B Federation | Two orgs trust each other's IdPs | Joint venture, partner access |
| Guest Access | Invite external user to your tenant | Azure AD B2B, Okta Inbound Federation |
| Contractor | Third-party staff access internal systems | Agency staff, outsourced teams |
| M&A Integration | Two orgs merging their identity systems | Post-acquisition access harmonisation |

---

## 6. IAM High Availability and Disaster Recovery

**Simple idea:** IAM is as critical as power and water — if authentication goes down, no one can log into anything. HA and DR design keeps it running even when hardware fails or entire data centres go offline.

**Analogy:** Hospitals have backup generators — the moment main power fails, the generator kicks in automatically. IAM HA works the same way.

```mermaid
flowchart TD
    subgraph Region1["🌍 Primary Region — Europe West"]
        subgraph AZ1["Availability Zone 1"]
            IDP1[🏛️ IdP Node 1\nActive]
            DB1[(Database\nPrimary)]
        end
        subgraph AZ2["Availability Zone 2"]
            IDP2[🏛️ IdP Node 2\nActive]
            DB2[(Database\nSynchronous\nReplica)]
        end
        LB1[⚖️ Load Balancer]
        IDP1 --> LB1
        IDP2 --> LB1
        DB1 <-->|Sync replication| DB2
    end

    subgraph Region2["🌍 DR Region — Europe North"]
        subgraph AZ3["Availability Zone 3"]
            IDP3[🏛️ IdP Node 3\nStandby / Warm]
            DB3[(Database\nAsync Replica)]
        end
        LB2[⚖️ Load Balancer\nDR]
        IDP3 --> LB2
        DB2 -->|Async replication| DB3
    end

    GLB[🌐 Global Load Balancer\nGeographic DNS routing]
    GLB --> LB1
    GLB -->|Failover only| LB2

    USERS([👥 All Users]) --> GLB

    FAIL{Primary Region\nFails?}
    FAIL -->|Yes| GLB
    LB1 -->|Healthy| FAIL
```

**HA / DR targets:**

| Metric | Definition | Typical IAM Target |
|---|---|---|
| **RTO** | Recovery Time Objective — how fast can we restore? | < 15 minutes |
| **RPO** | Recovery Point Objective — how much data can we lose? | < 1 minute (sync replication) |
| **Uptime SLA** | Availability percentage | 99.99% (52 min downtime/year) |
| **Failover mode** | Automatic or manual? | Automatic for regional failover |

---

## Putting It All Together — Reference Architecture

```mermaid
flowchart TD
    HR[(HR System\nAuthoritative Source)] --> IGA[IGA Platform\nSailPoint / Saviynt]

    IGA -->|Provisions| DIR[Directory\nActive Directory / LDAP]
    DIR --> IDP[Identity Provider\nOkta / Azure Entra ID]

    subgraph FED["Federation Layer"]
        BROKER[Identity Broker\nCross-domain trust]
        IDP <--> BROKER
    end

    IDP --> SSO[SSO / Token issuance]
    SSO --> APP1[Internal Apps]
    SSO --> APP2[Cloud Apps\nAWS / Azure / GCP]
    SSO --> APP3[Partner Apps]

    IGA -->|Privileged accounts| PAM[PAM Platform\nCyberArk / Delinea]
    PAM --> PRIV[Privileged Systems\nServers / DBs / Network]

    subgraph HA["HA / DR Infrastructure"]
        IDP
        DIR
    end

    AUDIT[(📋 Central Audit Log\nSIEM / SIEM)] 
    SSO --> AUDIT
    PAM --> AUDIT
    IGA --> AUDIT
```

---

## Quick Reference Cheat Sheet

| Pattern | When to Use | Key Trade-off |
|---|---|---|
| **Centralised IAM** | Single org, one cloud, simple setup | Easy to manage vs single point of failure |
| **Federated IAM** | Multiple orgs, B2B, M&A | Flexible vs complex trust maintenance |
| **HR as authoritative source** | Always — for employee identity | Automation vs dependency on HR data quality |
| **Identity broker** | Many IdPs + many apps | Reduces integrations vs adds a hop in auth flow |
| **Multi-tenant** | SaaS platforms, MSPs | Cost efficient vs isolation complexity |
| **Cross-domain trust** | Partners, contractors, M&A | Enables collaboration vs extends attack surface |
| **HA/DR** | Always — IAM is critical infra | Resilience vs cost and complexity |
