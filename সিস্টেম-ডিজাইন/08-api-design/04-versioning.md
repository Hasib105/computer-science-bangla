# API Versioning

## 🎯 কেন Versioning দরকার?

```
Breaking changes:
- Field removed
- Field renamed
- Response structure changed
- Endpoint deprecated

Old clients → Need old API
New clients → Need new API
```

## 📊 Versioning Strategies

### ১. URL Path Versioning
```
https://api.example.com/v1/users
https://api.example.com/v2/users

Pros:
✓ Simple, clear
✓ Easy caching
✓ Browser friendly

Cons:
✗ Not RESTful (URL = resource)
✗ Code duplication
```

### ২. Query Parameter
```
https://api.example.com/users?version=1
https://api.example.com/users?version=2

Pros:
✓ Optional parameter
✓ Easy to implement

Cons:
✗ Easy to forget
✗ Caching issues
```

### ৩. Header Versioning
```
GET /users
Accept: application/vnd.api.v1+json

GET /users
Accept: application/vnd.api.v2+json

Pros:
✓ Clean URLs
✓ RESTful

Cons:
✗ Not visible in URL
✗ Harder to test
```

### ৪. Content Negotiation
```
GET /users
Accept: application/json; version=1

Pros:
✓ Standard HTTP mechanism

Cons:
✗ Complex
✗ Tooling issues
```

## 💡 Best Practices

```
✓ URL path versioning (most common)
✓ Support at least 2 versions
✓ Deprecation warnings
✓ Clear documentation
✓ Sunset dates

Example deprecation:
HTTP/1.1 200 OK
Deprecation: true
Sunset: Sat, 31 Dec 2024 23:59:59 GMT
Link: <https://api.example.com/v2/users>; rel="successor-version"
```

## 📚 পরবর্তী টপিক

[Rate Limiting →](./05-rate-limiting.md)
