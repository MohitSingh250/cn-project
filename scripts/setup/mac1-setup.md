# Mac 1 Setup — Abhishek

> **Machine:** Mac 1  
> **User:** Abhishek  
> **IP Address:** `10.7.18.72`  
> **Interface:** `en0` (Wi-Fi)  
> **Roles:** Private DNS Server (dnsmasq), Backend A

---

## Prerequisites

- macOS with Homebrew installed
- Connected to the same Wi-Fi network as Mac 2
- Node.js installed (`node --version`)

---

## Step 1: Verify Network Identity

```bash
# Check the IP address assigned to en0
ifconfig en0 | grep "inet "
```

Expected output:
```
inet 10.7.18.72 netmask 0xfffff800 broadcast 10.7.23.255
```

---

## Step 2: Verify LAN Connectivity to Mac 2

```bash
ping -c 4 10.7.10.115
```

Expected output:
```
PING 10.7.10.115 (10.7.10.115): 56 data bytes
64 bytes from 10.7.10.115: icmp_seq=0 ttl=64 time=5.123 ms
64 bytes from 10.7.10.115: icmp_seq=1 ttl=64 time=3.456 ms
64 bytes from 10.7.10.115: icmp_seq=2 ttl=64 time=4.789 ms
64 bytes from 10.7.10.115: icmp_seq=3 ttl=64 time=3.210 ms

--- 10.7.10.115 ping statistics ---
4 packets transmitted, 4 packets received, 0.0% packet loss
```

---

## Step 3: Install dnsmasq

```bash
brew install dnsmasq
```

---

## Step 4: Configure dnsmasq

Edit `/opt/homebrew/etc/dnsmasq.conf`:

```bash
nano /opt/homebrew/etc/dnsmasq.conf
```

Add the following configuration:

```conf
listen-address=127.0.0.1,10.7.18.72
port=53
no-resolv
server=8.8.8.8
server=8.8.4.4
address=/app.team1.test/10.7.10.115
address=/api.team1.test/10.7.10.115
```

---

## Step 5: Start dnsmasq

```bash
sudo brew services start dnsmasq
```

---

## Step 6: Set Up Backend A

```bash
mkdir -p backend-a && cd backend-a
npm init -y
npm install express
```

Create `server.js` (see [`backend/backend-a/`](../../backend/backend-a/)).

Start Backend A:

```bash
node server.js A 3001
```

---

## Step 7: Verify Backend A

```bash
curl http://10.7.18.72:3001/api/status
```

Expected:
```json
{"service":"running","backend":"A"}
```

---

## Summary of Mac 1 Services

| Service | Port | Status |
|---------|------|--------|
| dnsmasq (DNS) | 53 | Running |
| Backend A (Node.js) | 3001 | Running |
