# Compliance for Financial Systems (PCI DSS, Audit Trails, Data Handling)

Financial and payment systems are bound by regulation and industry standards. For interviews, know the **intent** of each control and how it shows up in code and architecture. This page stays general; check the current standard text for exact requirement wording. See also [Secrets Management](secrets-management.md) and [Vulnerabilities & Secure Practices](index.md).

## Common frameworks

| Framework | Scope |
| :-- | :-- |
| **PCI DSS** (v4.0.x current; v3.2.1 retired) | Anyone who stores, processes or transmits cardholder data |
| SOX | Financial reporting controls and audit trails (US public companies) |
| GDPR / DPDP / CCPA | Personal data privacy, consent, erasure |
| PSD2 / SCA | Strong customer authentication for EU payments |
| SOC 2 / ISO 27001 | Organisation-level security controls and audits |

## PCI DSS essentials

PCI DSS has **12 principal requirements** grouped under six goals. v4.0 added a customised-approach option, broader MFA for access to the cardholder data environment (CDE), and targeted risk analysis.

| Goal | Requirement areas (general) |
| :-- | :-- |
| Secure network and systems | Network security controls (firewalls, segmentation); secure configurations (no vendor defaults) |
| Protect account data | Protect stored account data; encrypt transmission over open networks |
| Vulnerability management | Anti-malware; develop and maintain secure software and systems |
| Strong access control | Restrict access by need to know; identify and authenticate users (MFA); restrict physical access |
| Monitor and test | Log and monitor all access; test security regularly (scans, penetration tests) |
| Policy | Information security policy; third-party service provider management |

### Cardholder data handling

| Data element | May be stored? | Rule |
| :-- | :-- | :-- |
| PAN (card number) | Yes, if needed | Must be rendered unreadable (strong encryption, tokenization, truncation, keyed hashing); mask on display |
| Cardholder name, expiry | Yes | Protect together with PAN |
| CVV/CVC, full track data, PIN | **Never after authorisation** | Even encrypted |

**Reduce scope** (the best strategy):

- **Tokenization**: replace the PAN with a token from a PSP or vault; your systems hold only tokens.
- **Hosted fields / redirect pages**: card data never touches your servers.
- **Network segmentation**: isolate the CDE so the rest of the estate is out of scope.

```java
// Mask PAN for display and logs (never log a full PAN)
static String mask(String pan) {
    if (pan == null || pan.length() < 13) return "****";
    return "*".repeat(pan.length() - 4) + pan.substring(pan.length() - 4);
}
```

```java
// Field-level encryption (AES-GCM, fresh random IV per message)
byte[] iv = new byte[12];
new SecureRandom().nextBytes(iv);
Cipher c = Cipher.getInstance("AES/GCM/NoPadding");
c.init(Cipher.ENCRYPT_MODE, dataKey, new GCMParameterSpec(128, iv));
byte[] ct = c.doFinal(plaintext);   // store iv + ct; wrap dataKey with a KMS/Vault master key
```

Key management: separate data keys from key-encrypting keys, restrict custodians, rotate on schedule and on compromise (see [Secrets Management](secrets-management.md)).

### Encryption in transit and at rest

- TLS 1.2+ (prefer 1.3) for all external and CDE-internal traffic; disable weak ciphers.
- Disk/database encryption plus field-level encryption for PAN.
- Never send PAN over email or chat.

## Audit trails

An audit trail answers **who did what, when, from where, to which record, with what result**. It must be complete, tamper-resistant, time-synchronised and retained.

| Attribute | Practice |
| :-- | :-- |
| Content | User ID, action, resource ID, UTC timestamp, source IP, outcome, correlation ID |
| Never log | Full PAN, CVV, passwords, tokens, full PII |
| Integrity | Append-only store, restricted write access, hash chaining or WORM storage, central log server (SIEM) |
| Time | NTP-synchronised clocks |
| Retention | Per policy; PCI DSS asks for 12 months with the most recent 3 months immediately available |
| Review | Automated review and alerting for anomalies |

**Events to log**: authentication (success and failure), privilege changes, access to cardholder data, administrative actions, changes to audit settings, creation or deletion of system objects.

### Implementation options in Spring

```java
// Spring Data JPA auditing: who and when
@Entity
@EntityListeners(AuditingEntityListener.class)
class Payment {
    @CreatedBy String createdBy;
    @CreatedDate Instant createdAt;
    @LastModifiedBy String modifiedBy;
    @LastModifiedDate Instant modifiedAt;
}

@Configuration
@EnableJpaAuditing
class AuditConfig {
    @Bean
    AuditorAware<String> auditor() {
        return () -> Optional.ofNullable(SecurityContextHolder.getContext().getAuthentication())
                             .map(Authentication::getName);
    }
}
```

```java
// Spring Security authentication events
@Component
class AuthAudit {
    private static final Logger log = LoggerFactory.getLogger(AuthAudit.class);

    @EventListener
    void onFailure(AbstractAuthenticationFailureEvent e) {
        log.warn("AUTH_FAILURE user={} reason={}", e.getAuthentication().getName(),
                 e.getException().getClass().getSimpleName());
    }

    @EventListener
    void onSuccess(AuthenticationSuccessEvent e) {
        log.info("AUTH_SUCCESS user={}", e.getAuthentication().getName());
    }
}
```

- **Hibernate Envers** (`@Audited`) keeps entity revision history in `_AUD` tables, answering "what was the value before".
- For tamper evidence, publish audit events to an append-only Kafka topic or immutable store separate from the application DB.
- Business audit (domain events such as `PaymentRefunded`) differs from technical logs; keep both.

## Data handling and privacy

| Principle | Meaning |
| :-- | :-- |
| Data minimisation | Collect and keep only what you need |
| Retention and disposal | Define retention; securely delete or anonymise afterwards |
| Classification | Public / internal / confidential / restricted drives controls |
| Least privilege, need to know | Role-based access; MFA for administrative and CDE access |
| Separation of duties | No single person can change and deploy to production; approvals tracked |
| No production data in test | Use masked or synthetic data |
| Right to erasure | Design deletion across DB, backups, logs and event streams (crypto-shredding helps) |

**Secure SDLC for compliance**: code review, SAST and dependency scanning, change management with approvals, timely patching, regular penetration tests.

## Monetary correctness (often asked together)

- Use `BigDecimal` (or minor units as `long`), never `double`, for money.
- Idempotency keys on payment APIs avoid double charges.
- Immutable ledger entries: corrections are new compensating entries, never updates.

## Common interview questions

- **What reduces PCI scope the most?** Not touching card data: tokenization, hosted payment pages and network segmentation.
- **Can you store CVV?** No, not after authorisation, even encrypted.
- **How do you store a PAN if you must?** Strong encryption with managed keys, or tokenization; mask on display; never log it.
- **What belongs in an audit log?** Who, what, when, where, outcome, correlation ID; excluding sensitive values; append-only and time-synced.
- **Audit log vs application log?** Audit logs are a compliance record with integrity and retention needs; app logs are for debugging.
- **How do you make logs tamper-evident?** Central append-only store, restricted access, hash chains or WORM, alerting on gaps.
- **GDPR erasure with immutable logs or events?** Keep PII out of events (reference by ID) or use per-subject keys (crypto-shredding).

## References

- [PCI Security Standards Council: document library](https://www.pcisecuritystandards.org/document_library/)
- [OWASP Logging Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)
