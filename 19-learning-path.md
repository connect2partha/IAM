# IAM Learning Path

> A practical, structured guide for building your IAM knowledge — whatever your starting point or career goal.

---

## Learning IAM Is a Journey, Not a Sprint

IAM spans protocols, architecture, compliance, security, and operations. No one learns it all at once — and that's fine. The key is knowing **which path to take for your goal**.

Think of IAM learning like **building a house**:
- First you lay the foundations (core concepts, authentication, authorisation)
- Then you build the walls (protocols, SSO, directory services)
- Then you add the specialist rooms (PAM, IGA, cloud, zero trust)
- Finally you install the security systems (threat detection, compliance, architecture)

---

## 1. The Three-Level Learning Journey

```mermaid
flowchart TD
    START([🚀 Start Here]) --> B

    subgraph B["📗 Beginner — Build the Foundation\nEstimated: 4–6 weeks"]
        B1[Section 1\nFoundational Concepts\nIAM, PoLP, SoD, ZT, Lifecycle]
        B2[Section 2\nAuthentication\nMFA, Passwordless, Sessions]
        B3[Section 3\nAuthorisation\nRBAC, ABAC, DAC, MAC]
        B4[Section 4\nProtocols and Standards\nSAML, OAuth, OIDC, LDAP, JWT]
        B5[Section 5\nSingle Sign-On\nIdP, SP, SSO flows]
        B6[Section 6\nDirectory Services\nAD, LDAP, Entra ID]
        B1 --> B2 --> B3 --> B4 --> B5 --> B6
    end

    B --> MILE1([🏁 Milestone 1\nYou can explain how a user\nlogs in with SSO using SAML])

    MILE1 --> I

    subgraph I["📘 Intermediate — Go Deeper\nEstimated: 6–8 weeks"]
        I1[Section 7\nPrivileged Access Management\nPAM, vaulting, JIT]
        I2[Section 8\nIdentity Governance\nIGA, access reviews, lifecycle]
        I3[Section 9\nCloud IAM\nAWS / Azure / GCP IAM]
        I4[Section 10\nCustomer IAM\nCIAM, social login, GDPR]
        I5[Section 12\nAPI and Service Identity\nOAuth, mTLS, secrets]
        I6[Section 13\nPKI and Certificates\nX.509, CA, mTLS, revocation]
        I1 --> I2 --> I3 --> I4 --> I5 --> I6
    end

    I --> MILE2([🏁 Milestone 2\nYou can design an IAM\nsolution for a new application])

    MILE2 --> A

    subgraph A["📕 Advanced — Master the Craft\nEstimated: 6–10 weeks"]
        A1[Section 11\nZero Trust Architecture\nNIST 800-207, microsegmentation]
        A2[Section 14\nIdentity Threats\nAttack vectors, Kerberos attacks]
        A3[Section 15\nSecurity Monitoring\nUEBA, SIEM, anomaly detection]
        A4[Section 17\nIAM Architecture and Design\nPatterns, federation, HA/DR]
        A1 --> A2 --> A3 --> A4
    end

    A --> MILE3([🏁 Milestone 3\nYou can architect and secure\nan enterprise IAM programme])
```

---

## 2. Role-Based Learning Paths

Different roles need different knowledge. Start with where your role takes you.

```mermaid
flowchart TD
    ROLE([What is your role?]) --> R1{Security\nEngineer}
    ROLE --> R2{IAM\nAdministrator}
    ROLE --> R3{Compliance\nAnalyst / GRC}
    ROLE --> R4{Cloud\nEngineer}
    ROLE --> R5{Developer /\nSoftware Engineer}

    R1 -->|Start with| SE[Sections 1→2→3→4\nthen 11→14→15→17]
    R2 -->|Start with| IA[Sections 1→2→6→5\nthen 7→8→9→18]
    R3 -->|Start with| CA[Sections 1→8→7\nthen 16→15→17]
    R4 -->|Start with| CE[Sections 1→2→3→9\nthen 11→12→13]
    R5 -->|Start with| DE[Sections 1→2→3→4\nthen 10→12→13]
```

### Detailed Role Tracks

| Role | Core Sections | Specialist Sections | Skip / Skim |
|---|---|---|---|
| **Security Engineer** | 1, 2, 3, 11, 14, 15 | 4, 7, 17 | 10, 19, 20 |
| **IAM Administrator** | 1, 2, 5, 6, 7, 8 | 9, 18 | 11, 14 |
| **Compliance Analyst** | 1, 8, 16 | 7, 15, 17 | 12, 13 |
| **Cloud Engineer** | 1, 2, 3, 9, 12 | 11, 13 | 7, 10 |
| **Developer** | 1, 2, 3, 4, 10, 12 | 13, 9 | 7, 8 |
| **IAM Architect** | All sections | All sections | None |

---

## 3. Suggested Study Timeline

```mermaid
gantt
    title IAM Learning Timeline — Full Programme (28 weeks)
    dateFormat  YYYY-MM-DD
    axisFormat  Week %W

    section Beginner
    Section 1 — Foundational Concepts     :b1, 2025-01-06, 7d
    Section 2 — Authentication            :b2, after b1, 7d
    Section 3 — Authorisation             :b3, after b2, 7d
    Section 4 — Protocols and Standards   :b4, after b3, 7d
    Section 5 — Single Sign-On            :b5, after b4, 5d
    Section 6 — Directory Services        :b6, after b5, 5d

    section Intermediate
    Section 7 — PAM                       :i1, after b6, 7d
    Section 8 — IGA                       :i2, after i1, 7d
    Section 9 — Cloud IAM                 :i3, after i2, 7d
    Section 10 — CIAM                     :i4, after i3, 5d
    Section 12 — API Identity             :i5, after i4, 5d
    Section 13 — PKI                      :i6, after i5, 7d

    section Advanced
    Section 11 — Zero Trust               :a1, after i6, 7d
    Section 14 — Identity Threats         :a2, after a1, 7d
    Section 15 — Security Monitoring      :a3, after a2, 7d
    Section 17 — IAM Architecture         :a4, after a3, 7d

    section Compliance and Ecosystem
    Section 16 — Compliance               :c1, after a4, 7d
    Section 18 — Tools and Platforms      :c2, after c1, 7d
    Certifications study                  :cert, after c2, 14d
```

---

## 4. IAM Knowledge Domains — How They Connect

```mermaid
mindmap
  root((IAM\nKnowledge))
    Identity Foundation
      What is IAM
      Identity Lifecycle
      Least Privilege
      Zero Trust Principles
    Authentication
      Passwords and MFA
      Passwordless and FIDO2
      Session Management
      SSO
    Authorisation
      RBAC and ABAC
      Policy Models
      Cloud IAM
    Protocols
      SAML 2.0
      OAuth 2.0
      OIDC
      LDAP and Kerberos
      JWT
      SCIM
    Infrastructure
      Directory Services
      PKI and Certificates
      Secrets Management
      API Identity
    Governance
      IGA
      Access Reviews
      Separation of Duties
      Identity Lifecycle
    Security
      PAM
      Threat Vectors
      Zero Trust Architecture
      SIEM and UEBA
    Compliance
      SOX HIPAA GDPR
      PCI DSS
      SOC 2 ISO 27001
      NIST 800-53
    Ecosystem
      IAM Tools and Platforms
      Architecture Patterns
      HA and DR
```

---

## 5. Learning Resources by Type

| Resource Type | Best For | Examples |
|---|---|---|
| **Official documentation** | Protocols and standards | IETF RFCs (OAuth/OIDC), NIST publications, OASIS (SAML) |
| **Vendor docs** | Hands-on tool learning | Okta developer docs, Microsoft Learn, AWS IAM docs |
| **Online labs** | Practical skills | Okta Developer Edition (free), AWS free tier, Azure free account |
| **Books** | Deep theory | "Identity and Data Security for Web Development" (O'Reilly), CISSP study guides |
| **Certifications** | Validated credentials | SC-300, Okta Certified, AWS Security Specialty (see Section 20) |
| **CTF / practice** | Attack/defence | PentesterLab, HackTheBox, PortSwigger Web Academy (OAuth labs) |

---

## 6. How to Learn Effectively

```mermaid
flowchart LR
    READ[📖 Read the concept\nSimple explanation first] --> DIAGRAM

    DIAGRAM[🗺️ Draw or study\nthe diagram] --> LAB

    LAB[🔬 Set up a lab\nOkta dev / AWS free tier] --> APPLY

    APPLY[⚙️ Apply it\nBuild a real SSO integration] --> TEACH

    TEACH[🗣️ Explain it\nto someone else or write notes] --> REVIEW

    REVIEW[🔄 Review in 1 week\nspaced repetition] --> READ
```

**The best way to learn IAM is to build it:**
- Create a free Okta developer account and integrate a sample app with SAML SSO
- Set up AWS IAM and practice writing least-privilege policies
- Spin up a local AD or use Azure AD free tier
- Try the PortSwigger OAuth and OIDC labs for attack/defence understanding

---

## Quick Reference — Learning Path Summary

| Level | Sections | Estimated Time | Key Outcome |
|---|---|---|---|
| **Beginner** | 1, 2, 3, 4, 5, 6 | 4–6 weeks | Understand how authentication and SSO work |
| **Intermediate** | 7, 8, 9, 10, 12, 13 | 6–8 weeks | Design IAM for applications and cloud environments |
| **Advanced** | 11, 14, 15, 17 | 6–10 weeks | Architect secure, resilient enterprise IAM |
| **Compliance/GRC** | 16, 8, 7 | 4–5 weeks | Map IAM controls to regulatory requirements |
| **Ecosystem** | 18, 19, 20 | 2–3 weeks | Navigate the tool landscape and certifications |
