# Authentication

> Authentication answers the question: **"Can you prove you are who you claim to be?"**

Think of it like boarding a flight:
- Your **passport** is your identity
- The **security agent checking it** is authentication
- Once through, you're **authorized** to board your gate

---

## 2.1 Authentication Factors

There are four categories of proof you can use to authenticate. Using more than one makes it much harder for attackers.

```mermaid
flowchart TD
    AUTH([🔐 Authentication]) --> F1
    AUTH --> F2
    AUTH --> F3
    AUTH --> F4

    F1["🧠 Something You KNOW
    Passwords, PINs,
    Security questions"]

    F2["📱 Something You HAVE
    Hardware token, Smart card,
    TOTP app, Phone"]

    F3["👁️ Something You ARE
    Fingerprint, Face scan,
    Iris, Voice"]

    F4["📍 Somewhere You ARE
    GPS location, IP address,
    Network segment"]

    style F1 fill:#4a90d9,color:#fff
    style F2 fill:#7b68ee,color:#fff
    style F3 fill:#50c878,color:#fff
    style F4 fill:#f0a500,color:#fff
```

| Factor | Examples | Weakness |
|---|---|---|
| Something you know | Password, PIN | Can be guessed, stolen, phished |
| Something you have | Phone, YubiKey | Can be lost or stolen |
| Something you are | Fingerprint, face | Cannot be changed if compromised |
| Somewhere you are | IP address, GPS | Can be spoofed with VPN/proxies |

> **Multi-Factor Authentication (MFA)** = combining 2 or more different factor types. Two passwords is NOT MFA — they're the same factor type.

---

## 2.2 Authentication Mechanisms

### Password Policies

A strong password policy reduces the risk of guessing attacks.

```mermaid
flowchart LR
    subgraph Good["✅ Good Password Policy"]
        G1[Minimum 12 characters]
        G2[Mix of upper, lower, numbers, symbols]
        G3[Check against breach databases\nHaveIBeenPwned]
        G4[No forced periodic rotation\nunless compromised]
        G5[Block known bad passwords\npassword123, qwerty]
    end

    subgraph Bad["❌ Outdated / Weak Policy"]
        B1[Only 6–8 characters]
        B2[Force rotation every 90 days]
        B3[No breach checking]
        B4[Allow common passwords]
    end
```

---

### Multi-Factor Authentication (MFA)

**Simple idea:** Even if someone steals your password, they still need your second factor.

```mermaid
flowchart TD
    U([👤 User]) --> P[Enter Username & Password]
    P --> V1{Password\ncorrect?}
    V1 -->|No| DENY1([❌ Access Denied])
    V1 -->|Yes| MFA[Trigger 2nd Factor]

    MFA --> OPT1[📱 TOTP Code\nfrom Authenticator App]
    MFA --> OPT2[🔔 Push Notification\nApprove on phone]
    MFA --> OPT3[🔑 Hardware Key\nYubiKey tap]
    MFA --> OPT4[💬 SMS Code\nweakest option]

    OPT1 --> V2{2nd factor\nvalid?}
    OPT2 --> V2
    OPT3 --> V2
    OPT4 --> V2

    V2 -->|No| DENY2([❌ Access Denied])
    V2 -->|Yes| ACCESS([✅ Access Granted])
```

**Why SMS is weak:**
- Phone numbers can be hijacked via **SIM swapping** (attacker convinces carrier to transfer your number)
- SMS codes can be **intercepted** on SS7 network vulnerabilities
- Use TOTP apps (Google Authenticator, Authy) or hardware keys instead

---

### TOTP — Time-Based One-Time Password

TOTP generates a 6-digit code that changes every 30 seconds based on a shared secret and the current time.

```mermaid
sequenceDiagram
    participant U as 👤 User
    participant A as 📱 Authenticator App
    participant S as 🖥️ Server

    Note over U,S: Setup Phase (one time)
    S->>U: Display QR code containing secret key
    U->>A: Scan QR code
    A-->>A: Store secret key

    Note over U,S: Login Phase (every time)
    U->>S: Enter username + password
    S-->>S: Validate password ✓
    S->>U: Prompt for TOTP code

    A-->>A: Compute: TOTP = HMAC(secret, floor(time/30))
    U->>S: Enter 6-digit TOTP code

    S-->>S: Compute same TOTP independently
    S-->>S: Compare codes (allow ±1 window for clock drift)

    alt Codes match
        S->>U: ✅ Access Granted
    else Codes do not match
        S->>U: ❌ Access Denied
    end
```

---

### Passwordless Authentication — FIDO2 / WebAuthn / Passkeys

**Simple idea:** Instead of a password, your device holds a private key. The server holds the public key. You prove identity by signing a challenge — the private key never leaves your device.

> Like a wax seal: anyone can verify it's yours (public key), but only you have the stamp (private key).

```mermaid
flowchart TD
    subgraph Register["📋 Registration (One Time)"]
        R1([User clicks Register]) --> R2[Device creates key pair\nPrivate key stays on device\nPublic key sent to server]
        R2 --> R3[Server stores public key\nlinked to your account]
    end

    subgraph Login["🔐 Login (Every Time)"]
        L1([User clicks Login]) --> L2[Server sends a random challenge]
        L2 --> L3[Device asks for biometric\nor PIN to unlock private key]
        L3 --> L4[Device signs the challenge\nwith private key]
        L4 --> L5[Server verifies signature\nusing stored public key]
        L5 -->|Valid| L6([✅ Access Granted])
        L5 -->|Invalid| L7([❌ Access Denied])
    end
```

**Passkeys** = FIDO2 credentials that can sync across your devices via iCloud Keychain, Google Password Manager, etc.

---

### Hardware Security Keys (YubiKey, FIDO U2F)

A physical USB/NFC device you tap to authenticate. The private key is stored inside the hardware chip and **cannot be extracted**.

```mermaid
flowchart LR
    U([User]) -->|Plugs in / taps| K[🔑 YubiKey]
    K -->|Challenges| S[Server]
    S -->|Sends challenge| K
    K -->|Signs with hardware-bound key| S
    S -->|Verifies signature| ACCESS([✅ Authenticated])

    ATK([🏴‍☠️ Attacker steals password]) -->|Tries to login| S
    S -->|Requires hardware key tap| FAIL([❌ Cannot proceed\nno physical key])
```

---

### Certificate-Based Authentication (mTLS)

Both the client and server present certificates to prove their identity — commonly used for machine-to-machine authentication.

```mermaid
sequenceDiagram
    participant C as 💻 Client
    participant S as 🖥️ Server
    participant CA as 🏛️ Certificate Authority

    Note over C,CA: Setup — certificates issued by CA
    CA->>C: Issue client certificate (signed by CA)
    CA->>S: Issue server certificate (signed by CA)

    Note over C,S: mTLS Handshake
    C->>S: Hello + client certificate
    S-->>S: Verify client cert is signed by trusted CA ✓
    S->>C: Hello + server certificate
    C-->>C: Verify server cert is signed by trusted CA ✓
    C->>S: Encrypted session established
    S->>C: ✅ Mutual trust confirmed — data flows
```

---

## 2.3 Session Management

Once authenticated, the server needs to remember you are logged in. This is done with **session tokens**.

### Session Tokens vs Cookies

| Term | What it is |
|---|---|
| **Session token** | A random string the server uses to identify your logged-in session |
| **Cookie** | The browser's way of storing and sending the session token back to the server |

---

### Fixed vs Sliding Session Expiry

```mermaid
flowchart TD
    subgraph Fixed["⏱️ Fixed Expiry"]
        F1[Login at 9:00 AM] --> F2[Session expires at 10:00 AM]
        F2 --> F3[Must re-login\neven if actively using app]
        F3 --> F4[Good for high-security apps\nbanking, healthcare]
    end

    subgraph Sliding["🔄 Sliding Expiry"]
        S1[Login at 9:00 AM\n30-min timeout] --> S2[Active at 9:25 AM → resets to 9:55 AM]
        S2 --> S3[Active at 9:50 AM → resets to 10:20 AM]
        S3 --> S4[Idle for 30 min → session expires]
        S4 --> S5[Good for productivity apps\nemail, dashboards]
    end
```

---

### Access Tokens vs Refresh Tokens

```mermaid
flowchart TD
    LOGIN([👤 User Logs In]) --> AT[🎫 Access Token issued\nshort-lived: 15 min]
    LOGIN --> RT[🔄 Refresh Token issued\nlong-lived: 7–30 days]

    AT -->|Used for| API[API Calls]
    API -->|Token expires| EXP{Access Token\nExpired?}

    EXP -->|Yes| REF[Use Refresh Token\nto get new Access Token]
    REF --> VALID{Refresh Token\nstill valid?}
    VALID -->|Yes| NEWAT[✅ New Access Token issued]
    VALID -->|No| RELOGIN([🔁 Re-authenticate\nFull login required])

    EXP -->|No| CONTINUE[Continue API calls]
```

> **Why two tokens?** Access tokens are short-lived so stolen ones expire quickly. Refresh tokens are long-lived but only used rarely (at token expiry), reducing exposure.

---

### Token Revocation and Logout Propagation

**Problem:** Tokens are self-contained — the server can't "un-issue" them without extra work.

```mermaid
flowchart TD
    LOGOUT([👤 User clicks Logout]) --> RL[Revoke Refresh Token\nin token store]
    RL --> WAIT[Access token still valid\nfor up to 15 min]

    WAIT --> OPTIONS{How to handle\nactive access tokens?}

    OPTIONS --> O1[Short TTL\nAccept small window\nSimple, common approach]
    OPTIONS --> O2[Token blocklist\nCheck every request against\nrevocation list\nAccurate but adds latency]
    OPTIONS --> O3[Reference tokens\nOpaque tokens — always check\nintrospection endpoint]
```

---

## Quick Reference Cheat Sheet

| Concept | One-liner | Key Rule |
|---|---|---|
| **MFA** | Two or more different factor types | Password + TOTP = MFA; two passwords ≠ MFA |
| **TOTP** | Time-based 6-digit code from shared secret | Changes every 30 seconds; never reusable |
| **Passkeys / FIDO2** | Private key on device, public key on server | Private key never transmitted |
| **mTLS** | Both sides present certificates | Used for machine-to-machine trust |
| **Session token** | Server-side proof you're logged in | Store securely; use HttpOnly + Secure flags |
| **Access token** | Short-lived credential for API calls | Keep TTL short (5–15 min) |
| **Refresh token** | Long-lived token to get new access tokens | Store securely; revoke on logout |
| **SMS MFA** | One-time code via text message | Weak — vulnerable to SIM swapping |
| **Sliding expiry** | Session resets on activity | Good UX; use for low-risk apps |
| **Fixed expiry** | Session ends at set time regardless | Stronger security; use for sensitive apps |
