# Authentication ও Authorization

## 🎯 পার্থক্য বুঝুন

```
Authentication (AuthN):
"আপনি কে?" - পরিচয় যাচাই

Authorization (AuthZ):
"আপনি কি করতে পারবেন?" - অনুমতি যাচাই

┌──────────────────────────────────────────────────────┐
│                Login Flow                            │
├──────────────────────────────────────────────────────┤
│                                                      │
│  User: ahmed@mail.com                               │
│  Pass: ****                                         │
│         │                                           │
│         ▼                                           │
│  ┌────────────────┐                                 │
│  │ Authentication │ → "হ্যাঁ, এটি আহমেদ"             │
│  └────────────────┘                                 │
│         │                                           │
│         ▼                                           │
│  ┌────────────────┐                                 │
│  │ Authorization  │ → "আহমেদ admin নয়,             │
│  └────────────────┘    শুধু user role"               │
│                                                      │
└──────────────────────────────────────────────────────┘
```

## 🔐 Authentication Methods

### ১. Username & Password
```
সবচেয়ে সাধারণ পদ্ধতি।

User → Username + Password → Server
                               │
                               ▼
                          Hash Compare
                               │
                        ┌──────┴──────┐
                        ↓             ↓
                    Success        Failure
```

### ২. Token-based (JWT)
```
┌──────┐                    ┌──────┐
│Client│                    │Server│
└──┬───┘                    └──┬───┘
   │                           │
   │── Login (user/pass) ─────→│
   │                           │
   │←── JWT Token ─────────────│
   │                           │
   │── API + Bearer Token ────→│
   │                           │
   │←── Response ──────────────│
   │                           │
```

### ৩. OAuth 2.0
```
Third-party authentication (Google, Facebook)

User ──→ App ──→ Google ──→ User
  ↑                          │
  │                          │
  └── "Allow access?" ◄──────┘
           │
           ▼
      Token issued
```

### ৪. API Keys
```
Service-to-service authentication

GET /api/data
X-API-Key: sk_live_abc123

Simple কিন্তু কম secure।
```

### ৫. Multi-Factor Authentication (MFA)
```
Something you know (Password)
         +
Something you have (Phone/OTP)
         +
Something you are (Fingerprint)

┌──────────────────────────────────────┐
│           MFA Login Flow             │
├──────────────────────────────────────┤
│ Step 1: Password  ✓                  │
│ Step 2: OTP Code  ✓                  │
│ Step 3: Fingerprint ✓                │
│                                      │
│ Access Granted!                      │
└──────────────────────────────────────┘
```

## 🛡️ Authorization Models

### ১. Role-Based (RBAC)
```
Users → Roles → Permissions

User: আহমেদ
  └── Role: Admin
        └── Permissions: read, write, delete

User: করিম
  └── Role: Editor
        └── Permissions: read, write

User: রহিম
  └── Role: Viewer
        └── Permissions: read

┌──────────────────────────────────────┐
│              RBAC Matrix             │
├──────────┬────────┬────────┬─────────┤
│   Role   │  Read  │ Write  │ Delete  │
├──────────┼────────┼────────┼─────────┤
│  Admin   │   ✓    │   ✓    │   ✓     │
│  Editor  │   ✓    │   ✓    │   ✗     │
│  Viewer  │   ✓    │   ✗    │   ✗     │
└──────────┴────────┴────────┴─────────┘
```

### ২. Attribute-Based (ABAC)
```
Rules based on attributes।

Rule: "Department = Engineering AND 
       Location = Bangladesh AND
       Time = 9AM-6PM"
       
       → Allow access to code repo

More flexible কিন্তু complex।
```

### ৩. Permission-Based
```
Direct permissions to users।

User: আহমেদ
  └── Permissions:
        ├── users.read
        ├── users.write
        ├── orders.read
        └── reports.view
```

## 🔒 Password Security

### Hashing
```python
import bcrypt

# Password hash করা
password = "mypassword123"
salt = bcrypt.gensalt()
hashed = bcrypt.hashpw(password.encode(), salt)

# Verify করা
bcrypt.checkpw(password.encode(), hashed)  # True

# Database-এ store:
# $2b$12$LQv3c1yqBWVHxkd0LHAkCOYz6TtxMQJqhN8/X4.HJPiJxZsYxTpyi
```

### Password Policy
```
Strong Password Requirements:
✓ Minimum 8 characters
✓ At least 1 uppercase
✓ At least 1 lowercase
✓ At least 1 number
✓ At least 1 special character
✗ No common words
✗ No personal info
```

## 💡 Session Management

### Session-based
```
┌──────┐                    ┌──────┐
│Client│                    │Server│
└──┬───┘                    └──┬───┘
   │                           │
   │── Login ─────────────────→│
   │                           │── Create Session
   │←── Session ID (Cookie) ───│   (Store in Redis)
   │                           │
   │── Request + Cookie ──────→│
   │                           │── Validate Session
   │←── Response ──────────────│
```

### Token-based (Stateless)
```
┌──────┐                    ┌──────┐
│Client│                    │Server│
└──┬───┘                    └──┬───┘
   │                           │
   │── Login ─────────────────→│
   │                           │── Create JWT
   │←── JWT Token ─────────────│
   │                           │
   │── Request + JWT ─────────→│
   │                           │── Verify JWT signature
   │←── Response ──────────────│   (No DB lookup)
```

## ✅ Best Practices

```
Authentication:
✓ Use HTTPS everywhere
✓ Implement rate limiting on login
✓ Use strong password hashing (bcrypt/argon2)
✓ Implement MFA
✓ Lock account after failed attempts

Authorization:
✓ Principle of least privilege
✓ Validate on server-side
✓ Log all access attempts
✓ Regular permission audits

Session/Token:
✓ Short expiration times
✓ Secure cookie flags
✓ Token refresh mechanism
✓ Revocation capability
```

## 📚 পরবর্তী টপিক

[OAuth 2.0 ও JWT →](./02-oauth-jwt.md)
