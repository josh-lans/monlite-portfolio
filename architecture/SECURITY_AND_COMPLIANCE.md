# Security Architecture & Compliance

## Overview

MonLite has undergone an **8-phase enterprise security remediation** and **independent multi-AI verification audits** covering OWASP Top 10, SOC2, GDPR, and ISO27001 compliance requirements.

## Multi-AI Verification Approach

Security-critical features were independently reviewed across multiple AI platforms to catch blind spots:

| AI Platform | Audit Scope |
|-------------|-------------|
| **OpenAI Codex** | P0 security audit — OWASP Top 10, session management, injection vectors |
| **Claude** (Anthropic) | Architecture review, alarm engine integrity, RFC security |
| **Google Gemini** | Scalability analysis, connection pool tuning, query optimization |
| **GitHub Copilot** | Code-level security patterns, dependency vulnerability scanning |

This cross-model verification approach provides defense-in-depth against AI-specific blind spots. Each platform identifies different classes of issues.

## Security Hardening Timeline

### Phase 1-4: Foundation
- Content-Security-Policy and Permissions-Policy headers
- DOMPurify XSS sanitization on all user input
- CORS hardening with explicit origin allowlist
- ZIP path traversal guards on file upload
- Pydantic schema constraints (extra="forbid")
- Open redirect protection on OIDC logout

### Phase 5-6: Session & Auth
- HttpOnly cookie sessions (migrated from localStorage tokens)
- CSRF double-submit cookie protection
- Argon2 password hashing with configurable policy
- Account lockout after failed attempts
- Session inactivity timeout + max lifetime ceiling
- Concurrent session limits

### Phase 7: Enterprise (Codex P0 Audit)
- SSRF `is_private_ip()` fail-closed on DNS errors
- HMAC audit key required in production (RuntimeError at startup if missing)
- `DB_SSLMODE` auto-upgrades to "require" in production
- Session write throttling (~98% fewer DB writes)
- Rate limiter fail-closed in production
- DataTable row caps to prevent DOM bloat

### Phase 8: SSO & Access Control
- OIDC Authorization Code + PKCE (Okta, Microsoft Entra ID)
- JIT user provisioning from IdP claims
- Group-to-role mapping (IdP group → MonLite role)
- Group-to-team mapping (IdP group → company-scoped team)
- Force SSO mode (disable local login)
- IdP-managed MFA via SSO delegation

## Authentication Architecture

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Local      │     │    LDAP     │     │  OIDC SSO   │
│  (Argon2)    │     │  (AD/LDAP)  │     │ (Okta/Entra)│
└──────┬───────┘     └──────┬──────┘     └──────┬──────┘
       │                    │                    │
       └────────────────────┼────────────────────┘
                            │
                    ┌───────▼────────┐
                    │  Session Layer  │
                    │  HttpOnly Cookie│
                    │  + CSRF Token   │
                    └───────┬────────┘
                            │
                    ┌───────▼────────┐
                    │    RBAC Layer   │
                    │  4 roles + teams│
                    │  Company-scoped │
                    └────────────────┘
```

## RBAC Model

| Role | Capabilities |
|------|-------------|
| **System Admin** | Full access, user management, SSO config, system settings |
| **Engineer Admin** | Manage infrastructure within assigned companies, create runbooks |
| **Engineer** | View/acknowledge alarms, run test checks, use Luna |
| **Viewer** | Read-only dashboards and reports |

**Company-level isolation:** Users only see infrastructure belonging to their assigned companies. SQL subquery RBAC filters applied server-side (never client-side).

**Team-based elevation:** Engineers can be assigned to teams with elevated roles for specific companies (e.g., "SAP Team" with Engineer Admin access to "Acme Corp" only).

## Data Protection

| Layer | Protection |
|-------|-----------|
| **Credentials at rest** | Fernet encryption (AES-128-CBC + HMAC), 0600 key file permissions |
| **Audit logs** | HMAC-signed integrity verification (`MONLITE_AUDIT_HMAC_KEY`) |
| **Database** | Enforced PostgreSQL SSL in production (`DB_SSLMODE=require`) |
| **Collector communication** | HTTPS/TLS with optional certificate pinning |
| **Passwords** | Argon2id hashing with configurable policy (length, complexity, expiry) |
| **Sessions** | HttpOnly cookies, CSRF tokens, configurable lifetime/inactivity |

## Compliance Readiness

| Standard | Coverage |
|----------|---------|
| **OWASP Top 10** | All categories addressed across 8 phases |
| **SOC2** | Access control, audit logging, encryption, session management |
| **GDPR** | Data minimization, audit trail, configurable retention |
| **ISO27001** | Information security management controls |
| **ITAR** | Multi-tenant company-level isolation for regulated industries |
