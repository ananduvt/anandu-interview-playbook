# Security

## Authentication mechanisms
- **Session-cookie** — server stores session; cookie holds session id.
- **JWT (JSON Web Token)** — stateless, signed token (header.payload.signature); self-contained claims.
- **OAuth2 / OIDC** — delegated authorization; access + refresh tokens; OIDC adds identity (ID token).
- **API keys**, **mTLS** (mutual TLS), **HMAC request signing** (shared-secret signature per request).

## Authentication vs Authorization
- **AuthN** — verify identity. **AuthZ** — verify permissions (RBAC roles, scopes, ABAC).

## Hashing vs encryption
- **Hashing** — one-way, irreversible (passwords: bcrypt/argon2 + salt; integrity: SHA-256).
- **Encryption** — two-way with a key (confidentiality). Symmetric (AES) vs asymmetric (RSA/ECC).
- **HMAC** — keyed hash for message authenticity + integrity (symmetric secret).

## OWASP Top 10 (know these)
Broken access control · cryptographic failures · **injection** (SQL/command) · insecure design ·
security misconfiguration · vulnerable/outdated components · auth failures · data integrity failures ·
logging/monitoring failures · **SSRF**.

## Defenses
- Parameterized queries (prevent SQLi); input validation/output encoding (XSS); CSRF tokens.
- Least privilege; secrets in a vault (never in code); rotate credentials.
- TLS everywhere; mask/redact PII in logs; dependency scanning (CVE) + patching.
- Constant-time comparison for secrets/signatures; rate limiting.

## Spring Security
`SecurityFilterChain` bean; method security (`@PreAuthorize`, `@Secured`); OAuth2 resource server;
`BCryptPasswordEncoder` for passwords.
