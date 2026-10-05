# nginx Installation

> **Machine:** Mac 2 — Mohit (`10.7.10.115`)

---

## What Is nginx?

nginx (pronounced "engine-x") is a high-performance HTTP server and reverse proxy. In this project, it serves three critical roles:

1. **TLS Termination** — Handles HTTPS encryption/decryption
2. **Reverse Proxy** — Forwards client requests to backend servers
3. **Load Balancer** — Distributes requests evenly across backends

---

## Installation

### Using Homebrew

```bash
brew install nginx
```

### Verify installation

```bash
nginx -v
```

Expected:
```
nginx version: nginx/1.31.6
```

---

## Configuration File Location

```
/opt/homebrew/etc/nginx/nginx.conf
```

---

## Default Ports

| Port | Protocol | Usage |
|------|----------|-------|
| 8080 | HTTP | Default nginx (we don't use this) |
| 443 | HTTPS | Our TLS-enabled server |

> Port 443 requires `sudo` to bind (privileged port).

---

## Starting nginx

```bash
sudo nginx
```

Or via Homebrew services:

```bash
sudo brew services start nginx
```

---

## Stopping nginx

```bash
sudo nginx -s stop
```

---

## Reloading Configuration (without stopping)

```bash
sudo nginx -s reload
```

---

## Testing Configuration Syntax

Before restarting, always check for syntax errors:

```bash
sudo nginx -t
```

Expected:
```
nginx: the configuration file /opt/homebrew/etc/nginx/nginx.conf syntax is ok
nginx: configuration file /opt/homebrew/etc/nginx/nginx.conf test is successful
```
