# gRPC পরিচিতি

## 🎯 gRPC কি?

**gRPC (Google Remote Procedure Call)** হলো Google-এর তৈরি একটি high-performance RPC framework।

```
REST: Text-based (JSON)
gRPC: Binary (Protocol Buffers)

Performance: gRPC ≈ 10x faster than REST
```

## 📊 gRPC Features

```
✓ Binary protocol (faster)
✓ HTTP/2 (multiplexing)
✓ Strong typing
✓ Code generation
✓ Bidirectional streaming
✓ Built-in auth support
```

## 🔧 Protocol Buffers

```protobuf
// user.proto
syntax = "proto3";

package user;

// Message definitions
message User {
  int32 id = 1;
  string name = 2;
  string email = 3;
}

message GetUserRequest {
  int32 id = 1;
}

message CreateUserRequest {
  string name = 1;
  string email = 2;
}

// Service definition
service UserService {
  rpc GetUser(GetUserRequest) returns (User);
  rpc CreateUser(CreateUserRequest) returns (User);
  rpc ListUsers(Empty) returns (stream User);
}
```

## 📨 Communication Types

```
1. Unary: Client → Request → Server → Response

2. Server Streaming: Client → Request → Server → Stream of responses

3. Client Streaming: Client → Stream of requests → Server → Response

4. Bidirectional: Client ↔ Stream ↔ Server
```

## ⚡ gRPC vs REST

```
┌────────────────────┬──────────────────┬──────────────────────┐
│      Aspect        │      REST        │        gRPC          │
├────────────────────┼──────────────────┼──────────────────────┤
│ Protocol           │ HTTP/1.1         │ HTTP/2               │
│ Format             │ JSON (text)      │ Protobuf (binary)    │
│ Speed              │ Slower           │ Faster               │
│ Streaming          │ Limited          │ Full support         │
│ Browser support    │ Full             │ Limited              │
│ Learning curve     │ Easy             │ Moderate             │
│ Best for           │ Public APIs      │ Internal services    │
└────────────────────┴──────────────────┴──────────────────────┘
```

## 💡 কখন gRPC?

```
✓ Microservices communication
✓ Low latency required
✓ Streaming data
✓ Polyglot environments

✗ Browser clients
✗ Public APIs
✗ Simple applications
```

## 📚 পরবর্তী টপিক

[API Versioning →](./04-versioning.md)
