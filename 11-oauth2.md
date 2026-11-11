# OAuth 2.0

> The industry-standard framework for delegated authorisation — letting apps access resources on a user's behalf without ever seeing their password.

---

## 1. What is OAuth 2.0?

**OAuth 2.0** (RFC 6749, 2012) is a delegated authorisation framework. It lets a user grant a third-party application limited access to their resources — without giving the app their password.

**Real-world analogy — the hotel key card:**
> You check into a hotel. The front desk (Authorisation Server) gives you a key card (Access Token) that opens only your room and the gym (Scopes). It does NOT give you the master key (your password). When you check out, the key card stops working (token expiry/revocation).

```mermaid
flowchart LR
    USER([🏨 Guest / User]) -->|Checks in\nproves identity| DESK[Front Desk\nAuthorisation Server]
    DESK -->|Issues key card\nwith limited access| KEY[🔑 Key Card\nAccess Token]
    KEY -->|Opens only| ROOM[Your Room\nand Gym\nScoped Resources]
    KEY -->|Cannot open| MASTER[❌ Master Key\nYour Password]
```

> **Important:** OAuth 2.0 is an **authorisation** framework — it controls what an app can *do*. It is NOT an authentication protocol (it does not prove who you are). That is the job of **OpenID Connect (OIDC)**, which is built on top of OAuth 2.0.

---

## 2. Core Roles

OAuth 2.0 defines four roles that interact in every flow:

```mermaid
flowchart TD
    RO([👤 Resource Owner\nThe user who owns the data\nand grants permission])
    CLIENT([📱 Client\nThe app requesting access\ne.g. a mobile app or web app])
    AS([🏛️ Authorisation Server\nIssues tokens after\nauthenticating the user\ne.g. Okta, Azure Entra ID, Auth0])
    RS([🗄️ Resource Server\nThe API holding the data\ne.g. Google Drive API, GitHub API])

    RO -->|Grants permission to| CLIENT
    CLIENT -->|Requests token from| AS
    AS -->|Issues access token to| CLIENT
    CLIENT -->|Calls API with token| RS
    RS -->|Returns protected data| CLIENT
```

| Role | Who they are | Example |
|---|---|---|
| **Resource Owner** | The user | You, logging in |
| **Client** | The app | A photo-printing app |
| **Authorisation Server** | Issues tokens | Google Accounts, Okta |
| **Resource Server** | Holds the data | Google Photos API |

---

## 3. Tokens

### Access Token
- Short-lived credential (minutes to hours)
- Sent with every API call in the `Authorization: Bearer <token>` header
- Scoped to specific permissions

### Refresh Token
- Long-lived credential (days to months)
- Used to silently get a new access token when the current one expires
- Never sent to the resource server — only to the authorisation server

### Token Response Anatomy

```mermaid
flowchart LR
    REQ[Client requests token] -->|POST /token| AS[Authorisation Server]
    AS -->|Returns| RESP["{
  access_token: eyJhbGci...
  token_type: Bearer
  expires_in: 3600
  scope: read:emails write:files
  refresh_token: xK9p2mQ...
}"]
```

### Bearer vs DPoP (Proof-of-Possession)

```mermaid
flowchart TD
    subgraph Bearer["Bearer Token (Standard)"]
        B1[Token stolen from network] --> B2[Attacker replays token\nfrom any device ❌]
    end

    subgraph DPoP["DPoP — Proof of Possession"]
        D1[Token bound to client's\npublic/private key pair]
        D2[Each request signed\nwith private key]
        D1 --> D2 --> D3[Stolen token useless\nwithout the private key ✅]
    end
```

### Scopes

Scopes limit what the access token can do. The user sees and approves them on the consent screen.

| Scope | What it allows |
|---|---|
| `read:profile` | Read user's name and email |
| `write:files` | Create and edit files |
| `delete:posts` | Delete social media posts |
| `openid` | OIDC — authenticate the user |

---

## 4. Grant Types (Flows) in Depth

### 4.1 Authorization Code + PKCE (Recommended for all user-facing apps)

The most secure flow for apps where a human user is present. PKCE prevents auth code interception attacks.

```mermaid
sequenceDiagram
    actor User
    participant Client as Client App
    participant AS as Authorisation Server
    participant RS as Resource Server

    User->>Client: Click "Connect with GitHub"

    Note over Client: Generate code_verifier (random string)<br/>code_challenge = SHA256(code_verifier)

    Client->>AS: GET /authorize?<br/>client_id=...&redirect_uri=...&<br/>scope=read:profile&<br/>code_challenge=abc123&<br/>code_challenge_method=S256

    AS->>User: Show login + consent screen
    User->>AS: Login and approve scopes

    AS->>Client: Redirect to redirect_uri?code=AUTH_CODE

    Client->>AS: POST /token<br/>{ code: AUTH_CODE,<br/>  code_verifier: original_secret,<br/>  client_id: ... }

    Note over AS: Verify SHA256(code_verifier) == stored code_challenge ✅

    AS->>Client: { access_token, refresh_token, expires_in }

    Client->>RS: GET /api/profile<br/>Authorization: Bearer access_token

    RS->>Client: { user data }
```

---

### 4.2 Client Credentials (Machine-to-Machine)

No user involved. Used for service-to-service communication.

```mermaid
sequenceDiagram
    participant SVC as Service A\n(Client)
    participant AS as Authorisation Server
    participant API as Service B\n(Resource Server)

    SVC->>AS: POST /token<br/>{ grant_type: client_credentials,<br/>  client_id, client_secret,<br/>  scope: internal:read }

    AS->>SVC: { access_token, expires_in }

    SVC->>API: GET /internal/data<br/>Authorization: Bearer access_token

    API->>SVC: { data }
```

---

### 4.3 Device Authorization Grant (Browserless Devices)

For smart TVs, CLI tools, and IoT devices that cannot open a browser.

```mermaid
sequenceDiagram
    participant DEV as Smart TV / CLI
    participant AS as Authorisation Server
    participant USER as User's Phone/Browser

    DEV->>AS: POST /device_authorization<br/>{ client_id, scope }

    AS->>DEV: { device_code, user_code: "WDJB-MJHT",<br/>  verification_uri: "example.com/activate",<br/>  expires_in: 1800 }

    DEV->>USER: Display: "Go to example.com/activate\nEnter code: WDJB-MJHT"

    USER->>AS: Opens browser → enters code → logs in → approves

    loop Poll every 5 seconds
        DEV->>AS: POST /token { device_code, grant_type: device_code }
        AS->>DEV: { error: authorization_pending }
    end

    AS->>DEV: { access_token, refresh_token }
```

---

### 4.4 Implicit Flow ⛔ Deprecated

```mermaid
flowchart LR
    OLD["❌ Implicit Flow (Deprecated)
    Token returned directly in URL fragment
    #access_token=eyJ...
    
    Risks:
    • Token exposed in browser history
    • Token leaked via Referer header
    • Token visible to JavaScript on page
    
    DO NOT USE for new applications
    Use Authorization Code + PKCE instead"]
```

---

### 4.5 Resource Owner Password Credentials ⛔ Deprecated

```mermaid
flowchart LR
    OLD2["❌ ROPC Flow (Deprecated)
    App collects username + password directly
    
    Risks:
    • App can store or misuse credentials
    • Defeats the purpose of OAuth 2.0
    • No SSO, no MFA support
    
    DO NOT USE for new applications
    Use Authorization Code + PKCE instead"]
```

---

## 5. Scopes and Consent

```mermaid
flowchart TD
    APP([App requests access]) -->|Specifies scopes| AUTH[Authorisation Server]
    AUTH -->|Shows consent screen| USER([User])

    USER -->|Reviews what app will access| SCREEN["🖥️ Consent Screen
    
    'PrintPhotos app wants to:
    ✅ View your Google Photos
    ✅ Read your profile name
    ❌ NOT: Delete files
    ❌ NOT: Access Drive'"]

    SCREEN -->|Approves| GRANT[Token issued\nwith approved scopes]
    SCREEN -->|Denies| DENY[❌ Access denied\nNo token issued]

    GRANT --> API[App calls API\nwith scoped token]
```

**Incremental authorisation** — request only the scopes you need right now. Ask for more later when the user triggers a feature that needs them.

```mermaid
flowchart LR
    SIGNUP[User signs up] -->|Request only| S1[read:profile]
    S1 --> BASIC[Basic features work]
    BASIC -->|User clicks Upload| ASK2[Request write:files]
    ASK2 --> FULL[Upload feature works]
```

---

## 6. Token Introspection and Revocation

### Introspection (RFC 7662)
The Resource Server checks with the Authorisation Server whether a token is still valid.

```mermaid
sequenceDiagram
    participant CLIENT as Client
    participant RS as Resource Server
    participant AS as Authorisation Server

    CLIENT->>RS: GET /api/data<br/>Authorization: Bearer <token>

    RS->>AS: POST /introspect<br/>{ token: <token> }

    AS->>RS: { active: true,<br/>  scope: "read:data",<br/>  exp: 1735689600,<br/>  sub: "user123" }

    RS->>CLIENT: 200 OK { data }
```

### Revocation (RFC 7009)
Immediately invalidate an access or refresh token.

```mermaid
sequenceDiagram
    participant CLIENT as Client
    participant AS as Authorisation Server

    Note over CLIENT: User logs out or\nrevokes app access

    CLIENT->>AS: POST /revoke<br/>{ token: <refresh_token>,<br/>  client_id: ... }

    AS->>CLIENT: 200 OK

    Note over AS: Token marked invalid\nAll subsequent use fails
```

---

## 7. PKCE Deep Dive

PKCE (Proof Key for Code Exchange, RFC 7636) prevents an attacker who intercepts the auth code from exchanging it for a token.

```mermaid
sequenceDiagram
    participant CLIENT as Client App
    participant AS as Authorisation Server
    participant ATK as 🏴‍☠️ Attacker

    Note over CLIENT: Step 1: Generate PKCE pair
    Note over CLIENT: code_verifier = "dBjftJeZ4CVP-mB92K..."<br/>(random 43-128 char string)
    Note over CLIENT: code_challenge = BASE64URL(SHA256(code_verifier))<br/>= "E9Melhoa2OwvFrEMTJguCHaoeK..."

    CLIENT->>AS: GET /authorize?code_challenge=E9Mel...&code_challenge_method=S256
    AS->>CLIENT: Redirect with auth code=AUTH_CODE

    ATK-->>CLIENT: Intercepts AUTH_CODE ⚠️

    ATK->>AS: POST /token { code: AUTH_CODE } ← No code_verifier!
    AS->>ATK: ❌ 400 Error — code_verifier missing

    CLIENT->>AS: POST /token { code: AUTH_CODE, code_verifier: dBjftJeZ... }
    Note over AS: SHA256("dBjftJeZ...") == "E9Melhoa..." ✅
    AS->>CLIENT: ✅ { access_token, refresh_token }
```

---

## 8. OAuth 2.0 Security Best Practices (RFC 9700)

```mermaid
mindmap
  root((OAuth 2.0\nSecurity))
    Token Hygiene
      Short-lived access tokens
      Rotate refresh tokens on use
      Bind tokens with DPoP or mTLS
    Client Security
      Always use PKCE
      Validate redirect_uri exactly
      Use state parameter against CSRF
    Server Security
      Strict redirect_uri matching
      Rate-limit token endpoint
      Implement token revocation
    Storage
      Never store tokens in localStorage
      Use httpOnly cookies or secure storage
      Never log tokens
```

**OAuth 2.0 attack surface:**

```mermaid
flowchart TD
    subgraph Attacks["Common OAuth 2.0 Attacks"]
        A1[Open Redirector\nMalicious redirect_uri\nsteals auth code]
        A2[CSRF on Auth Endpoint\nForges authorisation request\nif state param missing]
        A3[Auth Code Interception\nIntercepted code redeemed\nbefore client — fixed by PKCE]
        A4[Token Leakage\nTokens in logs, URLs,\nor insecure storage]
        A5[Refresh Token Theft\nLong-lived token stolen\nfrom storage]
    end

    subgraph Mitigations["Mitigations"]
        M1[Exact redirect_uri validation]
        M2[Require state parameter]
        M3[Always use PKCE]
        M4[httpOnly cookies, no localStorage]
        M5[Rotate + bind refresh tokens]
    end

    A1 --> M1
    A2 --> M2
    A3 --> M3
    A4 --> M4
    A5 --> M5
```

---

## 9. OAuth 2.0 vs OAuth 1.0

| Feature | OAuth 1.0 | OAuth 2.0 |
|---|---|---|
| **Year** | 2010 (RFC 5849) | 2012 (RFC 6749) |
| **Signatures** | Required on every request (HMAC-SHA1) | Not required — relies on HTTPS |
| **Token types** | Single access token | Access + Refresh tokens |
| **Flows** | One flow | Multiple grant types |
| **Mobile support** | Poor | Designed for mobile and SPAs |
| **Complexity** | Complex to implement | Simpler (but more footguns) |
| **HTTPS requirement** | Optional (signatures compensate) | Mandatory |
| **Status** | Legacy, being replaced | Current standard |

---

## 10. Quick Reference

### Grant Type Selection

```mermaid
flowchart TD
    START{Who is\nthe client?} -->|Human user\npresent| BROWSER{Has a\nbrowser?}
    START -->|No user\nmachine only| CC[Client Credentials\n4.2]

    BROWSER -->|Yes| AUTH[Authorization Code\n+ PKCE\n4.1]
    BROWSER -->|No\nTV / CLI / IoT| DEVICE[Device Authorization\nGrant 4.3]
```

### OAuth 2.0 Endpoints

| Endpoint | Purpose |
|---|---|
| `/authorize` | Start the auth flow — redirect user here |
| `/token` | Exchange code or credentials for tokens |
| `/introspect` | Check if a token is active |
| `/revoke` | Invalidate a token |
| `/.well-known/oauth-authorization-server` | Discovery document |
| `/jwks` | Public keys to verify signed tokens |

### Security Checklist

- [ ] Use PKCE for all public clients
- [ ] Validate `redirect_uri` as exact match
- [ ] Use `state` parameter to prevent CSRF
- [ ] Store tokens in `httpOnly` cookies, not `localStorage`
- [ ] Use short-lived access tokens (≤ 1 hour)
- [ ] Rotate refresh tokens on every use
- [ ] Implement token revocation on logout
- [ ] Never log tokens or put them in URLs
- [ ] Use HTTPS everywhere — no exceptions
