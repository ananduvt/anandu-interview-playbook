# API Styles & Protocols

A quick comparison of the common ways services talk to each other.

## REST (Representational State Transfer)
- Architectural style over HTTP: **stateless**, client-server, cacheable, uniform interface, layered.
- Resources identified by URLs; manipulated via representations (JSON/XML); optional HATEOAS links.
- **Use for**: web/mobile APIs, cloud services — the default for CRUD-style resource APIs.

## Webhooks
- **Reverse API**: the server pushes an HTTP POST to a client-registered URL when an event occurs (no polling).
- Event-driven, real-time notifications.
- **Use for**: payment notifications, VCS/CI events, social updates.

## GraphQL
- Query language + runtime; clients request **exactly** the fields they need from a **single endpoint**.
- Strongly-typed schema; avoids over-/under-fetching.
- **Use for**: complex/nested data needs, mobile/front-end-driven data fetching.

## SOAP (Simple Object Access Protocol)
- XML-based messaging protocol over HTTP/SMTP; standardized, with WS-* specs (security, transactions).
- Platform/language independent, contract-first (WSDL).
- **Use for**: enterprise/legacy integrations, strict contracts, security-sensitive transactions.

## WebSocket
- **Full-duplex**, persistent connection over a single TCP connection; low latency.
- Event-driven push without polling.
- **Use for**: chat, live dashboards, online gaming, real-time collaboration.

## gRPC
- High-performance RPC over **HTTP/2** using **Protocol Buffers** (IDL + binary message format).
- Strongly typed, language-independent, supports bidirectional **streaming**.
- **Use for**: internal service-to-service calls where performance and strict contracts matter.

## Quick chooser
| Need | Style |
|------|-------|
| Standard resource CRUD API | REST |
| Client picks exact fields, nested data | GraphQL |
| Push notifications to third parties | Webhooks |
| High-perf internal RPC + streaming | gRPC |
| Real-time bidirectional | WebSocket |
| Enterprise contract + WS-* security | SOAP |
