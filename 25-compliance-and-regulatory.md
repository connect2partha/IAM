# IAM Compliance and Regulatory Frameworks

> Simple explanations of the major laws and standards that shape how organisations manage identity and access.

---

## Why Compliance Matters for IAM

Think of compliance frameworks as **building codes for identity security**. Just as a building code tells architects what fire exits are required, compliance frameworks tell organisations what access controls, audit trails, and identity practices they must have.

Non-compliance can mean fines, legal liability, or losing the right to operate in certain markets.

---

## 1. The Compliance Landscape

Different regulations apply to different industries. The same organisation may need to comply with multiple frameworks simultaneously.

```mermaid
flowchart TD
    subgraph Finance["🏦 Finance & Public Companies"]
        SOX[SOX\nSarbanes-Oxley]
        PCI[PCI DSS\nPayment Cards]
    end

    subgraph Health["🏥 Healthcare"]
        HIPAA[HIPAA\nHealth Data USA]
    end

    subgraph Privacy["🌍 Privacy / All Industries"]
        GDPR[GDPR\nEU Privacy]
        CCPA[CCPA\nCalifornia Privacy]
    end

    subgraph Tech["💻 Technology / SaaS"]
        SOC2[SOC 2 Type II\nService Organisations]
    end

    subgraph Universal["🌐 Universal / Government"]
        ISO[ISO/IEC 27001\nSecurity Management]
        NIST[NIST SP 800-53\nFederal / Best Practice]
    end
```

---

## 2. SOX — Sarbanes-Oxley Act

**Who it applies to:** Publicly traded US companies and their auditors.

**Simple analogy:** SOX is like requiring that two separate cashiers count the money — no single person controls the entire financial process.

**What it requires for IAM:**
- Separation of Duties on financial systems
- Quarterly access reviews (who has access to financial data)
- Audit trails showing who accessed what, when
- Controls preventing unauthorised changes to financial records

```mermaid
flowchart TD
    E([Employee]) --> REQ[Requests access\nto financial system]
    REQ --> MGR{Manager\nApproves?}
    MGR -->|No| DENY([Access Denied])
    MGR -->|Yes| SOD{SoD Check\nDoes this conflict\nwith existing roles?}
    SOD -->|Conflict found| DENY
    SOD -->|No conflict| GRANT[Access Granted\nLogged in audit trail]

    GRANT --> REVIEW[Quarterly\nAccess Review]
    REVIEW --> STILL{Still\nneeds access?}
    STILL -->|No| REVOKE[Access Revoked]
    STILL -->|Yes| RECERT[Recertified ✅]

    GRANT --> AUDIT[(📋 Immutable\nAudit Log)]
```

**Key IAM controls for SOX:**
- Role-based access to ERP systems (SAP, Oracle Financials)
- SoD rules: same person cannot create AND approve a payment
- Automated quarterly access certifications
- Privileged access monitoring for financial admins

---

## 3. HIPAA — Health Insurance Portability and Accountability Act

**Who it applies to:** US healthcare providers, health plans, healthcare clearinghouses, and their business associates.

**Simple analogy:** A doctor only reads the medical records they need for their patient right now — not every patient's records ever created.

**What it requires for IAM:**
- Minimum necessary access to PHI (Protected Health Information)
- Unique user identification (no shared accounts)
- Automatic session timeout
- Audit controls — logs of PHI access
- Breach notification within 60 days

```mermaid
flowchart TD
    HCP([Healthcare Professional]) --> LOGIN[Login with\nunique credentials]
    LOGIN --> ROLE{What is\nyour role?}
    ROLE -->|Doctor| D_ACCESS[Access to\nassigned patients only]
    ROLE -->|Nurse| N_ACCESS[Access to\nward patients only]
    ROLE -->|Billing| B_ACCESS[Access to\nbilling records only]
    ROLE -->|Admin| A_ACCESS[Access to\nadministrative data only]

    D_ACCESS --> PHI[(PHI Database)]
    N_ACCESS --> PHI
    B_ACCESS --> PHI
    A_ACCESS --> PHI

    PHI --> LOG[(Access Log\nevery read/write recorded)]
    LOG --> MONITOR[Security team\nmonitors for anomalies]
    MONITOR --> BREACH{Suspected\nbreach?}
    BREACH -->|Yes| NOTIFY[Notify affected individuals\nwithin 60 days]
    BREACH -->|No| CONTINUE([Continue normal operations])

    LOGIN --> TIMEOUT[Auto session\ntimeout after inactivity]
```

**Key IAM controls for HIPAA:**
- Role-based access mapped to job function
- No shared/generic accounts
- MFA for remote access to PHI
- Session timeout (typically 15 minutes of inactivity)
- Full PHI access logging and anomaly alerting

---

## 4. GDPR / CCPA — Data Privacy Regulations

**Who it applies to:**
- **GDPR**: Any organisation handling EU residents' personal data
- **CCPA**: Organisations handling California residents' personal data

**Simple analogy:** GDPR/CCPA is like giving people a remote control over their own data — they can pause it, rewind it, or delete it entirely.

**Data subject rights IAM must support:**

```mermaid
flowchart TD
    USER([Data Subject\nEU/CA Resident]) --> REQUEST[Submits rights request\nvia portal]

    REQUEST --> TYPE{Type of\nrequest?}

    TYPE -->|Right to Access| ACCESS[Export all personal data\nwithin 30 days]
    TYPE -->|Right to Deletion\nRight to be Forgotten| DELETE[Delete all personal data\nwithin 30 days]
    TYPE -->|Right to Rectification| FIX[Correct inaccurate data]
    TYPE -->|Restrict Processing| RESTRICT[Stop using data\nfor specified purpose]
    TYPE -->|Data Portability| EXPORT[Provide data in\nmachine-readable format]

    ACCESS --> VERIFY{Verify\nidentity first}
    DELETE --> VERIFY
    FIX --> VERIFY
    RESTRICT --> VERIFY
    EXPORT --> VERIFY

    VERIFY -->|Verified| FULFIL[Fulfil request]
    VERIFY -->|Not verified| REJECT[Reject — cannot\nconfirm identity]

    FULFIL --> LOG2[(Request log\nfor compliance evidence)]
```

**Key IAM controls for GDPR/CCPA:**
- Consent management — capture and record user consent
- Data minimisation — collect only what's needed
- Identity verification before fulfilling data requests
- Audit trail of all data access and changes
- Data retention policies — delete data after retention period
- Processor agreements — third-party vendors accessing data must comply

---

## 5. SOC 2 Type II

**Who it applies to:** Technology service providers and SaaS companies handling customer data.

**Simple analogy:** SOC 2 Type II is like a continuous CCTV recording — not just a photo of the building on one day, but proof that your security controls worked consistently over 6–12 months.

**The five Trust Service Criteria (TSC):**

```mermaid
mindmap
  root((SOC 2\nTrust Service\nCriteria))
    Security
      Logical access controls
      MFA enforcement
      Network controls
    Availability
      System uptime
      Incident response
      Capacity management
    Confidentiality
      Data encryption
      Access restrictions
      Disposal procedures
    Processing Integrity
      Complete and accurate processing
      Error handling
    Privacy
      Personal data handling
      Consent management
      Retention policies
```

**IAM evidence SOC 2 auditors look for:**

```mermaid
flowchart LR
    AUDIT([SOC 2 Auditor]) --> E1[Access review\nevidence\nquarterly certs]
    AUDIT --> E2[User provisioning\nlogs\nnew hire onboarding]
    AUDIT --> E3[Deprovisioning\nlogs\nleaver offboarding]
    AUDIT --> E4[MFA enrollment\nreports]
    AUDIT --> E5[Privileged access\nreview evidence]
    AUDIT --> E6[Password policy\nconfiguration screenshots]
    AUDIT --> E7[Security training\ncompletion records]

    E1 --> OPINION{Auditor\nOpinion}
    E2 --> OPINION
    E3 --> OPINION
    E4 --> OPINION
    E5 --> OPINION
    E6 --> OPINION
    E7 --> OPINION

    OPINION -->|Controls effective| PASS([✅ Unqualified Opinion\nSOC 2 Type II Report])
    OPINION -->|Gaps found| FAIL([⚠️ Qualified Opinion\nRemediation required])
```

---

## 6. ISO/IEC 27001 — A.9 Access Control

**Who it applies to:** Any organisation wanting to certify their information security management system (ISMS).

**Simple analogy:** ISO 27001 is the international standard for a security management system — like ISO 9001 is for quality management. A.9 is specifically the access control chapter.

**A.9 Access Control domains:**

```mermaid
flowchart TD
    A9[ISO/IEC 27001\nAnnex A.9\nAccess Control]

    A9 --> A91[A.9.1\nBusiness requirements\nof access control]
    A9 --> A92[A.9.2\nUser access management]
    A9 --> A93[A.9.3\nUser responsibilities]
    A9 --> A94[A.9.4\nSystem and application\naccess control]

    A91 --> A911[Access control policy\ndocumented and approved]
    A91 --> A912[Access to networks\nand network services]

    A92 --> A921[User registration\nand deregistration]
    A92 --> A922[User access provisioning]
    A92 --> A923[Privileged access rights]
    A92 --> A924[Secret authentication info\npassword management]
    A92 --> A925[Access rights review]
    A92 --> A926[Removal of access rights]

    A93 --> A931[Use of secret\nauthentication info]

    A94 --> A941[Information access restriction]
    A94 --> A942[Secure log-on procedures]
    A94 --> A943[Password management system]
    A94 --> A944[Privileged utility programs]
    A94 --> A945[Access control to source code]
```

---

## 7. NIST SP 800-53 — AC and IA Control Families

**Who it applies to:** US Federal agencies (mandatory), and widely adopted as best practice globally.

**Simple analogy:** NIST 800-53 is the most detailed security cookbook in existence — it lists exactly what security controls you should have, with multiple implementation levels.

```mermaid
flowchart TD
    subgraph AC["AC — Access Control Family"]
        AC1[AC-1 Policy and Procedures]
        AC2[AC-2 Account Management]
        AC3[AC-3 Access Enforcement]
        AC4[AC-4 Information Flow Enforcement]
        AC5[AC-5 Separation of Duties]
        AC6[AC-6 Least Privilege]
        AC7[AC-7 Unsuccessful Login Attempts]
        AC11[AC-11 Session Lock]
        AC17[AC-17 Remote Access]
        AC18[AC-18 Wireless Access]
    end

    subgraph IA["IA — Identification and Authentication Family"]
        IA1[IA-1 Policy and Procedures]
        IA2[IA-2 Identification and Authentication\nOrganisational Users]
        IA3[IA-3 Device Identification]
        IA4[IA-4 Identifier Management]
        IA5[IA-5 Authenticator Management]
        IA6[IA-6 Authenticator Feedback]
        IA8[IA-8 Non-Org User Auth]
        IA11[IA-11 Re-authentication]
    end

    AC2 -.->|feeds into| IA4
    AC3 -.->|enforces| IA2
    AC6 -.->|shaped by| IA5
```

**NIST control baselines:**

| Control | Low Baseline | Moderate Baseline | High Baseline |
|---|---|---|---|
| MFA (IA-2) | Privileged only | All users | All + phishing-resistant |
| Session Lock (AC-11) | 30 min | 15 min | 15 min |
| Account Review (AC-2) | Annual | Semi-annual | Quarterly |
| Least Privilege (AC-6) | Basic | Enforced | JIT + reviewed |

---

## 8. PCI DSS — Requirements 7 and 8

**Who it applies to:** Any organisation that stores, processes, or transmits payment card data.

**Simple analogy:** Think of a vault — not everyone in the bank gets into the vault (Requirement 7 — restrict access), and everyone going into the vault must sign in and show ID (Requirement 8 — authenticate).

```mermaid
flowchart TD
    subgraph REQ7["PCI DSS Requirement 7\nRestrict Access to System Components"]
        R7A[Define access control model\nbased on need-to-know]
        R7B[Document access roles\nfor cardholder data systems]
        R7C[Assign minimum necessary\naccess to each role]
        R7D[Deny all access\nby default]
        R7A --> R7B --> R7C --> R7D
    end

    subgraph REQ8["PCI DSS Requirement 8\nIdentify Users and Authenticate Access"]
        R8A[Unique ID for every user\nno shared accounts]
        R8B[Strong authentication\nMFA for all admin access]
        R8C[Password controls\nlength, complexity, history]
        R8D[Lock accounts after\n6 failed attempts]
        R8E[Session idle timeout\n15 minutes]
        R8F[Secure all credentials\nencrypt stored passwords]
        R8A --> R8B --> R8C --> R8D --> R8E --> R8F
    end

    REQ7 --> CDE[(Cardholder\nData Environment)]
    REQ8 --> CDE
```

**PCI DSS key IAM requirements summary:**

| Requirement | Control | Specifics |
|---|---|---|
| 7.1 | Access control model | Documented, need-to-know basis |
| 7.2 | Access control systems | Cover all system components |
| 8.2 | Unique IDs | No shared/generic accounts |
| 8.3 | MFA | Required for all admin and remote access |
| 8.3.6 | Password length | Minimum 12 characters |
| 8.3.9 | Password change | Every 90 days if no MFA |
| 8.6 | Service accounts | Managed, documented, reviewed |

---

## 9. The Compliance Audit Cycle

All frameworks follow a similar cycle. Build it once; evidence it continuously.

```mermaid
flowchart LR
    P[📋 Define Policy\nDocument IAM controls\nrequired by framework] --> I

    I[🔧 Implement Controls\nDeploy MFA, RBAC,\nprovisioning workflows] --> M

    M[📊 Monitor\nContinuous logging,\nalerts, dashboards] --> E

    E[📁 Collect Evidence\nAccess reviews, logs,\nscreenshots, reports] --> A

    A[🔍 Audit\nInternal or external\nauditor reviews evidence] --> R

    R{Findings?}
    R -->|Yes — gaps found| REM[🔨 Remediate\nFix controls,\nclose gaps]
    R -->|No — clean opinion| CERT([✅ Certification\nor Report issued])

    REM --> P
    CERT --> P
```

---

## Quick Reference Cheat Sheet

| Framework | Industry | Key IAM Requirements | Penalty for Non-Compliance |
|---|---|---|---|
| **SOX** | Public companies | SoD, quarterly access reviews, audit trails | Criminal charges for executives |
| **HIPAA** | Healthcare (US) | Minimum necessary access, unique IDs, PHI logging | $100–$50,000 per violation |
| **GDPR** | EU data (all industries) | Consent, data subject rights, breach notification | Up to 4% global annual revenue |
| **CCPA** | California data | Opt-out, deletion rights, disclosure | $100–$750 per consumer per incident |
| **SOC 2 Type II** | SaaS / tech providers | Logical access controls, evidence over time | Loss of customer trust / contracts |
| **ISO 27001** | Universal | A.9 access control policy and controls | Loss of certification |
| **NIST 800-53** | US Federal (+ best practice) | AC + IA control families, baselines | Loss of ATO (Authority to Operate) |
| **PCI DSS** | Payment card industry | Unique IDs, MFA, need-to-know, session controls | Fines + loss of card processing rights |
