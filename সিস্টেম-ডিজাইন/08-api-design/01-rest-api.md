# REST API ডিজাইন (REST API Design)

## 🎯 REST কি?

**REST (Representational State Transfer)** হলো ওয়েব সার্ভিস ডিজাইনের একটি আর্কিটেকচারাল স্টাইল।

## 📊 REST প্রিন্সিপল

### ১. Uniform Interface
```
Resource-based URLs:

✓ /users                   (সব ইউজার)
✓ /users/123               (একজন ইউজার)
✓ /users/123/orders        (একজন ইউজারের অর্ডার)

✗ /getUser?id=123          (Verb URL - ভুল)
✗ /createNewOrder          (Verb URL - ভুল)
```

### ২. HTTP Methods
```
┌────────┬──────────────────┬─────────────────────────────────┐
│ Method │     Action       │           Example               │
├────────┼──────────────────┼─────────────────────────────────┤
│ GET    │ Read (retrieve)  │ GET /users/123                  │
│ POST   │ Create           │ POST /users                     │
│ PUT    │ Update (replace) │ PUT /users/123                  │
│ PATCH  │ Update (partial) │ PATCH /users/123                │
│ DELETE │ Delete           │ DELETE /users/123               │
└────────┴──────────────────┴─────────────────────────────────┘
```

### ৩. Stateless
```
প্রতিটি রিকোয়েস্ট স্বয়ংসম্পূর্ণ।

✓ Request:
GET /users/123
Authorization: Bearer token123

✗ Server-side session এর উপর নির্ভর করা নয়
```

## 🔧 URL ডিজাইন

### রিসোর্স নেমিং
```
✓ Nouns (বিশেষ্য) ব্যবহার করুন:
/users
/products
/orders

✗ Verbs (ক্রিয়া) এড়িয়ে চলুন:
/getUsers
/createProduct
/deleteOrder
```

### নেস্টেড রিসোর্স
```
User-এর অর্ডার:
GET /users/123/orders

Order-এর আইটেম:
GET /orders/456/items

কিন্তু ৩+ লেভেল নেস্টিং এড়িয়ে চলুন।
```

### কুয়েরি প্যারামিটার
```
Filtering:
GET /users?status=active
GET /products?category=electronics&price_min=100

Pagination:
GET /users?page=2&limit=20

Sorting:
GET /products?sort=price&order=desc

Searching:
GET /products?search=iphone
```

## 📨 HTTP Status Codes

```
2xx Success:
├── 200 OK              → সফল রিকোয়েস্ট
├── 201 Created         → রিসোর্স তৈরি হয়েছে
├── 204 No Content      → সফল, কোনো কন্টেন্ট নেই

3xx Redirection:
├── 301 Moved Permanently
├── 304 Not Modified

4xx Client Error:
├── 400 Bad Request     → ভুল রিকোয়েস্ট ফরম্যাট
├── 401 Unauthorized    → অথেনটিকেশন দরকার
├── 403 Forbidden       → অনুমতি নেই
├── 404 Not Found       → রিসোর্স নেই
├── 422 Unprocessable   → ভ্যালিডেশন এরর
├── 429 Too Many Requests → Rate limit exceeded

5xx Server Error:
├── 500 Internal Error  → সার্ভার এরর
├── 502 Bad Gateway
├── 503 Service Unavailable
```

## 📦 Request/Response Format

### Request
```http
POST /users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer token123

{
  "name": "আহমেদ",
  "email": "ahmed@mail.com",
  "age": 25
}
```

### Response
```http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "id": 123,
  "name": "আহমেদ",
  "email": "ahmed@mail.com",
  "age": 25,
  "created_at": "2026-01-26T10:30:00Z"
}
```

### Error Response
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ]
  }
}
```

## 📖 Pagination

### Offset-based
```
GET /users?page=2&limit=20

Response:
{
  "data": [...],
  "pagination": {
    "page": 2,
    "limit": 20,
    "total": 100,
    "total_pages": 5
  }
}
```

### Cursor-based (Better for large datasets)
```
GET /users?cursor=abc123&limit=20

Response:
{
  "data": [...],
  "pagination": {
    "next_cursor": "def456",
    "has_more": true
  }
}
```

## 🔐 Authentication

### Bearer Token
```http
GET /users HTTP/1.1
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

### API Key
```http
GET /users HTTP/1.1
X-API-Key: your-api-key
```

## 💡 Best Practices

```
১. Versioning ব্যবহার করুন
   /api/v1/users
   /api/v2/users

২. HTTPS বাধ্যতামূলক করুন

৩. সঠিক HTTP status codes ব্যবহার করুন

৪. Pagination implement করুন

৫. Rate limiting যোগ করুন

৬. ভালো ডকুমেন্টেশন রাখুন (Swagger/OpenAPI)

৭. Consistent error format ব্যবহার করুন
```

## 📚 পরবর্তী টপিক

[GraphQL পরিচিতি →](./02-graphql.md)
