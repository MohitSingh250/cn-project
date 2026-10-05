# Task D — TLS Certificates

> Create a local Certificate Authority and generate server certificates for HTTPS.

---

## Objective

Establish a complete TLS certificate chain so nginx can serve HTTPS for our private domains.

---

## Implementation

### 1. Create Local CA

```bash
# Generate CA private key
openssl genrsa -out ca.key 2048

# Generate CA root certificate
openssl req -x509 -new -nodes \
  -key ca.key -sha256 -days 365 \
  -out ca.crt \
  -subj "/C=IN/ST=State/L=City/O=Team1/CN=Team1 Local CA"
```

### 2. Create SAN Configuration

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

### 3. Generate Server Certificate

```bash
# Generate server private key
openssl genrsa -out server.key 2048

# Generate CSR
openssl req -new -key server.key -out server.csr -config san.cnf

# Sign with CA
openssl x509 -req -in server.csr \
  -CA ca.crt -CAkey ca.key -CAcreateserial \
  -out server.crt -days 365 -sha256 \
  -extensions v3_req -extfile san.cnf
```

### 4. Trust the CA

```bash
sudo security add-trusted-cert -d -r trustRoot \
  -k /Library/Keychains/System.keychain ca.crt
```

---

## Certificate Chain

```
Team1 Local CA (Self-signed root)
    │
    └── app.team1.test (Server certificate)
            CN  = app.team1.test
            SAN = DNS:app.team1.test
```

---

## Verification

| Test | Command | Result |
|------|---------|--------|
| Chain valid | `openssl verify -CAfile ca.crt server.crt` | `server.crt: OK` ✅ |
| CN correct | `openssl x509 -in server.crt -noout -subject` | `CN = app.team1.test` ✅ |
| SAN present | `openssl x509 -in server.crt -noout -text \| grep DNS` | `DNS:app.team1.test` ✅ |
| TLS works | `curl -v https://app.team1.test --cacert ca.crt` | `SSL certificate verify ok` ✅ |

---

## Status: ✅ Complete
