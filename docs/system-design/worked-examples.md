# Worked Examples

Rehearse these end-to-end using the [framework](fundamentals.md): clarify → estimate → API → components →
data → scale → resilience → trade-offs.

## URL shortener (TinyURL)
- **Requirements**: shorten a URL, redirect; read-heavy; low latency; analytics optional.
- **API**: `POST /shorten {url}` → shortCode; `GET /{code}` → 301 redirect.
- **Encoding**: base62 of an auto-increment ID, or a hash (collision-check). 7 chars ≈ 3.5T URLs.
- **Storage**: key-value (code → url); heavy read → cache hot codes (Redis).
- **Scale**: stateless services behind LB; DB sharded by code; CDN/cache for redirects.
- **Trade-offs**: counter (needs coordination) vs hash (collisions); custom aliases.

## Rate limiter
- **Requirements**: allow N requests / window per user/IP; low latency; distributed.
- **Algorithm**: **token bucket** (smooth bursts) or sliding-window counter.
- **Storage**: Redis counters with TTL (atomic INCR + EXPIRE); key = user+window.
- **Placement**: API gateway / middleware. Return `429` + `Retry-After`.

## News feed
- **Requirements**: users follow others; show a merged, ranked timeline.
- **Fan-out on write** (push to followers' feeds — fast reads, expensive for celebrities) vs
  **fan-out on read** (merge at query time — cheap writes, slower reads). Hybrid for hot accounts.
- **Storage**: feed cache (Redis lists) + posts store; rank by recency/relevance.

## Chat / messaging
- **WebSocket** persistent connections; message queue for delivery; store per conversation.
- Delivery/read receipts; online presence; fan-out to devices; ordering per conversation.

## Tips
State assumptions and numbers out loud, justify each component, and call out trade-offs — the reasoning
matters more than a "correct" answer.
