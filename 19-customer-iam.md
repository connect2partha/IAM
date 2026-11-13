# Customer Identity and Access Management (CIAM)

[← Back to Index](README.md)

> CIAM is IAM designed for your *customers*, not your *employees*. The scale is bigger, the trust level is lower, and the user experience must be frictionless — because a bad login experience means lost revenue.

---

## 1. Enterprise IAM vs CIAM — Key Differences

**Enterprise IAM** manages 1,000–100,000 known employees on corporate devices.
**CIAM** manages millions of unknown customers on personal devices from anywhere.

```mermaid
flowchart LR
    subgraph ENT["🏢 Enterprise IAM"]
        E1[Known users\nemployees]
        E2[Managed devices\ncorporate laptops]
        E3[Moderate scale\n1K–100K users]
        E4[Security-first\ntight controls]
        E5[IT manages onboarding]
    end

    subgraph CIAM_BOX["🛍️ Customer IAM"]
        C1[Unknown users\ncustomers]
        C2[Unmanaged devices\npersonal phones]
        C3[Massive scale\nmillions of users]
        C4[UX-first\nfriction = churn]
        C5[Self-service onboarding]
    end
```

| Factor | Enterprise IAM | CIAM |
|---|---|---|
| **Scale** | Thousands | Millions |
| **Trust level** | High (employees) | Low (strangers) |
| **UX priority** | Security > UX | UX = Security |
| **Onboarding** | IT-driven | Self-service |
| **Compliance** | HR/SOX/HIPAA | GDPR/CCPA |
| **MFA** | Mandatory | Optional / risk-based |

---

## 2. Customer Registration and Progressive Profiling

**Simple idea:** Don't ask customers for 20 fields on the first visit. Collect data gradually as trust builds and as it becomes needed.

> Like a new friendship: you don't share your life story on day one. You share more as the relationship grows.

```mermaid
flowchart TD
    VISIT1[🛒 First Visit\nBrowsing the site] -->|No account needed yet| ANON[Anonymous session\ncookie for preferences only]

    ANON -->|Ready to purchase| SIGNUP[Sign up]
    SIGNUP --> BASIC[Collect minimal info\nemail + password only]
    BASIC --> VERIFY[📧 Email verification]
    VERIFY --> ACTIVE[✅ Active Account]

    ACTIVE -->|First purchase| COLLECT1[Collect shipping address\nonly what's needed for delivery]
    ACTIVE -->|Loyalty programme| COLLECT2[Collect name + preferences\nbetter recommendations]
    ACTIVE -->|Payment saved| COLLECT3[Collect payment info\nencrypted vault]

    COLLECT1 & COLLECT2 & COLLECT3 --> FULL[Full Customer Profile\nbuilt gradually over time]
```

---

## 3. Social Login and Account Linking

**Simple idea:** Let customers log in with their existing Google, Apple, or Facebook account — no new password to remember.

> Like showing your passport at a foreign hotel: a trusted authority has already verified you, so the hotel accepts it.

```mermaid
flowchart TD
    USER([👤 Customer]) -->|Clicks 'Sign in with Google'| APP[Your Application]
    APP -->|Redirects with OAuth request| GOOGLE[Google Auth Server]
    GOOGLE -->|Customer logs into Google| CONSENT[Customer grants consent\nshare email, name]
    CONSENT -->|Authorization code returned| APP

    APP -->|Exchange code for tokens| GOOGLE
    GOOGLE -->|ID token + access token| APP

    APP --> CHECK{Does email\nalready exist?}

    CHECK -->|New customer| CREATE[Create new account\nlinked to Google identity]
    CHECK -->|Existing account with same email| LINK[Link Google identity\nto existing account]
    CHECK -->|Existing account, different email| PROMPT[Prompt: log in with\nexisting account first to link]

    CREATE & LINK --> SESSION[✅ Logged in session]
```

---

## 4. Consent Management (GDPR)

**Simple idea:** Before collecting any personal data, get explicit consent. Record exactly what was consented to, when, and allow customers to change their mind.

> Like a restaurant menu allergen form — you tick what you're OK with, and they keep a copy.

```mermaid
flowchart TD
    VISIT[Customer visits site] --> BANNER[Consent banner displayed\ncookies, analytics, marketing]

    BANNER --> CHOICE{Customer choice}

    CHOICE -->|Accept all| FULL_CONSENT[Full consent recorded\ntimestamp + version]
    CHOICE -->|Reject non-essential| MINIMAL[Minimal consent\nessential cookies only]
    CHOICE -->|Customise| GRANULAR[Granular consent\nper category]

    FULL_CONSENT & MINIMAL & GRANULAR --> LOG[(Consent Store\nwho, what, when, version)]

    LOG --> ENFORCE[Enforce across all systems\nanalytics, email, ads]

    USER2([Customer]) -->|Changes mind later| SETTINGS[Consent preference centre]
    SETTINGS -->|Withdraws consent| UPDATE[Update consent store\nstop processing]
    UPDATE --> ENFORCE
```

---

## 5. Risk-Based / Adaptive Authentication

**Simple idea:** Make login easy when everything looks normal. Add extra checks (MFA) when something looks suspicious — without blocking legitimate users.

> Like a bouncer: regular regulars walk straight in; someone new wearing a hoodie at 2am gets ID-checked.

```mermaid
flowchart TD
    LOGIN[Customer Login Attempt] --> SIGNALS[Collect Risk Signals]

    SIGNALS --> S1[Device: known or new?]
    SIGNALS --> S2[Location: usual country or new?]
    SIGNALS --> S3[Time: normal hours or unusual?]
    SIGNALS --> S4[Behaviour: normal typing speed?]
    SIGNALS --> S5[IP reputation: clean or flagged?]

    S1 & S2 & S3 & S4 & S5 --> ENGINE[Risk Score Engine]

    ENGINE --> RISK{Risk Level}

    RISK -->|Low risk\nknown device, usual location| ALLOW[✅ Allow — no extra step\nseamless experience]
    RISK -->|Medium risk\nnew device, same country| MFA[📱 Step-up MFA\nSMS or authenticator app]
    RISK -->|High risk\nnew country, unusual hour| BLOCK[🔴 Block + alert\nor challenge question]

    MFA -->|Passes| ALLOW
    MFA -->|Fails| BLOCK
```

---

## 6. Right to Erasure (GDPR — Right to be Forgotten)

**Simple idea:** Customers can ask you to delete all their data. You must do it completely, across all systems, within 30 days.

```mermaid
flowchart TD
    USER([Customer]) -->|Submits deletion request| PORTAL[CIAM Preference Centre\nor support ticket]

    PORTAL --> VERIFY[Verify identity\nconfirm it's really them]
    VERIFY --> TICKET[Erasure ticket created\ntimer starts — 30 day SLA]

    TICKET --> CIAM_DEL[CIAM: delete account + profile]
    TICKET --> DB_DEL[App databases: delete records]
    TICKET --> EMAIL_DEL[Email platform: unsubscribe + delete]
    TICKET --> ANALYTICS[Analytics: anonymise data]
    TICKET --> BACKUP[Backup: flag for removal on next cycle]
    TICKET --> THIRD[Third-party vendors: notify to delete]

    CIAM_DEL & DB_DEL & EMAIL_DEL & ANALYTICS & BACKUP & THIRD --> CONFIRM[Confirmation email sent to customer]
    CONFIRM --> LOG[(Erasure audit record\nretained for legal proof)]

    style TICKET fill:#e74c3c,color:#fff
```

> **Important:** You can retain data you are legally required to keep (e.g., financial records for tax compliance), even after a deletion request.

---

## 7. CIAM Tools at a Glance

| Tool | Strength | Best For |
|---|---|---|
| **Auth0** | Developer-friendly, easy integration | Startups, B2C apps |
| **Okta CIAM** | Enterprise-grade, scalable | Large consumer platforms |
| **AWS Cognito** | AWS-native, low cost | AWS-hosted apps |
| **Firebase Auth** | Simple, mobile-first | Mobile apps, Google ecosystem |
| **Ping Identity** | Flexible, enterprise | Complex hybrid environments |
| **Akamai Identity Cloud** | Massive scale, CDN-integrated | High-traffic consumer sites |

---

## Quick Reference Cheat Sheet

| Concept | One-liner | Why It Matters |
|---|---|---|
| **CIAM vs Enterprise IAM** | Customers ≠ employees | Different trust, scale, and UX needs |
| **Progressive profiling** | Collect data gradually | Less friction = better conversion |
| **Social login** | Use Google/Apple identity | Fewer passwords to manage |
| **Account linking** | Connect social to existing account | Prevents duplicate accounts |
| **Consent management** | Record what data you're allowed to use | GDPR compliance |
| **Risk-based auth** | Add MFA only when it's risky | Security without friction |
| **Right to erasure** | Delete all data on request | GDPR obligation, 30-day SLA |
