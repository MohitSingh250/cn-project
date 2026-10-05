# dnsmasq Configuration

This directory contains the example configuration for `dnsmasq`, the private DNS server running on **Mac 1 (Abhishek — `10.7.18.72`)**.

---

## File

| File | Purpose |
|------|---------|
| `dnsmasq.conf.example` | Complete dnsmasq configuration with private domain records |

---

## What dnsmasq Does

`dnsmasq` acts as a lightweight DNS server on the local network. It resolves our private `.test` domains to the IP address of Mac 2, where nginx handles incoming requests.

| Domain | Resolves To | Meaning |
|--------|-------------|---------|
| `app.team1.test` | `10.7.10.115` | nginx on Mac 2 |
| `api.team1.test` | `10.7.10.115` | nginx on Mac 2 |

---

## Key Configuration Directives

| Directive | Value | Purpose |
|-----------|-------|---------|
| `listen-address` | `127.0.0.1,10.7.18.72` | Listen on localhost and LAN |
| `port` | `53` | Standard DNS port |
| `no-resolv` | — | Don't read `/etc/resolv.conf` |
| `server` | `8.8.8.8`, `8.8.4.4` | Upstream DNS for non-local queries |
| `address` | `/app.team1.test/10.7.10.115` | Private A record |
| `address` | `/api.team1.test/10.7.10.115` | Private A record |

---

## Installation Location

On macOS with Homebrew, the configuration file is typically at:

```
/opt/homebrew/etc/dnsmasq.conf
```

---

## Privacy Verification

These domains are **not publicly resolvable**:

```bash
dig @8.8.8.8 app.team1.test
# Returns: NXDOMAIN
```

They only resolve through our private DNS server:

```bash
dig @10.7.18.72 app.team1.test
# Returns: 10.7.10.115
```
