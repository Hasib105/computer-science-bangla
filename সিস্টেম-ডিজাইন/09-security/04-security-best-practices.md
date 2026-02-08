# Security Best Practices

## 🛡️ OWASP Top 10

```
1. Injection (SQL, NoSQL, OS)
2. Broken Authentication
3. Sensitive Data Exposure
4. XML External Entities (XXE)
5. Broken Access Control
6. Security Misconfiguration
7. Cross-Site Scripting (XSS)
8. Insecure Deserialization
9. Using Components with Known Vulnerabilities
10. Insufficient Logging & Monitoring
```

## 📊 Common Vulnerabilities

### ১. SQL Injection
```sql
-- Vulnerable:
query = "SELECT * FROM users WHERE id = " + user_input

-- Input: 1; DROP TABLE users;--
-- Result: Database destroyed!

-- Safe (Parameterized):
query = "SELECT * FROM users WHERE id = ?"
cursor.execute(query, (user_input,))
```

### ২. XSS (Cross-Site Scripting)
```html
<!-- Vulnerable -->
<div>Welcome, {{ user_input }}</div>

<!-- Input: <script>steal_cookies()</script> -->

<!-- Safe: Escape HTML -->
<div>Welcome, {{ escape(user_input) }}</div>
```

### ৩. CSRF (Cross-Site Request Forgery)
```
Attacker's page:
<form action="https://bank.com/transfer" method="POST">
  <input name="to" value="attacker">
  <input name="amount" value="10000">
</form>
<script>document.forms[0].submit()</script>

Protection: CSRF tokens
<input type="hidden" name="_csrf" value="random_token">
```

## ✅ Security Checklist

```
Authentication:
□ Strong password policy
□ MFA enabled
□ Account lockout
□ Secure password storage (bcrypt)

Authorization:
□ Principle of least privilege
□ Role-based access control
□ Validate on server-side

Data:
□ HTTPS everywhere
□ Encrypt sensitive data
□ Secure API keys & secrets
□ Input validation

Infrastructure:
□ Firewall configured
□ Regular security patches
□ DDoS protection
□ Regular backups

Monitoring:
□ Security logging
□ Intrusion detection
□ Alerting on anomalies
```

## 💡 Quick Wins

```
1. Use HTTPS only
2. Validate all inputs
3. Escape all outputs
4. Use parameterized queries
5. Keep dependencies updated
6. Enable security headers:
   - Content-Security-Policy
   - X-Frame-Options
   - X-Content-Type-Options
   - Strict-Transport-Security
```

---

🎉 সিকিউরিটি সেকশন সম্পূর্ণ!

[রিয়েল-ওয়ার্ল্ড ডিজাইন →](../10-real-world-designs/README.md)
