# Network Topology

The project utilizes a simple but realistic physical and logical network topology, running entirely over a shared Wi-Fi Local Area Network (LAN).

> **Phase 2 Update:** Both machines received new DHCP leases. All IP references below reflect the live Phase 2 addresses.

## Physical Topology

```
[ Wi-Fi Router / Campus AP ]
       │
       ├── (Wireless) ── [ Mac 1 - Abhishek Sharma (2401020081) ]
       │                   10.7.22.71
       └── (Wireless) ── [ Mac 2 - Mohit Singh (2401020098) ]
                           10.7.9.96
```

- Both machines are connected to the same wireless access point.
- They are on the same subnet (`255.255.248.0` / `/21`).
- Upstream DNS forwarders configured in `dnsmasq`: `10.5.7.1` (campus) and `8.8.8.8`.

## Logical Topology & IP Addressing

| Node | Interface | IP Address (Phase 2) | Subnet Mask | Broadcast Address |
|------|-----------|----------------------|-------------|-------------------|
| Mac 1 (Abhishek, 2401020081) | `en0` | `10.7.22.71` | `255.255.248.0` | `10.7.23.255` |
| Mac 2 (Mohit, 2401020098) | `en0` | `10.7.9.96` | `255.255.248.0` | `10.7.15.255` |

> Phase 1 (historical): Mac 1 was `10.7.18.72`, Mac 2 was `10.7.10.115`.

## Port Assignments (Phase 1 + Phase 2)

| Protocol | Port | Service | Host | Phase |
|----------|------|---------|------|-------|
| UDP | `53` | dnsmasq (Primary DNS) | `10.7.22.71` (Mac 1) | 1+2 |
| UDP | `53` | dnsmasq (Backup DNS) | `10.7.9.96` (Mac 2) | **2** |
| TCP | `443` | nginx (Primary Edge) | `10.7.9.96` (Mac 2) | 1+2 |
| TCP | `443` | nginx (Standby Edge) | `10.7.22.71` (Mac 1) | **2** |
| TCP | `3001` | Backend A | `10.7.22.71` (Mac 1) | 1+2 |
| TCP | `3002` | Backend B | `127.0.0.1` (Mac 2 loopback) | 1+2 |

## Network Paths

### Phase 1 + Phase 2 Normal Operation

1. **DNS Query Path (Primary):**
   - Client → Wi-Fi LAN → Mac 1 (`10.7.22.71:53`)

2. **DNS Query Path (Backup — Extension A):**
   - Client → Wi-Fi LAN → Mac 2 (`10.7.9.96:53`)

3. **HTTPS Request Path (Primary Edge):**
   - Client → Wi-Fi LAN → Mac 2 (`10.7.9.96:443`)

4. **Backend A Proxy Path:**
   - Mac 2 nginx (`10.7.9.96:443`) → Wi-Fi LAN → Mac 1 (`10.7.22.71:3001`)

5. **Backend B Proxy Path:**
   - Mac 2 nginx (`10.7.9.96:443`) → Loopback → Mac 2 (`127.0.0.1:3002`)

### Phase 2 — Edge Cutover (Extension E)

6. **HTTPS Request Path (Standby Edge — after DNS flip):**
   - Client → Wi-Fi LAN → Mac 1 (`10.7.22.71:443`)

7. **Backend A Proxy Path (via Standby Edge):**
   - Mac 1 nginx (`10.7.22.71:443`) → Loopback → Mac 1 (`127.0.0.1:3001`)

8. **Backend B Proxy Path (via Standby Edge):**
   - Mac 1 nginx (`10.7.22.71:443`) → Wi-Fi LAN → Mac 2 (`10.7.9.96:3002`)

## Firewall Rules Summary (Phase 2 — Extension C)

| Machine | Port | Rule | Allowed Sources |
|---------|------|------|----------------|
| Mac 1 | 3001 | `pf_backend_a.conf` | `lo0` (loopback) + `10.7.9.96` (nginx) |
| Mac 2 | 3002 | `pf_backend_b.conf` | `lo0` (loopback) only |

See [`config/firewall/`](../config/firewall/) for full rule files.
