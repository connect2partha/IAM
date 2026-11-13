# Identity Security Monitoring

[← Back to Index](README.md)

> Simple explanations with diagrams for how to detect and respond to identity-based threats in real time.

---

## 1. Why Monitor Identity?

Firewalls and antivirus protect the perimeter and devices. But if an attacker **steals valid credentials**, they bypass all of that — they look just like a legitimate user. The only way to catch them is to watch **how** identities behave.

> Think of it like a casino's eye-in-the-sky cameras. The cards and chips look the same whether you're a customer or a cheat. Only by watching patterns — are they counting cards? visiting the cashier too often? — can you spot trouble.

---

## 2. The Security Monitoring Pipeline

Identity events flow from many sources into a central system that detects threats and triggers automated responses.

```mermaid
flowchart LR
    subgraph Sources["📡 Identity Event Sources"]
        E1[Login attempts\nsuccess & failure]
        E2[MFA challenges\n& failures]
        E3[Password changes\n& resets]
        E4[Privileged access\nsessions]
        E5[Access to\nsensitive resources]
        E6[Account creation\n& modification]
    end

    subgraph Collection["📥 Log Collection"]
        SIEM[(SIEM\nSentinel / Splunk /\nChronicle)]
    end

    subgraph Detection["🔍 Detection"]
        RULES[Correlation Rules\nif X happens Y times → alert]
        UEBA[UEBA Engine\nAnomaly vs baseline]
    end

    subgraph Response["⚡ Response"]
        ALERT[Alert to analyst]
        SOAR[SOAR Playbook\nautomated actions]
    end

    Sources --> Collection
    Collection --> Detection
    Detection --> Response
```

---

## 3. Account Lockout Policy

A basic defence: if too many failed logins happen in a short time, lock the account temporarily.

> Like a PIN pad that blocks your card after 3 wrong attempts — it protects you, but if set too aggressively, it can lock out real users too.

```mermaid
flowchart TD
    LOGIN([User attempts login]) --> CRED{Credentials\ncorrect?}
    CRED -->|Yes| RESET[Reset fail counter\nto 0]
    RESET --> ACCESS([✅ Access granted])

    CRED -->|No| INC[Increment\nfail counter]
    INC --> THRESHOLD{Counter ≥\nthreshold?\ne.g. 5 fails}
    THRESHOLD -->|No| WAIT[Wait lockout\nobservation window\ne.g. 10 min]
    WAIT --> LOGIN

    THRESHOLD -->|Yes| LOCK[🔒 Account locked]
    LOCK --> NOTIFY[Notify user via email]
    LOCK --> UNLOCK{Unlock method}
    UNLOCK -->|Auto-unlock| TIMER[After lockout duration\ne.g. 30 min]
    UNLOCK -->|Manual unlock| ADMIN[Admin or\nself-service unlock]
    TIMER --> LOGIN
    ADMIN --> LOGIN

    subgraph Tuning["⚙️ Tuning Considerations"]
        T1[Too strict → legitimate users locked out\nHelp desk overwhelmed]
        T2[Too loose → password spray attacks succeed]
        T3[Sweet spot: 5-10 attempts per 10 min]
    end
```

---

## 4. Impossible Travel Detection

A login from London at 9:00 AM and then Sydney at 9:05 AM is physically impossible. This is a strong signal of account compromise.

> Like a hotel noticing a guest checked out at noon in New York, then tried to check in at noon in Tokyo — that's not humanly possible.

```mermaid
flowchart TD
    E1[Login event 1:\nUser alice@corp.com\nLocation: London, UK\nTime: 09:00 UTC] --> TRACK

    E2[Login event 2:\nUser alice@corp.com\nLocation: Sydney, AU\nTime: 09:05 UTC] --> TRACK

    TRACK[IAM / SIEM\ntracks login events\nper user] --> CALC[Calculate:\nDistance = 16,993 km\nTime gap = 5 minutes\nRequired speed = 203,916 km/h\n= IMPOSSIBLE]

    CALC --> ALERT[🚨 Impossible travel alert]
    ALERT --> ACTION{Automated\nresponse}
    ACTION --> BLOCK[Block active sessions]
    ACTION --> MFA[Require re-authentication\nwith step-up MFA]
    ACTION --> TICKET[Create security incident ticket]
    ACTION --> ANALYST[Notify security analyst]
```

**Common anomaly patterns to detect:**

| Signal | What it might mean |
|---|---|
| Impossible travel | Account takeover from another country |
| Login at 3 AM when user normally works 9-5 | Compromised credential or insider threat |
| Login from new country / anonymous VPN | Attacker hiding location |
| Sudden access to new, sensitive resources | Lateral movement or privilege escalation |
| Mass file download | Data exfiltration |

---

## 5. UEBA — User and Entity Behaviour Analytics

UEBA goes beyond simple rules. It **learns what "normal" looks like** for each user and device, then flags deviations.

> Like a bank fraud system that knows you usually buy coffee near home and flags when your card is suddenly used for electronics in another city.

```mermaid
flowchart TD
    subgraph Baseline["📊 Phase 1: Building Baseline (weeks 1-4)"]
        B1[Collect all user activity\nlogins, access, downloads, times] --> B2[ML model learns:\nnormal hours, locations,\nresources accessed, volumes]
        B2 --> B3[Baseline established\nper user and per peer group]
    end

    subgraph Detect["🔍 Phase 2: Ongoing Detection"]
        D1[New activity event arrives] --> D2{Deviates from\nbaseline?}
        D2 -->|No| D3[Low risk score\nnormal activity]
        D2 -->|Yes| D4[Calculate risk score\nbased on severity of deviation]
        D4 --> D5{Risk score\nthreshold?}
        D5 -->|Low| D6[Log and monitor]
        D5 -->|Medium| D7[Alert analyst]
        D5 -->|High| D8[Automated response\nstep-up MFA / block session]
    end

    Baseline --> Detect
```

**What UEBA analyses:**

```mermaid
mindmap
  root((UEBA Signals))
    Time Patterns
      Login hours
      Work days vs weekends
      Session duration
    Location Patterns
      IP address
      Country / city
      VPN vs corporate network
    Resource Patterns
      Which files accessed
      Which apps used
      Volume of data touched
    Privilege Patterns
      Role changes
      Unusual admin actions
      Access to sensitive systems
    Peer Group Comparison
      Does this user act like peers in same team?
      Outlier vs group norm
```

---

## 6. Privileged Account Audit Flow

Privileged accounts (admins, service accounts, root) need **extra scrutiny** because they can cause the most damage.

```mermaid
flowchart TD
    PA([Privileged User\nlogs into PAM tool]) --> CHECKOUT[Checks out\ncredential from vault]
    CHECKOUT --> SESSION[PAM records session\nkeystrokes, screen, commands]
    SESSION --> WORK[Performs admin work\non target system]
    WORK --> CHECKIN[Checks credential back in\nPAM rotates password]
    CHECKIN --> REVIEW[Session recording\nsaved for review]

    REVIEW --> ANALYTICS[Automated review:\nDid they run any risky commands?\nsudo rm -rf? large data exports?]
    ANALYTICS --> FLAG{Suspicious\nactivity?}
    FLAG -->|No| LOG[(Stored in\naudit log)]
    FLAG -->|Yes| ALERT[🚨 Alert security team]
    ALERT --> INVESTIGATE[Investigate and\ncreate incident]
```

---

## 7. SOAR Automated Response

**Security Orchestration, Automation and Response (SOAR)** automatically responds to alerts — faster than a human ever could.

> Like an automated sprinkler system: it doesn't wait for a firefighter to arrive — it starts responding the moment it detects fire.

```mermaid
sequenceDiagram
    participant SIEM as SIEM
    participant SOAR as SOAR Platform
    participant IAM as IAM System
    participant ITSM as Ticketing System
    participant SOC as SOC Analyst

    SIEM->>SOAR: Alert: Impossible travel detected\nUser: alice@corp.com
    Note over SOAR: Playbook triggered automatically

    SOAR->>IAM: Revoke all active sessions for alice
    IAM-->>SOAR: Sessions revoked ✅

    SOAR->>IAM: Force MFA re-enrolment
    IAM-->>SOAR: Flag set ✅

    SOAR->>ITSM: Create incident ticket\nPriority: High\nAssignee: SOC Team
    ITSM-->>SOAR: Ticket #INC-4821 created

    SOAR->>SOC: Email + Slack alert\n"Alice's account suspended — investigate"
    SOC->>SOAR: Analyst acknowledges alert

    Note over SOC: Analyst reviews evidence,\ndetermines if real attack or false positive
    SOC->>IAM: Restore access if false positive
```

---

## 8. Identity Monitoring Dashboard

What to track at a glance — the key metrics every security team should monitor.

```mermaid
flowchart TD
    subgraph DASH["🖥️ Identity Security Dashboard"]
        subgraph AUTH["Authentication Health"]
            M1[Failed Logins\nlast 24h]
            M2[MFA Challenge\nFailure Rate]
            M3[Locked Accounts\ncurrent count]
        end

        subgraph RISK["Risk Signals"]
            M4[Impossible Travel\nAlerts Today]
            M5[After-Hours\nLogins]
            M6[New Country\nAccess]
        end

        subgraph PRIV["Privileged Access"]
            M7[Active PAM\nSessions]
            M8[Privilege Escalations\nToday]
            M9[Dormant Admin\nAccounts]
        end

        subgraph LIFECYCLE["Lifecycle Hygiene"]
            M10[Inactive Accounts\n90+ days no login]
            M11[Accounts Without\nMFA Enrolled]
            M12[Stale Access\nPending Review]
        end
    end
```

**Key thresholds to alert on:**

| Metric | Alert threshold |
|---|---|
| Failed logins per user | > 5 in 10 minutes |
| Failed logins across org | > 1,000 in 1 hour (spray attack) |
| Impossible travel | Any occurrence |
| Dormant accounts | 90+ days inactive |
| Admin accounts without MFA | Any |
| Privileged session duration | > 8 hours continuous |

---

## Quick Reference Cheat Sheet

| Concept | Simple Meaning | Key Metric / Tool |
|---|---|---|
| **Account Lockout** | Temporarily block after failed attempts | 5-10 attempts / 10 min window |
| **Impossible Travel** | Physics-defying logins = compromise | Alert on any occurrence |
| **UEBA** | AI learns normal, flags abnormal | Risk score threshold-based alerting |
| **Privileged Audit** | Record everything admins do | PAM session recording + review |
| **SOAR** | Automated incident response | Playbooks: suspend → ticket → alert |
| **SIEM** | Central log aggregation + correlation | Sentinel, Splunk, Chronicle |
| **Dashboard KPIs** | Real-time identity health metrics | Failed logins, MFA failures, dormant accounts |
