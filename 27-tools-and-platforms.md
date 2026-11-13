# IAM Tools and Platforms

[← Back to Index](README.md)

> A practical guide to the IAM ecosystem — what each category of tool does, what the major players are, and how they fit together.

---

## The IAM Tool Landscape

No single tool does everything. IAM is an ecosystem of specialised products that work together. Understanding which tool belongs to which category — and why — is key to making good architecture decisions.

**Analogy:** Think of IAM tools like a hospital — you have doctors (identity providers), administrators (IGA), security guards (PAM), receptionists for walk-ins (CIAM), a pharmacy (secrets management), and medical records (directory). Each plays a distinct role.

---

## 1. The IAM Ecosystem Map

```mermaid
flowchart TD
    subgraph DIR["🗂️ Directory Services\nSource of Identity"]
        AD[Active Directory]
        LDAP[OpenLDAP]
        ENTRA[Azure Entra ID]
    end

    subgraph IGA_BOX["📋 Identity Governance\nWho should have what access?"]
        SP[SailPoint]
        SAV[Saviynt]
        OI[One Identity]
    end

    subgraph SSO_BOX["🔑 SSO / Enterprise IAM\nAuthenticate and authorise"]
        OKTA[Okta]
        PING[Ping Identity]
        ENTRA2[Azure Entra ID]
        FF[ForgeRock]
    end

    subgraph PAM_BOX["🔐 PAM\nProtect privileged accounts"]
        CA[CyberArk]
        DEL[Delinea]
        BT[BeyondTrust]
    end

    subgraph CIAM_BOX["👥 CIAM\nCustomer-facing identity"]
        AUTH0[Auth0]
        COG[AWS Cognito]
        FB[Firebase Auth]
    end

    subgraph SEC_BOX["🔒 Secrets Management\nStore credentials and keys"]
        VAULT[HashiCorp Vault]
        AWSSM[AWS Secrets Manager]
        AKV[Azure Key Vault]
    end

    subgraph CLOUD_BOX["☁️ Cloud IAM\nCloud resource access"]
        AWSIAM[AWS IAM]
        AZRBAC[Azure RBAC]
        GCPIAM[GCP IAM]
    end

    subgraph PKI_BOX["📜 PKI / Certificate Management"]
        EJBCA[EJBCA]
        ADCS[Microsoft ADCS]
        VEN[Venafi]
    end

    DIR -->|Feeds identities to| IGA_BOX
    DIR -->|Authenticates via| SSO_BOX
    IGA_BOX -->|Governs access in| SSO_BOX
    IGA_BOX -->|Governs access in| CLOUD_BOX
    SSO_BOX -->|Grants tokens to| CLOUD_BOX
    SSO_BOX -->|Manages| PAM_BOX
    PKI_BOX -->|Issues certs to| SSO_BOX
    PKI_BOX -->|Issues certs to| SEC_BOX
```

---

## 2. Typical Enterprise IAM Stack (Layers)

```mermaid
flowchart TD
    subgraph L0["Foundation Layer — Directory"]
        AD2[Active Directory / Azure Entra ID\nStores all identities and groups]
    end

    subgraph L1["Governance Layer — IGA"]
        IGA2[SailPoint / Saviynt\nAccess requests, reviews, provisioning policies]
    end

    subgraph L2["Authentication Layer — SSO / IdP"]
        IDP2[Okta / Ping / Entra\nAuthentication, MFA, SSO tokens]
    end

    subgraph L3["Privileged Layer — PAM"]
        PAM2[CyberArk / Delinea\nVault credentials, session recording, JIT access]
    end

    subgraph L4["Customer Layer — CIAM"]
        CIAM2[Auth0 / Cognito\nCustomer registration, social login, consent]
    end

    subgraph L5["Cloud Layer — Cloud IAM"]
        CLOUD2[AWS IAM / Azure RBAC / GCP IAM\nCloud resource permissions]
    end

    L0 --> L1 --> L2 --> L3
    L2 --> L4
    L2 --> L5
```

---

## 3. Category Deep-Dives

### Enterprise IAM / SSO

These are the central identity platforms — the "front door" for authentication across all enterprise apps.

| Tool | Best For | Deployment | Strength |
|---|---|---|---|
| **Okta** | Cloud-first, modern workforce | SaaS only | Largest app integration catalogue |
| **Azure Entra ID** | Microsoft-heavy orgs | Cloud (+ hybrid) | Native M365 integration, Conditional Access |
| **Ping Identity** | Large enterprise, complex federation | On-prem or cloud | Protocol flexibility, fine-grained policy |
| **ForgeRock** | Custom identity platforms, complex workflows | On-prem or cloud | Highly customisable, good for CIAM + workforce |

```mermaid
quadrantChart
    title Enterprise IAM — Cloud-Native vs On-Prem Strength
    x-axis On-Prem Strength --> Cloud-Native Strength
    y-axis Simple Deployment --> Complex Enterprise Features
    quadrant-1 Complex + Cloud-Native
    quadrant-2 Complex + On-Prem
    quadrant-3 Simple + On-Prem
    quadrant-4 Simple + Cloud-Native
    Okta: [0.90, 0.65]
    Azure Entra ID: [0.80, 0.70]
    Ping Identity: [0.55, 0.85]
    ForgeRock: [0.50, 0.90]
```

---

### Identity Governance and Administration (IGA)

IGA tools answer: "Who has access to what, should they have it, and how did they get it?"

```mermaid
flowchart LR
    HR2[(HR System)] -->|Joiner event| IGA3[IGA Platform\nSailPoint / Saviynt]
    IGA3 -->|Auto-provision| DIR2[Directory]
    IGA3 -->|Auto-provision| EMAIL2[Email]
    IGA3 -->|Auto-provision| APPS2[Business Apps]

    MGR([Manager]) -->|Access request| IGA3
    IGA3 -->|Approval workflow| APP_OWN([App Owner])
    APP_OWN -->|Approves| IGA3
    IGA3 -->|Grants access| APPS2

    IGA3 -->|Quarterly campaign| CERT[Access Certification\nCampaign]
    CERT -->|Review tasks sent to| MGR
    MGR -->|Certify or Revoke| CERT
    CERT -->|Revoke actions| APPS2
```

| Tool | Strength | Best For |
|---|---|---|
| **SailPoint** | Deep lifecycle automation, compliance reporting | Large enterprises, heavily regulated industries |
| **Saviynt** | Cloud-native IGA, good cloud app connectors | Cloud-first orgs, SaaS-heavy environments |
| **One Identity** | Strong AD governance | Microsoft-centric environments |

---

### Privileged Access Management (PAM)

PAM tools protect the most dangerous accounts — admin accounts, root access, service accounts — which attackers target most.

```mermaid
flowchart TD
    ADMIN([👤 Admin User]) --> PAM3[PAM Portal\nCyberArk / Delinea]
    PAM3 -->|Check out credential| VAULT[Password Vault\nRotates after each use]
    VAULT -->|Time-limited password| ADMIN
    ADMIN -->|Uses credential| TARGET[🖥️ Target Server\nor Database]
    TARGET -->|Session recorded| REC[(Session Recording)]
    VAULT -->|Auto-rotates password| VAULT
    PAM3 -->|Alerts on anomalies| SOC([Security Operations])
```

| Tool | Strength | Differentiator |
|---|---|---|
| **CyberArk** | Market leader, most features | Largest enterprise deployments, advanced threat analytics |
| **Delinea** (ex-Thycotic + Centrify) | Easier deployment | Good for mid-market, cloud-ready |
| **BeyondTrust** | Endpoint privilege management | Strong least-privilege for endpoints/workstations |

---

### CIAM — Customer Identity

Customer-facing identity is different from workforce identity. Scale is higher, UX matters more, and social login is expected.

```mermaid
flowchart LR
    CUST([👤 Customer]) --> CIAM3[CIAM Platform\nAuth0 / Cognito]
    CIAM3 -->|Social login| SOCIAL[Google / Apple\nFacebook / GitHub]
    CIAM3 -->|Password login| EMAIL3[Email + Password]
    CIAM3 -->|MFA option| MFA3[SMS / TOTP]
    CIAM3 -->|Issues token| APP3[Customer App\nor API]
    CIAM3 -->|Consent management| CONSENT[(GDPR Consent\nStore)]
    CIAM3 -->|Risk scoring| FRAUD[Fraud detection]
```

| Tool | Strength | Best For |
|---|---|---|
| **Auth0** (Okta) | Developer-friendly, flexible | SaaS products, developer teams |
| **AWS Cognito** | Native AWS integration, low cost at scale | AWS-native apps |
| **Firebase Auth** | Easy to set up, Google ecosystem | Mobile apps, startups |

---

### Secrets Management

Secrets are passwords, API keys, certificates, and tokens that applications need to function. Without a secrets manager, they end up hardcoded in source code — a major security risk.

```mermaid
flowchart TD
    DEV([Developer]) -->|Hardcoded secret ❌| CODE[Source Code\nGit Repository]
    CODE -->|Leaked if repo is public| BREACH([Data Breach])

    APP([Application]) -->|Request secret at runtime ✅| SM[Secrets Manager\nHashiCorp Vault\nAWS Secrets Manager\nAzure Key Vault]
    SM -->|Returns secret\nwith TTL| APP
    SM -->|Auto-rotates secret| SM
    SM -->|Audit log every access| AUDIT2[(Audit Log)]
```

| Tool | Strength | Best For |
|---|---|---|
| **HashiCorp Vault** | Most flexible, supports any secret type | Multi-cloud, hybrid, complex orgs |
| **AWS Secrets Manager** | Native AWS integration, auto-rotation | AWS-native workloads |
| **Azure Key Vault** | Native Azure integration, HSM-backed | Azure workloads, certificate management |

---

### Cloud IAM

Each cloud has its own native IAM system for controlling who can do what with cloud resources.

```mermaid
flowchart TD
    subgraph AWS["☁️ AWS IAM"]
        AWSU[IAM User / Role] -->|Attached| AWSP[IAM Policy\nJSON — Allow / Deny]
        AWSP --> AWSR[AWS Resource\nS3, EC2, RDS...]
    end

    subgraph AZURE["☁️ Azure RBAC"]
        AZID[User / Service Principal] -->|Assigned| AZR[Role\nOwner / Contributor / Reader]
        AZR -->|Scoped to| AZS[Scope\nSubscription / RG / Resource]
    end

    subgraph GCP["☁️ GCP IAM"]
        GCPU[Member\nUser / SA / Group] -->|Bound to| GCPR[Role\nPrimitive / Predefined / Custom]
        GCPR -->|On| GCPS[Resource\nProject / Bucket / VM]
    end
```

---

## 4. How Tools Connect in a Typical Enterprise

```mermaid
flowchart TD
    HR3[(HR System\nWorkday)] -->|Joiner/Mover/Leaver events| SP2[SailPoint IGA]
    SP2 -->|Provisions accounts| AD3[Active Directory]
    SP2 -->|Grants app access| OKTA2[Okta SSO]
    AD3 -->|Syncs to| OKTA2

    OKTA2 -->|SAML/OIDC| APP_A[App A — Salesforce]
    OKTA2 -->|SAML/OIDC| APP_B[App B — ServiceNow]
    OKTA2 -->|SAML/OIDC| APP_C[App C — AWS Console]

    SP2 -->|Privileged account lifecycle| CA2[CyberArk PAM]
    CA2 -->|Vaulted credentials| SERVERS[Servers / DBs]

    APP_C --> AWSIAM2[AWS IAM Roles\nAssumed via SAML federation]

    OKTA2 -->|All auth events| SIEM[SIEM\nSplunk / Sentinel]
    CA2 -->|Session recordings + logs| SIEM
    SP2 -->|Governance events| SIEM
```

---

## 5. Choosing the Right Tool

```mermaid
flowchart TD
    START([What is your use case?]) --> Q1{Who are\nyour users?}

    Q1 -->|Employees| Q2{Cloud-first\nor hybrid?}
    Q1 -->|Customers| CIAM4[CIAM\nAuth0 / Cognito / Firebase]
    Q1 -->|Privileged admins| PAM4[PAM\nCyberArk / Delinea]
    Q1 -->|Applications / services| SEC4[Secrets Manager\nVault / AWS SM / AKV]

    Q2 -->|Cloud-first| Q3{Microsoft\necosystem?}
    Q2 -->|Hybrid on-prem + cloud| Q4{Large enterprise\nwith compliance needs?}

    Q3 -->|Yes| ENT[Azure Entra ID]
    Q3 -->|No| OKT[Okta]

    Q4 -->|Yes| PING2[Ping Identity\nor ForgeRock]
    Q4 -->|No| OKT2[Okta or\nAzure Entra ID]

    Q1 -->|Need governance layer| IGA4{Heavily\nregulated?}
    IGA4 -->|Yes| SP3[SailPoint]
    IGA4 -->|Cloud-first| SAV2[Saviynt]
    IGA4 -->|Microsoft-focused| OI2[One Identity]
```

---

## Quick Reference Cheat Sheet

| Category | What it does | Leading Tools |
|---|---|---|
| **Enterprise SSO / IAM** | Authenticate employees, enforce MFA, enable SSO | Okta, Azure Entra ID, Ping, ForgeRock |
| **IGA** | Govern who has access, run access reviews, automate provisioning | SailPoint, Saviynt, One Identity |
| **PAM** | Protect privileged accounts, vault credentials, record sessions | CyberArk, Delinea, BeyondTrust |
| **CIAM** | Customer registration, social login, consent management | Auth0, AWS Cognito, Firebase Auth |
| **Secrets Management** | Store/rotate app secrets, prevent hardcoded credentials | HashiCorp Vault, AWS Secrets Manager, Azure Key Vault |
| **Cloud IAM** | Control access to cloud resources natively | AWS IAM, Azure RBAC, GCP IAM |
| **Directory** | Store and manage identity objects | Active Directory, OpenLDAP, Azure Entra ID |
| **PKI** | Issue and manage X.509 certificates | EJBCA, Microsoft ADCS, Venafi |
