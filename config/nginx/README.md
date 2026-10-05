# nginx Configuration

This directory contains the example configuration for `nginx`, the HTTPS edge server running on **Mac 2 (Mohit — `10.7.10.115`)**.

---

## File

| File | Purpose |
|------|---------|
| `nginx.conf.example` | Complete nginx config with TLS, reverse proxy, and load balancing |

---

## What nginx Does

nginx serves three roles in this project:

### 1. TLS Termination
- Listens on port `443` with SSL enabled
- Uses the self-signed certificate from our local CA (`Team1 Local CA`)
- Negotiates `TLSv1.3` with clients
- Decrypts HTTPS and forwards plain HTTP to backends

### 2. Reverse Proxy
- Forwards requests to the `backends` upstream pool
- Passes original client headers (`Host`, `X-Real-IP`, `X-Forwarded-For`)

### 3. Round-Robin Load Balancer
- Distributes requests evenly between Backend A and Backend B
- Default nginx behavior: no configuration needed beyond listing the servers

---

## Upstream Pool

```nginx
upstream backends {
    server 10.7.18.72:3001;    # Backend A — Mac 1
    server 127.0.0.1:3002;     # Backend B — Mac 2
}
```

---

## Server Names

```nginx
server_name  app.team1.test api.team1.test;
```

Both domains are handled by the same server block.

---

## Verified Behavior

| Property | Value |
|----------|-------|
| Server version | `nginx/1.31.6` |
| TLS version | `TLSv1.3` |
| Certificate CN | `app.team1.test` |
| Issuer | `Team1 Local CA` |
| Load balancing | Round-robin (alternating A/B) |

---

## Installation Location

On macOS with Homebrew:

```
/opt/homebrew/etc/nginx/nginx.conf
```

> **Note:** Update the `ssl_certificate` and `ssl_certificate_key` paths to match your actual certificate locations.
