# Creating the Local Certificate Authority

> **Machine:** Mac 2 — Mohit (`10.7.10.115`)

---

## Why a Local CA?

Since our domains (`app.team1.test`, `api.team1.test`) are private and not publicly registered, no public Certificate Authority (like Let's Encrypt) will issue certificates for them.

We create our own **local CA** (`Team1 Local CA`) to:
1. Sign our own server certificates
2. Establish a trusted certificate chain
3. Enable `curl` and browsers to verify HTTPS without errors

---

## Step 1: Generate the CA Private Key

```bash
openssl genrsa -out ca.key 2048
```

This creates a 2048-bit RSA private key for the CA.

> **⚠️ Security:** `ca.key` is the root of trust. Never share or commit it.

---

## Step 2: Generate the CA Root Certificate

```bash
openssl req -x509 -new -nodes \
  -key ca.key \
  -sha256 \
  -days 365 \
  -out ca.crt \
  -subj "/C=IN/ST=State/L=City/O=Team1/CN=Team1 Local CA"
```

### Parameters Explained

| Parameter | Value | Purpose |
|-----------|-------|---------|
| `-x509` | — | Output a self-signed certificate (not a CSR) |
| `-new` | — | Generate a new certificate |
| `-nodes` | — | No passphrase on the key |
| `-key ca.key` | CA private key | Sign with our CA key |
| `-sha256` | — | Use SHA-256 hash algorithm |
| `-days 365` | 1 year | Certificate validity |
| `-out ca.crt` | CA certificate | Output file |
| `-subj` | Distinguished Name | Identifies this CA as "Team1 Local CA" |

---

## Step 3: Trust the CA on macOS

### Add to Keychain Access

```bash
sudo security add-trusted-cert -d -r trustRoot \
  -k /Library/Keychains/System.keychain ca.crt
```

Or manually:
1. Open **Keychain Access**
2. Drag `ca.crt` into the **System** keychain
3. Double-click the certificate
4. Expand **Trust**
5. Set **When using this certificate** to **Always Trust**

---

## Files Created

| File | Purpose | Size |
|------|---------|------|
| `ca.key` | CA private key | ~1.7 KB |
| `ca.crt` | CA root certificate | ~1.3 KB |

---

## Verification

```bash
openssl x509 -in ca.crt -text -noout | head -20
```

Should show:
```
Issuer: C = IN, ST = State, L = City, O = Team1, CN = Team1 Local CA
Subject: C = IN, ST = State, L = City, O = Team1, CN = Team1 Local CA
```

The Issuer and Subject are the same — this is a self-signed root certificate.
