# Task B — Private DNS

> Set up a private DNS server to resolve custom domains within the LAN.

---

## Objective

Deploy `dnsmasq` on Mac 1 as a private DNS server that resolves `app.team1.test` and `api.team1.test` to Mac 2's IP address (`10.7.10.115`).

---

## Implementation

### 1. Install dnsmasq on Mac 1

```bash
brew install dnsmasq
```

### 2. Configure Private Records

Edit `/opt/homebrew/etc/dnsmasq.conf`:

```conf
listen-address=127.0.0.1,10.7.18.72
port=53
no-resolv
server=8.8.8.8
server=8.8.4.4
address=/app.team1.test/10.7.10.115
address=/api.team1.test/10.7.10.115
```

### 3. Start the Service

```bash
sudo brew services start dnsmasq
```

### 4. Configure Mac 2 to Use Private DNS

```bash
sudo networksetup -setdnsservers Wi-Fi 10.7.18.72
```

---

## Verification

| Test | Command | Result |
|------|---------|--------|
| Private resolution | `dig @10.7.18.72 app.team1.test` | `10.7.10.115` ✅ |
| API resolution | `dig @10.7.18.72 api.team1.test` | `10.7.10.115` ✅ |
| Public isolation | `dig @8.8.8.8 app.team1.test` | `NXDOMAIN` ✅ |

---

## DNS Architecture

```
Mac 2 (Client)
    │
    │ DNS Query: "app.team1.test?"
    │ UDP port 53
    │
    ▼
Mac 1 — dnsmasq (10.7.18.72:53)
    │
    ├── Match: app.team1.test → 10.7.10.115
    └── No match → Forward to 8.8.8.8
```

---

## OSI Layers Involved

| Layer | What Happens |
|-------|-------------|
| Application (L7) | DNS protocol (query type A, response with IP) |
| Transport (L4) | UDP port 53 |
| Network (L3) | IP packet from `10.7.10.115` to `10.7.18.72` |
| Data Link (L2) | Wi-Fi frame between two MacBooks |

---

## Status: ✅ Complete
