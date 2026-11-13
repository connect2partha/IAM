# Multi Factor Authentication (MFA)

[← Back to Index](README.md)

> A deep dive into every MFA method, how each works under the hood, where each breaks, and how to choose the right one.

---

## 1. What is MFA and Why It Matters

A password alone is no longer enough. Billions of credentials are leaked every year from data breaches. Attackers buy these lists and try them automatically against every major service — a technique called **credential stuffing**.

> **Analogy:** An ATM requires your physical card AND your PIN. Even if someone knows your PIN, they cannot withdraw money without the card. MFA applies the same principle to digital logins.

**Microsoft research:** MFA blocks over **99.9% of automated account compromise attacks**, even when passwords are known.

```mermaid
flowchart TD
    ATK([🏴‍☠️ Attacker has\nyour password]) --> TRY[Attempts login]

    subgraph NoMFA["❌ No MFA"]
        TRY --> IN1([Access granted immediately])
    end

    subgraph WithMFA["✅ With MFA"]
        TRY --> FACTOR{Second factor\nrequired}
        FACTOR -->|Attacker doesn't\nhave your phone| DENY([Access denied ✅])
        FACTOR -->|You approve| GRANT([Access granted])
    end
```

### Why passwords fail on their own

```mermaid
flowchart LR
    B1[Data breach\nat another site] --> CRED[Your password\nexposed]
    B2[Phishing email] --> CRED
    B3[Malware / keylogger] --> CRED
    B4[Password reuse\nacross sites] --> CRED

    CRED --> ATK[Attacker tries it\non your work account]
    ATK -->|No MFA| BREACH([💥 Account compromised])
    ATK -->|MFA enabled| BLOCK([🛡️ Blocked])
```

---

## 2. MFA Factors and Strength Hierarchy

MFA factors fall into four categories. **Using factors from two different categories** is what makes authentication strong.

| Factor | Category | Examples |
|---|---|---|
| Something you **know** | Knowledge | Password, PIN, security question |
| Something you **have** | Possession | Phone, hardware key, smart card |
| Something you **are** | Inherence | Fingerprint, face, iris |
| Somewhere you **are** | Location | Geolocation, IP range, network |

### Strength spectrum — weakest to strongest

```mermaid
flowchart LR
    SMS["📱 SMS OTP\n⭐ Weakest\nSIM-swappable\nSS7 attacks"] -->
    TOTP["🔢 TOTP App\n⭐⭐ Weak-Medium\nPhishable\nDevice theft risk"] -->
    PUSH["🔔 Push Notification\n⭐⭐⭐ Medium\nFatigue attacks\nphishable session"] -->
    HWKEY["🔑 Hardware Key\n⭐⭐⭐⭐ Strong\nPhishing-resistant\norigin-bound"] -->
    PASSKEY["🛡️ Passkey\n⭐⭐⭐⭐⭐ Strongest\nPhishing-resistant\nno shared secret"]
```

> **Key insight:** SMS and TOTP can be intercepted in real time by a skilled attacker. Hardware keys and passkeys cannot — they are cryptographically bound to the legitimate site.

---

## 3. OTP Methods in Detail

### 3.1 TOTP — Time-Based One-Time Password

TOTP generates a 6-digit code that changes every 30 seconds, using a **shared secret** seeded during setup and the **current time** as inputs.

> **Analogy:** Both you and the server are watching the same clock and doing the same calculation. As long as your clocks are in sync, you get the same number.

**How it works (simplified):**

```
Code = HOTP(secret, floor(current_unix_time / 30))
     = HMAC-SHA1(secret, time_counter) → truncate to 6 digits
```

```mermaid
sequenceDiagram
    actor User
    participant App as Authenticator App
    participant Server

    Note over User,Server: Setup (one time)
    Server->>User: Display QR code containing secret key
    User->>App: Scan QR code — secret stored on device

    Note over User,Server: Every login
    User->>Server: Enter username + password
    Server->>User: Prompt for TOTP code
    App->>App: Compute HMAC(secret, time_window)
    App->>User: Display 6-digit code
    User->>Server: Submit code
    Server->>Server: Compute same HMAC independently
    Server->>User: Codes match ✅ — access granted
```

**Apps:** Google Authenticator, Microsoft Authenticator, Authy, 1Password

**Weakness:** The code can be phished in real time — an attacker's fake login page can relay the code to the real site within the 30-second window.

---

### 3.2 HOTP — Counter-Based OTP

HOTP works the same as TOTP but uses a **counter** instead of time. The counter increments each time a code is generated.

```mermaid
flowchart LR
    SECRET[Shared Secret] --> HMAC
    COUNTER[Counter\nIncrements each use] --> HMAC
    HMAC[HMAC-SHA1] --> CODE[6-digit OTP]

    CODE --> SERVER{Server checks\nwith same counter}
    SERVER -->|Match| OK[✅ Authenticated\nCounter advanced]
    SERVER -->|No match| DRIFT[⚠️ Counter drift\nResync required]
```

**Risk:** If the user generates codes without submitting them, the client counter advances ahead of the server — causing sync failures. TOTP is more common today because time-sync is automatic.

---

### 3.3 SMS OTP

A one-time code is sent via text message to the user's registered phone number.

```mermaid
sequenceDiagram
    actor User
    participant Server
    participant Carrier as Mobile Carrier
    participant Phone as User's Phone

    User->>Server: Enter username + password
    Server->>Carrier: Send SMS OTP to +44-xxx-xxxx
    Carrier->>Phone: SMS: "Your code is 847291"
    Phone->>User: Shows SMS
    User->>Server: Submits 847291
    Server->>User: ✅ Access granted
```

**Why SMS is weak:**

```mermaid
flowchart TD
    ATK([Attacker]) --> SIM[SIM Swap\nSocially engineers carrier\nto port number to attacker's SIM]
    ATK --> SS7[SS7 Attack\nExploits telecom signalling\nprotocol to intercept SMS]
    ATK --> PHISH[Real-time phishing\nRelays code to real site\nwithin seconds]

    SIM & SS7 & PHISH --> BYPASS([MFA bypassed ❌])
```

| Scenario | Use SMS OTP? |
|---|---|
| Consumer app, low risk | Acceptable (better than nothing) |
| Corporate/enterprise login | Avoid — use TOTP or hardware key |
| Privileged / admin access | Never — use hardware key |
| Regulated industries (banking, healthcare) | Avoid — regulators increasingly reject SMS |

---

## 4. Push Notifications

Push MFA sends a notification to the user's registered smartphone app. The user taps **Approve** or **Deny**.

> **Analogy:** Someone rings your doorbell — you look through the peephole, recognise the person, and decide whether to let them in.

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant Server
    participant MFAService as MFA Service\n(Duo / Okta / MS Auth)
    participant Phone as User's Phone

    User->>Browser: Enter username + password
    Browser->>Server: Credentials submitted
    Server->>MFAService: Trigger push to user's device
    MFAService->>Phone: Push notification:\n"Login attempt from London\nApprove or Deny?"
    User->>Phone: Taps Approve ✅
    Phone->>MFAService: Approval confirmed
    MFAService->>Server: Second factor passed
    Server->>Browser: Access granted
```

### MFA Fatigue Attack

An attacker who has the user's password repeatedly sends push notifications hoping the user taps **Approve** by mistake or out of frustration.

```mermaid
flowchart TD
    ATK([Attacker has password]) --> SPAM[Sends 20 push\nnotifications at 2am]
    SPAM --> USER{User response}
    USER -->|Confused / tired\ntaps Approve| BREACH([💥 Account compromised])
    USER -->|Ignores all| SAFE([✅ Safe])

    subgraph Defences["Defences Against Fatigue"]
        D1[Number Matching\nUser must type the\nnumber shown on screen]
        D2[Additional Context\nShows app name + location]
        D3[Rate Limiting\nLock after N push denials]
        D4[Fraud Alert\nAuto-lock on unusual volume]
    end
```

**Number matching** is the most effective defence: the login screen shows a number (e.g. "47") and the push notification asks "Type the number shown" — an attacker cannot know it.

---

## 5. Hardware Security Keys (FIDO U2F / FIDO2)

A hardware security key is a physical USB/NFC device that uses **public-key cryptography** to authenticate. It is **phishing-resistant** because the key is cryptographically bound to the exact website domain.

> **Analogy:** A wax seal that only fits one specific envelope — it cannot be copied or used elsewhere.

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant Server
    participant Key as Hardware Key\n(YubiKey / Titan)

    Note over User,Key: Registration (one time)
    Server->>Browser: Request key registration
    Browser->>Key: Generate key pair for this origin\n(e.g. accounts.google.com)
    Key->>Browser: Public key
    Browser->>Server: Store public key ✅

    Note over User,Key: Every login
    User->>Browser: Enter username + password
    Server->>Browser: Send random challenge
    Browser->>Key: Sign challenge with private key\n(for this exact origin)
    Key->>User: Touch the key 👆
    User->>Key: Touch confirmed
    Key->>Browser: Signed challenge
    Browser->>Server: Return signature
    Server->>Server: Verify with stored public key ✅
    Server->>Browser: Access granted
```

**Why hardware keys cannot be phished:**

The private key is bound to the **exact origin** (domain). A fake site like `accounts-g00gle.com` gets a different key pair — the signature it receives is useless on the real site.

```mermaid
flowchart LR
    REAL["✅ Real site\naccounts.google.com\nKey signs correctly"] --> PASS[Login succeeds]
    FAKE["❌ Phishing site\naccounts-g00gle.com\nKey refuses to sign\n(wrong origin)"] --> FAIL[Login fails\nAttacker gets nothing]
```

**Examples:** YubiKey (Yubico), Google Titan Key, Apple Touch ID key (FIDO2 built into MacBook)

---

## 6. Passkeys (Passwordless MFA)

A **passkey** is a FIDO2 credential stored on your device (or synced via cloud) that replaces both the password and the second factor in a single step.

> **Analogy:** Your fingerprint on your phone IS the key — you don't need to remember anything or carry a separate device.

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant Server
    participant Device as Device\n(Phone / Laptop)

    Note over User,Device: Registration (one time)
    Server->>Browser: Request passkey creation
    Browser->>Device: Generate FIDO2 key pair
    Device->>User: Authenticate locally\n(Face ID / fingerprint / PIN)
    User->>Device: Biometric confirmed
    Device->>Browser: Public key + attestation
    Browser->>Server: Store public key ✅

    Note over User,Device: Every login
    User->>Browser: Click "Sign in with passkey"
    Server->>Browser: Send challenge
    Browser->>Device: Sign challenge
    Device->>User: Authenticate locally\n(Face ID / fingerprint)
    User->>Device: Biometric confirmed
    Device->>Browser: Signed challenge
    Browser->>Server: Verify signature ✅
    Server->>Browser: Access granted — no password ever sent
```

**Key properties:**

| Property | Detail |
|---|---|
| **No shared secret** | Private key never leaves the device |
| **Phishing-resistant** | Bound to exact origin domain |
| **Biometric is local** | Face/fingerprint verified on device — never sent to server |
| **Synced passkeys** | Stored in iCloud Keychain / Google Password Manager — available across devices |
| **Device-bound passkeys** | Stay on one device only — higher security, less convenience |

---

## 7. Adaptive / Risk-Based MFA

Not every login carries the same risk. Requiring a hardware key to check a news feed is unnecessary friction. Requiring only a password to approve a wire transfer is negligent.

**Risk-based MFA** dynamically adjusts the required authentication strength based on context signals.

```mermaid
flowchart TD
    LOGIN([Login attempt]) --> SIGNALS

    subgraph SIGNALS["Risk Signals Collected"]
        S1[Device: known or new?]
        S2[Location: usual or new country?]
        S3[Time: business hours or 3am?]
        S4[Action: view dashboard or delete account?]
        S5[Network: corporate VPN or Tor exit node?]
        S6[Behaviour: matches historical pattern?]
    end

    SIGNALS --> SCORE{Risk Score}

    SCORE -->|Low risk\nKnown device + location\nnormal hours| L1[Password only\nor silent auth]
    SCORE -->|Medium risk\nNew device or location| L2[Password +\nTOTP or Push]
    SCORE -->|High risk\nNew country + unusual hours\nor high-value action| L3[Password +\nHardware key]
    SCORE -->|Very high risk\nTor / flagged IP\nimpossible travel| L4[Block and\nalert security team]
```

### Step-Up Authentication

A user is already logged in but attempts a sensitive action — the system challenges them again.

```mermaid
flowchart LR
    USER([Logged-in User]) --> ACTION[Attempts to:\napprove payment\nor delete data\nor change MFA settings]
    ACTION --> STEPUP{Step-up\nrequired}
    STEPUP --> RE_AUTH[Re-authenticate\nwith stronger factor]
    RE_AUTH -->|Passes| ALLOW([Action permitted])
    RE_AUTH -->|Fails| DENY([Action denied\nAlert raised])
```

---

## 8. MFA in Enterprise Scenarios

### Conditional Access (Azure Entra ID)

Conditional Access policies evaluate signals at login time and enforce the right level of authentication.

```mermaid
flowchart TD
    REQ([Access Request]) --> CA[Conditional Access\nPolicy Engine]

    CA --> C1{Who is the user?\nEmployee / Guest / Admin}
    C1 --> C2{What app?\nEmail / Finance / Admin portal}
    C2 --> C3{What device?\nCompliant / Unmanaged / Unknown}
    C3 --> C4{Where from?\nCorporate network / Home / New country}

    C4 -->|All trusted signals| ALLOW[Allow — no MFA]
    C4 -->|Mixed signals| MFA_REQ[Require MFA]
    C4 -->|High risk signals| BLOCK[Block access]
    C4 -->|Privileged action| STRONG[Require hardware key]

    MFA_REQ --> COMPLY{User passes MFA?}
    COMPLY -->|Yes| GRANT([Access granted])
    COMPLY -->|No| DENY([Access denied])
```

### MFA for Privileged Accounts

Admin and privileged accounts face stricter MFA requirements — they are the highest-value targets.

```mermaid
flowchart LR
    subgraph Standard["Standard User"]
        U1([Employee]) -->|TOTP or Push OK| A1[Business apps]
    end

    subgraph Privileged["Privileged / Admin"]
        U2([IT Admin]) -->|Hardware key required| PA[Admin portals]
        U2 -->|Hardware key + PIM activation| PB[Azure admin roles]
        U2 -->|Hardware key + session recording| PC[PAM — server access]
    end
```

---

## 9. MFA Bypass and Attacks

### AiTM — Adversary-in-the-Middle Phishing

The most dangerous attack against TOTP and push MFA. The attacker acts as a real-time proxy.

```mermaid
sequenceDiagram
    actor User
    participant Fake as Attacker's\nFake Site
    participant Real as Real Site\n(e.g. Microsoft 365)

    User->>Fake: Visits phishing link\nEnters username + password
    Fake->>Real: Relays credentials to real site
    Real->>Fake: Requests MFA code
    Fake->>User: Shows fake MFA prompt
    User->>Fake: Enters TOTP code
    Fake->>Real: Relays TOTP code immediately
    Real->>Fake: Issues session cookie ✅
    Fake->>Fake: Steals session cookie
    Note over Fake: Attacker now has valid session\nand bypasses MFA entirely
```

**Defence — phishing-resistant MFA only:**

```mermaid
flowchart LR
    AITM([AiTM attack\nusing TOTP]) -->|TOTP relayed in time| BYPASS[MFA bypassed ❌]

    PASSKEY([Passkey / Hardware key]) -->|Bound to real domain\nFake site gets useless signature| BLOCKED[Attack fails ✅]
```

### Attack Summary and Defences

| Attack | Affected Methods | Defence |
|---|---|---|
| Credential stuffing | Password only | MFA of any kind |
| SIM swapping | SMS OTP | Switch to TOTP or hardware key |
| SS7 interception | SMS OTP | Switch to app-based MFA |
| AiTM phishing | SMS, TOTP, Push | Passkeys or hardware keys only |
| MFA fatigue | Push notifications | Number matching, rate limiting |
| Device theft | TOTP app, passkeys | Device PIN / biometric lock |

---

## 10. MFA Recovery and Fallback

A strong MFA method is undermined if the **account recovery path is weak**.

```mermaid
flowchart TD
    LOST([User loses\nMFA device]) --> OPTIONS{Recovery\noptions}

    OPTIONS --> BC[Backup codes\nOne-time codes generated\nat MFA setup]
    OPTIONS --> SEC[Secondary device\nPreviously registered backup phone]
    OPTIONS --> IT[IT helpdesk\nIdentity verification required]
    OPTIONS --> EMAIL[Recovery email\nWeaker — email could be compromised]

    BC -->|Each code single-use| ACCESS([Regain access])
    SEC --> ACCESS
    IT -->|Verify with employee ID\nvideo call, manager approval| ACCESS
    EMAIL -->|Risky if email is compromised| RISK([⚠️ Potential weakness])

    ACCESS --> RESET[Re-enrol MFA\non new device]
```

**Backup code risks:**

```mermaid
flowchart LR
    CODES[Backup codes\ngenerated at setup] --> STORED{Where stored?}

    STORED -->|Password manager| SAFE[✅ Acceptable]
    STORED -->|Printed and locked away| SAFE
    STORED -->|Screenshot in cloud photos| RISKY[⚠️ Risky]
    STORED -->|Plain text file on desktop| BREACH([❌ Dangerous])
    STORED -->|Never saved| LOCKED([❌ Locked out])
```

> **Rule:** Backup codes are as sensitive as passwords. Store them in a password manager or physical safe — never in an email or unencrypted file.

---

## 11. Quick Reference

| Method | Phishing Resistant | SIM-Swap Resistant | UX Friction | Cost | Best For |
|---|---|---|---|---|---|
| **SMS OTP** | ❌ No | ❌ No | Low | Low | Low-risk consumer apps only |
| **TOTP App** | ❌ No | ✅ Yes | Medium | Free | General enterprise use |
| **Push (no number match)** | ❌ No | ✅ Yes | Low | Medium | General use — enable number matching |
| **Push (number matching)** | ⚠️ Partial | ✅ Yes | Low-Med | Medium | Good enterprise default |
| **Hardware Key (FIDO2)** | ✅ Yes | ✅ Yes | Low | £25–50/key | Privileged accounts, high-value targets |
| **Passkey** | ✅ Yes | ✅ Yes | Very Low | Free | Modern apps, consumer + enterprise |

### Choosing the right MFA method

```mermaid
flowchart TD
    START([Choose MFA method]) --> Q1{Is phishing\nresistance required?}

    Q1 -->|Yes — privileged\nor high-value account| Q2{Budget for\nhardware keys?}
    Q2 -->|Yes| HW[Hardware Key\nFIDO2 / YubiKey]
    Q2 -->|No / BYOD| PK[Passkey\non personal device]

    Q1 -->|No — standard\nbusiness account| Q3{Push MFA\navailable?}
    Q3 -->|Yes| PUSH2[Push with\nnumber matching]
    Q3 -->|No| TOTP2[TOTP app\nGoogle / MS Authenticator]

    Q1 -->|Consumer app\nlow risk| SMS2[SMS OTP\nor Social login]
```
