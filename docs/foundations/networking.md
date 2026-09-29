# Networking

## OSI vs TCP/IP model
| OSI layer | Example |
|-----------|---------|
| Application | HTTP, DNS, SMTP |
| Presentation | TLS, encoding |
| Session | connection management |
| Transport | **TCP, UDP** |
| Network | **IP**, routing |
| Data link | Ethernet, MAC |
| Physical | cables, signals |

TCP/IP model collapses these into: Application, Transport, Internet, Link.

## TCP vs UDP
- **TCP** — connection-oriented, reliable, ordered, flow/congestion control (3-way handshake). Web, APIs, DB.
- **UDP** — connectionless, best-effort, low latency, no ordering. Streaming, gaming, DNS, VoIP.

## HTTP essentials
- Methods: GET (safe, idempotent), POST, PUT (idempotent), PATCH, DELETE (idempotent).
- Status codes: 2xx success, 3xx redirect, 4xx client error, 5xx server error.
- HTTP/1.1 → HTTP/2 (multiplexing, header compression) → HTTP/3 (QUIC over UDP).
- HTTPS = HTTP over **TLS** (confidentiality + integrity + server auth via certificates).

## DNS
Resolves domain → IP. Chain: resolver → root → TLD → authoritative. Records: A/AAAA, CNAME, MX, TXT, NS.

## Other essentials
- **Load balancer** — distributes traffic across instances (L4 vs L7).
- **Ports** — 80 HTTP, 443 HTTPS, 22 SSH, 3306 MySQL, 5432 Postgres.
- **Latency vs throughput**; **CDN / edge** for static content close to users.
