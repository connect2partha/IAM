# Protocols and Standards

[← Back to Index](README.md)

> Protocols are the **agreed languages** that systems use to handle identity, authentication, and authorization. Without them, every app would need its own custom login system.

Think of protocols like electrical plugs — they define a standard so any device (identity provider) can plug into any socket (application) around the world.

---

## 4.1 Federated Identity Protocols

Federated identity lets users log in once and access multiple systems — even across different organisations — without sharing passwords.

---

### SAML 2.0 — Security Assertion Markup Language

SAML is an XML-based protocol for exchanging authentication data between an **Identity Provider (IdP)** and a **Service Provider (SP)**.

> Like a notarised letter: the IdP writes a signed XML "assertion" saying "I vouch for this user." The SP trusts the letter because it's signed by a known IdP.

**SP-Initiated SSO** (most common — user starts at the app):

```mermaid
sequenceDiagram
    participant U as 👤 User
    participant SP as 🌐 Service Provider (App)
    participant IdP as 🏛️ Identity Provider (Okta/ADFS)

    U->>SP: Visit app (not logged in)
    SP-->>SP: Generate AuthnRequest
    SP->>U: Redirect to IdP with AuthnRequest
    U->>IdP: Arrive at IdP login page
    U->>IdP: Enter credentials + MFA
    IdP-->>IdP: Validate credentials
    IdP-->>IdP: Create signed SAML Assertion (XML)
    IdP->>U: POST SAML Response to SP's ACS URL
    U->>SP: Browser delivers SAML Response
    SP-->>SP: Verify signature using IdP's public certificate
    SP-->>SP: Extract user attributes from assertion
    SP->>U: ✅ Logged in — session created
```

**IdP-Initiated SSO** (user starts at the IdP portal, e.g., company intranet):

```mermaid
sequenceDiagram
    participant U as 👤 User
    participant IdP as 🏛️ Identity Provider
    participant SP as 🌐 Service Provider

    U->>IdP: Login to company portal
    IdP->>U: Show list of available apps
    U->>IdP: Click on target app
    IdP-->>IdP: Create SAML Assertion (no AuthnRequest)
    IdP->>U: POST SAML Response to SP
    U->>SP: Browser delivers SAML Response
    SP->>U: ✅ Logged in
```

---

### OAuth 2.0 — Open Authorization

OAuth is an **authorization** protocol — it lets a third-party app access resources on your behalf, without giving it your password.

> Like a valet key: you give the valet a key that opens the car door but cannot open the boot or start the engine.

**Authorization Code Flow + PKCE** (for web and mobile apps):

```mermaid
sequenceDiagram
    participant U as 👤 User
    participant App as 📱 Client App
    participant AS as 🔐 Authorization Server
    participant API as 🌐 Resource Server (API)

    App-->>App: Generate code_verifier and code_challenge (PKCE)
    U->>App: Click "Login with Google"
    App->>U: Redirect to Authorization Server\nwith code_challenge
    U->>AS: Login and consent screen
    U-->>AS: Approve permissions
    AS->>App: Return authorization code
    App->>AS: Exchange code + code_verifier for tokens
    AS-->>AS: Verify code_verifier matches code_challenge
    AS->>App: Return access_token + refresh_token
    App->>API: Call API with access_token
    API->>App: Return protected data ✅
```

**Client Credentials Flow** (machine-to-machine, no user involved):

```mermaid
sequenceDiagram
    participant SVC as ⚙️ Service / Microservice
    participant AS as 🔐 Authorization Server
    participant API as 🌐 Resource Server

    SVC->>AS: POST /token\nclient_id + client_secret + grant_type=client_credentials
    AS-->>AS: Validate client credentials
    AS->>SVC: Return access_token
    SVC->>API: Call API with access_token
    API->>SVC: Return data ✅
```

**Device Authorization Flow** (for TVs, CLIs with no browser):

```mermaid
sequenceDiagram
    participant D as 📺 Device / CLI
    participant AS as 🔐 Authorization Server
    participant U as 👤 User (on phone/laptop)

    D->>AS: Request device code
    AS->>D: Return device_code + user_code + verification_uri
    D->>U: Display: "Go to example.com/activate, enter: ABCD-1234"
    U->>AS: Visit URL on phone, enter user_code, login + approve
    D-->>AS: Poll for access token every few seconds
    AS->>D: Return access_token once user approves ✅
```

---

### OpenID Connect (OIDC)

OIDC is built **on top of OAuth 2.0** — it adds **authentication**. While OAuth only says "you can access this resource," OIDC also says "here's who the user is."

> OAuth = car valet key. OIDC = valet key **plus** a business card that says who gave the key.

```mermaid
sequenceDiagram
    participant U as 👤 User
    participant App as 📱 Client App
    participant OP as 🔐 OpenID Provider (IdP)
    participant UI as 🌐 UserInfo Endpoint

    U->>App: Click "Sign in with Google"
    App->>U: Redirect to OP with scope=openid+profile+email
    U->>OP: Login and consent
    OP->>App: Return authorization code
    App->>OP: Exchange code for tokens
    OP->>App: Return id_token (JWT) + access_token

    Note over App: id_token contains: sub, email, name, iat, exp
    App-->>App: Verify id_token signature
    App-->>App: Extract user identity from id_token

    opt Need more user info
        App->>UI: Call /userinfo with access_token
        UI->>App: Return full user profile
    end

    App->>U: ✅ Logged in as user@example.com
```

---

### When to Use SAML vs OAuth vs OIDC

```mermaid
flowchart TD
    Q{What do you need?} --> A1[Enterprise SSO\nbetween companies]
    Q --> A2[Let a 3rd-party app\naccess my API]
    Q --> A3[Login with Google/Facebook\nin your app]
    Q --> A4[Machine-to-machine\nAPI access]

    A1 --> SAML[🏛️ SAML 2.0\nXML-based, legacy enterprise]
    A3 --> OIDC[🔐 OpenID Connect\nOAuth 2.0 + identity layer]
    A2 --> OAUTH[⚙️ OAuth 2.0\nAuthorization only]
    A4 --> CC[⚙️ OAuth 2.0\nClient Credentials Flow]
```

| Protocol | Purpose | Token Format | Best For |
|---|---|---|---|
| **SAML 2.0** | Authentication + SSO | XML Assertion | Enterprise B2B, legacy apps |
| **OAuth 2.0** | Authorization (access delegation) | Access token (opaque or JWT) | API access, mobile apps |
| **OIDC** | Authentication (built on OAuth) | ID token (JWT) + access token | Consumer login, modern web/mobile |

---

## 4.2 Directory and Provisioning

### LDAP — Lightweight Directory Access Protocol

LDAP is a protocol for querying and managing a **directory** — a hierarchical database of users, groups, and resources.

```mermaid
flowchart TD
    subgraph DIT["LDAP Directory Information Tree (DIT)"]
        ROOT[dc=company,dc=com]
        ROOT --> OU1[ou=Users]
        ROOT --> OU2[ou=Groups]
        ROOT --> OU3[ou=Computers]

        OU1 --> U1["cn=Alice Smith
        dn: cn=Alice Smith,ou=Users,dc=company,dc=com
        mail: alice@company.com
        memberOf: cn=Finance,ou=Groups"]

        OU1 --> U2["cn=Bob Jones
        dn: cn=Bob Jones,ou=Users,dc=company,dc=com"]

        OU2 --> G1["cn=Finance
        member: cn=Alice Smith,..."]
    end
```

**LDAP Search Query example:**
```
Base DN:  ou=Users,dc=company,dc=com
Filter:   (&(objectClass=person)(department=Finance))
Result:   Returns all people in Finance department
```

---

### Active Directory (AD)

AD is Microsoft's implementation of LDAP + Kerberos — the dominant enterprise directory service.

```mermaid
flowchart TD
    subgraph Forest["AD Forest (top-level trust boundary)"]
        subgraph DomA["Domain: corp.company.com"]
            DC1[Domain Controller 1]
            DC2[Domain Controller 2]
            U1([Users & Computers])
            G1[Group Policy Objects]
        end

        subgraph DomB["Domain: eu.company.com"]
            DC3[Domain Controller 3]
            U2([EU Users])
        end

        DomA <-->|Bidirectional Trust| DomB
    end
```

---

### Kerberos Authentication

Kerberos is used by Active Directory for authenticating users on a Windows network. It uses **tickets** instead of passwords.

> Like a theme park wristband system: show your ticket at the gate, get a wristband for each ride — you never show your password to each ride.

```mermaid
sequenceDiagram
    participant U as 👤 User
    participant AS as 🔐 AS (Authentication Service)
    participant TGS as 🎫 TGS (Ticket Granting Service)
    participant SVC as 🖥️ File Server

    Note over U,AS: Step 1 — Get Ticket Granting Ticket (TGT)
    U->>AS: Send username (password hash used to encrypt)
    AS->>U: Return TGT (encrypted with KDC secret key)

    Note over U,TGS: Step 2 — Get Service Ticket
    U->>TGS: Present TGT + request access to File Server
    TGS-->>TGS: Validate TGT
    TGS->>U: Return Service Ticket (for File Server only)

    Note over U,SVC: Step 3 — Access the Service
    U->>SVC: Present Service Ticket
    SVC-->>SVC: Decrypt ticket using its own secret
    SVC->>U: ✅ Access Granted — password never sent to service
```

---

### SCIM 2.0 — System for Cross-domain Identity Management

SCIM automates user provisioning and deprovisioning between systems (e.g., Workday → Okta → Salesforce).

```mermaid
sequenceDiagram
    participant HR as 👥 HR System (Workday)
    participant IDP as 🏛️ Identity Provider (Okta)
    participant APP as 🌐 Target App (Salesforce)

    Note over HR,APP: New Employee Joins
    HR->>IDP: SCIM POST /Users\n{name, email, department, role}
    IDP-->>IDP: Create identity record
    IDP->>APP: SCIM POST /Users\n{mapped attributes}
    APP->>IDP: 201 Created — user provisioned ✅

    Note over HR,APP: Employee Changes Department
    HR->>IDP: SCIM PATCH /Users/{id}\n{department: "Finance"}
    IDP->>APP: SCIM PATCH /Users/{id}\n{update attributes}

    Note over HR,APP: Employee Leaves
    HR->>IDP: SCIM DELETE /Users/{id}
    IDP->>APP: SCIM DELETE /Users/{id}
    APP->>IDP: 204 No Content — user deprovisioned ✅
```

---

### RADIUS — Remote Authentication Dial-In User Service

RADIUS handles authentication for network access — commonly used for Wi-Fi, VPN, and 802.1X port-based access control.

```mermaid
sequenceDiagram
    participant D as 💻 Device
    participant AP as 📡 Access Point / Switch
    participant RADIUS as 🔐 RADIUS Server (NPS)
    participant AD as 🏛️ Active Directory

    D->>AP: Connect to corporate Wi-Fi
    AP->>RADIUS: RADIUS Access-Request\n(username, EAP credentials)
    RADIUS->>AD: Validate user credentials
    AD->>RADIUS: Valid ✓
    RADIUS->>AP: RADIUS Access-Accept\n(+ VLAN assignment, policies)
    AP->>D: ✅ Network access granted on VLAN 10
```

---

## 4.3 Token Formats

### JWT — JSON Web Token

A JWT is a compact, URL-safe token consisting of three Base64-encoded parts separated by dots.

```
eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJ1c2VyMTIzIn0.signature
      HEADER                    PAYLOAD              SIGNATURE
```

```mermaid
flowchart LR
    subgraph JWT["JWT Structure: header.payload.signature"]
        H["📋 HEADER
        {
          alg: RS256,
          typ: JWT
        }
        Base64URL encoded"]

        P["📦 PAYLOAD (Claims)
        {
          sub: user123,
          email: alice@co.com,
          role: admin,
          iat: 1699999999,
          exp: 1700003599
        }
        Base64URL encoded"]

        S["🔏 SIGNATURE
        RS256(
          base64(header) +
          '.' +
          base64(payload),
          private_key
        )"]
    end

    H -->|dot| P -->|dot| S
```

**JWT `alg` pitfalls:**

```mermaid
flowchart TD
    ALG{Algorithm\nchoice} --> NONE["alg: none\n❌ No signature — anyone\ncan forge the token"]
    ALG --> HS256["alg: HS256\nSymmetric — same secret\nfor sign + verify\n⚠️ Server must keep secret safe"]
    ALG --> RS256["alg: RS256\nAsymmetric — private key signs,\npublic key verifies\n✅ Recommended"]

    NONE --> DANGER[Never accept alg:none\nCheck explicitly]
    HS256 --> CAUTION[OK if secret is\nrotated regularly]
    RS256 --> SAFE[✅ Best practice]
```

---

### SAML Assertion Structure

A SAML Assertion is an XML document signed by the IdP that describes who the user is.

```mermaid
flowchart TD
    subgraph SAML["SAML Assertion (XML)"]
        A1[Issuer — who created this\nIdP entity ID]
        A2[Subject — who this is about\nNameID: alice@company.com]
        A3[Conditions — validity window\nNotBefore / NotOnOrAfter]
        A4[AuthnStatement — how they authenticated\nPasswordProtectedTransport]
        A5[AttributeStatement — user attributes\nemail, groups, role]
        A6[Signature — IdP signs the whole thing\nXMLDSig with X.509 cert]
    end
```

---

### Opaque Tokens and Introspection

An **opaque token** is a random string with no embedded data — the resource server must call the authorization server to find out what it means.

```mermaid
sequenceDiagram
    participant App as 📱 Client App
    participant API as 🌐 Resource Server
    participant AS as 🔐 Authorization Server

    App->>API: Call API with opaque token\nAuthorization: Bearer a1b2c3d4e5...
    API-->>API: Token is opaque — cannot decode locally
    API->>AS: POST /introspect\n{token: a1b2c3d4e5...}
    AS->>API: Return token info\n{active: true, sub: user123, scope: read:invoices, exp: ...}
    API-->>API: Validate scope and expiry
    API->>App: Return requested data ✅
```

| Token Type | Pros | Cons |
|---|---|---|
| **JWT (self-contained)** | No introspection call needed; fast | Cannot revoke before expiry |
| **Opaque** | Revocable instantly | Requires introspection call every request |

---

## Quick Reference Cheat Sheet

| Protocol / Standard | One-liner | Key Rule |
|---|---|---|
| **SAML 2.0** | XML-based enterprise SSO | IdP signs assertions; SP verifies with public cert |
| **OAuth 2.0** | Authorization framework — delegate access | Use PKCE for public clients; never implicit flow |
| **OIDC** | Authentication layer on top of OAuth 2.0 | ID token = who you are; access token = what you can do |
| **LDAP** | Query/manage hierarchical directory | DN identifies entries; filter queries find them |
| **Kerberos** | Ticket-based auth used by Active Directory | TGT from KDC → Service Ticket → access service |
| **SCIM 2.0** | Automate user provisioning | Standard REST API for create/update/delete users |
| **RADIUS** | Network access authentication | Used for Wi-Fi, VPN, 802.1X |
| **JWT** | Self-contained signed token | Use RS256; validate `alg`, `exp`, `iss`, `aud` |
| **Opaque token** | Random string; meaning stored server-side | Requires introspection; revocable |
| **SAML Assertion** | Signed XML vouching for the user | Verify signature; check NotOnOrAfter |
