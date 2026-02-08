# OAuth 2.0 ও JWT

## 🎯 OAuth 2.0 কি?

**OAuth 2.0** হলো একটি authorization framework যা third-party apps-কে limited access দেয়।

```
Use Case:
User → "Login with Google" → App gets user data

User ডিরেক্টলি password দেয় না app-কে।
Google token দেয়, app সেই token দিয়ে data access করে।
```

## 📊 OAuth 2.0 Flow

### Authorization Code Flow
```
┌──────────────────────────────────────────────────────────────────┐
│                     OAuth 2.0 Flow                               │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  User        App           Auth Server        Resource Server   │
│   │           │                 │                    │          │
│   │──────────→│                 │                    │          │
│   │  1. Login │                 │                    │          │
│   │           │────────────────→│                    │          │
│   │           │ 2. Redirect to  │                    │          │
│   │           │    Auth Server  │                    │          │
│   │←──────────│←────────────────│                    │          │
│   │           │                 │                    │          │
│   │ 3. User logs in & grants    │                    │          │
│   │    permission               │                    │          │
│   │────────────────────────────→│                    │          │
│   │                             │                    │          │
│   │ 4. Auth code to App         │                    │          │
│   │←─────────── redirect ───────│                    │          │
│   │           │                 │                    │          │
│   │           │ 5. Exchange     │                    │          │
│   │           │    code for     │                    │          │
│   │           │    token        │                    │          │
│   │           │────────────────→│                    │          │
│   │           │←───────────────│                    │          │
│   │           │  Access Token   │                    │          │
│   │           │                 │                    │          │
│   │           │ 6. Access API   │                    │          │
│   │           │ with token      │                    │          │
│   │           │────────────────────────────────────→│          │
│   │           │←────────────────────────────────────│          │
│   │           │  Protected Data │                    │          │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

## 🔐 JWT (JSON Web Token)

```
JWT = Header.Payload.Signature

Example:
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJ1c2VyX2lkIjoiMTIzIiwibmFtZSI6IkFobWVkIiwiZXhwIjoxNjQwMDAwMDAwfQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

### JWT Structure
```
Header:
{
  "alg": "HS256",
  "typ": "JWT"
}

Payload:
{
  "user_id": "123",
  "name": "Ahmed",
  "role": "admin",
  "exp": 1640000000  // Expiration
}

Signature:
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret
)
```

### JWT Verification
```
┌────────────────────────────────────────────────────┐
│               JWT Verification                     │
├────────────────────────────────────────────────────┤
│                                                    │
│  Client sends: Authorization: Bearer <jwt>        │
│                                                    │
│  Server:                                           │
│  1. Split JWT into parts                          │
│  2. Decode header & payload                       │
│  3. Verify signature with secret                  │
│  4. Check expiration                              │
│  5. Extract user info from payload                │
│                                                    │
│  No database lookup needed!                       │
│                                                    │
└────────────────────────────────────────────────────┘
```

## 🔄 Access Token vs Refresh Token

```
Access Token:
- Short-lived (15 min - 1 hour)
- Used for API calls
- Stored in memory

Refresh Token:
- Long-lived (days - weeks)
- Used to get new access token
- Stored securely (httpOnly cookie)

Flow:
1. Login → Access + Refresh tokens
2. API call with Access token
3. Access token expires
4. Use Refresh token to get new Access token
5. Continue API calls
```

## 💡 Best Practices

```
✓ Short expiry for access tokens
✓ Secure storage for refresh tokens
✓ Use HTTPS always
✓ Validate token on every request
✓ Implement token revocation
```

## 📚 পরবর্তী টপিক

[HTTPS ও Encryption →](./03-https-encryption.md)
