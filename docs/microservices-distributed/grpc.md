# gRPC vs REST in Depth

gRPC is a contract-first RPC framework that runs over **HTTP/2** and serialises messages with **Protocol Buffers** (protobuf). It is the usual choice for internal, high-throughput service-to-service calls. For the one-line overview and the style chooser see [API Styles](../spring-web/api-styles.md#grpc).

## gRPC vs REST

| Aspect | REST (JSON/HTTP) | gRPC |
|--------|------------------|------|
| Contract | Optional (OpenAPI) | Mandatory `.proto` file |
| Payload | Text JSON | Binary protobuf (smaller, faster to parse) |
| Transport | HTTP/1.1 or HTTP/2 | HTTP/2 only (multiplexing, header compression) |
| Streaming | Awkward (SSE, WebSocket) | Built in (4 call types) |
| Browser support | Native | Needs gRPC-Web proxy |
| Debugging | curl, browser | grpcurl, special tooling |
| Caching / CDN | Easy (GET, ETag) | Not HTTP-cache friendly |
| Codegen | Optional | Stubs for many languages |

## Protobuf Contract

```protobuf
syntax = "proto3";

package orders.v1;
option java_multiple_files = true;
option java_package = "com.example.orders.v1";

service OrderService {
  rpc GetOrder (GetOrderRequest) returns (Order);                       // unary
  rpc WatchOrders (WatchRequest) returns (stream Order);                // server streaming
  rpc UploadItems (stream Item) returns (UploadSummary);                // client streaming
  rpc Chat (stream ChatMsg) returns (stream ChatMsg);                   // bidirectional
}

message GetOrderRequest { string order_id = 1; }
message Order {
  string order_id = 1;
  string status   = 2;
  repeated Item items = 3;
}
message Item { string sku = 1; int32 qty = 2; }
```

**Evolution rules:** field numbers are the wire identity. Never reuse or renumber a field; add new fields with new numbers; mark removed ones `reserved`. Unknown fields are ignored, so adding fields is backward compatible. Version the package (`orders.v1`) for breaking changes.

## Four Call Types

| Type | Shape | Example use |
|------|-------|-------------|
| Unary | 1 request, 1 response | Get order |
| Server streaming | 1 request, N responses | Price feed, progress updates |
| Client streaming | N requests, 1 response | Bulk upload, telemetry batch |
| Bidirectional | N requests, N responses (independent) | Chat, live sync |

## Java Server and Client

```java
// Server (grpc-java; Spring Boot starters such as Spring gRPC wrap this)
public class OrderServiceImpl extends OrderServiceGrpc.OrderServiceImplBase {
    @Override
    public void getOrder(GetOrderRequest req, StreamObserver<Order> out) {
        Order order = Order.newBuilder().setOrderId(req.getOrderId()).setStatus("PAID").build();
        out.onNext(order);
        out.onCompleted();
    }

    @Override
    public void watchOrders(WatchRequest req, StreamObserver<Order> out) {
        for (Order o : repo.recent()) out.onNext(o);   // push many responses
        out.onCompleted();
    }
}

// Client with a deadline
ManagedChannel channel = ManagedChannelBuilder.forAddress("orders", 9090).usePlaintext().build();
OrderServiceGrpc.OrderServiceBlockingStub stub = OrderServiceGrpc.newBlockingStub(channel);
try {
    Order o = stub.withDeadlineAfter(500, TimeUnit.MILLISECONDS)
                  .getOrder(GetOrderRequest.newBuilder().setOrderId("42").build());
} catch (StatusRuntimeException e) {
    if (e.getStatus().getCode() == Status.Code.DEADLINE_EXCEEDED) { /* fallback */ }
}
```

Create the `ManagedChannel` once and reuse it; it multiplexes many calls over one connection. `usePlaintext()` is for local use only; use TLS in production.

## Deadlines, Cancellation and Errors

- **Deadline** is an absolute point in time for the whole call. It propagates downstream when the server passes the incoming `Context` on, so a chain of calls shares one budget. Prefer deadlines to per-hop timeouts.
- **Cancellation:** when the client cancels or the deadline expires, the server's `Context` is cancelled. Long-running handlers should check `Context.current().isCancelled()`.
- **Status codes** replace HTTP codes: `OK`, `INVALID_ARGUMENT`, `NOT_FOUND`, `DEADLINE_EXCEEDED`, `UNAVAILABLE` (retryable), `UNAUTHENTICATED`, `PERMISSION_DENIED`, `INTERNAL`.
- **Interceptors** (client and server) handle auth metadata, logging, tracing and metrics.
- **Load balancing:** long-lived HTTP/2 connections defeat L4 balancers; use client-side balancing, a service mesh or an L7 proxy such as Envoy.

## When to Choose Which

- gRPC: internal east-west calls, polyglot services, streaming, tight latency, strict contracts.
- REST: public APIs, browser clients, easy caching and debugging, third-party consumers.
- Often both: REST at the edge (API gateway), gRPC inside.

## Common Interview Questions

- **Why is gRPC faster?** Binary protobuf, HTTP/2 multiplexing and header compression, generated code.
- **How do you evolve a proto?** Add fields with new numbers, never reuse numbers, use `reserved`, version the package.
- **Why do deadlines matter?** Without them a slow dependency holds threads indefinitely; propagated deadlines stop wasted work downstream.
- **Why is gRPC hard from browsers?** Browsers cannot control HTTP/2 framing and trailers; gRPC-Web with a proxy is needed.
- **Retries?** Only for idempotent calls and `UNAVAILABLE`; see [Idempotency Keys](data-consistency-patterns.md#idempotency-keys-for-write-apis).
