# TLS Certificate Configuration

This directory documents the TLS certificate setup for the project.

> **⚠️ Important:** Actual private keys (`.key` files) and CA keys are excluded from this repository via `.gitignore`. Never commit private keys to version control.

---

## Certificate Chain

```
Team1 Local CA (Root)
    │
    └── app.team1.test (Server Certificate)
            SAN: app.team1.test
```

---

## Certificate Details

| Property | Value |
|----------|-------|
| **Common Name (CN)** | `app.team1.test` |
| **Subject Alternative Name (SAN)** | `app.team1.test` |
| **Issuer** | `Team1 Local CA` |
| **Protocol** | `TLSv1.3` |
| **Verification** | `SSL certificate verify ok` |

---

## Files Created During Setup

| File | Purpose | Committed? |
|------|---------|------------|
| `ca.key` | CA private key | ❌ Never |
| `ca.crt` | CA root certificate | ✅ Can be shared |
| `server.key` | Server private key | ❌ Never |
| `server.csr` | Certificate signing request | ❌ Intermediate |
| `server.crt` | Signed server certificate | ✅ Can be shared |
| `san.cnf` | OpenSSL SAN config | ✅ Yes |

---

## How Certificates Were Generated

See [`scripts/tls/`](../../scripts/tls/) for the complete step-by-step guide:

1. [`local-ca.md`](../../scripts/tls/local-ca.md) — Creating the local Certificate Authority
2. [`certificate-generation.md`](../../scripts/tls/certificate-generation.md) — Generating the server certificate with SAN
3. [`certificate-verification.md`](../../scripts/tls/certificate-verification.md) — Verifying the certificate chain

---

## Trust Configuration

The CA root certificate (`ca.crt`) must be trusted on client machines:

- **macOS**: Added to Keychain Access and marked as "Always Trust"
- **curl**: Use `--cacert ca.crt` flag, or trust system-wide

This allows `curl` and browsers to verify the certificate chain without warnings.
