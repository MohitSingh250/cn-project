# Certificate Generation

> **Machine:** Mac 2 — Mohit (`10.7.10.115`)

---

## Step 1: Generate the Server Private Key

```bash
openssl genrsa -out server.key 2048
```

---

## Step 2: Create the SAN Configuration File

Modern TLS requires **Subject Alternative Names (SAN)**. Browsers and `curl` check SAN, not just CN.

Create `san.cnf`:

```ini
[req]
default_bits       = 2048
prompt             = no
default_md         = sha256
distinguished_name = dn
req_extensions     = v3_req

[dn]
C  = IN
ST = State
L  = City
O  = Team1
CN = app.team1.test

[v3_req]
subjectAltName = @alt_names

[alt_names]
DNS.1 = app.team1.test
```

### Why SAN?

| Field | Purpose |
|-------|---------|
| `CN` (Common Name) | Legacy — some older clients still check this |
| `SAN` (Subject Alternative Name) | Modern standard — all current TLS clients require this |

Without SAN, `curl` would report: `SSL: no alternative certificate subject name matches target host name`

---

## Step 3: Generate the Certificate Signing Request (CSR)

```bash
openssl req -new \
  -key server.key \
  -out server.csr \
  -config san.cnf
```

---

## Step 4: Sign the CSR with the Local CA

```bash
openssl x509 -req \
  -in server.csr \
  -CA ca.crt \
  -CAkey ca.key \
  -CAcreateserial \
  -out server.crt \
  -days 365 \
  -sha256 \
  -extensions v3_req \
  -extfile san.cnf
```

### Parameters

| Parameter | Purpose |
|-----------|---------|
| `-CA ca.crt` | Sign with our CA certificate |
| `-CAkey ca.key` | Use the CA private key |
| `-CAcreateserial` | Auto-generate serial number |
| `-extensions v3_req` | Include the SAN extension |
| `-extfile san.cnf` | Read SAN config from this file |

---

## Files Created

| File | Purpose | Committed? |
|------|---------|------------|
| `server.key` | Server private key | ❌ Never |
| `server.csr` | Certificate signing request | ❌ Intermediate |
| `server.crt` | Signed server certificate | ✅ Can be shared |
| `san.cnf` | SAN configuration | ✅ Yes |

---

## Certificate Chain

```
Team1 Local CA (ca.crt)
    │
    └── app.team1.test (server.crt)
            CN  = app.team1.test
            SAN = DNS:app.team1.test
```

---

## Verification

```bash
openssl x509 -in server.crt -text -noout | grep -A 2 "Subject Alternative Name"
```

Expected:
```
X509v3 Subject Alternative Name:
    DNS:app.team1.test
```
