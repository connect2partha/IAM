# Zero Trust Architecture (ZTA)

> Zero Trust is a security philosophy: stop trusting based on location (like "inside the network") and start trusting based on verified identity, device health, and context — every single time.

---

## 1. The Old Way vs Zero Trust

**The old model:** Build a strong wall around the network. Everyone inside the wall is trusted.
**The problem:** Attackers can get inside (phishing, VPN compromise, insider threat). Once inside — total freedom.

**Zero Trust says:** There is no "inside". Verify everyone, every time, regardless of where they are.

> Old model = castle with a moat. Zero Trust = every room in the castle has its own lock, and you need ID to enter each one.

```mermaid
flowchart TD
    subgraph OLD["❌ Old Perimeter Model — Castle and Moat"]
        MOAT[🏰 Firewall / VPN\nStrong outer wall]
        MOAT -->|Once inside...| TRUSTED[Everything trusted\nMove freely between all systems]
        BREACH[Attacker bypasses wall] --> TRUSTED
        TRUSTED --> DATA[💀 Full access to data]
    end

    subgraph ZT["✅ Zero Trust — Every Door Checked"]
        USER2([User / Device]) --> CHECK1{Identity verified?\nMFA}
        CHECK1 -->|Yes| CHECK2{Device healthy?\nEDR / MDM}
        CHECK2 -->|Yes| CHECK3{Authorised for\nthis resource?}
        CHECK3 -->|Yes| ACCESS2[✅ Narrow access granted\nthis resource only]
        CHECK3 -->|No| DENY2[❌ Denied]
        CHECK1 -->|No| DENY2
        CHECK2 -->|No| DENY2
        ACCESS2 --> MONITOR[Continuously monitored]
    end
```

---

## 2. NIST SP 800-207 — ZTA Reference Architecture

The US government's official ZTA standard defines three core components.

```mermaid
flowchart TD
    subgraph CTRL["Control Plane"]
        PE[Policy Engine\nMakes trust decisions\nbased on policy + signals]
        PA[Policy Administrator\nIssues / revokes session tokens\ncommunicates with PEP]
        PE <-->|Trust decision| PA
    end

    subgraph DATA["Data Plane"]
        PEP[Policy Enforcement Point\nGateway / proxy that sits in front of resources]
    end

    USER([Subject\nUser / Device / Workload]) --> PEP
    PEP <-->|Request evaluation| PA

    PA -->|Allow — issue token| PEP
    PA -->|Deny| PEP

    PEP -->|Allowed traffic| RESOURCE[(Protected Resource\nApp / Data / Service)]

    SIGNALS[Trust Signals\nIdentity provider, MDM, SIEM, threat intel] --> PE
```

| Component | Role | Real-world Example |
|---|---|---|
| **Policy Engine (PE)** | Decides allow or deny | Entra ID Conditional Access rules |
| **Policy Administrator (PA)** | Manages session, issues access tokens | Okta / Azure AD |
| **Policy Enforcement Point (PEP)** | Sits between subject and resource; enforces the decision | Reverse proxy, ZTNA gateway, App Gateway |

---

## 3. Trust Signal Evaluation

**Simple idea:** Every access request is scored using multiple signals. The policy engine weighs them all to decide: allow, allow with extra step, or deny.

```mermaid
flowchart TD
    REQ[Access Request] --> SIG[Collect Trust Signals]

    SIG --> ID_SIG[Identity Signal\nIs MFA passed?\nIs account compromised?]
    SIG --> DEV_SIG[Device Signal\nIs device MDM-enrolled?\nIs antivirus up to date?\nIs OS patched?]
    SIG --> NET_SIG[Network Signal\nKnown IP?\nTor / VPN / proxy?]
    SIG --> BEH_SIG[Behaviour Signal\nNormal access pattern?\nImpossible travel?]
    SIG --> DATA_SIG[Data Sensitivity\nHow sensitive is this resource?]

    ID_SIG & DEV_SIG & NET_SIG & BEH_SIG & DATA_SIG --> ENGINE[Policy Engine\nCompute trust score]

    ENGINE --> DECISION{Trust Level}

    DECISION -->|High trust| ALLOW[✅ Allow direct access]
    DECISION -->|Medium trust| STEPUP[📱 Step-up: require MFA]
    DECISION -->|Low trust| RESTRICT[⚠️ Read-only or limited access]
    DECISION -->|Very low trust| BLOCK[🔴 Block + alert security]
```

---

## 4. Microsegmentation

**Simple idea:** Instead of one big flat network, split it into tiny zones. Each zone has its own access rules. Compromise in one zone cannot spread to others.

> Like a submarine with watertight compartments. If one floods, the others stay dry.

```mermaid
flowchart TD
    subgraph FLAT["❌ Flat Network — Lateral Movement Risk"]
        ATK([Attacker]) -->|Compromises one server| SERVER1[Web Server]
        SERVER1 -->|Free access| SERVER2[App Server]
        SERVER1 -->|Free access| SERVER3[Database]
        SERVER1 -->|Free access| SERVER4[Finance System]
        SERVER1 -->|Free access| SERVER5[HR System]
    end

    subgraph MICRO["✅ Microsegmented Network"]
        ATK2([Attacker]) -->|Compromises| WEB[Web Server\nZone: DMZ]
        WEB -->|Requires explicit allow rule| APP[App Server\nZone: App Tier]
        APP -->|Requires explicit allow rule| DB[(Database\nZone: Data Tier)]
        WEB -->|❌ No path| FIN[Finance System\nZone: Finance]
        WEB -->|❌ No path| HR[HR System\nZone: HR]
        APP -->|❌ No path| FIN
        APP -->|❌ No path| HR
    end
```

### Segmentation layers

```mermaid
mindmap
  root((Microsegmentation))
    Network Layer
      VLAN isolation
      Firewall rules
      SDN policies
    Workload Layer
      Container network policies
      Kubernetes NetworkPolicy
      Service mesh mTLS
    Application Layer
      API gateway controls
      App-layer firewall
      Zero Trust Network Access
    Data Layer
      Encryption at rest
      Column-level security
      Data classification tags
```

---

## 5. Continuous Verification

**Simple idea:** Trust is not granted once at login and never checked again. It is continuously re-evaluated throughout the session.

> Like a nightclub: they check your ID at the door, but security also watches you inside. If you start trouble, you're out.

```mermaid
flowchart TD
    LOGIN[Initial Login\nMFA + device check] -->|Trust granted| SESSION[Active Session]

    SESSION --> CONTINUOUS[Continuous Monitoring]

    CONTINUOUS --> CHECK1{Identity still valid?\naccount not compromised?}
    CONTINUOUS --> CHECK2{Device still healthy?\nno malware detected?}
    CONTINUOUS --> CHECK3{Behaviour still normal?\nno anomaly detected?}

    CHECK1 & CHECK2 & CHECK3 -->|All OK| SESSION

    CHECK1 -->|Account flagged| REAUTH[Force re-authentication]
    CHECK2 -->|Device compromised| TERM[Terminate session\nquarantine device]
    CHECK3 -->|Anomaly detected| STEPUP2[Step-up MFA\nor session review]

    REAUTH -->|Passes| SESSION
    REAUTH -->|Fails| TERM
```

---

## 6. Device Trust

**Simple idea:** Only allow access from devices that are known, managed, and healthy.

```mermaid
flowchart TD
    DEVICE([Device requests access]) --> CHECK[Device Trust Check]

    CHECK --> ENROLL{Is device\nMDM-enrolled?}
    ENROLL -->|No| DENY[❌ Denied\nEnrol device first]
    ENROLL -->|Yes| PATCH{Is OS patched\nand up to date?}

    PATCH -->|No| RESTRICT[⚠️ Limited access\nupdates required]
    PATCH -->|Yes| AV{Is antivirus\nrunning and current?}

    AV -->|No| RESTRICT
    AV -->|Yes| CERT{Does device\nhave valid cert?]
    CERT -->|No| DENY
    CERT -->|Yes| ALLOW[✅ Device trusted\nFull access allowed]

    ALLOW --> MONITOR[Device continuously monitored\nvia EDR]
```

---

## 7. BeyondCorp — Google's Zero Trust Model

Google built BeyondCorp after the Aurora breach in 2010. The core idea: **no VPN**. All access goes through an access proxy that enforces identity + device trust.

```mermaid
flowchart TD
    EMP([Employee]) -->|On any network\nhome, café, office| PROXY[Access Proxy\nclassify-myapp.corp.google.com]

    PROXY --> IDcheck{Identity\nverified?}
    IDcheck -->|No| DENY[❌ Blocked]
    IDcheck -->|Yes| DEVcheck{Device\nMDM-enrolled + cert?}

    DEVcheck -->|No| DENY
    DEVcheck -->|Yes| APP[Internal Application]

    APP --> LOG2[(Access logged)]

    VPN[❌ No VPN needed\nor used]
```

> **Key insight:** An employee on the internet and an attacker on the corporate network get the same treatment — both must prove identity and device trust.

---

## 8. Microsoft Zero Trust / SASE

```mermaid
flowchart TD
    IDENTITY[Entra ID\nIdentity plane] --> CA[Conditional Access\nPolicy engine]
    DEVICE[Intune / Defender\nDevice plane] --> CA
    APP[Defender for Cloud Apps\nApp plane] --> CA
    NET[Entra Private Access\nEntra Internet Access] --> CA

    CA --> GRANT[Grant access with controls]
    CA --> BLOCK2[Block access]
    CA --> SESSION_CTRL[Grant with session controls\ne.g. read-only in browser]

    GRANT & SESSION_CTRL --> AUDIT2[(Unified Audit Log\nSentinel SIEM)]
```

---

## Quick Reference Cheat Sheet

| Concept | One-liner | Practical Example |
|---|---|---|
| **Never trust, always verify** | Location doesn't equal trust | VPN users still get MFA challenged |
| **Least privilege** | Minimum access, time-limited | PIM for admin roles |
| **Assume breach** | Act as if attackers are already inside | Segment network, encrypt everything |
| **Policy Engine** | The brain that decides allow/deny | Entra Conditional Access |
| **Policy Enforcement Point** | The gate that blocks/allows traffic | Reverse proxy, ZTNA gateway |
| **Microsegmentation** | Watertight compartments | Kubernetes NetworkPolicy |
| **Device trust** | Only managed, healthy devices allowed | MDM enrollment required |
| **Continuous verification** | Trust re-evaluated every request | Session risk scoring |
| **BeyondCorp** | No VPN, all via access proxy | Identity + device cert required |
