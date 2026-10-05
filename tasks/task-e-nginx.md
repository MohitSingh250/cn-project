# Task E — nginx Reverse Proxy

> Configure nginx as the edge server, reverse proxy, and TLS termination point.

---

## Objective

Set up nginx on Mac 2 (`10.7.10.115`) to intercept all HTTPS requests for `app.team1.test` and `api.team1.test`, decrypt the TLS traffic, and forward the requests to our backend pool.

---

## Implementation

### 1. Install nginx

```bash
brew install nginx
```

### 2. Configure nginx

Edit `/opt/homebrew/etc/nginx/nginx.conf`:

```nginx
worker_processes 1;

events {
    worker_connections 1024;
}

http {
    # Define the upstream backend pool
    upstream backends {
        server 10.7.18.72:3001;    # Backend A on Mac 1
        server 127.0.0.1:3002;     # Backend B on Mac 2
    }

    server {
        listen 443 ssl;
        server_name app.team1.test api.team1.test;

        # TLS Configuration
        ssl_certificate /path/to/server.crt;
        ssl_certificate_key /path/to/server.key;
        ssl_protocols TLSv1.2 TLSv1.3;

        # Reverse Proxy Configuration
        location / {
            proxy_pass http://backends;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
```

### 3. Start nginx

```bash
sudo nginx
```

---

## Verification

```bash
curl -v https://app.team1.test/api/status --cacert ca.crt
```

Result:
- **TLS Handshake** succeeds (`TLSv1.3`).
- **HTTP Response** is `200 OK`.
- **Server Header** is `nginx/1.31.6`.
- **Response Body** comes from one of the backends (`{"service":"running","backend":"A"}`).

---

## OSI Layers Involved

| Layer | nginx Role |
|-------|------------|
| Application (L7) | Parses HTTP request, forwards HTTP to backends, adds proxy headers |
| Presentation (L6) | Decrypts TLS (HTTPS → HTTP) |
| Transport (L4) | Listens on TCP port 443 |

---

## Status: ✅ Complete
