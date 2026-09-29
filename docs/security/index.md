# Vulnerabilities & Secure Practices

Complements auth topics in [Spring & Web → Security](../spring-web/security.md).

## OWASP Top 10 (2021)
1. Broken access control
2. Cryptographic failures
3. **Injection** (SQL, command, LDAP)
4. Insecure design
5. Security misconfiguration
6. Vulnerable & outdated components
7. Identification & authentication failures
8. Software & data integrity failures
9. Security logging & monitoring failures
10. **Server-Side Request Forgery (SSRF)**

## Common attacks & defenses
| Attack | Defense |
|--------|---------|
| SQL injection | parameterized queries / prepared statements, ORM |
| XSS | output encoding, CSP, sanitize input |
| CSRF | anti-CSRF tokens, SameSite cookies |
| SSRF | allow-list outbound hosts, block internal ranges |
| Broken access control | server-side authz checks, deny by default |
| Sensitive data exposure | TLS, encryption at rest, mask PII in logs |

## Dependency / supply-chain security
- Scan dependencies for CVEs (OWASP Dependency-Check, Snyk, WIZ, PRISMA); **pin non-vulnerable versions**.
- Keep libraries patched; watch advisories (e.g. Log4Shell). Reference: [CISA KEV catalog](https://www.cisa.gov/known-exploited-vulnerabilities-catalog).

## Secure coding principles
Least privilege · defense in depth · fail securely · validate at boundaries · never trust client input ·
secrets in a vault (not code) · constant-time comparison for secrets · rotate credentials · audit logging.
