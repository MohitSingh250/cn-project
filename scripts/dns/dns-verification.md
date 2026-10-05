# DNS Verification

> **Run from:** Mac 2 — Mohit (`10.7.10.115`)

---

## Test 1: Private DNS Resolution

Query our private DNS server for `app.team1.test`:

```bash
dig @10.7.18.72 app.team1.test
```

**Expected output (relevant section):**

```
;; ANSWER SECTION:
app.team1.test.     0    IN    A    10.7.10.115
```

**What this proves:**
- dnsmasq on Mac 1 is running and reachable
- `app.team1.test` resolves to `10.7.10.115` (Mac 2)
- The DNS query travels over the LAN (UDP port 53)

---

## Test 2: API Domain Resolution

```bash
dig @10.7.18.72 api.team1.test
```

**Expected output:**

```
;; ANSWER SECTION:
api.team1.test.     0    IN    A    10.7.10.115
```

---

## Test 3: Public DNS Isolation (NXDOMAIN)

Query Google's public DNS for our private domain:

```bash
dig @8.8.8.8 app.team1.test
```

**Expected output:**

```
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN
```

**What this proves:**
- `app.team1.test` does **not** exist in public DNS
- The domain is truly private, resolvable only through our dnsmasq server
- This is a critical security/isolation property of the system

---

## Test 4: Upstream Forwarding

Verify that non-local queries are forwarded to Google DNS:

```bash
dig @10.7.18.72 google.com
```

**Expected:** A valid A record for `google.com` (proves upstream forwarding works).

---

## Summary

| Test | Command | Expected Result |
|------|---------|----------------|
| Private resolution | `dig @10.7.18.72 app.team1.test` | `10.7.10.115` |
| API resolution | `dig @10.7.18.72 api.team1.test` | `10.7.10.115` |
| Public DNS isolation | `dig @8.8.8.8 app.team1.test` | `NXDOMAIN` |
| Upstream forwarding | `dig @10.7.18.72 google.com` | Valid A record |
