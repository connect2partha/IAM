# Privileged Access Management (PAM)

> PAM is about locking down the most powerful keys to your kingdom — and knowing exactly who used them, when, and why.

---

## What is Privileged Access?

A regular employee has access to their email and a few business apps. A **privileged user** has access to the systems that run the business — servers, databases, network devices, cloud infrastructure.

> Think of a regular key vs a **master key**. A master key opens every door. PAM is the system that controls who gets the master key, for how long, and watches everything they do with it.

```mermaid
flowchart LR
    subgraph Regular["👤 Regular User Access"]
        RU([Employee]) --> EMAIL[Email]
        RU --> HR[HR Portal]
        RU --> CRM[CRM]
    end

    subgraph Privileged["🔑 Privileged Access"]
        PU([Admin]) --> DC[Domain Controllers]
        PU --> DB[Production Databases]
        PU --> INFRA[Network Infrastructure]
        PU --> CLOUD[Cloud Root Accounts]
        PU --> SEC[Security Systems]
    end

    RISK[/"⚠️ Privileged accounts are\nthe #1 target for attackers"/]
    Privileged -.-> RISK
```

---

## 1. Types of Privileged Accounts

```mermaid
flowchart TD
    PA[Privileged Accounts] --> HUMAN[Human Accounts]
    PA --> NONHUMAN[Non-Human Accounts]

    HUMAN --> LA[Local Admin\nBuilt-in admin on each machine\n🔴 High Risk]
    HUMAN --> DA[Domain Admin\nFull control of AD domain\n🔴 Critical Risk]
    HUMAN --> EA[Enterprise Admin\nFull control of entire AD forest\n🔴 Critical Risk]
    HUMAN --> CA[Cloud Admin\nRoot / Global Admin in cloud\n🔴 Critical Risk]

    NONHUMAN --> SVC[Service Accounts\nApps authenticating to other systems\n🟡 Medium Risk]
    NONHUMAN --> SA[Scheduled Task Accounts\nAutomation and scripts\n🟡 Medium Risk]
    NONHUMAN --> API[API Keys\nService-to-service calls\n🟡 Medium Risk]
    NONHUMAN --> SSH[SSH Keys\nServer-to-server access\n🟡 Medium Risk]
```

| Account Type | Risk | Why It's Dangerous |
|---|---|---|
| Domain Admin | Critical | Full AD control — can create backdoor accounts |
| Local Admin | High | Can install malware, bypass endpoint controls |
| Service Account | Medium | Often has no expiry, password never changed |
| Root / Cloud Admin | Critical | Can destroy or exfiltrate entire infrastructure |
| SSH Key | Medium | Often forgotten, gives persistent server access |

---

## 2. Credential Vaulting — Check-In / Check-Out

Instead of giving admins a permanent password, PAM tools store credentials in a **vault** and issue them temporarily — like a hotel key card that expires at checkout.

```mermaid
flowchart TD
    ADMIN([👤 Admin]) -->|1. Request access to Production DB| PAM[🏦 PAM Vault\nCyberArk / Delinea]

    PAM -->|2. Verify identity + approval| AUTH{Approved?}
    AUTH -->|No| DENY([❌ Denied + Alert])
    AUTH -->|Yes| CHECKOUT[3. Check out credential\nTemp password issued\nValid for 2 hours]

    CHECKOUT -->|4. Connect via PAM proxy| TARGET[(🗄️ Production Database)]

    TARGET -->|5. Session complete| CHECKIN[6. Check-in\nSession ends]
    CHECKIN --> ROTATE[7. Auto-rotate password\nOld credential invalid ♻️]
    ROTATE --> LOG[(📋 Full session\naudit log retained)]
```

> **Why auto-rotate?** Even if the admin copied the password, it's useless after their session ends.

---

## 3. Just-in-Time (JIT) Privileged Access

**JIT access** means privilege is granted only when needed, for a limited time — then automatically removed. No permanent standing privilege.

> Old model: "Alice is permanently a Domain Admin."
> JIT model: "Alice can be a Domain Admin for the next 30 minutes to complete this task."

```mermaid
flowchart TD
    ADMIN([👤 Admin]) -->|Submit request:\nNeed Domain Admin\nto patch 3 servers\nEstimate: 30 min| REQUEST[Access Request Portal]

    REQUEST --> APPROVAL{Auto-approval\nor manager?}
    APPROVAL -->|Low risk — auto-approved| GRANT
    APPROVAL -->|High risk — manager reviews| MANAGER([👨‍💼 Manager])
    MANAGER -->|Approves| GRANT
    MANAGER -->|Denies| DENY([❌ Denied])

    GRANT[✅ Domain Admin role\ngranted for 30 minutes]
    GRANT --> SESSION[Admin completes task]
    SESSION --> TIMER{Time elapsed?}
    TIMER -->|30 min reached| REVOKE[🔒 Role automatically\nrevoked]
    TIMER -->|Extension requested| APPROVAL
    REVOKE --> AUDIT[(📋 Audit log:\nwho, what, when, why)]
```

### Standing Privilege vs JIT

```mermaid
flowchart LR
    subgraph Standing["❌ Standing Privilege (Risky)"]
        A([Admin]) -->|Always has| DA[Domain Admin]
        DA -->|If account compromised| BREACH[Attacker has\nDomain Admin 24/7]
    end

    subgraph JIT["✅ Just-in-Time (Safe)"]
        B([Admin]) -->|Requests when needed| VAULT[PAM Vault]
        VAULT -->|Grants for 30 min| DA2[Domain Admin]
        DA2 -->|Auto-revoked| NONE[No privilege\nafter task]
    end
```

---

## 4. Session Monitoring and Recording

Every privileged session is **proxied through the PAM system**, recorded, and can be reviewed or terminated in real time.

> Like a **security camera** on every admin action — not to spy, but to investigate incidents and prove compliance.

```mermaid
flowchart LR
    ADMIN([👤 Privileged User]) -->|Connect via PAM proxy| PAM_PROXY[🎥 PAM Session Proxy\nCyberArk PSM / Delinea]

    PAM_PROXY -->|Passes connection through| TARGET1[🖥️ Windows Server]
    PAM_PROXY -->|Passes connection through| TARGET2[🐧 Linux Server]
    PAM_PROXY -->|Passes connection through| TARGET3[🗄️ Database]

    PAM_PROXY --> RECORD[Session Recording\nKeystrokes, mouse, screen]
    PAM_PROXY --> METADATA[Session Metadata\nUser, time, target, duration]
    PAM_PROXY --> REALTIME[Real-time Monitoring\nSecurity team can watch live]
    PAM_PROXY --> ALERT[Alerting\nSuspicious commands trigger alert]

    RECORD & METADATA --> AUDIT[(📋 Immutable\nAudit Archive)]
    REALTIME -->|Terminate if needed| KILL[🛑 Kill Session]
```

---

## 5. Break-Glass (Emergency) Accounts

What if the PAM system itself goes down? Or there's a critical incident at 3am? **Break-glass accounts** are emergency backdoors — powerful, rarely used, and tightly controlled.

> Like the **fire extinguisher** behind glass on the wall — only break it in an emergency, and everyone will know you did.

```mermaid
flowchart TD
    INCIDENT[🚨 Critical Incident\nPAM system unavailable\nor major outage]

    INCIDENT --> CHECK{Normal admin\naccess available?}
    CHECK -->|Yes| NORMAL[Use normal PAM workflow]
    CHECK -->|No — genuine emergency| BG[Break-Glass Process]

    BG --> DUAL[Two-person rule:\nBoth people must be present\nand authorise together]
    DUAL --> VAULT2[Physical / offline vault\nEnvelope in safe or\npassword manager with dual keys]
    VAULT2 --> ACCESS[Retrieve break-glass\ncredential]
    ACCESS --> USE[Use emergency account\nto resolve incident]
    USE --> REPORT[📧 Immediate notification\nto CISO + Security team]
    USE --> AUDIT2[(📋 All actions logged\nManual audit review)]
    USE --> ROTATE2[Password rotated\nimmediately after use ♻️]
```

---

## 6. PAM Tools Ecosystem

```mermaid
mindmap
  root((PAM Tools))
    Credential Vaulting
      CyberArk Vault
      Delinea Secret Server
      BeyondTrust Password Safe
    Session Management
      CyberArk PSM
      Delinea Connection Manager
      BeyondTrust Remote Support
    Secrets Management
      HashiCorp Vault
      AWS Secrets Manager
      Azure Key Vault
    Cloud PAM
      CyberArk Cloud Entitlements
      Delinea Cloud Suite
      AWS IAM Access Analyzer
    Open Source
      HashiCorp Vault
      Teleport
      Boundary
```

---

## 7. Without PAM vs With PAM

```mermaid
flowchart TD
    subgraph NoPAM["❌ Without PAM"]
        A1[Admins have permanent\nshared passwords]
        A2[Credentials stored in\nspreadsheets or sticky notes]
        A3[No audit trail of\nwho did what]
        A4[Passwords never rotated]
        A5[If attacker steals creds\nthey have unlimited access forever]

        A1 --> A5
        A2 --> A5
        A4 --> A5
    end

    subgraph WithPAM["✅ With PAM"]
        B1[Just-in-time access\nonly when needed]
        B2[Credentials vaulted\nauto-rotated after use]
        B3[Full session recording\nand audit trail]
        B4[Stolen credential becomes\nuseless after session ends]
        B5[Break-glass process\nfor genuine emergencies]

        B1 --> B4
        B2 --> B4
    end
```

---

## Quick Reference Cheat Sheet

| Concept | One-liner | Key Rule |
|---|---|---|
| **Privileged Account** | Account with elevated system access | Treat every privileged account as a critical asset |
| **Credential Vault** | Secure store for privileged passwords | Check out, use, check in — then auto-rotate |
| **JIT Access** | Privilege only when needed | No standing privilege; time-bound elevation |
| **Session Recording** | Record all privileged sessions | PAM proxy captures keystrokes, screen, metadata |
| **Break-Glass** | Emergency backdoor account | Dual-person rule, immediate notification, rotate after use |
| **Service Account** | Non-human privileged identity | Treat like any privileged account — vault and rotate |
| **CyberArk** | Leading enterprise PAM tool | Vault, session management, threat analytics |
| **HashiCorp Vault** | Secrets management, cloud-native PAM | Great for DevOps / cloud-native environments |
