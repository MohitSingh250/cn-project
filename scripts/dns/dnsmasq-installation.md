# dnsmasq Installation

> **Machine:** Mac 1 — Abhishek (`10.7.18.72`)

---

## What Is dnsmasq?

`dnsmasq` is a lightweight DNS forwarder and DHCP server. In this project, it acts as a **private DNS server** that resolves our custom `.test` domains to internal IP addresses.

---

## Why dnsmasq?

| Feature | Benefit |
|---------|---------|
| Lightweight | Minimal resource usage on Mac |
| Simple configuration | Single config file |
| Wildcard support | `address=` directive handles entire domains |
| LAN-accessible | Listens on the LAN interface for other machines |
| Upstream forwarding | Non-local queries forwarded to Google DNS |

---

## Installation

### Using Homebrew

```bash
brew install dnsmasq
```

### Verify installation

```bash
dnsmasq --version
```

Expected output (version may vary):
```
Dnsmasq version 2.90  Copyright (c) 2000-2024 Simon Kelley
```

---

## Configuration File Location

```
/opt/homebrew/etc/dnsmasq.conf
```

---

## Starting dnsmasq

```bash
sudo brew services start dnsmasq
```

> **Note:** `sudo` is required because dnsmasq binds to port 53, which is a privileged port (< 1024).

### Verify it's running

```bash
sudo brew services list | grep dnsmasq
```

Expected:
```
dnsmasq started root /Library/LaunchDaemons/homebrew.mxcl.dnsmasq.plist
```

---

## Network Layer

| OSI Layer | Role |
|-----------|------|
| Application (L7) | DNS protocol — query/response for domain names |
| Transport (L4) | UDP port 53 (standard DNS) |
| Network (L3) | IP — reachable at `10.7.18.72` from the LAN |
