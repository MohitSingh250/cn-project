# nginx Configuration

> **Machine:** Mac 2 — Mohit (`10.7.10.115`)

---

## Configuration File

```bash
sudo nano /opt/homebrew/etc/nginx/nginx.conf
```

---

## Configuration Breakdown

### 1. Upstream Block — Load Balancer Pool

```nginx
upstream backends {
    server 10.7.18.72:3001;    # Backend A — Mac 1 (Abhishek)
    server 127.0.0.1:3002;     # Backend B — Mac 2 (Mohit, localhost)
}
```

**What this does:**
- Defines two backend servers as an upstream group called `backends`
- nginx distributes requests across these servers using **round-robin** by default
- Backend A is on a separate physical machine (cross-LAN)
- Backend B is on the same machine (loopback)

---

### 2. Server Block — HTTPS Listener

```nginx
server {
    listen       443 ssl;
    server_name  app.team1.test api.team1.test;
```

- Listens on port `443` with SSL/TLS enabled
- Responds to requests for both `app.team1.test` and `api.team1.test`

---

### 3. SSL/TLS Configuration

```nginx
    ssl_certificate      /path/to/certs/server.crt;
    ssl_certificate_key  /path/to/certs/server.key;

    ssl_protocols        TLSv1.2 TLSv1.3;
    ssl_ciphers          HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
```

| Directive | Purpose |
|-----------|---------|
| `ssl_certificate` | Path to the signed server certificate |
| `ssl_certificate_key` | Path to the server private key |
| `ssl_protocols` | Allow TLSv1.2 and TLSv1.3 only |
| `ssl_ciphers` | Use strong ciphers, exclude weak ones |
| `ssl_prefer_server_ciphers` | Server chooses the cipher, not the client |

---

### 4. Reverse Proxy Location

```nginx
    location / {
        proxy_pass http://backends;

        proxy_set_header Host              $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
```

| Directive | Purpose |
|-----------|---------|
| `proxy_pass` | Forward requests to the upstream pool |
| `Host` | Preserve the original hostname |
| `X-Real-IP` | Pass the client's real IP to backends |
| `X-Forwarded-For` | Chain of proxies the request passed through |
| `X-Forwarded-Proto` | Tell the backend the original protocol (https) |

---

## Request Flow

```
Client                    nginx                        Backend
  │                         │                             │
  │──── HTTPS (TLS 1.3) ──→│                             │
  │                         │──── HTTP (plain) ──────────→│
  │                         │                             │
  │                         │←──── HTTP Response ─────────│
  │←──── HTTPS Response ────│                             │
```

nginx decrypts the TLS at the edge and forwards plain HTTP to backends. This is called **TLS termination**.

---

## Apply Changes

```bash
sudo nginx -t        # Test configuration
sudo nginx -s reload # Reload without downtime
```
