# HTTPS ও Encryption

## 🎯 HTTPS কি?

**HTTPS = HTTP + TLS/SSL**

```
HTTP (Insecure):
Client ─── "password123" ───→ Server
           ↑
    Attacker can read!

HTTPS (Secure):
Client ─── 🔒 Encrypted 🔒 ───→ Server
           ↑
    Attacker sees: "a8f2k3..."
```

## 📊 TLS Handshake

```
┌────────────────────────────────────────────────────────────────┐
│                     TLS Handshake                              │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  Client                                      Server            │
│    │                                           │               │
│    │── 1. Client Hello ───────────────────────→│               │
│    │   (Supported cipher suites)               │               │
│    │                                           │               │
│    │←── 2. Server Hello ───────────────────────│               │
│    │   (Chosen cipher, Certificate)            │               │
│    │                                           │               │
│    │── 3. Verify Certificate ─────────────────→│               │
│    │   (Check with CA)                         │               │
│    │                                           │               │
│    │── 4. Key Exchange ───────────────────────→│               │
│    │   (Pre-master secret)                     │               │
│    │                                           │               │
│    │←── 5. Finished ───────────────────────────│               │
│    │   (Both have session keys)                │               │
│    │                                           │               │
│    │══ 6. Encrypted Communication ════════════│               │
│    │                                           │               │
└────────────────────────────────────────────────────────────────┘
```

## 🔐 Encryption Types

### Symmetric Encryption
```
Same key for encrypt & decrypt।

Key: "secret123"

Plaintext ─[Key]→ Ciphertext ─[Key]→ Plaintext

Algorithms: AES, DES, 3DES

Problem: Key exchange difficult
```

### Asymmetric Encryption
```
Public key encrypt, Private key decrypt।

┌──────────────────────────────────────────┐
│                                          │
│  Public Key (Anyone can have)            │
│  ┌─────────────────┐                     │
│  │ 🔓 Encrypt only │                     │
│  └─────────────────┘                     │
│                                          │
│  Private Key (Secret)                    │
│  ┌─────────────────┐                     │
│  │ 🔒 Decrypt only │                     │
│  └─────────────────┘                     │
│                                          │
└──────────────────────────────────────────┘

Algorithms: RSA, ECC

Use: Key exchange, Digital signatures
```

## 🔧 Encryption at Rest & Transit

```
At Rest (Stored data):
Database → Encrypted storage
Files → Encrypted disk (AES-256)

In Transit (Moving data):
Client ← HTTPS/TLS → Server
Service A ← mTLS → Service B
```

## 💡 Best Practices

```
✓ Use HTTPS everywhere
✓ TLS 1.2+ only
✓ Strong cipher suites
✓ HSTS header
✓ Certificate pinning (mobile)
✓ Regular certificate rotation
```

## 📚 পরবর্তী টপিক

[Security Best Practices →](./04-security-best-practices.md)
