# OAuth 1.0

> The original delegated authorisation standard — how apps were first allowed to act on behalf of users without ever seeing their passwords.

---

## 1. What is OAuth 1.0?

Before OAuth existed, apps that needed access to your data would simply ask for your username and password — storing them, using them forever, and giving you no way to revoke access without changing your password.

**OAuth 1.0** (RFC 5849, 2010) solved this by introducing *delegated authorisation*: a user can grant a third-party app limited access to their data on another service, without sharing credentials.

> **Analogy:** A valet key — it only unlocks the car door and starts the engine. It cannot open the glove box, the boot, or the garage. You hand the valet the special key, not your master key.

```mermaid
flowchart LR
    subgraph Before["❌ Before OAuth"]
        USER1([You]) -->|Give username\n& password| APP1[Third-Party App]
        APP1 -->|Full access\nforever| API1[(Your Data)]
    end

    subgraph After["✅ With OAuth 1.0"]
        USER2([You]) -->|Approve limited\naccess| APP2[Third-Party App]
        APP2 -->|Access Token\nlimited scope| API2[(Your Data)]
        USER2 -->|Can revoke\nanytime| APP2
    end
```

**Why it was created:**
- Twitter needed apps to post tweets on users' behalf without storing passwords
- Early APIs (Flickr, Google, Netflix) faced the same problem
- No standard existed — every service invented its own scheme

---

## 2. Core Concepts

```mermaid
flowchart TD
    subgraph Roles["The Three Roles"]
        RO([🧑 Resource Owner\nThe User\nOwns the data, grants permission])
        C([📱 Client\nThe Third-Party App\nWants access to the data])
        S([🖥️ Server\nThe API + Auth Server\nHolds the data, issues tokens])
    end

    subgraph Credentials["Credentials & Tokens"]
        CK[Consumer Key\nIdentifies the app — public]
        CS[Consumer Secret\nApp's private signing key]
        RT[Request Token\nTemporary — used during auth flow]
        V[Verifier\nOne-time code proving user approved]
        AT[Access Token\nLong-lived — used to call the API]
    end

    C -->|Uses| CK & CS
    RO -->|Grants| V
    S -->|Issues| RT & AT
```

| Term | What it is |
|---|---|
| **Resource Owner** | The user whose data is being accessed |
| **Client** | The app requesting access |
| **Server** | The service hosting the data and handling auth |
| **Consumer Key** | Public ID of the client app |
| **Consumer Secret** | Secret key the client uses to sign requests |
| **Request Token** | Temporary token used only during the auth handshake |
| **Verifier** | Proof that the user approved access |
| **Access Token** | The credential used to call the API after authorisation |

---

## 3. OAuth 1.0 Flow (3-Legged)

The "3 legs" are the three parties: Resource Owner, Client, and Server.

```mermaid
sequenceDiagram
    actor User as 🧑 Resource Owner
    participant App as 📱 Client App
    participant Server as 🖥️ Auth + API Server

    Note over App,Server: Step 1 — Get a Request Token
    App->>Server: POST /request_token\n(signed with Consumer Key + Secret)
    Server-->>App: oauth_token (Request Token)\noauth_token_secret

    Note over User,Server: Step 2 — User Authorises
    App->>User: Redirect to\n/authorize?oauth_token=...
    User->>Server: Logs in and approves access
    Server-->>User: Redirect back to App callback URL\nwith oauth_verifier

    Note over App,Server: Step 3 — Exchange for Access Token
    App->>Server: POST /access_token\n(Request Token + Verifier,\nsigned with Consumer Secret)
    Server-->>App: oauth_token (Access Token)\noauth_token_secret

    Note over App,Server: Step 4 — Call the API
    App->>Server: GET /api/resource\n(signed with Access Token)
    Server-->>App: Protected resource data ✅
```

---

## 4. Request Signing

Every API call in OAuth 1.0 is **cryptographically signed**. This means even if someone intercepts the request, they cannot replay or forge it.

```mermaid
flowchart TD
    subgraph BUILD["Building the Signature Base String"]
        M[HTTP Method\ne.g. GET] --> SBS
        URL[Normalised URL\ne.g. https://api.example.com/data] --> SBS
        PARAMS[Sorted Parameters\noauth_consumer_key, oauth_nonce,\noauth_timestamp, oauth_token, ...] --> SBS
        SBS[Signature Base String\nGET&https%3A%2F%2F...&oauth_...]
    end

    subgraph SIGN["Signing"]
        SBS --> HMAC[HMAC-SHA1\nusing Consumer Secret\n+ Token Secret as key]
        HMAC --> SIG[oauth_signature\ne.g. tR3+Ty81lMeYAr/Fid0kMTYa/WM=]
    end

    subgraph REQUEST["Final Request Header"]
        SIG --> HDR["Authorization: OAuth\noauth_consumer_key='...',\noauth_nonce='...',\noauth_signature='...',\noauth_timestamp='...',\noauth_token='...'"]
    end
```

**Why nonce and timestamp?**
- **Nonce** — a unique random value per request; the server rejects any repeated nonce
- **Timestamp** — requests older than a few minutes are rejected
- Together they prevent **replay attacks** — capturing a valid request and re-sending it

---

## 5. OAuth 1.0a (Revised)

The original OAuth 1.0 had a **session fixation vulnerability**: an attacker could inject their own Request Token into the user's authorisation flow and hijack the resulting Access Token.

**OAuth 1.0a** (the revision) fixed this:

```mermaid
sequenceDiagram
    participant App
    participant Server
    actor User

    Note over App,Server: OAuth 1.0 (vulnerable)
    App->>Server: POST /request_token\n(no callback URL)
    Server-->>App: Request Token
    Note over App: Attacker could substitute\ntheir own Request Token here ❌

    Note over App,Server: OAuth 1.0a (fixed)
    App->>Server: POST /request_token\noauth_callback=https://myapp.com/callback
    Server-->>App: Request Token\n+ oauth_callback_confirmed=true ✅
    Note over App,Server: Callback is locked in — attacker\ncannot substitute a different token
    App->>User: Redirect with confirmed Request Token
    User->>Server: Approves
    Server-->>App: Redirects to confirmed callback\nwith Verifier
```

> **Rule:** Always use OAuth **1.0a**, never the original 1.0.

---

## 6. Problems with OAuth 1.0

```mermaid
mindmap
  root((OAuth 1.0\nProblems))
    Cryptographic Complexity
      Every request must be signed
      HMAC-SHA1 hard to implement correctly
      Parameter encoding rules are strict
      Many libraries got it wrong
    No Token Standards
      No standard token format
      No standard expiry mechanism
      Refresh not standardised
    Mobile Unfriendly
      Native apps cannot keep Consumer Secret safe
      No suitable flow for mobile / desktop apps
    Mixed Concerns
      Mixes authentication and authorisation
      No standard for user identity
    Scalability
      Stateful — server must store nonces
      Cannot use stateless bearer tokens
```

---

## 7. OAuth 1.0 vs OAuth 2.0 — Why the Change

```mermaid
flowchart LR
    subgraph O1["OAuth 1.0"]
        A1[Signed every request\nHMAC-SHA1]
        A2[One flow for all clients]
        A3[Stateful nonce tracking]
        A4[No token format standard]
        A5[Complex — few correct implementations]
    end

    subgraph O2["OAuth 2.0"]
        B1[TLS handles transport security\nBearer tokens — no signing]
        B2[Multiple flows\nfor different client types]
        B3[Stateless JWT tokens\npossible]
        B4[JWT as standard token format]
        B5[Simpler — many correct\nimplementations]
    end

    O1 -->|Replaced by| O2
```

| Feature | OAuth 1.0 | OAuth 2.0 |
|---|---|---|
| **Request signing** | Required (HMAC-SHA1) | Not required — relies on TLS |
| **Token format** | Opaque string | Bearer token (often JWT) |
| **Client types** | One flow for all | Separate flows (web, mobile, machine) |
| **Mobile support** | Poor | Good (PKCE flow) |
| **Complexity** | High | Lower |
| **Nonce tracking** | Required server-side | Not needed |
| **Still used?** | Legacy only | Current standard |

---

## 8. Where OAuth 1.0 Still Exists

```mermaid
flowchart LR
    LEGACY[OAuth 1.0 Still Found In] --> T1[Twitter / X API v1\nlegacy endpoints]
    LEGACY --> T2[Some older enterprise\nand government APIs]
    LEGACY --> T3[NetSuite, SAP\nlegacy integrations]

    LEGACY --> AVOID[❌ Not recommended\nfor new implementations]
    AVOID --> USE[✅ Use OAuth 2.0\n+ PKCE instead]
```

> If you encounter OAuth 1.0 in an integration, check whether the provider offers an OAuth 2.0 endpoint. Almost all do now.

---

## 9. Quick Reference

### OAuth 1.0 Three-Legged Flow Summary

| Step | Action | Who does it |
|---|---|---|
| 1 | Request a temporary Request Token | Client → Server |
| 2 | Redirect user to authorisation page | Client → User |
| 3 | User approves, gets Verifier | User → Server → Client |
| 4 | Exchange Request Token + Verifier for Access Token | Client → Server |
| 5 | Use Access Token to call protected API | Client → Server |

### Key Terms Cheat Sheet

| Term | Simple Meaning |
|---|---|
| **Consumer Key** | Your app's public ID |
| **Consumer Secret** | Your app's private signing key — never expose |
| **Request Token** | Temporary token for the auth handshake only |
| **Verifier** | Proof the user said yes |
| **Access Token** | The credential your app uses to call the API |
| **Token Secret** | Paired with Access Token for signing API calls |
| **Nonce** | Random unique value — prevents request replay |
| **HMAC-SHA1** | The signing algorithm used to protect every request |
| **3-legged** | Involves three parties: user, app, and server |
| **OAuth 1.0a** | The fixed version — always use this, never plain 1.0 |
