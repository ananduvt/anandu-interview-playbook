# Secrets Management

A **secret** is any credential that grants access: DB passwords, API keys, TLS private keys, signing keys, OAuth client secrets. Secrets management means storing them encrypted, delivering them only to authorised workloads, rotating them, and auditing every access. Related: [Vulnerabilities & Secure Practices](index.md), [Spring Cloud](../microservices-distributed/spring-cloud.md), [Compliance for Financial Systems](compliance-pci.md).

## Where secrets must NOT live

| Anti-pattern | Why it fails |
| :-- | :-- |
| Hard-coded in source / committed `application.yml` | Lives forever in Git history, visible to everyone with repo access |
| Baked into Docker images | Anyone who can pull the image can read layers |
| Plain environment variables | Leak via `/proc`, crash dumps, `docker inspect`, `/actuator/env` |
| Plain Kubernetes `Secret` | Only base64-encoded; encrypt etcd at rest and restrict RBAC |
| Shared long-lived admin password | No attribution, rotation breaks everyone |

## Options at a glance

| Option | Good for | Caveat |
| :-- | :-- | :-- |
| Env vars / mounted files from the platform | Simple deployments | Static; rotation needs restart |
| Spring Cloud Config `{cipher}` values | Encrypting values in a Git-backed config repo | Key management is on you; no dynamic secrets |
| HashiCorp Vault | Dynamic secrets, PKI, transit encryption, audit | Another critical service to run (HA, unseal) |
| Cloud managers (AWS Secrets Manager, Azure Key Vault, GCP Secret Manager) | Cloud-native apps, built-in rotation and IAM | Cloud lock-in |
| Kubernetes External Secrets / CSI driver | Sync manager secrets into pods | Synced copy is still a K8s Secret (unless mounted via CSI) |

## Spring Cloud Config `{cipher}` encryption

Config Server can encrypt values in the config repo so Git never holds plaintext.

```yaml
# Config Server bootstrap: symmetric key (prefer an RSA keystore in production)
encrypt:
  key: ${CONFIG_ENCRYPT_KEY}      # injected from the platform, never committed
```

```bash
# Encrypt through the Config Server endpoint (/decrypt works the same way)
curl -X POST http://config-server:8888/encrypt -d 'my-db-password'
# -> 682bc583f4641835fa2db009355293665...  (placeholder output)
```

```yaml
# In the Git config repo (application-prod.yml)
spring:
  datasource:
    password: '{cipher}682bc583f4641835fa2db009355293665...'
```

- The server decrypts before serving by default; the client receives plaintext over HTTP, so **always use TLS** and secure `/encrypt` and `/decrypt` endpoints.
- Set `spring.cloud.config.server.encrypt.enabled=false` to ship ciphertext and let clients decrypt (clients then need the key).
- Asymmetric: use a keystore (`encrypt.keyStore.location`, `.password`, `.alias`) so the public key encrypts and only the server holds the private key.
- Key rotation means re-encrypting every value. Weakness: it is static encryption, not dynamic secrets.

## Spring Cloud Vault

Vault is an identity-based secrets engine: authenticate, get a time-limited token, read secrets by policy.

| Concept | Meaning |
| :-- | :-- |
| Secrets engines | KV (static), Database (dynamic creds), PKI (certs), Transit (encryption as a service) |
| Auth methods | Token, AppRole, Kubernetes, AWS/IAM, JWT/OIDC, LDAP |
| Policies | Path-based ACLs (`path "secret/data/orders/*" { capabilities = ["read"] }`) |
| Leases | Every dynamic secret has a TTL; renew or it is revoked |
| Seal / unseal | Data encrypted at rest; unseal via Shamir key shares or auto-unseal with a cloud KMS |
| Audit devices | Log every request and response (values hashed) |

```bash
vault kv put secret/orders db.password='<placeholder>'
vault kv get secret/orders
vault read database/creds/orders-role   # dynamic user + password, auto-expires
```

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-vault-config</artifactId>
</dependency>
```

```yaml
spring:
  config:
    import: vault://
  cloud:
    vault:
      uri: https://vault.internal:8200
      authentication: KUBERNETES          # or APPROLE
      kubernetes:
        role: orders-service
      kv:
        enabled: true
        backend: secret
        default-context: orders           # reads secret/orders
```

```java
@Value("${db.password}") String password;   // resolved from Vault like any property
```

Dynamic database credentials use the `spring.cloud.vault.database.*` properties (role and backend) together with the `spring-cloud-vault-config-databases` module. Config Server can also use Vault as a backend (`vault` profile), so clients keep talking only to Config Server.

### Sample use cases

1. **Dynamic DB credentials**: each pod gets a unique DB user with a short TTL; a leak self-expires and access is attributable.
2. **Encryption as a service (Transit)**: app sends plaintext, Vault returns ciphertext, so the key never reaches the app (good for PAN/PII fields).
3. **Internal PKI**: short-lived mTLS certificates for service-to-service traffic.
4. **Cloud credentials**: temporary cloud credentials issued per job.

## Cloud secret managers (AWS example)

```yaml
# Spring Cloud AWS 3.x
spring:
  config:
    import: aws-secretsmanager:/prod/orders/db
```

An IAM role on the pod or instance controls access, so there is no bootstrap secret. Managed rotation uses a function that updates the secret and the database together.

## Rotation

| Strategy | How |
| :-- | :-- |
| Dynamic secrets | Vault issues short-lived credentials; nothing to rotate |
| Dual credentials (blue/green) | Two valid users; rotate the standby, switch, then rotate the old one |
| Scheduled rotation | Secrets Manager / Vault rotates on a timer |
| Emergency rotation | Revoke, reissue, redeploy; practise it |

Apps must pick up new values without downtime:

- Re-read on connection failure, or use a pool that supports credential refresh.
- `@RefreshScope` + `/actuator/refresh` works for Config Server properties, but does not safely rebuild a `DataSource` by default, so test it.
- Spring Cloud Vault renews leases and can re-request dynamic credentials on expiry.

## The "secret zero" problem

Something must authenticate the app to Vault in the first place. Solve it with **platform identity** (Kubernetes service account JWT, cloud IAM role, managed identity), or AppRole with a response-wrapped one-time `secret_id` delivered by CI. Avoid a static token in an env var.

## Practical rules for Java apps

- Never log secrets. Actuator `/env` and `/configprops` sanitise values by default; tighten with `management.endpoint.env.show-values`.
- Prefer `char[]` for passwords you handle directly and clear it afterwards; a `String` stays in the heap until GC.
- Use pre-commit and CI secret scanners (gitleaks, trufflehog). If a secret leaks, **rotate it**; rewriting Git history is not enough.
- Least privilege: one secret path/role per service, separate dev/stage/prod.
- Encrypt in transit (TLS) and at rest; audit access.

## Common interview questions

- **Why not environment variables?** They are visible to anything that can read the process environment or dumps, never rotate by themselves, and often end up in logs. Better than hard-coding, not best.
- **`{cipher}` vs Vault?** `{cipher}` encrypts static values stored in Git; Vault adds access policies, auditing, leases, dynamic secrets and rotation.
- **What are dynamic secrets?** Credentials created on demand with a TTL, unique per consumer, revoked on expiry.
- **A secret was committed to Git. What now?** Revoke and rotate immediately, then clean history, then audit usage. Add scanning.
- **How does an app authenticate to Vault?** Platform identity (Kubernetes auth, IAM) or AppRole; this answers "secret zero".
- **Are K8s Secrets safe?** Base64 is not encryption; enable etcd encryption at rest, tight RBAC, or use External Secrets / CSI with a real manager.

## References

- [Spring Cloud Vault](https://docs.spring.io/spring-cloud-vault/reference/)
- [Spring Cloud Config: Encryption and Decryption](https://docs.spring.io/spring-cloud-config/reference/server/encryption-and-decryption.html)
- [Vault documentation](https://developer.hashicorp.com/vault/docs)
