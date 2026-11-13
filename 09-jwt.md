# JSON Web Tokens (JWT)

[← Back to Index](README.md)

> A compact, self-contained token format used across OAuth 2.0, OIDC, and API authentication — in plain language with diagrams.

---

## 1. What is a JWT?

A **JSON Web Token (JWT)** is a compact, URL-safe token that carries information (called *claims*) about a subject — typically a user or service.

Think of it like a **signed letter**:
- Anyone can open and read the letter (Base64-encoded, not encrypted)
- Only the original signer could have produced the valid signature
- The recipient verifies the signature before trusting the contents

```mermaid
flowchart LR
    ISSUER([🏛️ Identity Provider\nIssues the JWT]) -->|Signs with private key| JWT[📄 JWT Token]
    JWT -->|Sent to| CLIENT([👤 User / App])
    CLIENT -->|Presents JWT| API([🔒 API / Resource])
    API -->|Verifies signature\nwith public key| TRUST([✅ Trusts the claims])
```

A JWT has exactly **three parts**, separated by dots:

```
eyJhbGciOiJSUzI1NiJ9  .  eyJzdWIiOiJ1c2VyMTIzIn0  .  SflKxwRJSMeKKF2QT4fwpMeJf36P
      HEADER          .             PAYLOAD             .         SIGNATURE
```

---

## 2. JWT Structure

### 2.1 The Three Parts

```mermaid
flowchart TD
    subgraph JWT["JWT = Header . Payload . Signature"]
        H["HEADER\n(Base64URL encoded)\n---\n{\n  'alg': 'RS256',\n  'typ': 'JWT'\n}"]
        P["PAYLOAD\n(Base64URL encoded)\n---\n{\n  'sub': 'user123',\n  'iss': 'https://auth.example.com',\n  'exp': 1735689600,\n  'roles': ['admin']\n}"]
        S["SIGNATURE\n---\nRS256(\n  base64url(header) + '.' +\n  base64url(payload),\n  privateKey\n)"]
    end

    H --> DOT1[.] --> P --> DOT2[.] --> S
```

### 2.2 Header
Declares the token type and signing algorithm.

| Field | Meaning | Example |
|---|---|---|
| `alg` | Signing algorithm | `RS256`, `HS256`, `ES256` |
| `typ` | Token type | `JWT` |
| `kid` | Key ID — which key was used | `key-2024-01` |

### 2.3 Payload (Claims)
The claims are the data the token carries. Base64URL-encoded — **not encrypted** — so never put secrets here.

### 2.4 Signature
Computed over `base64url(header) + "." + base64url(payload)` using the algorithm declared in the header. Prevents tampering.

---

## 3. JWT Signing Algorithms

### Symmetric — HS256 (HMAC-SHA256)
One shared secret is used to both **sign** and **verify**. Simple but requires the secret to be shared with every verifier.

### Asymmetric — RS256 / ES256
A **private key signs**; a **public key verifies**. The IdP keeps the private key secret; anyone can get the public key from the JWKS endpoint.

```mermaid
flowchart LR
    subgraph Symmetric["HS256 — Symmetric"]
        S_IDP([IdP]) -->|Signs with SECRET| S_JWT[JWT]
        S_JWT -->|Verifies with same SECRET| S_API([API])
        S_NOTE[⚠️ Secret must be\nshared with every verifier]
    end

    subgraph Asymmetric["RS256 — Asymmetric"]
        A_IDP([IdP]) -->|Signs with PRIVATE KEY| A_JWT[JWT]
        A_JWT -->|Verifies with PUBLIC KEY\nfrom JWKS endpoint| A_API([API])
        A_NOTE[✅ Private key never leaves IdP\nPublic key is openly shareable]
    end
```

```mermaid
sequenceDiagram
    participant API
    participant IdP

    API->>IdP: GET /.well-known/jwks.json
    IdP-->>API: { keys: [ { kty: RSA, n: ..., e: ... } ] }

    Note over API: JWT arrives in request

    API->>API: Decode header → find kid
    API->>API: Look up matching public key
    API->>API: Verify signature with public key
    API->>API: ✅ Signature valid — trust claims
```

### The `alg: none` Vulnerability
An attacker removes the signature and sets `"alg": "none"`. A naive library may skip verification entirely.

> **Rule:** Always explicitly allowlist accepted algorithms. Never accept `alg: none`.

---

## 4. JWT Validation Steps

Always validate every step — skipping any one step opens an attack vector.

```mermaid
flowchart TD
    RECV([JWT Received]) --> D1{Decode\nbase64url parts}
    D1 -->|Malformed| REJECT([❌ Reject])

    D1 --> D2{Verify\nsignature}
    D2 -->|Invalid| REJECT

    D2 --> D3{Check exp\nnot expired?}
    D3 -->|Expired| REJECT

    D3 --> D4{Check nbf\ntoken active yet?}
    D4 -->|Too early| REJECT

    D4 --> D5{Check iss\ntrusted issuer?}
    D5 -->|Unknown issuer| REJECT

    D5 --> D6{Check aud\ntoken for this service?}
    D6 -->|Wrong audience| REJECT

    D6 --> D7{Check jti\nnot replayed?}
    D7 -->|Already used| REJECT

    D7 --> ACCEPT([✅ Token Valid\nTrust the claims])
```

---

## 5. JWT Claims in Depth

### Registered Claims (standardised by RFC 7519)

| Claim | Full Name | Meaning | Example |
|---|---|---|---|
| `iss` | Issuer | Who issued the token | `https://auth.example.com` |
| `sub` | Subject | Who the token is about | `user-uuid-123` |
| `aud` | Audience | Who the token is for | `api.example.com` |
| `exp` | Expiration | Unix timestamp — token invalid after this | `1735689600` |
| `iat` | Issued At | Unix timestamp — when issued | `1735686000` |
| `nbf` | Not Before | Token not valid before this time | `1735686000` |
| `jti` | JWT ID | Unique token identifier — prevents replay | `uuid-abc-123` |

### Custom / Private Claims

```mermaid
flowchart LR
    subgraph Payload["Example Decoded JWT Payload"]
        ISS["iss: 'https://auth.acme.com'"]
        SUB["sub: 'emp-00847'"]
        AUD["aud: 'crm.acme.com'"]
        EXP["exp: 1735689600  ← expires"]
        IAT["iat: 1735686000  ← issued"]
        JTI["jti: 'a1b2c3d4'  ← unique ID"]
        ROLE["roles: ['sales-manager']  ← custom"]
        DEPT["department: 'EMEA-Sales'  ← custom"]
        EMAIL["email: 'alice@acme.com'  ← custom"]
    end
```

> **Size warning:** JWTs are sent on **every HTTP request** in the Authorization header. Keep payloads small — use a reference (user ID) rather than embedding large datasets.

---

## 6. JWT Lifecycle

```mermaid
flowchart LR
    LOGIN([User Logs In]) --> ISSUE[IdP issues\nAccess Token\nshort-lived: 5–60 min\n+\nRefresh Token\nlong-lived: days–weeks]

    ISSUE --> USE[Client uses\nAccess Token\non each API call]

    USE --> CHECK{Access Token\nExpired?}
    CHECK -->|No| USE
    CHECK -->|Yes| REFRESH[Client sends\nRefresh Token]

    REFRESH --> RCHECK{Refresh Token\nvalid & not revoked?}
    RCHECK -->|Yes| NEWTOKEN[New Access Token\nissued]
    RCHECK -->|No| RELOGIN([User must log in again])

    NEWTOKEN --> USE
```

```mermaid
sequenceDiagram
    actor User
    participant Client
    participant AuthServer
    participant API

    User->>Client: Login
    Client->>AuthServer: POST /token (credentials)
    AuthServer-->>Client: access_token (15min) + refresh_token (7d)

    Client->>API: GET /data\nAuthorization: Bearer <access_token>
    API-->>Client: 200 OK + data

    Note over Client,API: 15 minutes later — access token expired

    Client->>API: GET /data\nAuthorization: Bearer <expired_token>
    API-->>Client: 401 Unauthorized

    Client->>AuthServer: POST /token\ngrant_type=refresh_token
    AuthServer-->>Client: new access_token + (optionally) new refresh_token

    Client->>API: GET /data\nAuthorization: Bearer <new_access_token>
    API-->>Client: 200 OK + data
```

---

## 7. Where JWTs Are Used

```mermaid
mindmap
  root((JWT Usage))
    OAuth 2.0
      Access tokens
      Refresh tokens
    OpenID Connect
      ID tokens
      UserInfo responses
    API Authentication
      Bearer token in\nAuthorization header
      API gateway validation
    SSO
      SAML alternative\nfor modern apps
    Microservices
      Service-to-service\ntoken propagation
    CLI Tools
      Short-lived tokens\nfor CI/CD pipelines
```

**In an HTTP request:**

```
GET /api/v1/invoices HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
```

---

## 8. JWT Security Pitfalls

### `alg: none` Attack
Attacker modifies header to `"alg": "none"` and strips the signature. Vulnerable libraries skip verification.

### Algorithm Confusion Attack (RS256 → HS256)
Server expects RS256. Attacker changes header to HS256 and signs with the **public key** as the HMAC secret. A confused library verifies it successfully.

```mermaid
sequenceDiagram
    participant Attacker
    participant Server

    Note over Attacker: Knows server's RS256 public key (it's public!)

    Attacker->>Attacker: Craft payload with elevated roles
    Attacker->>Attacker: Set alg: HS256
    Attacker->>Attacker: Sign with public key as HMAC secret

    Attacker->>Server: Send forged JWT

    Note over Server: Vulnerable library sees alg=HS256\nUses public key to verify HMAC\n✅ Verification passes!

    Server-->>Attacker: 200 OK — unauthorized access granted
```

### Full Pitfall Table

| Pitfall | Risk | Mitigation |
|---|---|---|
| `alg: none` | Signature bypass | Allowlist algorithms; reject `none` |
| Algorithm confusion | Forge tokens | Pin the expected algorithm server-side |
| Weak HS256 secret | Brute-force offline | Use 256-bit+ random secret |
| No `exp` claim | Token never expires | Always set a short expiry |
| Sensitive data in payload | Data exposure | JWT is encoded, not encrypted — use opaque tokens or JWE for sensitive data |
| No revocation | Stolen token stays valid | Use short expiry + refresh token rotation; maintain a blocklist for critical apps |
| Missing `aud` check | Token accepted by wrong service | Always validate `aud` matches your service |

---

## 9. JWT vs Opaque Tokens

```mermaid
flowchart TD
    subgraph JWT_FLOW["JWT — Self-Contained"]
        J1([API receives JWT]) --> J2[Decode + verify\nsignature locally]
        J2 --> J3[Check claims\nexp, iss, aud]
        J3 --> J4([✅ No network call needed])
        J4 --> J5[⚠️ Cannot revoke\nuntil expiry]
    end

    subgraph OPQ_FLOW["Opaque Token — Reference"]
        O1([API receives token]) --> O2[POST /introspect\nto auth server]
        O2 --> O3([Auth server checks\nif token is active])
        O3 --> O4([✅ Instantly revocable])
        O4 --> O5[⚠️ Network call\non every request]
    end
```

| | JWT | Opaque Token |
|---|---|---|
| **Validation** | Local — no network call | Requires introspection endpoint call |
| **Revocation** | Difficult — wait for expiry or maintain blocklist | Instant — delete from server store |
| **Performance** | Fast — stateless | Slower — introspection call per request |
| **Self-contained** | Yes — carries user info | No — just a reference key |
| **Best for** | Short-lived access tokens, microservices | Long-lived sessions, high-security apps |

---

## 10. Quick Reference

### Registered Claims Cheat Sheet

| Claim | Type | Required? | Purpose |
|---|---|---|---|
| `iss` | String (URI) | Recommended | Identifies the issuer |
| `sub` | String | Recommended | Identifies the subject |
| `aud` | String / Array | Recommended | Intended recipient(s) |
| `exp` | NumericDate | Always set | Expiry time |
| `iat` | NumericDate | Recommended | Issue time |
| `nbf` | NumericDate | Optional | Not-valid-before time |
| `jti` | String | Recommended | Unique token ID (replay prevention) |

### Algorithm Selection Guide

| Use Case | Recommended Algorithm | Why |
|---|---|---|
| Single server, simple setup | HS256 | Symmetric, fast — only one verifier |
| Multiple services / microservices | RS256 or ES256 | Asymmetric — services only need public key |
| High-security / IoT | ES256 | Smaller key size, same security as RS256 |
| Never | `none` | No signature — completely unsafe |

### JWT Security Checklist

- [ ] Algorithm explicitly pinned server-side (never accept `alg: none`)
- [ ] `exp` claim always set (keep access tokens short: 5–15 min)
- [ ] `aud` claim validated against your service identifier
- [ ] `iss` claim validated against trusted issuer list
- [ ] `jti` tracked to prevent token replay (for high-security flows)
- [ ] No sensitive PII or secrets in payload (Base64 is not encryption)
- [ ] JWKS endpoint used for public key distribution (not hardcoded keys)
- [ ] Refresh tokens rotated on use and stored securely
- [ ] Token blocklist in place for logout / critical revocation flows
