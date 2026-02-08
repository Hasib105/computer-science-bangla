# API Gateway

## 🎯 API Gateway কি?

**API Gateway** হলো মাইক্রোসার্ভিস আর্কিটেকচারে একটি single entry point যা সব client request handle করে।

```
Without API Gateway:
Client → User Service
Client → Order Service
Client → Product Service
(Client কে সব service জানতে হয়)

With API Gateway:
Client → API Gateway → User Service
                    → Order Service
                    → Product Service
(Client শুধু Gateway জানে)
```

## 🏗️ API Gateway Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│     ┌──────────┐    ┌──────────┐    ┌──────────┐              │
│     │  Mobile  │    │   Web    │    │ Partner  │              │
│     │   App    │    │   App    │    │   API    │              │
│     └────┬─────┘    └────┬─────┘    └────┬─────┘              │
│          │               │               │                     │
│          └───────────────┼───────────────┘                     │
│                          ↓                                     │
│          ┌───────────────────────────────┐                    │
│          │         API Gateway           │                    │
│          │  ┌─────────────────────────┐ │                    │
│          │  │ • Authentication        │ │                    │
│          │  │ • Rate Limiting         │ │                    │
│          │  │ • Request Routing       │ │                    │
│          │  │ • Load Balancing        │ │                    │
│          │  │ • Response Caching      │ │                    │
│          │  │ • Protocol Translation  │ │                    │
│          │  └─────────────────────────┘ │                    │
│          └───────────────┬───────────────┘                    │
│                          │                                     │
│         ┌────────────────┼────────────────┐                   │
│         ↓                ↓                ↓                   │
│    ┌─────────┐     ┌─────────┐     ┌─────────┐               │
│    │  User   │     │  Order  │     │ Product │               │
│    │ Service │     │ Service │     │ Service │               │
│    └─────────┘     └─────────┘     └─────────┘               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 📊 API Gateway Responsibilities

### ১. Request Routing
```
/users/* → User Service
/orders/* → Order Service
/products/* → Product Service

GET /api/users/123
         │
         ↓
API Gateway: Route to User Service
         │
         ↓
User Service: Return user data
```

### ২. Authentication/Authorization
```
Request → API Gateway → Verify JWT → Forward to Service

┌─────────┐    ┌─────────────┐    ┌──────────┐
│ Client  │───→│ API Gateway │───→│ Service  │
└─────────┘    │             │    └──────────┘
               │ 1. Check JWT │
               │ 2. Validate  │
               │ 3. Forward   │
               └─────────────┘
```

### ৩. Rate Limiting
```
User: 100 requests/minute allowed

Request 1-100: ✓ Allowed
Request 101+: ✗ 429 Too Many Requests

┌────────────────────────────────┐
│ Rate Limit: 100 req/min        │
│ User: user_123                 │
│ Current: 98                    │
│ Status: OK                     │
└────────────────────────────────┘
```

### ৪. Response Aggregation
```
Single request → Multiple services → Combined response

GET /api/dashboard

API Gateway:
├── User Service → User info
├── Order Service → Recent orders
└── Product Service → Recommendations

Response: Combined JSON
```

## 🔧 Popular API Gateways

```
Open Source:
- Kong
- Nginx
- Traefik
- Express Gateway

Cloud:
- AWS API Gateway
- Azure API Management
- Google Cloud Endpoints
```

## 💡 Best Practices

```
✓ Keep gateway lightweight
✓ Implement circuit breakers
✓ Cache responses when possible
✓ Use health checks
✓ Centralized logging
✗ Don't put business logic in gateway
```

## 📚 পরবর্তী টপিক

[সার্ভিস ডিসকভারি →](./05-service-discovery.md)
