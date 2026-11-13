# Directory Services

[← Back to Index](README.md)

> A directory service is the phonebook of your IT environment — it stores who exists, what groups they belong to, and what they're allowed to do.

---

## What is a Directory Service?

Imagine your company has 5,000 employees. Every app — email, HR system, file server, VPN — needs to know who those people are, what their role is, and whether they're still employed.

A **directory service** is the central store for all that identity data. Apps ask the directory instead of maintaining their own user database.

```mermaid
flowchart LR
    DS[(🗂️ Directory Service\nSingle Source of Truth)]

    DS --> A1[Email Server]
    DS --> A2[VPN]
    DS --> A3[HR Portal]
    DS --> A4[File Shares]
    DS --> A5[Wi-Fi / 802.1X]

    HR[👥 HR System] -->|Authoritative identity data| DS
```

---

## 1. Active Directory (AD)

**Active Directory Domain Services (AD DS)** is Microsoft's on-premises directory service. It organises users and computers into a hierarchy.

> Think of AD like a **company org chart**, but for IT. At the top is the company (forest), then departments (domains), then teams (OUs).

### AD Structure

```mermaid
flowchart TD
    FOREST[🌲 Forest: contoso.com\nTop-level security boundary]

    FOREST --> TREE1[🌳 Domain Tree: contoso.com]
    FOREST --> TREE2[🌳 Domain Tree: fabrikam.com\nSeparate acquired company]

    TREE1 --> DOM1[🏢 Domain: contoso.com\nRoot domain]
    TREE1 --> DOM2[🏢 Domain: eu.contoso.com\nChild domain — Europe]
    TREE1 --> DOM3[🏢 Domain: us.contoso.com\nChild domain — USA]

    DOM1 --> OU1[📁 OU: Finance]
    DOM1 --> OU2[📁 OU: Engineering]
    DOM1 --> OU3[📁 OU: HR]

    OU1 --> U1([👤 Alice])
    OU1 --> U2([👤 Bob])
    OU2 --> U3([👤 Carol])
    OU1 --> GPO1[🔧 GPO: Finance Security Policy]
```

### AD Replication Between Sites

Companies have offices in multiple cities. Each site has its own **Domain Controller (DC)**. Changes replicate between them.

```mermaid
flowchart LR
    subgraph London["🏢 London Site"]
        DC_LON[Domain Controller\nLondon]
    end

    subgraph NYC["🏢 New York Site"]
        DC_NYC[Domain Controller\nNew York]
    end

    subgraph SG["🏢 Singapore Site"]
        DC_SG[Domain Controller\nSingapore]
    end

    DC_LON <-->|Replication\nvia Site Link| DC_NYC
    DC_NYC <-->|Replication\nvia Site Link| DC_SG
    DC_LON <-->|Replication\nvia Site Link| DC_SG

    note1[/"AD replicates: users, groups, GPOs, passwords\nScheduled intervals or immediate for urgent changes\n(e.g. password change, account lockout)"/]
```

---

## 2. Azure Entra ID (formerly Azure AD)

**Azure Entra ID** is Microsoft's cloud-based identity platform. It is NOT just AD in the cloud — it uses different protocols (OIDC/OAuth instead of Kerberos/LDAP) and has different concepts.

> Think of Entra ID as the **cloud evolution** of Active Directory — designed for internet-scale, SaaS apps, and remote workers.

### Entra ID Tenant Structure

```mermaid
flowchart TD
    TENANT[☁️ Entra ID Tenant\ncontoso.onmicrosoft.com\nTop-level security boundary]

    TENANT --> USERS[👥 Users]
    TENANT --> GROUPS[👨‍👩‍👦 Groups]
    TENANT --> APPS[📱 Enterprise Applications\nSaaS registrations]
    TENANT --> DEVICES[💻 Devices\nAzure AD joined / registered]
    TENANT --> ROLES[🔑 Directory Roles\nGlobal Admin, etc.]

    USERS --> U1([Alice — Member])
    USERS --> U2([Bob@partner.com — Guest])

    APPS --> APP1[Salesforce]
    APPS --> APP2[Slack]
    APPS --> APP3[Custom Internal App]
```

### Azure AD B2B — External Partner Access

B2B (Business-to-Business) lets partners use their own identity to access your resources — no need to create accounts for them.

```mermaid
flowchart LR
    subgraph PartnerOrg["🏢 Partner Org\npartner.com"]
        EXT([👤 Carol\ncarol@partner.com])
        EXT_IDP[Partner IdP\nEntra ID or other]
    end

    subgraph YourOrg["🏢 Your Org\ncontoso.com"]
        INVITE[📧 Admin sends\nguest invitation]
        GUEST[Guest Account\ncarol@partner.com#EXT#]
        APP[SharePoint / Teams\nInternal App]
    end

    INVITE -->|Invitation email| EXT
    EXT -->|Accepts invite| GUEST
    EXT -->|Authenticates at| EXT_IDP
    EXT_IDP -->|Token passed to| APP
    GUEST -->|Access governed by| APP
```

### Azure AD B2C — Customer Identity

B2C (Business-to-Consumer) is for external customers — think millions of users registering on your website.

```mermaid
flowchart TD
    U([🛒 Customer]) --> CHOICE{How do you\nwant to sign in?}

    CHOICE -->|Email + Password| LOCALACCOUNT[Local Account\nin B2C tenant]
    CHOICE -->|Google| GOOGLE[Google\nSocial Identity Provider]
    CHOICE -->|Facebook| FB[Facebook\nSocial Identity Provider]
    CHOICE -->|Apple| APPLE[Apple\nSocial Identity Provider]

    LOCALACCOUNT --> B2C[Azure AD B2C\nCustom Policies\n/ User Flows]
    GOOGLE --> B2C
    FB --> B2C
    APPLE --> B2C

    B2C -->|Issues token| YOURAPP[Your App /\nAPI]
```

### Hybrid Identity — On-Prem AD + Azure Entra ID

Most companies start with on-prem AD and gradually move to cloud. **Azure AD Connect** bridges the two.

```mermaid
flowchart LR
    subgraph OnPrem["🏢 On-Premises"]
        AD[Active Directory\ncontoso.com]
        DC[Domain Controllers]
        USERS_OP[On-prem Users\nGroups, Devices]
    end

    subgraph Sync["🔄 Azure AD Connect\nSync Engine"]
        CONNECT[AD Connect\nSync Service]
        PHS[Password Hash Sync\nor Pass-through Auth\nor Federation]
    end

    subgraph Cloud["☁️ Azure Cloud"]
        ENTRA[Azure Entra ID\ncontoso.onmicrosoft.com]
        M365[Microsoft 365]
        SAAS[SaaS Apps\nSalesforce, Slack...]
    end

    AD --> CONNECT
    CONNECT -->|Syncs identities| ENTRA
    PHS -->|Auth method| ENTRA
    ENTRA --> M365
    ENTRA --> SAAS
```

---

## 3. OpenLDAP — LDAP Directory

**LDAP (Lightweight Directory Access Protocol)** is the open standard that Active Directory is also built on. OpenLDAP is the open-source implementation.

> LDAP is the **language** directories speak. AD is a directory that speaks LDAP (among others).

### LDAP Directory Information Tree (DIT)

```mermaid
flowchart TD
    ROOT["dc=contoso,dc=com\n(Root)"]

    ROOT --> OU_USERS["ou=Users"]
    ROOT --> OU_GROUPS["ou=Groups"]
    ROOT --> OU_SERVICES["ou=ServiceAccounts"]

    OU_USERS --> U1["cn=Alice Smith\nuid=asmith\nmail=alice@contoso.com\nuserPassword=..."]
    OU_USERS --> U2["cn=Bob Jones\nuid=bjones\nmail=bob@contoso.com"]

    OU_GROUPS --> G1["cn=Finance\nmember=uid=asmith,..."]
    OU_GROUPS --> G2["cn=Engineering\nmember=uid=bjones,..."]

    OU_SERVICES --> SVC1["cn=appservice\nuid=appsvc"]
```

> Every entry in LDAP has a **Distinguished Name (DN)** — a unique path.
> Example: `cn=Alice Smith,ou=Users,dc=contoso,dc=com`

### How an App Authenticates Against LDAP

```mermaid
sequenceDiagram
    participant App
    participant LDAP as LDAP Server

    App->>LDAP: Bind (service account credentials)
    LDAP-->>App: Bind success

    App->>LDAP: Search: find user where uid=asmith
    LDAP-->>App: Return DN: cn=Alice,ou=Users,dc=contoso,dc=com

    App->>LDAP: Bind as Alice (DN + password she typed)
    LDAP-->>App: Bind success = password correct ✅

    App->>LDAP: Search: get Alice's group memberships
    LDAP-->>App: memberOf: Finance, AllStaff

    App-->>App: Grant access based on groups
```

---

## 4. Cloud Directory Services

```mermaid
flowchart TD
    subgraph AWS["☁️ AWS"]
        AWSDS[AWS Directory Service]
        AWSDS --> MAD[AWS Managed\nMicrosoft AD]
        AWSDS --> ADSIMPLE[AD Connector\nProxy to on-prem AD]
        AWSDS --> SIMAD[Simple AD\nSamba-based]
    end

    subgraph GCP["☁️ Google Cloud"]
        GCPDIR[Cloud Identity\nGoogle Workspace Directory]
        GCPDIR --> GCPIDP[Identity Platform\nFirebase Auth / CIAM]
    end

    subgraph USE["Common Use Cases"]
        EC2[EC2 Instances\njoined to domain]
        SAML2[SAML federation\nfor GCP resources]
        GCPCIAM[Customer identity\nfor GCP apps]
    end

    MAD --> EC2
    GCPDIR --> SAML2
    GCPIDP --> GCPCIAM
```

---

## Directory Services Comparison

| Feature | On-Prem AD DS | Azure Entra ID | OpenLDAP |
|---|---|---|---|
| **Protocol** | Kerberos, LDAP, NTLM | OIDC, OAuth 2.0, SAML | LDAP, LDAPS |
| **Location** | On-premises | Cloud (SaaS) | On-prem or cloud |
| **Management** | Group Policy (GPOs) | Conditional Access, Intune | Manual / overlays |
| **Device join** | Domain join | Azure AD join / Hybrid | N/A |
| **Best for** | Traditional corp IT | Cloud-first / hybrid | Linux, open-source apps |
| **MFA** | Via ADFS / NPS | Built-in | Via RADIUS/LDAP proxy |
| **Cost** | Windows Server license | Per-user subscription | Free / open-source |

---

## Quick Reference Cheat Sheet

| Concept | One-liner | Key Rule |
|---|---|---|
| **AD Forest** | Top-level security boundary | Cross-forest trust = explicit configuration needed |
| **AD Domain** | Replication and policy boundary | GPOs apply at domain/OU level |
| **OU** | Organisational container | Used to scope GPOs and delegate admin |
| **DC** | Domain Controller | Holds a copy of AD, handles authentication |
| **Entra ID** | Cloud identity platform | OIDC/OAuth, not Kerberos/LDAP |
| **B2B** | Guest access for partners | They authenticate at their own IdP |
| **B2C** | Consumer identity at scale | Supports social login, millions of users |
| **Hybrid Join** | On-prem AD + cloud Entra ID | Azure AD Connect syncs identities |
| **LDAP DN** | Unique path to a directory entry | `cn=Alice,ou=Users,dc=contoso,dc=com` |
| **OpenLDAP** | Open-source LDAP server | Protocol-compatible with AD for simple lookups |
