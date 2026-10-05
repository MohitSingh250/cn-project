# Certificate Verification

> **Machine:** Mac 2 — Mohit (`10.7.10.115`)

---

## Test 1: Verify Certificate Contents

```bash
openssl x509 -in server.crt -text -noout
```

**Key fields to check:**

| Field | Expected Value |
|-------|---------------|
| Subject | `CN = app.team1.test` |
| Issuer | `CN = Team1 Local CA` |
| Validity | Not expired |
| SAN | `DNS:app.team1.test` |
| Signature Algorithm | `sha256WithRSAEncryption` |

---

## Test 2: Verify Certificate Chain

```bash
openssl verify -CAfile ca.crt server.crt
```

**Expected output:**
```
server.crt: OK
```

This proves:
- `server.crt` was signed by `ca.crt`
- The certificate chain is valid
- If the CA is trusted, all certificates it signed will be trusted

---

## Test 3: Verify via HTTPS (curl)

```bash
curl -v https://app.team1.test/api/status --cacert ca.crt
```

**Expected TLS output (relevant lines):**

```
* TLSv1.3 (OUT), TLS handshake, Client hello
* TLSv1.3 (IN), TLS handshake, Server hello
* TLSv1.3 (IN), TLS handshake, Certificate
* TLSv1.3 (IN), TLS handshake, Finished
* SSL certificate verify ok.
*  subject: C=IN; ST=State; L=City; O=Team1; CN=app.team1.test
*  issuer: C=IN; ST=State; L=City; O=Team1; CN=Team1 Local CA
*  SSL certificate verify ok.
```

**Key observations:**
- TLS version: `TLSv1.3`
- Certificate subject: `CN=app.team1.test`
- Certificate issuer: `CN=Team1 Local CA`
- Verification: `SSL certificate verify ok.`

---

## Test 4: Check SAN Extension

```bash
openssl x509 -in server.crt -text -noout | grep -A 2 "Subject Alternative Name"
```

**Expected:**
```
X509v3 Subject Alternative Name:
    DNS:app.team1.test
```

---

## What Each Test Proves

| Test | Proves |
|------|--------|
| Certificate contents | Correct CN, issuer, and SAN |
| Chain verification | CA signed the server certificate |
| HTTPS curl | End-to-end TLS works through nginx |
| SAN check | Modern TLS hostname validation will succeed |
