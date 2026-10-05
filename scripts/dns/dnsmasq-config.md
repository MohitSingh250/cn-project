# dnsmasq Configuration

> **Machine:** Mac 1 — Abhishek (`10.7.18.72`)

---

## Configuration File

Edit `/opt/homebrew/etc/dnsmasq.conf`:

```bash
nano /opt/homebrew/etc/dnsmasq.conf
```

---

## Complete Configuration

```conf
# Listen on localhost and LAN interface
listen-address=127.0.0.1,10.7.18.72

# Standard DNS port
port=53

# Don't read /etc/resolv.conf for upstream servers
no-resolv

# Upstream DNS servers for non-local queries
server=8.8.8.8
server=8.8.4.4

# Private domain records
address=/app.team1.test/10.7.10.115
address=/api.team1.test/10.7.10.115
```

---

## Configuration Breakdown

### `listen-address=127.0.0.1,10.7.18.72`

Makes dnsmasq listen on:
- `127.0.0.1` — For local DNS queries on Mac 1 itself
- `10.7.18.72` — For DNS queries from Mac 2 (and any other LAN client)

### `port=53`

Standard DNS port. All DNS clients use port 53 by default.

### `no-resolv`

Prevents dnsmasq from reading `/etc/resolv.conf` for upstream servers. We explicitly define upstream servers instead.

### `server=8.8.8.8` / `server=8.8.4.4`

Queries for domains NOT matching our private records are forwarded to Google's public DNS servers.

### `address=/app.team1.test/10.7.10.115`

Creates an A record:
- `app.team1.test` → `10.7.10.115`

This means any DNS query for `app.team1.test` returns the IP of Mac 2, where nginx is running.

### `address=/api.team1.test/10.7.10.115`

Same pattern for the API domain.

---

## Restart After Configuration Changes

```bash
sudo brew services restart dnsmasq
```

---

## DNS Flow

```
Client queries: app.team1.test
    │
    ▼
dnsmasq (10.7.18.72:53)
    │
    ├── Matches "app.team1.test" → Returns 10.7.10.115
    │
    └── No match (e.g., google.com) → Forwards to 8.8.8.8
```
