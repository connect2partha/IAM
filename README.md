# Identity and Access Management — Learning Index

## Quick Navigation

| # | Section | Document |
|---|---|---|
| 1 | Foundational Concepts | [01-foundational-concepts.md](01-foundational-concepts.md) |
| 2 | Authentication | [02-authentication.md](02-authentication.md) |
| 3 | Authorization | [03-authorization.md](03-authorization.md) |
| 4 | Protocols and Standards | [04-protocols-and-standards.md](04-protocols-and-standards.md) |
| 5 | Single Sign-On (SSO) | [05-single-sign-on.md](05-single-sign-on.md) |
| 6 | Directory Services | [06-directory-services.md](06-directory-services.md) |
| 7 | Privileged Access Management (PAM) | [07-privileged-access-management.md](07-privileged-access-management.md) |
| 8 | Identity Governance and Administration (IGA) | [08-identity-governance-administration.md](08-identity-governance-administration.md) |
| 9 | Cloud IAM | [09-cloud-iam.md](09-cloud-iam.md) |
| 10 | Customer Identity and Access Management (CIAM) | [10-customer-iam.md](10-customer-iam.md) |
| 11 | Zero Trust Architecture | [11-zero-trust-architecture.md](11-zero-trust-architecture.md) |
| 12 | API and Service Identity | [12-api-and-service-identity.md](12-api-and-service-identity.md) |
| 13 | PKI and Certificates | [13-pki-and-certificates.md](13-pki-and-certificates.md) |
| 14 | Identity Threats and Attack Vectors | [14-identity-threats.md](14-identity-threats.md) |
| 15 | Identity Security Monitoring | [15-identity-security-monitoring.md](15-identity-security-monitoring.md) |
| 16 | Compliance and Regulatory Frameworks | [16-compliance-and-regulatory.md](16-compliance-and-regulatory.md) |
| 17 | IAM Architecture and Design Patterns | [17-iam-architecture-and-design.md](17-iam-architecture-and-design.md) |
| 18 | Key Tools and Platforms | [18-tools-and-platforms.md](18-tools-and-platforms.md) |
| 19 | Recommended Learning Path | [19-learning-path.md](19-learning-path.md) |
| 20 | Certifications to Consider | [20-certifications.md](20-certifications.md) |

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

## [2. Authentication](02-authentication.md)

### 2.1 Factors
- Something you know — passwords, PINs, security questions
- Something you have — hardware tokens, smart cards, TOTP apps
- Something you are — biometrics (fingerprint, face, iris)
- Somewhere you are — geolocation, IP-based controls

### 2.2 Mechanisms
- Password policies — complexity, rotation, breach detection (HaveIBeenPwned)
- Multi-Factor Authentication (MFA) — TOTP, HOTP, push, SMS (and why SMS is weak)
- Passwordless — FIDO2, WebAuthn, passkeys
- Certificate-based authentication (PKI, smart cards, mTLS)
- Hardware security keys — YubiKey, FIDO U2F

### 2.3 Session Management
- Session tokens, cookies, sliding vs fixed expiration
- Refresh tokens vs access tokens
- Token revocation and logout propagation

---

## [3. Authorization](03-authorization.md)

### 3.1 Access Control Models
- **DAC** — Discretionary Access Control (owner sets permissions)
- **MAC** — Mandatory Access Control (labels/clearances, Bell-LaPadula)
- **RBAC** — Role-Based Access Control (roles → permissions)
- **ABAC** — Attribute-Based Access Control (policy rules over attributes)
- **PBAC** — Policy-Based Access Control
- **ReBAC** — Relationship-Based Access Control (Google Zanzibar model)

### 3.2 Access Control Concepts
- Subjects, objects, permissions
- Role hierarchies and role explosion
- Just-in-Time (JIT) and Just-Enough-Access (JEA)
- Implicit deny vs explicit deny
- Policy evaluation order and conflict resolution

---

## [4. Protocols and Standards](04-protocols-and-standards.md)

### 4.1 Federated Identity
- **SAML 2.0** — assertions, IdP-initiated vs SP-initiated SSO, metadata exchange
- **OAuth 2.0** — authorization flows (Authorization Code, Client Credentials, Device, PKCE)
- **OpenID Connect (OIDC)** — ID token, UserInfo endpoint, discovery
- **WS-Federation** — passive requestor profile, ADFS context

### 4.2 Directory and Provisioning
- **LDAP** — DIT structure, DN, attributes, search filters, LDAPS
- **Active Directory** — forests, domains, trusts, GPOs, Kerberos integration
- **SCIM 2.0** — automated provisioning/deprovisioning, schema, operations
- **Kerberos** — AS, TGS, TGT, service tickets, delegation (constrained vs unconstrained)
- **RADIUS** — NPS, 802.1X, EAP variants

### 4.3 Token Formats
- **JWT** (JSON Web Token) — header, payload, signature, `alg` pitfalls (`none`, RS256 vs HS256)
- **SAML Assertions** — XML structure, signing, encryption
- **Opaque tokens** — introspection endpoint (RFC 7662)

---

## [5. Single Sign-On (SSO)](05-single-sign-on.md)
- IdP vs SP roles
- SSO session vs application session
- SSO logout — front-channel vs back-channel
- Enterprise SSO vs Customer SSO (CIAM)
- Social login (OAuth with Google, Apple, GitHub)

---

## [6. Directory Services](06-directory-services.md)
- Active Directory DS — schema, replication, sites
- Azure Entra ID (Azure AD) — tenants, B2B, B2C, hybrid join
- OpenLDAP — configuration DIT, overlays, ACLs
- Cloud directory services — AWS Directory Service, Google Cloud Directory

---

## [7. Privileged Access Management (PAM)](07-privileged-access-management.md)
- Privileged accounts — local admin, domain admin, service accounts, root
- Credential vaulting — check-in / check-out, password rotation
- Just-in-Time privileged access — time-bounded elevation
- Session monitoring and recording
- Break-glass (emergency) accounts and procedures
- Tools — CyberArk, Delinea, BeyondTrust, HashiCorp Vault

---

## [8. Identity Governance and Administration (IGA)](08-identity-governance-administration.md)
- Identity lifecycle automation — provisioning, reconciliation, deprovisioning
- Access request and approval workflows
- Access certifications / access reviews (periodic recertification)
- Role management and role mining
- Separation of Duties enforcement
- Audit logging and reporting for compliance
- Tools — SailPoint, Saviynt, One Identity, Microsoft Identity Governance

---

## [9. Cloud IAM](09-cloud-iam.md)
- **AWS IAM** — users, groups, roles, policies (identity vs resource), trust policies, SCPs, Permission Boundaries
- **Azure RBAC** — built-in roles, custom roles, management groups, PIM (Privileged Identity Management)
- **GCP IAM** — projects, folders, Org policies, Workload Identity Federation
- Cross-cloud federation and workload identity
- Service accounts and workload identity best practices
- Instance metadata and IMDS credential exposure risks

---

## [10. Customer Identity and Access Management (CIAM)](10-customer-iam.md)
- Consumer-facing vs enterprise IAM differences
- Progressive profiling and consent management
- GDPR / privacy considerations — data minimization, right to deletion
- Social login and account linking
- Fraud detection and risk-based authentication
- Tools — Auth0, Okta CIAM, Ping Identity, AWS Cognito, Firebase Auth

---

## [11. Zero Trust Architecture](11-zero-trust-architecture.md)
- Core tenets — verify explicitly, least privilege, assume breach
- Microsegmentation — network, application, data layers
- Continuous verification and adaptive authentication
- Device trust and endpoint health signals
- NIST SP 800-207 reference architecture
- Microsoft SASE / Entra + Defender integration
- BeyondCorp (Google's ZT model)

---

## [12. API and Service Identity](12-api-and-service-identity.md)
- API keys — issuance, rotation, scoping, risks
- Service-to-service authentication — mTLS, signed JWTs, Workload Identity
- OAuth 2.0 Client Credentials flow for machine identities
- Service mesh identity — SPIFFE/SPIRE, Istio mTLS
- Secrets management — HashiCorp Vault, AWS Secrets Manager, Azure Key Vault

---

## [13. PKI and Certificates](13-pki-and-certificates.md)
- X.509 certificate structure — SAN, CN, extensions
- Certificate Authorities — root, intermediate, issuing CA
- Certificate lifecycle — issuance, renewal, revocation (CRL, OCSP)
- mTLS — client certificate authentication
- Code signing and document signing

---

## [14. Identity Threats and Attack Vectors](14-identity-threats.md)
- Credential stuffing and password spraying
- Phishing and adversary-in-the-middle (AiTM) attacks
- Pass-the-Hash / Pass-the-Ticket (Kerberos attacks)
- Golden Ticket and Silver Ticket attacks
- Token theft and replay
- OAuth misuse — open redirectors, implicit flow risks
- SAML vulnerabilities — XML signature wrapping
- Insider threat and privilege abuse
- SIM swapping (MFA bypass)

---

## [15. Identity Security Monitoring](15-identity-security-monitoring.md)
- Failed login detection and account lockout policy
- Anomalous behavior detection — impossible travel, unusual hours
- Privileged account activity auditing
- Identity threat detection with SIEM/SOAR (Sentinel, Splunk, Chronicle)
- UEBA — User and Entity Behavior Analytics

---

## [16. Compliance and Regulatory Frameworks](16-compliance-and-regulatory.md)
- **SOX** — segregation of duties, audit trails for financial systems
- **HIPAA** — minimum necessary access, PHI access logging
- **GDPR / CCPA** — data subject rights, consent, identity data handling
- **SOC 2 Type II** — logical access controls, review evidence
- **ISO/IEC 27001** — A.9 Access Control domain
- **NIST SP 800-53** — AC (Access Control), IA (Identification and Authentication) families
- **PCI DSS** — Requirement 7 (access restriction) and 8 (identification and authentication)

---

## [17. IAM Architecture and Design Patterns](17-iam-architecture-and-design.md)
- Centralized vs federated identity
- Authoritative source of identity (HR system as system of record)
- Identity hub / identity broker pattern
- Multi-tenant IAM design
- Cross-domain trust and federation design
- High availability and DR for IAM services

---

## [18. Key Tools and Platforms (Ecosystem Map)](18-tools-and-platforms.md)

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

## [19. Recommended Learning Path](19-learning-path.md)

| Level | Sections |
|---|---|
| Beginner | 1, 2, 3, 4, 5, 6 |
| Intermediate | 7, 8, 9, 10, 12, 13 |
| Advanced | 11, 14, 15, 17 |
| Compliance / GRC track | 16, 8 (governance aspects), 7 |

---

## [20. Certifications to Consider](20-certifications.md)
- **CISSP** — broad security including access control domain
- **CISM** — management-focused, includes identity governance
- **CompTIA Security+** — foundational, covers authentication and access
- **Okta Certified Professional / Administrator**
- **Microsoft SC-300** — Identity and Access Administrator
- **AWS Certified Security Specialty** — cloud IAM focus
- **CyberArk Defender / Sentry** — PAM-specific
