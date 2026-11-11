# Single Sign-On (SSO)

> Log in once, access everything — no more password fatigue.

---

## What is SSO?

Imagine a **theme park wristband**. You buy one ticket at the gate, get a wristband, and all rides let you in without buying another ticket. SSO works the same way — log in once, and all your apps just work.

Without SSO, every app is a separate ride with its own ticket booth.

```mermaid
flowchart LR
    subgraph WithoutSSO["❌ Without SSO — Login to Every App"]
        U1([👤 User]) -->|Login 1| A1[Email App]
        U1 -->|Login 2| A2[HR Portal]
        U1 -->|Login 3| A3[CRM]
        U1 -->|Login 4| A4[Finance Tool]
        U1 -->|Login 5| A5[Collaboration App]
    end

    subgraph WithSSO["✅ With SSO — Login Once, Access All"]
        U2([👤 User]) -->|One Login| IDP[Identity Provider\nSSO Portal]
        IDP --> B1[Email App]
        IDP --> B2[HR Portal]
        IDP --> B3[CRM]
        IDP --> B4[Finance Tool]
        IDP --> B5[Collaboration App]
    end
```

---

## 1. IdP vs SP Roles

Two players make SSO work:

| Role | Full Name | Simple Meaning | Example |
|---|---|---|---|
| **IdP** | Identity Provider | The gatekeeper who verifies who you are | Okta, Azure Entra ID, Google Workspace |
| **SP** | Service Provider | The app you want to use | Salesforce, Slack, Jira |

> Think of IdP as your **passport office** and SP as the **foreign country**. The country (SP) trusts the passport (token) issued by your government (IdP).

```mermaid
flowchart LR
    U([👤 User]) --> SP[Service Provider\ne.g. Salesforce]
    SP -->|I don't know you, go prove identity| IDP[Identity Provider\ne.g. Okta]
    IDP -->|User logs in here| U
    IDP -->|Issues token: This is Alice, she's verified| SP
    SP -->|Welcome, Alice!| U

    style IDP fill:#4a90d9,color:#fff
    style SP fill:#50c878,color:#000
```

---

## 2. SSO Login Flow (Step by Step)

Here is what actually happens behind the scenes when you click "Sign In":

```mermaid
sequenceDiagram
    actor User
    participant SP as Service Provider<br/>(e.g. Slack)
    participant IdP as Identity Provider<br/>(e.g. Okta)

    User->>SP: Visit app / click Login
    SP->>SP: Check — does user have a session?
    SP-->>User: No session. Redirect to IdP
    User->>IdP: Arrive at IdP login page
    IdP->>User: Show login form (username + MFA)
    User->>IdP: Enter credentials
    IdP->>IdP: Validate credentials
    IdP-->>SP: Send signed token (SAML assertion or OIDC token)
    SP->>SP: Validate token signature
    SP-->>User: Welcome! Session created ✅
```

---

## 3. SSO Session vs Application Session

There are **two separate sessions** running when you use SSO. Understanding both prevents confusion around logout.

```mermaid
flowchart TD
    subgraph IdPSession["🔑 IdP Session (SSO Session)"]
        IS[Created when you log in to IdP\nTypically 8–24 hours\nStored as a cookie on IdP domain\nShared across ALL apps]
    end

    subgraph AppSession["🎫 Application Session (SP Session)"]
        AS1[App A Session\n1–2 hours]
        AS2[App B Session\n30 minutes]
        AS3[App C Session\n4 hours]
    end

    IdPSession -->|Authenticates & creates| AS1
    IdPSession -->|Authenticates & creates| AS2
    IdPSession -->|Authenticates & creates| AS3

    note1[/"⚠️ App session can expire even when SSO session is active\nApp will silently re-authenticate using active IdP session"/]
```

---

## 4. SSO Logout — Front-Channel vs Back-Channel

Logging out is trickier than logging in. You must kill **all sessions** — both at the IdP and at every SP.

### Front-Channel Logout
The browser is used as the messenger — the IdP redirects the user's browser to each SP's logout URL.

```mermaid
sequenceDiagram
    actor User
    participant IdP
    participant SP1 as Slack (SP1)
    participant SP2 as Jira (SP2)

    User->>IdP: Click Logout
    IdP->>IdP: Invalidate IdP session
    IdP-->>User: Redirect browser to SP1 logout URL
    User->>SP1: Browser hits SP1 logout endpoint
    SP1->>SP1: Invalidate SP1 session
    SP1-->>User: Redirect browser to SP2 logout URL
    User->>SP2: Browser hits SP2 logout endpoint
    SP2->>SP2: Invalidate SP2 session
    SP2-->>User: Redirect back to IdP
    IdP-->>User: Logout complete ✅
```

### Back-Channel Logout
The IdP calls each SP directly — no browser involved. More reliable, but SP must expose a logout endpoint.

```mermaid
sequenceDiagram
    actor User
    participant IdP
    participant SP1 as Slack (SP1)
    participant SP2 as Jira (SP2)

    User->>IdP: Click Logout
    IdP->>IdP: Invalidate IdP session
    IdP->>SP1: HTTP POST logout token (server-to-server)
    SP1->>SP1: Invalidate SP1 session
    IdP->>SP2: HTTP POST logout token (server-to-server)
    SP2->>SP2: Invalidate SP2 session
    IdP-->>User: Logout complete ✅

    Note over IdP,SP2: No browser redirects needed<br/>More reliable — works even if browser is closed
```

| | Front-Channel | Back-Channel |
|---|---|---|
| Messenger | User's browser | Direct server call |
| Reliability | Lower — browser must visit each SP | Higher — server-to-server |
| Privacy | Browser URLs visible | Hidden from browser |
| Complexity | Simpler to implement | Requires logout endpoint on each SP |

---

## 5. Social Login (OAuth with Google, Apple, GitHub)

**Social login** lets users reuse their existing accounts (Google, Apple, GitHub) instead of creating new credentials.

> Think of it as "vouching". You say "I know this person" and your friend (Google) vouches for you at the new app.

```mermaid
flowchart TD
    U([👤 User]) -->|Clicks Sign in with Google| APP[Your App\nService Provider]
    APP -->|Redirect to Google with client_id + scope| GOOGLE[Google\nOAuth Server]
    GOOGLE -->|Show consent screen| U
    U -->|User approves| GOOGLE
    GOOGLE -->|Redirect back with auth code| APP
    APP -->|Exchange auth code for tokens\n(server-to-server)| GOOGLE
    GOOGLE -->|Returns: access_token + id_token| APP
    APP -->|Extract identity from id_token| IDENTITY[User: alice@gmail.com\nName: Alice]
    IDENTITY -->|Find or create local account| DB[(App User Database)]
    DB --> SESSION([✅ Session Created — Alice is logged in])
```

### What information does social login share?

```mermaid
mindmap
  root((Social Login\nData Shared))
    Basic Profile
      Name
      Email
      Profile picture
    Optional Scopes
      Calendar access
      Contacts
      Drive files
    Not Shared
      Password
      Payment info
      Private messages
```

---

## 6. Enterprise SSO vs Customer SSO (CIAM)

These look similar but serve very different audiences.

```mermaid
flowchart TD
    subgraph Enterprise["🏢 Enterprise SSO"]
        E1[Audience: Employees & contractors]
        E2[Scale: Hundreds to tens of thousands]
        E3[Identity source: Corporate HR / AD]
        E4[Trust level: High — known users]
        E5[UX priority: Security & compliance]
        E6[Examples: Okta, Azure Entra ID, Ping]
    end

    subgraph CIAM["🌍 Customer IAM — CIAM"]
        C1[Audience: External customers / consumers]
        C2[Scale: Millions of users]
        C3[Identity source: Self-registration + social login]
        C4[Trust level: Lower — anonymous users]
        C5[UX priority: Frictionless experience]
        C6[Examples: Auth0, AWS Cognito, Firebase Auth]
    end

    SSO[Single Sign-On\nCore Concept] --> Enterprise
    SSO --> CIAM
```

| Dimension | Enterprise SSO | CIAM |
|---|---|---|
| Users | Employees | Customers |
| Scale | Thousands | Millions |
| Identity Source | HR / AD | Self-registration, social |
| Onboarding | IT-provisioned | Self-service |
| Key concern | Compliance, audit | Conversion, UX |
| MFA | Mandatory | Optional / adaptive |

---

## Putting It All Together

```mermaid
flowchart LR
    U([👤 User]) --> CHOICE{Login\nmethod?}

    CHOICE -->|Enterprise user| IDP[IdP\nOkta / Entra ID]
    CHOICE -->|Social login| SOCIAL[Google / Apple\nGitHub OAuth]

    IDP -->|SAML / OIDC token| SP1[App A]
    IDP -->|SAML / OIDC token| SP2[App B]
    IDP -->|SAML / OIDC token| SP3[App C]

    SOCIAL -->|id_token| APP[Your App]

    SP1 & SP2 & SP3 --> SESSION[✅ SSO Session\nOne login, all apps]
    APP --> SESSION2[✅ App Session\nSocial identity linked]

    SESSION --> LOGOUT{Logout?}
    LOGOUT -->|Front-channel| FCL[Browser visits each SP]
    LOGOUT -->|Back-channel| BCL[IdP calls each SP directly]
```

---

## Quick Reference Cheat Sheet

| Concept | One-liner | Key Rule |
|---|---|---|
| **IdP** | Verifies identity | Issues the signed token |
| **SP** | Provides the service | Trusts and consumes the token |
| **SSO Session** | IdP-level session | Shared across all apps, longer-lived |
| **App Session** | SP-level session | Per-app, shorter-lived |
| **Front-channel logout** | Browser-based logout relay | Simpler but less reliable |
| **Back-channel logout** | Server-to-server logout | Reliable, requires logout endpoint |
| **Social login** | Use Google/Apple/GitHub identity | OAuth + OIDC under the hood |
| **Enterprise SSO** | Employees, compliance-first | Okta, Entra ID, Ping |
| **CIAM** | Customers, UX-first | Auth0, Cognito, Firebase |
