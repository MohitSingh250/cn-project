# Task A — Network Identity

> Establish the network identity of both machines on the LAN.

---

## Objective

Identify the IP addresses and interfaces of both MacBooks, verify they are on the same LAN, and confirm bidirectional connectivity.

---

## Implementation

### Mac 1 — Abhishek

```bash
ifconfig en0 | grep "inet "
```

Result:
```
inet 10.7.18.72 netmask 0xfffff800 broadcast 10.7.23.255
```

### Mac 2 — Mohit

```bash
ifconfig en0 | grep "inet "
```

Result:
```
inet 10.7.10.115 netmask 0xfffff800 broadcast 10.7.15.255
```

---

## Connectivity Verification

### Mac 1 → Mac 2

```bash
ping -c 4 10.7.10.115
```

Result: **4 packets transmitted, 4 received, 0.0% packet loss**

### Mac 2 → Mac 1

```bash
ping -c 4 10.7.18.72
```

Result: **4 packets transmitted, 4 received, 0.0% packet loss**

---

## Network Identity Table

| Property | Mac 1 (Abhishek) | Mac 2 (Mohit) |
|----------|-------------------|----------------|
| IP Address | `10.7.18.72` | `10.7.10.115` |
| Interface | `en0` | `en0` |
| Network | Wi-Fi LAN | Wi-Fi LAN |
| Connectivity | ✅ Bidirectional | ✅ Bidirectional |

---

## OSI Layers Involved

| Layer | What Happens |
|-------|-------------|
| Physical (L1) | Wi-Fi radio transmission |
| Data Link (L2) | MAC frame encapsulation on `en0` |
| Network (L3) | ICMP Echo Request/Reply (ping) over IP |

---

## Status: ✅ Complete
