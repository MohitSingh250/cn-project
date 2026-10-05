# Mac 2 Setup — Mohit

> **Machine:** Mac 2  
> **User:** Mohit Singh  
> **IP Address:** `10.7.10.115`  
> **Interface:** `en0` (Wi-Fi)  
> **Roles:** nginx HTTPS Edge / Reverse Proxy / Load Balancer, Backend B

---

## Prerequisites

- macOS with Homebrew installed
- Connected to the same Wi-Fi network as Mac 1
- Node.js installed (`node --version`)

---

## Step 1: Verify Network Identity

```bash
ifconfig en0 | grep "inet "
```

Expected output:
```
inet 10.7.10.115 netmask 0xfffff800 broadcast 10.7.15.255
```

---

## Step 2: Verify LAN Connectivity to Mac 1

```bash
ping -c 4 10.7.18.72
```

Expected output:
```
PING 10.7.18.72 (10.7.18.72): 56 data bytes
64 bytes from 10.7.18.72: icmp_seq=0 ttl=64 time=4.567 ms
64 bytes from 10.7.18.72: icmp_seq=1 ttl=64 time=3.890 ms
64 bytes from 10.7.18.72: icmp_seq=2 ttl=64 time=5.012 ms
64 bytes from 10.7.18.72: icmp_seq=3 ttl=64 time=3.345 ms

--- 10.7.18.72 ping statistics ---
4 packets transmitted, 4 packets received, 0.0% packet loss
```

---

## Step 3: Configure DNS Resolution

Set Mac 2 to use Mac 1's dnsmasq as its DNS server:

**System Preferences → Network → Wi-Fi → Advanced → DNS**

Add DNS server: `10.7.18.72`

Or via command line:

```bash
sudo networksetup -setdnsservers Wi-Fi 10.7.18.72
```

---

## Step 4: Install nginx

```bash
brew install nginx
```

---

## Step 5: Generate TLS Certificates

See [`scripts/tls/`](../tls/) for the complete certificate generation guide.

Summary:
1. Create a local CA (`Team1 Local CA`)
2. Generate a server certificate for `app.team1.test` with SAN
3. Sign it with the local CA
4. Trust the CA in macOS Keychain

---

## Step 6: Configure nginx

Edit `/opt/homebrew/etc/nginx/nginx.conf`:

```bash
nano /opt/homebrew/etc/nginx/nginx.conf
```

See [`config/nginx/nginx.conf.example`](../../config/nginx/nginx.conf.example) for the complete configuration.

Key sections:
- Upstream pool with Backend A (`10.7.18.72:3001`) and Backend B (`127.0.0.1:3002`)
- SSL/TLS with the generated certificates
- Reverse proxy with header forwarding

---

## Step 7: Start nginx

```bash
sudo brew services start nginx
```

Or:

```bash
sudo nginx
```

---

## Step 8: Set Up Backend B

```bash
mkdir -p backend-b && cd backend-b
npm init -y
npm install express
```

Create `server.js` (see [`backend/backend-b/`](../../backend/backend-b/)).

Start Backend B:

```bash
node server.js B 3002
```

---

## Step 9: Verify Backend B

```bash
curl http://127.0.0.1:3002/api/status
```

Expected:
```json
{"service":"running","backend":"B"}
```

---

## Step 10: Verify End-to-End HTTPS

```bash
curl -v https://app.team1.test/api/status --cacert /path/to/ca.crt
```

Expected: TLS 1.3 handshake, HTTP 200, alternating `X-Backend` headers.

---

## Summary of Mac 2 Services

| Service | Port | Status |
|---------|------|--------|
| nginx (HTTPS) | 443 | Running |
| Backend B (Node.js) | 3002 | Running |
