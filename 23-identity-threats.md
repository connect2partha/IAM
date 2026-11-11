# Identity Threats and Attack Vectors

> Simple explanations with diagrams for how attackers target identity systems — and how to defend against them.

---

## 1. Why Identity is the Primary Attack Target

Identity is the new perimeter. 80%+ of breaches involve compromised credentials. If an attacker can steal or forge an identity, they don't need to break through firewalls — they just walk in through the front door.

```mermaid
mindmap
  root((Identity Attacks))
    Credential Attacks
      Credential Stuffing
      Password Spraying
      Brute Force
    Session Attacks
      Token Theft
      Session Hijacking
      Cookie Stealing
    Protocol Attacks
      Pass-the-Hash
      Golden Ticket
      SAML Wrapping
      OAuth Misuse
    Social Engineering
      Phishing
      AiTM Proxy
      SIM Swapping
    Insider Threats
      Privilege Abuse
      Data Exfiltration
      Credential Sharing
```

---

## 2. Credential Stuffing

Attackers take **username/password pairs from leaked databases** and try them against other websites. Because people reuse passwords, many attempts succeed.

> Like trying every key from a stolen bunch to see which doors they open elsewhere.

```mermaid
flowchart TD
    A[Attacker buys leaked\ncredential database\ne.g. from dark web] --> B[Loads millions of\nusername:password pairs]
    B --> C[Automated tool tries each\npair against target site]
    C --> D{Login\nsuccessful?}
    D -->|Yes| E[Account taken over\nData stolen / sold]
    D -->|No| F[Try next credential]
    F --> C

    subgraph Defence["✅ Defences"]
        D1[MFA — stolen password alone not enough]
        D2[Breach detection — flag known leaked passwords]
        D3[Rate limiting — slow down automated attempts]
        D4[CAPTCHA on login]
    end
```

---

## 3. Password Spraying

Instead of trying many passwords against one account (brute force, which triggers lockout), attacker tries **one common password against many accounts**.

> Like a thief trying "1234" on every locker in the building — most will fail, but a few will open.

```mermaid
flowchart LR
    ATK([Attacker]) --> P[One common password\ne.g. Summer2024!]
    
    P -->|Try against| U1[user1@company.com]
    P -->|Try against| U2[user2@company.com]
    P -->|Try against| U3[user3@company.com]
    P -->|Try against| U4[hundreds more users...]

    U1 --> R1[Fail]
    U2 --> R2[✅ SUCCESS — account taken]
    U3 --> R3[Fail]
    U4 --> R4[Fail...]

    subgraph Defence["✅ Defences"]
        D1[MFA — password alone not enough]
        D2[Unusual login pattern detection]
        D3[Ban common passwords at registration]
        D4[UEBA — detect spray pattern across accounts]
    end
```

---

## 4. Phishing and Adversary-in-the-Middle (AiTM)

Modern phishing doesn't just steal passwords — it steals **session cookies** using a proxy between the user and the real site, bypassing MFA entirely.

> Like a fake receptionist who takes your ID, makes a copy of your visitor badge, then hands you through to the real reception — you got in, but they now have your badge.

```mermaid
sequenceDiagram
    participant U as Victim User
    participant P as Attacker's Proxy\n(e.g. Evilginx)
    participant R as Real Login Page\n(e.g. Microsoft 365)

    U->>P: Clicks phishing link\nthinks it's real Microsoft
    P->>R: Forwards request (proxy)
    R-->>P: Real login page
    P-->>U: Shows real login page

    U->>P: Enters username + password + MFA code
    P->>R: Forwards credentials in real time
    R-->>P: Issues session cookie ← STOLEN HERE
    P-->>U: Shows "logged in" (passes through)

    Note over P: Attacker now has session cookie
    Note over P: MFA was bypassed — it was used for the REAL login
    P->>R: Uses stolen cookie directly
    R-->>P: Full access ✅ for attacker
```

**Defence:** Phishing-resistant MFA (FIDO2/passkeys) — these bind authentication to the legitimate domain, so the proxy can't replay the credential.

---

## 5. Pass-the-Hash

On Windows systems, passwords are stored as **NTLM hashes**. An attacker who gains admin access can dump these hashes from memory and use them to authenticate — **without ever knowing the actual password**.

> Like stealing a photocopy of a key — you don't know what the original looks like, but the photocopy still opens the lock.

```mermaid
flowchart TD
    ATK([Attacker gains\ninitial foothold\ne.g. via phishing]) --> DUMP[Dump NTLM hashes\nfrom LSASS memory\ne.g. using Mimikatz]
    DUMP --> HASH[Has hash for\nadmin account:\nAAD3B435... etc]
    HASH --> PTH[Use hash directly\nin authentication request\nno password needed]
    PTH --> ACCESS[Lateral movement\nto other systems\nas admin user]

    subgraph Defence["✅ Defences"]
        D1[Credential Guard — protects LSASS memory]
        D2[Privileged Access Workstations]
        D3[Network segmentation — limit lateral movement]
        D4[Disable NTLM, enforce Kerberos / NTLMv2]
    end
```

---

## 6. Kerberos Golden Ticket Attack

The most powerful Active Directory attack. Attacker compromises the **KRBTGT account** (the Kerberos ticket-granting secret) and can forge tickets for **any user** in the domain.

> Like stealing the master key-making machine at a hotel — you can now make a key card for any room, for any fake guest, that never expires.

```mermaid
flowchart TD
    ATK([Attacker]) --> COMP[Compromise Domain Controller\nor steal KRBTGT hash]
    COMP --> KRBTGT[Obtain KRBTGT\npassword hash]
    KRBTGT --> FORGE[Forge a Kerberos TGT\nfor any user, any group\ne.g. Domain Admin]
    FORGE --> TICKET[Golden Ticket\nvalid for 10+ years\nany username, any privileges]
    TICKET --> ACCESS[Access any system\nin the domain\nwithout any real account]

    subgraph Silver["Silver Ticket — Limited Variant"]
        S1[Compromise service account hash\nonly] --> S2[Forge TGS for\nthat specific service only]
        S2 --> S3[Access just that service\nmore stealthy]
    end

    subgraph Defence["✅ Defences"]
        D1[Rotate KRBTGT password twice\nafter domain compromise]
        D2[Monitor for TGTs with unusual\nlifetimes or attributes]
        D3[Tier 0 isolation for DCs]
        D4[Detect Mimikatz / DCSync activity]
    end
```

---

## 7. Token Theft and Replay

An attacker steals a valid **JWT or session token** and replays it to gain access — often after compromising a device or intercepting traffic.

> Like someone stealing your hotel keycard while you're at the pool — they can enter your room without knowing your name or booking reference.

```mermaid
flowchart TD
    U([Legitimate User\nlogs in]) --> TOKEN[Auth server issues\nJWT access token]
    TOKEN --> BROWSER[Token stored in browser\nlocalStorage or cookie]

    ATK([Attacker]) --> XSS[Exploits XSS vulnerability\nin web app]
    XSS --> STEAL[Steals token\nfrom localStorage]
    STEAL --> REPLAY[Sends stolen token\nto API as Bearer token]
    REPLAY --> ACCESS[API accepts token —\nattacker authenticated as user ✅]

    subgraph Defence["✅ Defences"]
        D1[Store tokens in httpOnly cookies\nnot localStorage — XSS-safe]
        D2[Short token expiry — limit window]
        D3[Token binding — tie token to TLS session]
        D4[Fix XSS vulnerabilities — CSP headers]
        D5[Refresh token rotation — detect reuse]
    end
```

---

## 8. OAuth Open Redirector Attack

In OAuth, after authentication the server redirects to a `redirect_uri`. If the app doesn't validate this strictly, an attacker can steal the **authorization code**.

> Like a fake postal worker intercepting a parcel — they redirect the delivery to their address instead of yours.

```mermaid
flowchart TD
    ATK([Attacker crafts malicious URL]) --> URL["Authorization URL with\nredirect_uri=https://evil.com/steal"]
    URL --> VICTIM[Victim clicks phishing link\nto legitimate auth server]
    VICTIM --> AUTH[Auth server authenticates user\nuser completes MFA]
    AUTH --> REDIR[Auth server follows redirect_uri\nwithout validating it ❌]
    REDIR --> EVIL[Authorization code sent\nto attacker's evil.com]
    EVIL --> EXCHANGE[Attacker exchanges code\nfor access token]
    EXCHANGE --> ACCESS[Full access to victim's account]

    subgraph Defence["✅ Defences"]
        D1[Register exact redirect_uris — no wildcards]
        D2[Validate redirect_uri strictly server-side]
        D3[Use PKCE — ties code to original requester]
    end
```

---

## 9. SAML XML Signature Wrapping

SAML assertions are XML documents. Attackers can **modify the assertion content** while keeping the signature valid by exploiting how the XML is parsed.

> Like photocopying a signed contract and inserting extra pages — the signature is real, but the document says something different now.

```mermaid
flowchart TD
    LEGIT[IdP issues SAML assertion:\nUser = alice, Role = employee\nSigned ✅] --> ATK

    subgraph ATK["Attacker manipulates XML"]
        WRAP[Wraps original signed assertion\nin outer XML element]
        WRAP --> INJECT[Injects new unsigned assertion:\nUser = alice, Role = admin]
        INJECT --> SUBMIT[Submits wrapped XML to SP]
    end

    subgraph SP["Service Provider — Vulnerable Parser"]
        PARSE[Parser finds the signature\nin the outer wrapper — valid ✅]
        PARSE --> PROCESS[But processes the\ninjected inner assertion]
        PROCESS --> ESCALATE[Alice gets admin access ❌]
    end

    ATK --> SP

    subgraph Defence["✅ Defences"]
        D1[Use schema-validated SAML libraries]
        D2[Canonicalise XML before signature verification]
        D3[Reject assertions with extra XML wrapping]
        D4[Use modern OIDC/JWT instead of SAML where possible]
    end
```

---

## 10. SIM Swapping (MFA Bypass)

Attacker social-engineers a mobile carrier to transfer the victim's phone number to the attacker's SIM card — then receives SMS OTPs.

> Like convincing a bank that you lost your card and getting a new one sent to your address — except it's the attacker's address.

```mermaid
flowchart TD
    ATK([Attacker]) --> INFO[Researches victim\nLinkedIn, social media\ndata broker sites]
    INFO --> CALL[Calls mobile carrier:\n"Hi, I'm Alice. Lost my phone.\nPlease transfer to new SIM."]
    CALL --> SOCIAL[Social engineers\ncustomer service rep]
    SOCIAL --> PORT[Carrier ports Alice's number\nto attacker's SIM ❌]
    PORT --> SMS[Attacker receives\nall of Alice's SMS messages]
    SMS --> OTP[Receives SMS OTP\nfor Alice's bank / email]
    OTP --> RESET[Resets Alice's passwords\nusing OTP + account recovery]
    RESET --> ACCESS[Full account takeover]

    subgraph Defence["✅ Defences"]
        D1[Use TOTP app or hardware key\nnot SMS MFA]
        D2[Set carrier account PIN / port-out freeze]
        D3[Use FIDO2 passkeys — not phone-number-based]
    end
```

---

## 11. Common Techniques of Identity Theft

Identity theft happens when someone uses another person's personal information — without consent — to impersonate them, gain access, or commit fraud. It affects individuals and organisations alike.

> Like someone stealing your wallet, your ID card, and your signature — then pretending to be you everywhere.

```mermaid
mindmap
  root((Identity Theft\nTechniques))
    Digital
      Phishing
        Email phishing
        Smishing SMS
        Vishing voice call
      Malware
        Keyloggers
        Spyware
        RATs
      Data Breach
        Dark web purchase
        Credential dumps
      Account Takeover
        Password reset abuse
        SIM swap
    Physical
      Mail theft
      Dumpster diving
      Stolen wallet / documents
      Shoulder surfing
    Social Engineering
      Pretexting
      Impersonation
      Romance scams
    Financial
      Card skimming
      ATM skimmers
      Synthetic identity fraud
```

### 11.1 Phishing — Email, SMS, and Voice

The most common identity theft technique. Attacker impersonates a trusted organisation to trick the victim into revealing credentials or personal information.

```mermaid
flowchart TD
    subgraph Email["📧 Email Phishing"]
        E1[Fake email from 'your bank'] --> E2[Victim clicks link]
        E2 --> E3[Fake login page — credentials stolen]
    end

    subgraph SMS["📱 Smishing (SMS Phishing)"]
        S1["'Your package is held — verify identity'"] --> S2[Victim clicks SMS link]
        S2 --> S3[Personal details harvested]
    end

    subgraph Voice["📞 Vishing (Voice Phishing)"]
        V1["Caller: 'This is HMRC / IRS / bank security'"] --> V2[Victim provides SSN, DOB, password]
        V2 --> V3[Caller uses info for fraud]
    end
```

### 11.2 Data Breaches and Dark Web Purchase

Attackers buy **leaked personal data** from past breaches — name, email, password, SSN, date of birth, address — and use it to impersonate victims.

```mermaid
flowchart LR
    BREACH[Company data breach\ne.g. healthcare, retail, credit bureau] --> DUMP[Database of records\nname, email, SSN, DOB, address]
    DUMP --> DARKWEB[Sold on dark web\nfor cents per record]
    DARKWEB --> ATK[Attacker buys records]
    ATK --> FRAUD1[Opens credit card\nin victim's name]
    ATK --> FRAUD2[Files false tax return\nfor refund]
    ATK --> FRAUD3[Takes over existing accounts\nusing personal info for verification]
```

### 11.3 Malware — Keyloggers and Spyware

Malicious software silently records everything the victim types or views.

| Malware Type | What it steals | How it gets installed |
|---|---|---|
| **Keylogger** | Passwords, credit card numbers, messages | Email attachment, fake software download |
| **Spyware** | Browser history, saved passwords, files | Bundled with free software |
| **RAT (Remote Access Trojan)** | Full system access, screenshots, webcam | Phishing, cracked software |
| **Info-stealer** (Redline, Vidar) | Browser cookies, saved credentials | Malvertising, game cheats |

### 11.4 Physical Identity Theft

Not all identity theft is digital. Physical methods remain common.

```mermaid
flowchart TD
    subgraph Physical["Physical Identity Theft Methods"]
        MAIL[Mail Theft\nSteal credit card offers,\nbank statements, tax documents]
        DUMPSTER[Dumpster Diving\nRecover discarded documents\nwith account numbers or SSNs]
        SHOULDER[Shoulder Surfing\nWatch victim type PIN\nor password in public]
        WALLET[Stolen Wallet\nPhysical ID, credit cards,\ninsurance cards]
    end

    subgraph Use["What attackers do with physical documents"]
        F1[Open new credit accounts]
        F2[Access existing bank accounts]
        F3[Obtain medical care using victim's insurance]
        F4[File fraudulent tax returns]
    end

    Physical --> Use
```

**Defence:** Shred documents before disposal. Use a P.O. box or mail lock. Use a PIN cover keypad when entering PINs.

### 11.5 Social Engineering and Pretexting

Attacker invents a believable scenario (pretext) to trick the victim or an organisation's staff into revealing information.

```mermaid
sequenceDiagram
    participant ATK as Attacker
    participant REP as Customer Service Rep
    participant ACCT as Victim's Account

    ATK->>REP: "Hi, I'm John Smith, DOB 15/03/1985.\nI've lost access to my account."
    REP->>ATK: "Can you confirm your address?"
    ATK->>REP: Provides address (found on LinkedIn / data broker)
    REP->>ATK: "Security question: mother's maiden name?"
    ATK->>REP: Provides answer (found on social media)
    REP-->>ATK: Resets account credentials
    ATK->>ACCT: Full access to victim's account
```

**Common pretexts:** "I'm from IT support", "I'm from your bank's fraud team", "I'm a government official".

### 11.6 Card Skimming

Physical devices attached to ATMs or payment terminals capture card data and PINs.

```mermaid
flowchart LR
    SKIM[Skimmer device\nfitted over ATM card slot] --> MAG[Reads magnetic stripe\nas victim inserts card]
    CAM[Hidden camera\nor fake keypad overlay] --> PIN[Records PIN\nas victim types]
    MAG & PIN --> CLONE[Attacker clones card]
    CLONE --> FRAUD[Makes withdrawals\nor purchases]
```

**Defence:** Check for loose card readers, cover keypad when typing PIN, use contactless payments (chip+PIN, Apple Pay).

### 11.7 Synthetic Identity Fraud

Attacker **combines real and fake information** to create a new fictitious identity — often using a real SSN with a fake name and DOB.

```mermaid
flowchart TD
    REAL[Real SSN\ne.g. from a child or\nrecently deceased person] --> SYNTH
    FAKE[Fake name, DOB, address] --> SYNTH[Synthetic Identity Created]
    SYNTH --> CREDIT[Apply for credit\nwith no history]
    CREDIT --> BUILD[Slowly build credit score\nover months or years]
    BUILD --> BUST[Bust-out fraud:\nmax out all credit lines\nthen disappear]
```

> Hardest fraud to detect because the SSN belongs to a real (but different) person. Banks often can't tell until the bust-out.

### 11.8 Account Takeover (ATO)

Attacker gains control of an existing legitimate account using stolen credentials or identity information.

```mermaid
flowchart TD
    ATK([Attacker]) --> METHOD{How they\ngain access}
    METHOD -->|Leaked credentials| CRED[Credential stuffing\nor password spraying]
    METHOD -->|Personal info| RESET[Password reset\nusing DOB, SSN, security questions]
    METHOD -->|SIM swap| SIM[Receives SMS OTP\nafter porting victim's number]
    METHOD -->|Phishing| PHISH[Victim enters creds\non fake site]

    CRED & RESET & SIM & PHISH --> ATO[Account Taken Over]

    ATO --> CHANGE[Change email + phone\nlock victim out]
    ATO --> DRAIN[Drain funds / points / data]
    ATO --> PIVOT[Use account to\nattack contacts]
```

### 11.9 Identity Theft Defence Summary

```mermaid
flowchart TD
    subgraph Prevent["Prevention"]
        P1[Use strong unique passwords\n+ password manager]
        P2[Enable MFA — preferably\nFIDO2 not SMS]
        P3[Freeze credit reports\nat all bureaus]
        P4[Shred documents\nwith personal info]
        P5[Monitor accounts\nfor unusual activity]
    end

    subgraph Detect["Detection"]
        D1[Credit monitoring alerts]
        D2[Bank transaction notifications]
        D3[Annual free credit report check]
        D4[Have I Been Pwned — monitor\nfor leaked email/password]
    end

    subgraph Respond["Response"]
        R1[Report to bank / card issuer immediately]
        R2[File police report]
        R3[Report to national agency\ne.g. FTC, Action Fraud, ACSC]
        R4[Place fraud alert on credit file]
        R5[Document all fraudulent transactions]
    end

    Prevent --> Detect --> Respond
```

---

## 12. Insider Threat and Privilege Abuse

Not all threats come from outside. Employees, contractors, or admins with legitimate access can misuse it.

```mermaid
flowchart TD
    subgraph Types["Insider Threat Types"]
        MALI[Malicious Insider\nDeliberately steals / sabotages]
        NEGLI[Negligent Insider\nAccidentally exposes data]
        COMPR[Compromised Insider\nAccount taken over by attacker]
    end

    subgraph Indicators["🚨 Warning Signs"]
        I1[Accessing data outside\nnormal work hours]
        I2[Downloading large volumes\nof files]
        I3[Accessing systems outside\nnormal job scope]
        I4[Privilege escalation attempts]
        I5[Disabling audit logs]
    end

    subgraph Defence["✅ Defences"]
        D1[Least privilege — minimise what anyone can access]
        D2[Separation of duties — no single person controls all]
        D3[UEBA — detect anomalous access patterns]
        D4[DLP — detect and block data exfiltration]
        D5[PAM — record privileged sessions]
    end
```

---

## Quick Reference Cheat Sheet

### Technical Attack Vectors

| Attack | How it works | Key Defence |
|---|---|---|
| **Credential Stuffing** | Leaked creds tried against target | MFA + breach detection |
| **Password Spraying** | One common password vs many users | MFA + anomaly detection |
| **AiTM Phishing** | Proxy steals session cookie, bypasses MFA | FIDO2 / passkeys |
| **Pass-the-Hash** | NTLM hash used instead of password | Credential Guard, disable NTLM |
| **Golden Ticket** | Forged Kerberos TGT with KRBTGT hash | Protect DC, rotate KRBTGT |
| **Token Theft** | Stolen JWT replayed | httpOnly cookies, short expiry |
| **OAuth Open Redirect** | Auth code stolen via bad redirect_uri | Exact URI registration + PKCE |
| **SAML Wrapping** | Modified XML with valid signature | Schema-validated SAML libraries |
| **SIM Swapping** | Phone number ported to attacker | Use TOTP/FIDO2, not SMS MFA |
| **Insider Threat** | Legitimate user abuses access | Least privilege + UEBA + PAM |

### Common Identity Theft Techniques

| Technique | Method | Key Defence |
|---|---|---|
| **Email Phishing** | Fake email → fake login page → credentials stolen | Security awareness + MFA |
| **Smishing** | Fake SMS → link → personal info harvested | Never click SMS links from unknown senders |
| **Vishing** | Fake phone call → victim reveals SSN, DOB, password | Verify caller identity independently |
| **Data Breach / Dark Web** | Leaked personal data bought and used for fraud | Credit freeze + HIBP monitoring |
| **Keylogger / Malware** | Records keystrokes, steals saved passwords | Antivirus + avoid pirated software |
| **Mail Theft** | Physical statements / card offers stolen | Shred documents + mail lock |
| **Dumpster Diving** | Documents with account numbers recovered | Shred everything with personal info |
| **Shoulder Surfing** | PIN / password watched in public | Cover keypad, use privacy screen |
| **Social Engineering** | Pretext to trick staff into resetting account | Knowledge-based auth is weak; use strong MFA |
| **Card Skimming** | Physical device reads card + captures PIN | Use contactless; cover keypad |
| **Synthetic Identity** | Real SSN + fake name used to open accounts | SSN + biometric verification at account opening |
| **Account Takeover** | Multiple methods used to hijack existing account | MFA + anomaly detection + credit freeze |
