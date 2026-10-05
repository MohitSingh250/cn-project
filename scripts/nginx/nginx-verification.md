# nginx Verification

> **Machine:** Mac 2 — Mohit (`10.7.10.115`)

---

## Test 1: Configuration Syntax

```bash
sudo nginx -t
```

Expected:
```
nginx: the configuration file /opt/homebrew/etc/nginx/nginx.conf syntax is ok
nginx: configuration file /opt/homebrew/etc/nginx/nginx.conf test is successful
```

---

## Test 2: Process Running

```bash
ps aux | grep nginx
```

Expected: nginx master and worker processes visible.

---

## Test 3: Port 443 Listening

```bash
sudo lsof -i :443
```

Expected: nginx listening on port 443.

---

## Test 4: HTTPS Response

```bash
curl -v https://app.team1.test/api/status --cacert /path/to/ca.crt
```

**Expected output (key lines):**

```
< HTTP/1.1 200 OK
< Server: nginx/1.31.6
< X-Backend: A
```

or

```
< X-Backend: B
```

---

## Test 5: Server Version Header

```
Server: nginx/1.31.6
```

This confirms nginx is handling the request, not the backends directly.

---

## Verified Properties

| Property | Value |
|----------|-------|
| Version | `nginx/1.31.6` |
| HTTPS port | `443` |
| TLS version | `TLSv1.3` |
| Upstream backends | 2 (Backend A + B) |
| Load balancing | Round-robin |
