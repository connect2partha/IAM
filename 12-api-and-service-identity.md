# API and Service Identity

> Simple explanations with diagrams for how machines and services prove who they are — not humans.

---

## 1. Human vs Machine Identity

We usually think of IAM for people — employees logging in with usernames and passwords. But modern applications are made of **hundreds of services** that also need to talk to each other securely.

> Think of it like this: a human shows their employee badge at the door. A service needs its own "badge" too — one that can't be forgotten, stolen from a desk, or shared around.

```mermaid
flowchart TD
    subgraph Human["👤 Human Identity"]
        H1[Person logs in] --> H2[Username + Password + MFA]
        H2 --> H3[Gets session token]
        H3 --> H4[Accesses app]
    end

    subgraph Machine["⚙️ Machine / Service Identity"]
        M1[Service starts up] --> M2[Uses API key / cert / workload identity]
        M2 --> M3[Gets access token]
        M3 --> M4[Calls another service]
    end

    Human -.->|Same goal: prove identity| Machine
```

---

## 2. API Keys

An **API key** is a secret string that a service includes in its requests to prove its identity — like a password for a machine.

> Like a gym membership card. You hand it to the reader, it verifies you're a member, and lets you in.

### API Key Lifecycle

```mermaid
flowchart LR
    A([Admin creates API key]) --> B[Scope it\nread-only? write? which endpoints?]
    B --> C[Distribute to service\nvia env var or vault]
    C --> D{Service uses key\nfor API calls}
    D --> E[Rotate key periodically\nor on suspected leak]
    E --> F[Revoke old key\nafter cutover]
    F --> D
```

### Risks of API Keys

| Risk | What happens |
|---|---|
| **No expiry** | A leaked key works forever |
| **Over-scoped** | Key can do more than the service needs |
| **Stored in code** | Accidentally committed to git — public forever |
| **Shared between services** | One leak compromises everything that uses it |
| **No audit trail** | Can't tell which service made which call |

---

## 3. OAuth 2.0 Client Credentials Flow

This is the standard way for a **service to get a token** without any human involvement.

> Think of it as a service calling the front desk, saying "I'm Service A, here's my ID and secret", and getting a visitor badge to access another floor.

```mermaid
sequenceDiagram
    participant SA as Service A
    participant AS as Auth Server
    participant SB as Service B

    SA->>AS: POST /token\nclient_id + client_secret + scope
    AS-->>SA: access_token (JWT, short-lived)

    SA->>SB: GET /api/resource\nAuthorization: Bearer <token>
    SB->>AS: Verify token (or validate JWT signature locally)
    AS-->>SB: Token valid, subject = Service A
    SB-->>SA: 200 OK — resource returned
```

**Key points:**
- No user involved — purely machine-to-machine
- Token is **short-lived** (minutes/hours), limiting damage if stolen
- Service authenticates with `client_id` + `client_secret` (or certificate)

---

## 4. Mutual TLS (mTLS)

Normal TLS: **your browser verifies the server's certificate** (is this really google.com?).
mTLS: **both sides verify each other** — the server also checks the client's certificate.

> Like a spy movie where both agents must show their credentials before either talks.

```mermaid
sequenceDiagram
    participant C as Client Service
    participant S as Server Service

    C->>S: ClientHello — I want to connect
    S-->>C: ServerHello + Server Certificate
    C->>C: Verify server cert\n(signed by trusted CA?)
    C-->>S: Client Certificate
    S->>S: Verify client cert\n(signed by trusted CA?)
    S-->>C: TLS Handshake Complete ✅

    Note over C,S: All traffic now encrypted\nBoth sides verified each other
    C->>S: Encrypted API request
    S-->>C: Encrypted response
```

**Why mTLS over API keys?**
- Certificate is tied to a cryptographic keypair — can't be copy-pasted like a string
- Automatically expired and rotated
- Identity is verified at the **transport layer**, before any application logic runs

---

## 5. Service Mesh Identity — SPIFFE / SPIRE

In a **service mesh** (e.g., Kubernetes with Istio), every microservice automatically gets a cryptographic identity called a **SVID (SPIFFE Verifiable Identity Document)**.

> Like a government issuing each citizen a chip-and-pin ID card automatically at birth. Every citizen (service) gets one, and they use it to prove identity to each other.

```mermaid
flowchart TD
    SPIRE[SPIRE Server\nTrusted Identity Authority]

    subgraph K8s["Kubernetes Cluster"]
        A1[Service A\nSVID: spiffe://company/service-a] 
        A2[Service B\nSVID: spiffe://company/service-b]
        A3[Service C\nSVID: spiffe://company/service-c]
        
        AGENT[SPIRE Agent\non each node]
    end

    SPIRE -->|Issues SVIDs| AGENT
    AGENT -->|Delivers SVID cert| A1
    AGENT -->|Delivers SVID cert| A2
    AGENT -->|Delivers SVID cert| A3

    A1 -->|mTLS using SVID| A2
    A2 -->|mTLS using SVID| A3
```

**Benefits:**
- Zero manually managed secrets between services
- Certs automatically rotated (every hour by default)
- Works across clouds, clusters, and VMs

---

## 6. Secrets Management

Services need **database passwords, API keys, TLS certificates** — these are secrets. Hardcoding them is dangerous. A secrets manager stores and delivers them securely.

> Think of it like a secure safe-deposit box at a bank. The service proves its identity, the bank hands over the contents, and the box auto-reseals with a new combination after each opening.

```mermaid
flowchart TD
    APP[Application / Service] -->|1. Authenticate\ne.g. via K8s service account| VAULT[(Secrets Manager\nHashiCorp Vault /\nAWS Secrets Manager /\nAzure Key Vault)]

    VAULT -->|2. Verify identity| POLICY{Authorised\nfor this secret?}
    POLICY -->|No| DENY([❌ Denied])
    POLICY -->|Yes| RETURN[3. Return short-lived\ndynamic secret]

    RETURN --> APP
    APP --> USE[4. Use secret\ne.g. DB password]

    VAULT --> ROTATE[5. Auto-rotate\nsecret on schedule]
    ROTATE --> VAULT
```

### Secrets Management Tools Compared

| Tool | Best for | Key feature |
|---|---|---|
| **HashiCorp Vault** | Multi-cloud, on-prem | Dynamic secrets, fine-grained policies |
| **AWS Secrets Manager** | AWS-native apps | Automatic RDS password rotation |
| **Azure Key Vault** | Azure-native apps | Managed HSM, tight Azure RBAC |

---

## 7. Static API Keys vs Dynamic Tokens

```mermaid
flowchart TD
    subgraph Static["🔑 Static API Key (Risky)"]
        S1[Created once] --> S2[Never expires]
        S2 --> S3[Stored in config file / env]
        S3 --> S4[Leaked in git commit]
        S4 --> S5[Attacker uses key\nforever ❌]
    end

    subgraph Dynamic["🎫 Dynamic Short-lived Token (Safe)"]
        D1[Service authenticates\nat startup] --> D2[Gets token\nvalid 15 minutes]
        D2 --> D3[Uses token for calls]
        D3 --> D4{Token expired?}
        D4 -->|Yes| D1
        D4 -->|No| D3
        D2 --> D5[Even if stolen,\nexpires soon ✅]
    end
```

---

## Quick Reference Cheat Sheet

| Concept | Simple Meaning | Best Practice |
|---|---|---|
| **API Key** | Password for a service | Scope tightly, rotate regularly, never commit to code |
| **Client Credentials Flow** | Machine gets OAuth token | Use short expiry, prefer cert-based client auth |
| **mTLS** | Both sides show certificates | Use in service-to-service comms |
| **SPIFFE/SPIRE** | Auto-issued identity for every service | Use in Kubernetes / microservices |
| **Secrets Manager** | Secure vault for passwords/keys | Use dynamic secrets, never hardcode |
| **Dynamic tokens** | Short-lived, auto-expiring access | Always prefer over long-lived static keys |
