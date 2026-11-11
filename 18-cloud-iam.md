# Cloud IAM

> Cloud IAM controls who (or what) can do what to which cloud resources. Each major cloud provider has its own model, but the core idea is the same: attach policies to identities, then those identities can act on resources.

---

## 1. Why Cloud IAM is Different

**Traditional IAM:** You manage users in Active Directory inside your data centre.
**Cloud IAM:** Everything — servers, databases, functions, storage — is an API call. Every API call needs an authorised identity.

> Think of cloud IAM as the bouncer for every door in a skyscraper that has millions of doors, all accessible from the internet.

---

## 2. AWS IAM

### Core Building Blocks

```mermaid
flowchart TD
    ACCOUNT[AWS Account] --> USERS[IAM Users\nlong-term credentials]
    ACCOUNT --> GROUPS[IAM Groups\ncontainers for users]
    ACCOUNT --> ROLES[IAM Roles\ntemporary credentials, assumed by anyone]
    ACCOUNT --> POLICIES[IAM Policies\nJSON documents defining Allow/Deny]

    GROUPS --> POLICIES
    USERS --> POLICIES
    ROLES --> POLICIES

    POLICIES --> RESOURCES[AWS Resources\nS3, EC2, RDS, Lambda...]
```

> **Key rule:** Users and roles do nothing on their own. Policies define what they can do.

### Identity Policy vs Resource Policy

| Policy Type | Attached To | Example |
|---|---|---|
| **Identity Policy** | User / Group / Role | "This role can read S3 bucket X" |
| **Resource Policy** | The resource itself | "This S3 bucket can be read by role Y" |
| **SCP** | AWS Organization OU | "No account in this OU can disable CloudTrail" |

### Role Assumption — How AWS Services Get Credentials

```mermaid
sequenceDiagram
    participant EC2 as EC2 Instance
    participant META as Instance Metadata Service
    participant STS as AWS STS
    participant S3 as S3 Bucket

    EC2->>META: GET /latest/meta-data/iam/security-credentials/
    META-->>EC2: Returns role name attached to instance
    EC2->>STS: AssumeRole (via instance profile)
    STS-->>EC2: Temporary credentials (Access Key + Secret + Token, ~1hr TTL)
    EC2->>S3: s3:GetObject with temporary credentials
    S3-->>EC2: Returns object
```

> **Best practice:** Never use long-term IAM user access keys on EC2. Always use IAM roles with temporary credentials.

### AWS Organizations and Service Control Policies (SCPs)

```mermaid
flowchart TD
    MGMT[Management Account] --> ROOT[Root]
    ROOT --> OU1[OU: Production]
    ROOT --> OU2[OU: Development]
    ROOT --> OU3[OU: Security]

    OU1 --> ACC1[Account: Prod-App]
    OU1 --> ACC2[Account: Prod-DB]
    OU2 --> ACC3[Account: Dev-App]
    OU3 --> ACC4[Account: Security-Tools]

    SCP1[SCP: Deny disable CloudTrail] -->|Applies to all| ROOT
    SCP2[SCP: Restrict regions to eu-west-1] -->|Applies to| OU1
    SCP3[SCP: Allow dev instance types only] -->|Applies to| OU2

    style SCP1 fill:#e74c3c,color:#fff
    style SCP2 fill:#e74c3c,color:#fff
    style SCP3 fill:#e74c3c,color:#fff
```

> **SCPs are guardrails, not grants.** An SCP can only restrict; it cannot grant access. Access is the intersection of the SCP and the identity policy.

### Permission Boundaries

```mermaid
flowchart LR
    subgraph WITHOUT["Without Permission Boundary"]
        ADMIN([Admin user]) -->|Creates role| NEWROLE[New role with AdministratorAccess]
        NEWROLE --> ANYTHING[Can do anything ❌]
    end

    subgraph WITH["With Permission Boundary"]
        ADMIN2([Admin user]) -->|Creates role| NEWROLE2[New role with AdministratorAccess]
        NEWROLE2 -->|But bounded by| PB[Permission Boundary:\nonly S3 and DynamoDB]
        PB --> LIMITED[Can only access S3 + DynamoDB ✅]
    end
```

---

## 3. Azure RBAC and Entra ID

### Azure Management Hierarchy

```mermaid
flowchart TD
    TENANT[Entra ID Tenant\nIdentity boundary] --> MG_ROOT[Root Management Group]
    MG_ROOT --> MG1[Management Group: Corp]
    MG_ROOT --> MG2[Management Group: Sandbox]

    MG1 --> SUB1[Subscription: Prod]
    MG1 --> SUB2[Subscription: Non-Prod]
    MG2 --> SUB3[Subscription: Dev]

    SUB1 --> RG1[Resource Group: web-rg]
    SUB1 --> RG2[Resource Group: db-rg]

    RG1 --> RES1[VM: web-01]
    RG1 --> RES2[App Service: api]
    RG2 --> RES3[SQL Server: prod-db]
```

> **RBAC roles are inherited downward.** A role assigned at Management Group level applies to all subscriptions, resource groups, and resources below it.

### Azure Built-in RBAC Roles

| Role | Can Read | Can Write | Can Manage Access |
|---|---|---|---|
| **Owner** | ✅ | ✅ | ✅ |
| **Contributor** | ✅ | ✅ | ❌ |
| **Reader** | ✅ | ❌ | ❌ |
| **User Access Administrator** | ✅ | ❌ | ✅ |

### Azure PIM — Privileged Identity Management

```mermaid
flowchart TD
    USER([👤 User]) -->|Needs admin access| PORTAL[Opens PIM portal]
    PORTAL --> REQUEST[Requests elevation\nspecify reason + duration]

    REQUEST --> APPROVE{Approval required?}
    APPROVE -->|Auto-approve| ACTIVATE[Role activated immediately]
    APPROVE -->|Needs approval| MGRAPPROVE([Manager / Admin approves])
    MGRAPPROVE --> ACTIVATE

    ACTIVATE --> TIMER[⏱️ Time-limited admin role\ne.g., 2 hours]
    TIMER --> WORK[User performs privileged task]
    TIMER -->|Time expires| EXPIRE[Role automatically deactivated]

    WORK & EXPIRE --> AUDIT[(Azure Audit Log\nwho, what, when, why)]
```

---

## 4. GCP IAM

### GCP Resource Hierarchy

```mermaid
flowchart TD
    ORG[Organisation\ne.g., company.com] --> FOLDER1[Folder: Engineering]
    ORG --> FOLDER2[Folder: Finance]

    FOLDER1 --> PROJ1[Project: backend-prod]
    FOLDER1 --> PROJ2[Project: backend-dev]
    FOLDER2 --> PROJ3[Project: finance-tools]

    PROJ1 --> RES1[GCS Bucket]
    PROJ1 --> RES2[GKE Cluster]
    PROJ1 --> RES3[BigQuery Dataset]
```

### GCP IAM Concepts

| Concept | Meaning |
|---|---|
| **Principal** | Who — a user, service account, or group |
| **Role** | What — a collection of permissions |
| **Policy** | Binds a principal to a role on a resource |

> GCP IAM uses **allow policies** (additive). There are no explicit deny policies in standard GCP IAM — the most permissive binding wins (but Org Policies can restrict).

---

## 5. Workload Identity Federation

**Simple idea:** Instead of creating long-lived service account keys, let external workloads (GitHub Actions, on-prem servers, another cloud) exchange a short-lived identity token for cloud access.

> Like a temporary visitor pass: show your company ID, get a timed badge, return it when done. No permanent key card issued.

```mermaid
flowchart TD
    WL[External Workload\ne.g. GitHub Actions runner] -->|Has OIDC token issued by GitHub| TOKEN[Short-lived JWT token]

    TOKEN -->|Exchange at| FED[Cloud Workload Identity\nFederation endpoint]

    FED --> VERIFY{Verify token\nsignature + claims}
    VERIFY -->|Valid| TEMP[Issue temporary cloud credentials\n~1hr TTL]
    VERIFY -->|Invalid| DENY[❌ Denied]

    TEMP --> ACCESS[Access cloud resources\nS3 / GCS / Azure Storage]

    style TOKEN fill:#27ae60,color:#fff
    style TEMP fill:#27ae60,color:#fff
```

### Why this matters: No more long-lived keys

```mermaid
flowchart LR
    subgraph OLD["❌ Old Way — Long-lived Keys"]
        KEY[Service Account Key file\nin GitHub Secrets] -->|Stored forever| RISK[If leaked: permanent access\nuntil manually rotated]
    end

    subgraph NEW["✅ New Way — Workload Identity Federation"]
        OIDC[GitHub OIDC token\nlasts minutes] -->|Exchange for| TEMP2[Temp cloud credentials\n1hr TTL, auto-expire]
        TEMP2 --> SAFE[No secret to leak or rotate ✅]
    end
```

---

## 6. IMDS Credential Exposure Risk

**Simple idea:** Cloud VMs automatically get IAM credentials via the Instance Metadata Service (IMDS). If an attacker runs code on your VM (e.g., via SSRF), they can steal those credentials.

```mermaid
flowchart TD
    VM[EC2 / GCE / Azure VM] --> APP[Web Application running on VM]
    APP --> SSRF{⚠️ SSRF Vulnerability\nin app code}

    ATTACKER([🏴‍☠️ Attacker]) -->|Crafted request to trigger SSRF| APP
    SSRF -->|App fetches| IMDS[Instance Metadata Service\nhttp://169.254.169.254/]
    IMDS -->|Returns| CREDS[IAM Role Credentials\nAccessKeyId + SecretKey + Token]
    CREDS -->|Stolen by attacker| ATTACKER

    ATTACKER -->|Uses stolen creds| CLOUD[Cloud API calls\nunder VM's IAM role]
```

### Mitigations

```mermaid
flowchart LR
    M1[Enable IMDSv2\nRequires session token — SSRF can't get it] --> SAFE[✅ Protected]
    M2[Least-privilege IAM role\nVM only has what it needs] --> SAFE
    M3[Block outbound 169.254.169.254\nat app/network level] --> SAFE
    M4[No IMDSv1 on new instances\nenforce via SCP/Policy] --> SAFE
```

---

## 7. Cross-Cloud Identity — Key Patterns

```mermaid
flowchart TD
    subgraph PATTERN1["Pattern 1: Workload Identity Federation"]
        P1_WL[Workload on Cloud A] -->|OIDC token| P1_FED[Cloud B Federation]
        P1_FED --> P1_ACCESS[Access Cloud B resources]
    end

    subgraph PATTERN2["Pattern 2: Centralized IdP Federation"]
        P2_IDP[Corporate Entra ID] -->|SAML/OIDC federation| P2_AWS[AWS IAM Identity Center]
        P2_IDP -->|SAML/OIDC federation| P2_GCP[GCP Workforce Identity]
        P2_USER([Employee]) --> P2_IDP
        P2_IDP --> P2_USER
    end
```

---

## Quick Reference Cheat Sheet

| Concept | AWS | Azure | GCP |
|---|---|---|---|
| **Identity units** | Users, Roles, Groups | Users, Service Principals, Managed Identities | Users, Service Accounts, Groups |
| **Policy attachment** | Identity + Resource policies | RBAC role assignments | IAM bindings (principal + role) |
| **Org-level guardrails** | SCPs | Azure Policy + Management Groups | Org Policies |
| **Privileged access** | IAM roles + temporary keys | PIM (time-bounded activation) | PAM (Privileged Access Manager) |
| **Workload identity** | IAM Roles for EC2/EKS/Lambda | Managed Identity | Workload Identity |
| **Federated external workloads** | OIDC Federation | Workload Identity Federation | Workload Identity Federation |
| **Audit logs** | CloudTrail | Azure Monitor / Activity Log | Cloud Audit Logs |
