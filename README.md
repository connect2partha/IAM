# Identity and Access Management — Learning Index

## Quick Navigation

| # | Section | Document |
|---|---|---|
| 1 | Foundational Concepts | [01-foundational-concepts.md](01-foundational-concepts.md) |
| 2 | Managing Digital Identities | [02-managing-digital-identities.md](02-managing-digital-identities.md) |
| 3 | Authentication | [03-authentication.md](03-authentication.md) |
| 4 | Multi Factor Authentication (MFA) | [04-multi-factor-authentication.md](04-multi-factor-authentication.md) |
| 5 | Authorization | [05-authorization.md](05-authorization.md) |
| 6 | Protocols and Standards | [06-protocols-and-standards.md](06-protocols-and-standards.md) |
| 7 | Cryptography | [07-cryptography.md](07-cryptography.md) |
| 8 | Hashing | [08-hashing.md](08-hashing.md) |
| 9 | JSON Web Tokens (JWT) | [09-jwt.md](09-jwt.md) |
| 10 | OAuth 1.0 | [10-oauth1.md](10-oauth1.md) |
| 11 | OAuth 2.0 | [11-oauth2.md](11-oauth2.md) |
| 12 | OpenID Connect (OIDC) | [12-openid-connect.md](12-openid-connect.md) |
| 13 | Single Sign-On (SSO) | [13-single-sign-on.md](13-single-sign-on.md) |
| 14 | Directory Services | [14-directory-services.md](14-directory-services.md) |
| 15 | Privileged Access Management (PAM) | [15-privileged-access-management.md](15-privileged-access-management.md) |
| 16 | Identity Governance and Administration (IGA) | [16-identity-governance-administration.md](16-identity-governance-administration.md) |
| 17 | User Provisioning | [17-user-provisioning.md](17-user-provisioning.md) |
| 18 | Cloud IAM | [18-cloud-iam.md](18-cloud-iam.md) |
| 19 | Customer Identity and Access Management (CIAM) | [19-customer-iam.md](19-customer-iam.md) |
| 20 | Zero Trust Architecture | [20-zero-trust-architecture.md](20-zero-trust-architecture.md) |
| 21 | API and Service Identity | [21-api-and-service-identity.md](21-api-and-service-identity.md) |
| 22 | PKI and Certificates | [22-pki-and-certificates.md](22-pki-and-certificates.md) |
| 23 | Identity Threats and Attack Vectors | [23-identity-threats.md](23-identity-threats.md) |
| 24 | Identity Security Monitoring | [24-identity-security-monitoring.md](24-identity-security-monitoring.md) |
| 25 | Compliance and Regulatory Frameworks | [25-compliance-and-regulatory.md](25-compliance-and-regulatory.md) |
| 26 | IAM Architecture and Design Patterns | [26-iam-architecture-and-design.md](26-iam-architecture-and-design.md) |
| 27 | Key Tools and Platforms | [27-tools-and-platforms.md](27-tools-and-platforms.md) |
| 28 | Recommended Learning Path | [28-learning-path.md](28-learning-path.md) |
| 29 | Certifications to Consider | [29-certifications.md](29-certifications.md) |

---

## [1. Foundational Concepts](01-foundational-concepts.md)
- What is IAM — identity, authentication, authorization, accountability
- Principle of Least Privilege (PoLP)
- Separation of Duties (SoD)
- Defense in Depth
- Zero Trust Architecture (ZTA) — NIST SP 800-207
- Identity Lifecycle (joiner / mover / leaver)
- Trust models — transitive, explicit, implicit

---

## [2. Managing Digital Identities](02-managing-digital-identities.md)
- What is a digital identity — attributes, credentials, entitlements
- Types of identities — human, machine, service, privileged, NPE
- Identity attributes — core, work, and security attributes
- Identity Assurance Levels (IAL 1 / 2 / 3) — NIST 800-63
- Identity stores and repositories — LDAP, AD, cloud directory, CIAM
- Authoritative source of identity — HR as system of record
- Identity federation — trusting identities across organisations
- Non-Person Entities (NPEs) — apps, services, devices, pipelines
- Identity lifecycle management — requested → active → archived
- Identity governance — preventing drift, orphans, and SoD conflicts

---

## [3. Authentication](03-authentication.md)

### 3.1 Factors
- Something you know — passwords, PINs, security questions
- Something you have — hardware tokens, smart cards, TOTP apps
- Something you are — biometrics (fingerprint, face, iris)
- Somewhere you are — geolocation, IP-based controls

### 3.2 Mechanisms
- Password policies — complexity, rotation, breach detection (HaveIBeenPwned)
- Multi-Factor Authentication (MFA) — TOTP, HOTP, push, SMS (and why SMS is weak)
- Passwordless — FIDO2, WebAuthn, passkeys
- Certificate-based authentication (PKI, smart cards, mTLS)
- Hardware security keys — YubiKey, FIDO U2F

### 3.3 Session Management
- Session tokens, cookies, sliding vs fixed expiration
- Refresh tokens vs access tokens
- Token revocation and logout propagation

---

## [4. Multi Factor Authentication (MFA)](04-multi-factor-authentication.md)
- Why MFA matters — the broken state of passwords alone
- MFA factor hierarchy — strength comparison from SMS to passkeys
- TOTP and HOTP — how time-based and counter-based OTPs work
- SMS OTP — how it works and why it is weak (SIM swap, SS7)
- Push notifications — flow, MFA fatigue attacks, number matching defence
- Hardware security keys — FIDO U2F / FIDO2, phishing-resistant by design
- Passkeys — passwordless MFA, biometric binding, cross-device sync
- Adaptive / risk-based MFA — step-up authentication by risk score
- MFA in enterprise — Conditional Access, always-on vs step-up
- MFA bypass attacks — AiTM phishing, push bombing, SIM swapping
- MFA recovery — backup codes, secondary device, weak recovery risks

---

## [5. Authorization](05-authorization.md)

### 5.1 Access Control Models
- **DAC** — Discretionary Access Control (owner sets permissions)
- **MAC** — Mandatory Access Control (labels/clearances, Bell-LaPadula)
- **RBAC** — Role-Based Access Control (roles → permissions)
- **ABAC** — Attribute-Based Access Control (policy rules over attributes)
- **PBAC** — Policy-Based Access Control
- **ReBAC** — Relationship-Based Access Control (Google Zanzibar model)

### 5.2 Access Control Concepts
- Subjects, objects, permissions
- Role hierarchies and role explosion
- Just-in-Time (JIT) and Just-Enough-Access (JEA)
- Implicit deny vs explicit deny
- Policy evaluation order and conflict resolution

---

## [6. Protocols and Standards](06-protocols-and-standards.md)

### 6.1 Federated Identity
- **SAML 2.0** — assertions, IdP-initiated vs SP-initiated SSO, metadata exchange
- **OAuth 2.0** — authorization flows (Authorization Code, Client Credentials, Device, PKCE)
- **OpenID Connect (OIDC)** — ID token, UserInfo endpoint, discovery
- **WS-Federation** — passive requestor profile, ADFS context

### 6.2 Directory and Provisioning
- **LDAP** — DIT structure, DN, attributes, search filters, LDAPS
- **Active Directory** — forests, domains, trusts, GPOs, Kerberos integration
- **SCIM 2.0** — automated provisioning/deprovisioning, schema, operations
- **Kerberos** — AS, TGS, TGT, service tickets, delegation (constrained vs unconstrained)
- **RADIUS** — NPS, 802.1X, EAP variants

### 6.3 Token Formats
- **JWT** (JSON Web Token) — header, payload, signature, `alg` pitfalls (`none`, RS256 vs HS256)
- **SAML Assertions** — XML structure, signing, encryption
- **Opaque tokens** — introspection endpoint (RFC 7662)

---

## [7. Cryptography](07-cryptography.md)
- What cryptography provides — confidentiality, integrity, authenticity, non-repudiation
- Symmetric encryption — AES, shared key, authenticated encryption (AES-GCM)
- Asymmetric encryption — RSA, ECC, public/private key pair
- Digital signatures — sign with private key, verify with public key
- Key exchange — Diffie-Hellman, ECDH, Perfect Forward Secrecy
- TLS handshake — where asymmetric and symmetric crypto meet
- Encryption in IAM — passwords at rest, session tokens, TLS in transit, JWT signing
- Key management lifecycle — generate, distribute, store, rotate, revoke, destroy
- Common cryptographic attacks — brute force, padding oracle, downgrade, weak algorithms
- Algorithm cheat sheet — AES-256-GCM, RSA-2048, ECDSA P-256, Ed25519

---

## [8. Hashing](08-hashing.md)
- What is hashing — one-way, deterministic, fixed-size output (fingerprint analogy)
- Common hash algorithms — MD5 (broken), SHA-1 (deprecated), SHA-256, SHA-3, BLAKE2
- Why plain hashing is not enough for passwords — rainbow table attacks
- Salting — random salt defeats pre-computed tables
- Password hashing algorithms — bcrypt, PBKDF2, scrypt, Argon2id
- HMAC — hash-based message authentication, OAuth 1.0 signing, webhook verification
- Hashing in digital signatures — why sign a hash, not the full message
- Hashing for integrity — file checksums, certificate fingerprints, Git commits, Docker digests
- Hash in IAM protocols — JWT HS256, RS256, SAML XML-DSIG, TOTP (HMAC-SHA1)
- Hashing vs encryption — when to use which

---

## [9. JSON Web Tokens (JWT)](09-jwt.md)
- What is a JWT — compact, self-contained signed token
- JWT structure — Header . Payload . Signature (Base64URL encoded)
- Registered claims — iss, sub, aud, exp, iat, nbf, jti
- Signing algorithms — HS256 (symmetric), RS256 / ES256 (asymmetric)
- JWT validation pipeline — signature → expiry → issuer → audience
- JWT lifecycle — issuance, use, expiry, refresh
- Where JWTs are used — OAuth 2.0, OIDC, API Bearer tokens
- Security pitfalls — `alg: none`, algorithm confusion, weak secrets, no expiry
- JWT vs Opaque tokens — stateless vs revocable

---

## [10. OAuth 1.0](10-oauth1.md)
- What OAuth 1.0 solved — delegated access without sharing passwords
- Core concepts — Consumer Key/Secret, Request Token, Access Token, Verifier
- 3-Legged OAuth flow — Request Token → User Authorisation → Access Token
- Request signing — HMAC-SHA1 signature, nonce, timestamp
- OAuth 1.0a — session fixation fix
- Limitations — cryptographic complexity, not mobile-friendly
- Why OAuth 2.0 replaced it

---

## [11. OAuth 2.0](11-oauth2.md)
- Core roles — Resource Owner, Client, Authorisation Server, Resource Server
- Tokens — Access Token, Refresh Token, scopes
- Authorization Code + PKCE — the recommended flow for user-facing apps
- Client Credentials — machine-to-machine, no user involved
- Device Authorization Grant — browserless devices (TV, CLI)
- Implicit and ROPC flows — deprecated, never use for new apps
- Scopes and consent — requesting only what is needed
- Token introspection and revocation
- PKCE deep dive — code_verifier, code_challenge, replay prevention
- OAuth 2.0 security best practices — RFC 9700

---

## [12. OpenID Connect (OIDC)](12-openid-connect.md)
- What OIDC adds to OAuth 2.0 — the identity layer (authentication)
- The ID Token — JWT with claims about the authenticated user
- Standard claims — iss, sub, aud, exp, iat, nonce, acr, amr
- Authorization Code flow with OIDC — full sequence
- UserInfo endpoint — fetching additional profile claims
- OIDC Discovery — `.well-known/openid-configuration`, JWKS URI
- OIDC Scopes — openid, profile, email, phone, address, offline_access
- OIDC in practice — social login, enterprise SSO, mobile apps
- OIDC vs SAML — when to use which
- ID token validation checklist

---

## [13. Single Sign-On (SSO)](13-single-sign-on.md)
- What is SSO — problems solved and risks if done poorly
- Core concepts — IdP, SP, assertion, trust relationship
- SAML 2.0 — SP-initiated and IdP-initiated flows, metadata exchange
- OIDC / OAuth SSO — Authorization Code + PKCE, token types, ID token anatomy
- SAML vs OIDC — decision flowchart and comparison table
- SSO session management — IdP session vs SP session, timeout policies
- SSO logout — front-channel and back-channel in depth, why logout is hard
- Enterprise SSO patterns — cloud-only, hybrid (AD + Entra ID), header-based legacy
- Social login — account linking, consent flow
- Enterprise SSO vs CIAM — employees vs customers
- SSO security — blast radius, session attacks, redirect URI validation
- SSO for APIs — token propagation, machine-to-machine (Client Credentials)

---

## [14. Directory Services](14-directory-services.md)
- Active Directory DS — schema, replication, sites
- Azure Entra ID (Azure AD) — tenants, B2B, B2C, hybrid join
- OpenLDAP — configuration DIT, overlays, ACLs
- Cloud directory services — AWS Directory Service, Google Cloud Directory

---

## [15. Privileged Access Management (PAM)](15-privileged-access-management.md)
- Privileged accounts — local admin, domain admin, service accounts, root
- Credential vaulting — check-in / check-out, password rotation
- Just-in-Time privileged access — time-bounded elevation
- Session monitoring and recording
- Break-glass (emergency) accounts and procedures
- Tools — CyberArk, Delinea, BeyondTrust, HashiCorp Vault

---

## [16. Identity Governance and Administration (IGA)](16-identity-governance-administration.md)
- Identity lifecycle automation — provisioning, reconciliation, deprovisioning
- Access request and approval workflows
- Access certifications / access reviews (periodic recertification)
- Role management and role mining
- Separation of Duties enforcement
- Audit logging and reporting for compliance
- Tools — SailPoint, Saviynt, One Identity, Microsoft Identity Governance

---

## [17. User Provisioning](17-user-provisioning.md)
- What is user provisioning — manual vs automated
- The provisioning lifecycle — Joiner, Mover, Leaver
- Provisioning methods — role-based, request-based, JIT, HR-driven
- SCIM 2.0 — standard API for create / update / delete across apps
- Deprovisioning — orphaned accounts and offboarding flows
- Reconciliation — detecting and fixing access drift
- Provisioning governance — role profiles, approvals, access reviews, SoD
- Patterns by organisation size — small to enterprise

---

## [18. Cloud IAM](18-cloud-iam.md)
- **AWS IAM** — users, groups, roles, policies (identity vs resource), trust policies, SCPs, Permission Boundaries
- **Azure RBAC** — built-in roles, custom roles, management groups, PIM (Privileged Identity Management)
- **GCP IAM** — projects, folders, Org policies, Workload Identity Federation
- Cross-cloud federation and workload identity
- Service accounts and workload identity best practices
- Instance metadata and IMDS credential exposure risks

---

## [19. Customer Identity and Access Management (CIAM)](19-customer-iam.md)
- Consumer-facing vs enterprise IAM differences
- Progressive profiling and consent management
- GDPR / privacy considerations — data minimization, right to deletion
- Social login and account linking
- Fraud detection and risk-based authentication
- Tools — Auth0, Okta CIAM, Ping Identity, AWS Cognito, Firebase Auth

---

## [20. Zero Trust Architecture](20-zero-trust-architecture.md)
- Core tenets — verify explicitly, least privilege, assume breach
- Microsegmentation — network, application, data layers
- Continuous verification and adaptive authentication
- Device trust and endpoint health signals
- NIST SP 800-207 reference architecture
- Microsoft SASE / Entra + Defender integration
- BeyondCorp (Google's ZT model)

---

## [21. API and Service Identity](21-api-and-service-identity.md)
- API keys — issuance, rotation, scoping, risks
- Service-to-service authentication — mTLS, signed JWTs, Workload Identity
- OAuth 2.0 Client Credentials flow for machine identities
- Service mesh identity — SPIFFE/SPIRE, Istio mTLS
- Secrets management — HashiCorp Vault, AWS Secrets Manager, Azure Key Vault

---

## [22. PKI and Certificates](22-pki-and-certificates.md)
- X.509 certificate structure — SAN, CN, extensions
- Certificate Authorities — root, intermediate, issuing CA
- Certificate lifecycle — issuance, renewal, revocation (CRL, OCSP)
- mTLS — client certificate authentication
- Code signing and document signing

---

## [23. Identity Threats and Attack Vectors](23-identity-threats.md)
- Credential stuffing and password spraying
- Phishing and adversary-in-the-middle (AiTM) attacks
- Pass-the-Hash / Pass-the-Ticket (Kerberos attacks)
- Golden Ticket and Silver Ticket attacks
- Token theft and replay
- OAuth misuse — open redirectors, implicit flow risks
- SAML vulnerabilities — XML signature wrapping
- SIM swapping (MFA bypass)
- **Common techniques of identity theft** — phishing, smishing, vishing, data breach, malware/keyloggers, mail theft, dumpster diving, shoulder surfing, social engineering/pretexting, card skimming, synthetic identity fraud, account takeover
- Insider threat and privilege abuse

---

## [24. Identity Security Monitoring](24-identity-security-monitoring.md)
- Failed login detection and account lockout policy
- Anomalous behavior detection — impossible travel, unusual hours
- Privileged account activity auditing
- Identity threat detection with SIEM/SOAR (Sentinel, Splunk, Chronicle)
- UEBA — User and Entity Behavior Analytics

---

## [25. Compliance and Regulatory Frameworks](25-compliance-and-regulatory.md)
- **SOX** — segregation of duties, audit trails for financial systems
- **HIPAA** — minimum necessary access, PHI access logging
- **GDPR / CCPA** — data subject rights, consent, identity data handling
- **SOC 2 Type II** — logical access controls, review evidence
- **ISO/IEC 27001** — A.9 Access Control domain
- **NIST SP 800-53** — AC (Access Control), IA (Identification and Authentication) families
- **PCI DSS** — Requirement 7 (access restriction) and 8 (identification and authentication)

---

## [26. IAM Architecture and Design Patterns](26-iam-architecture-and-design.md)
- Centralized vs federated identity
- Authoritative source of identity (HR system as system of record)
- Identity hub / identity broker pattern
- Multi-tenant IAM design
- Cross-domain trust and federation design
- High availability and DR for IAM services

---

## [27. Key Tools and Platforms (Ecosystem Map)](27-tools-and-platforms.md)

| Category | Tools |
|---|---|
| Enterprise IAM / SSO | Okta, Ping Identity, Azure Entra ID, ForgeRock |
| IGA | SailPoint, Saviynt, One Identity |
| PAM | CyberArk, Delinea, BeyondTrust |
| CIAM | Auth0, AWS Cognito, Firebase Auth |
| Secrets | HashiCorp Vault, AWS Secrets Manager, Azure Key Vault |
| Cloud IAM | AWS IAM, Azure RBAC, GCP IAM |
| Directory | Active Directory, OpenLDAP, Azure Entra ID |
| PKI | EJBCA, Microsoft ADCS, Venafi |

---

## [28. Recommended Learning Path](28-learning-path.md)

| Level | Sections |
|---|---|
| Beginner | 1, 2, 3, 4, 5, 6 |
| Intermediate | 7, 8, 9, 10, 11, 12, 13, 14, 15 |
| Advanced | 16, 17, 18, 19, 20, 21, 22, 23 |
| Compliance / GRC track | 25, 16, 15 |

---

## [29. Certifications to Consider](29-certifications.md)
- **CISSP** — broad security including access control domain
- **CISM** — management-focused, includes identity governance
- **CompTIA Security+** — foundational, covers authentication and access
- **Okta Certified Professional / Administrator**
- **Microsoft SC-300** — Identity and Access Administrator
- **AWS Certified Security Specialty** — cloud IAM focus
- **CyberArk Defender / Sentry** — PAM-specific
