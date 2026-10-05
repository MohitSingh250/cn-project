# DNS Tests

---

## Test 1: Private Domain Resolution

```bash
dig @10.7.18.72 app.team1.test
```

### Verified Result

```
;; ANSWER SECTION:
app.team1.test.     0    IN    A    10.7.10.115

;; SERVER: 10.7.18.72#53(10.7.18.72)
```

**✅ PASS** — `app.team1.test` resolves to `10.7.10.115` via our private DNS server.

---

## Test 2: API Domain Resolution

```bash
dig @10.7.18.72 api.team1.test
```

### Verified Result

```
;; ANSWER SECTION:
api.team1.test.     0    IN    A    10.7.10.115
```

**✅ PASS** — `api.team1.test` resolves to `10.7.10.115`.

---

## Test 3: Public DNS Isolation

```bash
dig @8.8.8.8 app.team1.test
```

### Verified Result

```
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN
```

**✅ PASS** — Domain does not exist in Google's public DNS. Our domain is truly private.

---

## Test 4: Bidirectional LAN Connectivity

### Mac 1 → Mac 2

```bash
# Run on Mac 1
ping -c 4 10.7.10.115
```

**Result:** 4 packets transmitted, 4 received, 0.0% packet loss.

### Mac 2 → Mac 1

```bash
# Run on Mac 2
ping -c 4 10.7.18.72
```

**Result:** 4 packets transmitted, 4 received, 0.0% packet loss.

**✅ PASS** — Full bidirectional LAN connectivity confirmed.

---

## What These Tests Prove

| Concept | Evidence |
|---------|----------|
| Private DNS works | `app.team1.test` → `10.7.10.115` |
| DNS is private | Google DNS returns NXDOMAIN |
| LAN is functional | 0% packet loss both directions |
| dnsmasq serves cross-LAN | Mac 2 queries Mac 1's DNS successfully |
| Correct A records | Both domains resolve to nginx on Mac 2 |
