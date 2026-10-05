# HTTPS Tests

---

## Test 1: TLS Handshake Verification

```bash
curl -v https://app.team1.test/api/status --cacert /path/to/ca.crt
```

### Verified Result

```
* TLSv1.3 (OUT), TLS handshake, Client hello
* TLSv1.3 (IN), TLS handshake, Server hello
* TLSv1.3 (IN), TLS handshake, Certificate
* TLSv1.3 (IN), TLS handshake, Finished
* SSL certificate verify ok.
```

**✅ PASS** — TLS 1.3 handshake completed successfully.

---

## Test 2: Certificate Verification

From the same `curl -v` output:

```
*  subject: C=IN; ST=State; L=City; O=Team1; CN=app.team1.test
*  issuer: C=IN; ST=State; L=City; O=Team1; CN=Team1 Local CA
*  SSL certificate verify ok.
```

### Verified Properties

| Property | Value |
|----------|-------|
| Common Name (CN) | `app.team1.test` |
| Subject Alternative Name (SAN) | `app.team1.test` |
| Issuer | `Team1 Local CA` |
| TLS Version | `TLSv1.3` |
| Verification | `SSL certificate verify ok.` |

**✅ PASS** — Certificate chain is valid. The server certificate was issued by our trusted local CA.

---

## Test 3: HTTP Response Through HTTPS

```bash
curl -s https://app.team1.test/api/status --cacert /path/to/ca.crt
```

### Verified Result

```json
{"service":"running","backend":"A"}
```

Response headers:
```
HTTP/1.1 200 OK
Server: nginx/1.31.6
X-Backend: A
```

**✅ PASS** — Full HTTPS request/response cycle working. nginx terminates TLS and proxies to a backend.

---

## Test 4: DNS Resolution for HTTPS

```bash
curl -v https://app.team1.test/api/status --cacert /path/to/ca.crt 2>&1 | grep "Connected to"
```

### Verified Result

```
* Connected to app.team1.test (10.7.10.115) port 443
```

**✅ PASS** — `app.team1.test` resolves to `10.7.10.115` (nginx on Mac 2) before the TLS handshake begins.

---

## What These Tests Prove

| Concept | Evidence |
|---------|----------|
| TLS 1.3 | Handshake uses TLSv1.3 |
| Valid certificate | `SSL certificate verify ok` |
| Correct CN/SAN | Matches `app.team1.test` |
| Trusted CA | Issuer is `Team1 Local CA` |
| End-to-end HTTPS | HTTP 200 through encrypted channel |
| nginx serving | `Server: nginx/1.31.6` header |
