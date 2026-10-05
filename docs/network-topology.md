# Network Topology

The project utilizes a simple but realistic physical and logical network topology, running entirely over a shared Wi-Fi Local Area Network (LAN).

## Physical Topology

```
[ Wi-Fi Router ]
       │
       ├── (Wireless) ── [ Mac 1 - Abhishek ]
       │
       └── (Wireless) ── [ Mac 2 - Mohit ]
```

- Both machines are connected to the same wireless access point.
- They are on the same subnet (`255.255.248.0` / `/21`).

## Logical Topology & IP Addressing

| Node | Interface | IP Address | Subnet Mask | Broadcast Address |
|------|-----------|------------|-------------|-------------------|
| Mac 1 | `en0` | `10.7.18.72` | `255.255.248.0` | `10.7.23.255` |
| Mac 2 | `en0` | `10.7.10.115` | `255.255.248.0` | `10.7.15.255` |

## Port Assignments

| Protocol | Port | Service | Host |
|----------|------|---------|------|
| UDP | `53` | dnsmasq | `10.7.18.72` (Mac 1) |
| TCP | `443` | nginx | `10.7.10.115` (Mac 2) |
| TCP | `3001` | Backend A | `10.7.18.72` (Mac 1) |
| TCP | `3002` | Backend B | `127.0.0.1` (Mac 2 Localhost) |

## Network Paths

1. **DNS Query Path:**
   - Mac 2 (`10.7.10.115`) → Wi-Fi LAN → Mac 1 (`10.7.18.72:53`)

2. **HTTPS Request Path:**
   - Client → Wi-Fi LAN → Mac 2 (`10.7.10.115:443`)

3. **Backend A Proxy Path:**
   - Mac 2 (`10.7.10.115:443`) → Wi-Fi LAN → Mac 1 (`10.7.18.72:3001`)

4. **Backend B Proxy Path:**
   - Mac 2 (`10.7.10.115:443`) → Loopback → Mac 2 (`127.0.0.1:3002`)
