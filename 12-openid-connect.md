# OpenID Connect (OIDC)

[← Back to Index](README.md)

> OIDC is the identity layer built on top of OAuth 2.0 — it answers not just "what can you do?" but "who are you?"

---

## 1. What is OpenID Connect?

**OAuth 2.0** is an authorisation framework — it lets an app act on your behalf (e.g. post to Twitter). But OAuth 2.0 never defined a standard way to verify *who* you are.

**OpenID Connect (OIDC)** fixes that. It adds an identity layer on top of OAuth 2.0 so apps can authenticate users and get profile information in a standard way.

> Analogy: OAuth 2.0 gives you a hotel key card that opens your room. OIDC also tells the hotel your name, booking reference, and check-out date — it identifies you, not just grants you access.

```mermaid
flowchart LR
    OAUTH["OAuth 2.0\n'What can you do?'\nAuthorisation only\nIssues: Access Token"]
    OIDC["OpenID Connect\n'Who are you?'\nAuthentication + Authorisation\nIssues: ID Token + Access Token"]

    OAUTH -->|Extends| OIDC

    subgraph Adds["OIDC adds on top of OAuth 2.0"]
        A1[ID Token — JWT about the user]
        A2[UserInfo Endpoint — fetch profile]
        A3[openid scope — triggers OIDC]
        A4[Discovery — .well-known/openid-configuration]
    end
    OIDC --> Adds
```

---

## 2. How OIDC Extends OAuth 2.0

OIDC makes four key additions to OAuth 2.0:

```mermaid
flowchart TD
    subgraph OAuth2["OAuth 2.0 Baseline"]
        O1[Authorization Endpoint]
        O2[Token Endpoint]
        O3[Access Token]
        O4[Refresh Token]
    end

    subgraph OIDCAdds["OIDC Additions"]
        N1["ID Token\n(signed JWT about the user)"]
        N2["UserInfo Endpoint\nGET /userinfo → profile claims"]
        N3["openid scope\nTriggers identity layer"]
        N4["Discovery Document\n/.well-known/openid-configuration"]
        N5["JWKS URI\nPublic keys to verify ID Token"]
    end

    OAuth2 -->|OIDC builds on| OIDCAdds
```

| OAuth 2.0 | OIDC adds |
|---|---|
| Access Token (what you can do) | ID Token (who you are) |
| Token Endpoint | UserInfo Endpoint |
| `scope: read write` | `scope: openid profile email` |
| No standard discovery | `.well-known/openid-configuration` |

---

## 3. The ID Token

The **ID Token** is a signed JWT issued by the Identity Provider (IdP) after the user authenticates. It is proof of authentication — not for accessing APIs.

```mermaid
flowchart LR
    subgraph IDToken["ID Token Structure (JWT)"]
        H["Header\n{\n  alg: RS256,\n  kid: key-id-123\n}"]
        P["Payload\n{\n  iss: https://idp.example.com,\n  sub: user-abc-123,\n  aud: my-app-client-id,\n  exp: 1700000000,\n  iat: 1699996400,\n  auth_time: 1699996400,\n  nonce: abc123,\n  email: alice@example.com,\n  name: Alice Smith\n}"]
        S["Signature\nRSA/ECDSA signed\nby IdP private key"]
    end

    H --> DOT1[.] --> P --> DOT2[.] --> S
```

**Key claims explained:**

| Claim | Meaning | Why it matters |
|---|---|---|
| `iss` | Issuer — who issued the token | Validate it matches your IdP |
| `sub` | Subject — unique user ID | Stable identifier — never changes |
| `aud` | Audience — intended recipient | Must match your client ID |
| `exp` | Expiry timestamp | Reject tokens past this time |
| `iat` | Issued at timestamp | Token freshness check |
| `auth_time` | When user actually authenticated | For max_age enforcement |
| `nonce` | Random value from your request | Prevents replay attacks |
| `acr` | Authentication Context Class Ref | What auth method was used |
| `amr` | Authentication Methods References | e.g. `["pwd","mfa"]` |

```mermaid
flowchart LR
    IDT[ID Token\nWho you ARE\nPresented to the app\nNever sent to APIs]
    AT[Access Token\nWhat you can DO\nSent to resource servers / APIs]
    RT[Refresh Token\nGet new access tokens\nStored securely, long-lived]

    style IDT fill:#4a90d9,color:#fff
    style AT fill:#50c878,color:#fff
    style RT fill:#f0a500,color:#fff
```

---

## 4. OIDC Authentication Flows

### 4.1 Authorization Code Flow (Recommended)

The most secure flow — the token never passes through the browser.

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant App as Client App
    participant IdP as Identity Provider

    User->>Browser: Visit app
    Browser->>App: Request protected page
    App->>Browser: Redirect to IdP\n?response_type=code\n&scope=openid profile email\n&redirect_uri=...\n&state=xyz\n&nonce=abc

    Browser->>IdP: Follow redirect — login page
    User->>IdP: Enter credentials + MFA
    IdP->>Browser: Redirect back to app\n?code=AUTH_CODE&state=xyz

    Browser->>App: Deliver auth code

    App->>IdP: POST /token\nclient_id + client_secret + code
    IdP-->>App: ID Token + Access Token\n(+ Refresh Token if offline_access)

    App->>App: Validate ID Token\n(sig, iss, aud, exp, nonce)

    opt Get more user info
        App->>IdP: GET /userinfo\nAuthorization: Bearer {access_token}
        IdP-->>App: User profile claims
    end

    App->>Browser: Set session cookie
    Browser->>User: Access granted ✅
```

### 4.2 Authorization Code + PKCE (for public clients)

Used by SPAs and mobile apps that cannot keep a client secret.

```mermaid
sequenceDiagram
    participant App as Mobile/SPA App
    participant IdP

    App->>App: Generate code_verifier (random)\ncode_challenge = SHA256(code_verifier)

    App->>IdP: /authorize\n?code_challenge=...\n&code_challenge_method=S256\n&scope=openid profile

    IdP-->>App: auth code

    App->>IdP: POST /token\ncode + code_verifier (plain text)

    IdP->>IdP: SHA256(code_verifier)\nmust match stored code_challenge

    IdP-->>App: ID Token + Access Token ✅
```

### 4.3 Implicit Flow (Deprecated — do not use)

```mermaid
flowchart LR
    A[App redirects to IdP] -->|response_type=id_token token| B[IdP returns tokens in URL fragment]
    B --> C[Tokens exposed in browser history\nand server logs ❌]
    C --> D[Use Authorization Code + PKCE instead ✅]
```

---

## 5. UserInfo Endpoint

After authentication, the app can call the **UserInfo endpoint** using the access token to fetch additional profile claims.

```mermaid
sequenceDiagram
    participant App
    participant IdP as IdP /userinfo

    App->>IdP: GET /userinfo\nAuthorization: Bearer {access_token}

    IdP->>IdP: Validate access token\nCheck openid scope

    IdP-->>App: 200 OK\n{\n  "sub": "user-abc-123",\n  "name": "Alice Smith",\n  "given_name": "Alice",\n  "family_name": "Smith",\n  "email": "alice@example.com",\n  "email_verified": true,\n  "picture": "https://...",\n  "phone_number": "+447700000000",\n  "locale": "en-GB"\n}
```

> **Note:** Claims in the ID Token are minimal by design. The UserInfo endpoint provides richer profile data without bloating every token.

---

## 6. OIDC Discovery

OIDC providers publish a **discovery document** so clients can configure themselves automatically without hardcoding endpoints.

```mermaid
sequenceDiagram
    participant App
    participant IdP

    App->>IdP: GET /.well-known/openid-configuration

    IdP-->>App: {\n  "issuer": "https://idp.example.com",\n  "authorization_endpoint": ".../authorize",\n  "token_endpoint": ".../token",\n  "userinfo_endpoint": ".../userinfo",\n  "jwks_uri": ".../jwks.json",\n  "scopes_supported": ["openid","profile","email"],\n  "response_types_supported": ["code"],\n  "id_token_signing_alg_values_supported": ["RS256"]\n}

    App->>IdP: GET /jwks.json
    IdP-->>App: Public keys (JWK Set)\nUsed to verify ID Token signatures

    App->>App: Store endpoints + public keys\nReady to authenticate users
```

---

## 7. OIDC Scopes and Claims

The `openid` scope is **mandatory** — including it is what makes an OAuth 2.0 request become an OIDC authentication request.

```mermaid
flowchart TD
    subgraph Scopes["OIDC Scopes → Claims Returned"]
        S1["openid\n(required)\n→ sub"]
        S2["profile\n→ name, given_name, family_name,\n   picture, website, gender,\n   birthdate, locale, updated_at"]
        S3["email\n→ email, email_verified"]
        S4["phone\n→ phone_number,\n   phone_number_verified"]
        S5["address\n→ formatted, street_address,\n   locality, region, postal_code,\n   country"]
        S6["offline_access\n→ refresh_token issued"]
    end
```

**Example request with scopes:**
```
GET /authorize
  ?client_id=my-app
  &scope=openid profile email
  &response_type=code
  &redirect_uri=https://app.example.com/callback
  &nonce=random-value
```

---

## 8. OIDC in Practice

### 8.1 Social Login ("Sign in with Google")

```mermaid
sequenceDiagram
    actor User
    participant App
    participant Google as Google IdP

    User->>App: Click "Sign in with Google"
    App->>Google: Redirect\n?scope=openid email profile\n&client_id=...

    User->>Google: Google login (if not already signed in)
    Google->>User: Consent screen\n"App wants: your name and email"
    User->>Google: Allow

    Google->>App: Auth code
    App->>Google: Exchange code → ID Token + Access Token
    App->>App: Validate ID Token\nExtract sub, email, name

    App->>User: Logged in as Alice ✅
```

### 8.2 Enterprise SSO via Okta / Entra ID

```mermaid
sequenceDiagram
    actor Employee
    participant App as Corporate App
    participant Okta as Okta / Entra ID

    Employee->>App: Access app
    App->>Okta: Redirect to IdP\n?scope=openid profile email groups

    Employee->>Okta: SSO login (already authenticated via MFA)
    Okta->>App: ID Token\n(includes groups, department, role claims)

    App->>App: Map groups to app roles\nGrant access based on claims

    App->>Employee: Access granted ✅
```

### 8.3 Mobile App (Authorization Code + PKCE)

```mermaid
flowchart LR
    MOB([Mobile App]) -->|PKCE flow, opens system browser| IDP[IdP]
    IDP -->|Auth code via custom URL scheme| MOB
    MOB -->|Exchange code + verifier| IDP
    IDP -->|ID Token + Access Token| MOB
    MOB -->|Stored in secure keystore| TOKEN[(Tokens)]
```

---

## 9. OIDC vs SAML

```mermaid
flowchart TD
    Q1{What type of\napplication?} -->|Modern web / mobile / API| OIDC_REC[Use OIDC ✅]
    Q1 -->|Enterprise legacy / browser-only| SAML_REC[Use SAML ✅]

    Q2{Do you need\nmobile support?} -->|Yes| OIDC_REC
    Q2 -->|No| EITHER[Either works]

    Q3{Does the SP only\nsupport SAML?} -->|Yes| SAML_REC
    Q3 -->|No| OIDC_REC
```

| Feature | OIDC | SAML 2.0 |
|---|---|---|
| Token format | JWT (JSON) | XML Assertion |
| Transport | REST / HTTP | Browser redirect / POST |
| Mobile support | Native ✅ | Poor ❌ |
| API access | Built-in (access token) | Needs workaround |
| Setup complexity | Low | Medium–High |
| Enterprise adoption | Growing | Dominant (legacy) |
| Spec complexity | Moderate | High |
| Discovery | Automatic | Manual metadata exchange |

---

## 10. OIDC Security Considerations

```mermaid
flowchart TD
    RECEIVE[Receive ID Token] --> V1{Verify\nsignature\nusing JWKS}
    V1 -->|Invalid| REJECT([❌ Reject])
    V1 -->|Valid| V2{Check iss\nmatches IdP}
    V2 -->|No match| REJECT
    V2 -->|Match| V3{Check aud\nmatches client_id}
    V3 -->|No match| REJECT
    V3 -->|Match| V4{Check exp\nnot expired}
    V4 -->|Expired| REJECT
    V4 -->|Valid| V5{Check nonce\nmatches sent value}
    V5 -->|No match| REJECT
    V5 -->|Match| V6{Check auth_time\nif max_age used}
    V6 -->|Too old| REJECT
    V6 -->|Valid| ACCEPT([✅ Token Valid])
```

**Security checklist:**

| Check | Why |
|---|---|
| Verify signature (RS256/ES256) | Ensures token came from the real IdP |
| Validate `iss` | Prevents tokens from rogue IdPs |
| Validate `aud` | Prevents token substitution attacks |
| Check `exp` | Reject stale tokens |
| Validate `nonce` | Prevents replay attacks |
| Use PKCE | Protects auth code from interception |
| Validate `redirect_uri` | Prevents open redirect attacks |
| Never use `alg: none` | Unsigned tokens are always invalid |

---

## 11. Quick Reference

### OIDC Endpoints

| Endpoint | Purpose |
|---|---|
| `/.well-known/openid-configuration` | Discovery — all other endpoints |
| `/authorize` | Start authentication, get auth code |
| `/token` | Exchange code for ID + Access Token |
| `/userinfo` | Fetch user profile claims |
| `/jwks.json` | Public keys to verify ID Token |
| `/logout` | End session at IdP |

### ID Token Claims Cheat Sheet

| Claim | Required | Description |
|---|---|---|
| `iss` | ✅ | Issuer URL |
| `sub` | ✅ | Unique user identifier |
| `aud` | ✅ | Your client_id |
| `exp` | ✅ | Expiry (Unix timestamp) |
| `iat` | ✅ | Issued at (Unix timestamp) |
| `nonce` | If sent | Replay prevention value |
| `auth_time` | If requested | When user authenticated |
| `acr` | Optional | Auth context (e.g. MFA level) |
| `amr` | Optional | Methods used (pwd, mfa, etc.) |

### OIDC vs OAuth 2.0 vs SAML

| | OIDC | OAuth 2.0 | SAML 2.0 |
|---|---|---|---|
| Purpose | AuthN + AuthZ | AuthZ only | AuthN |
| Token | ID Token (JWT) + Access Token | Access Token | XML Assertion |
| Standard body | OpenID Foundation | IETF | OASIS |
| Discovery | ✅ Automatic | ❌ | ❌ Manual metadata |
| Mobile-friendly | ✅ | ✅ | ❌ |
| Enterprise legacy | Growing | N/A | Dominant |
