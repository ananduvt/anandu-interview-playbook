# Web Services

See also [API Styles & Protocols](api-styles.md) for REST/GraphQL/gRPC/SOAP/WebSocket comparison.

## SOAP vs REST
| | SOAP | REST |
|--|------|------|
| Type | Protocol | Architectural style |
| Format | XML only | JSON, XML, … |
| Contract | WSDL (strict) | OpenAPI (looser) |
| Transport | HTTP, SMTP | HTTP |
| State | can be stateful | stateless |
| Standards | WS-* (security, tx) | HTTP semantics |
| Best for | enterprise, strict contracts, security | web/mobile APIs, simplicity |

## REST maturity (Richardson)
Level 0 (RPC over HTTP) → 1 (resources) → 2 (HTTP verbs + status codes, most APIs) → 3 (HATEOAS).

## Authentication vs Authorization
- **Authentication** — *who are you?* (login, credentials, tokens).
- **Authorization** — *what can you do?* (roles/permissions/scopes).
- Order: authenticate first, then authorize. See [Security](security.md).

## API design essentials
- Nouns + HTTP verbs; URI versioning (`/v1/`); consistent error envelope; meaningful status codes.
- Pagination (offset vs cursor); idempotency for retries; mask sensitive data; document with OpenAPI/Swagger.
