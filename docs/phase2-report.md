# Phase 2 Final Report — Harden, Recover, and Troubleshoot

> **Team:** Sambhav
> **Course:** Computer Networks
> **Phase:** Phase 2

---

## 1. What Changed from Phase 1

| Dimension | Phase 1 | Phase 2 |
|-----------|---------|---------|
| Mac 1 IP | `10.7.18.72` | `10.7.22.71` (new DHCP lease) |
| Mac 2 IP | `10.7.10.115` | `10.7.9.96` (new DHCP lease) |
| DNS | Primary on Mac 1 only | Primary on Mac 1 + **Backup on Mac 2** (`dnsmasq`) |
| DNS TTL | Unset (default) | **30 seconds** (`local-ttl=30`, `host-record` TTL=30) |
| Firewall | None | **`pf` rules** blocking direct backend port access |
| nginx upstream | No health params | `max_fails=2 fail_timeout=5s` + `proxy_next_upstream` |
| Edge | Single nginx on Mac 2 | Mac 2 (primary) + **Mac 1 standby** `nginx` on :443 |
| Failover | Manual | Automatic: nginx retries other backend within 2s timeout |

---

## 2. Network Parameters

| Node | IP | Services |
|------|-----|---------|
| Mac 1 (Abhishek, 2401020081) | `10.7.22.71` | Primary DNS :53, Backend A :3001, Standby nginx :443 |
| Mac 2 (Mohit Singh, 2401020098) | `10.7.9.96` | Primary Edge :443, Backup DNS :53, Backend B :3002 |
| Upstream DNS forwarders | `10.5.7.1`, `8.8.8.8` | Campus DNS, Google DNS |
| Private domains | `app.team1.test`, `api.team1.test` | Both → `10.7.9.96` (Phase 2 normal) |

---

## 3. Extension A — Backup DNS

**Objective:** Prove that if primary `dnsmasq` on Mac 1 (`10.7.22.71`) is stopped, Mac 2's backup `dnsmasq` on `10.7.9.96` continues to resolve private domains without interruption.

### Setup

**Mac 2 (`10.7.9.96`) — Backup DNS config** ([`config/dnsmasq/dnsmasq-mac2.conf`](../config/dnsmasq/dnsmasq-mac2.conf)):
```ini
listen-address=127.0.0.1,10.7.9.96
bind-interfaces
no-resolv
server=10.5.7.1
server=8.8.8.8
local-ttl=30
host-record=app.team1.test,10.7.9.96,30
host-record=api.team1.test,10.7.9.96,30
```

### Commands Executed

```bash
# Mac 1: Stop primary DNS
sudo brew services stop dnsmasq

# Mac 2: Confirm backup DNS still resolves
dscacheutil -flushcache; sudo killall -HUP mDNSResponder
dig @10.7.9.96 app.team1.test +short

# Mac 2: Confirm HTTPS still works via backup DNS
curl -sSI --cacert ~/certs/ca.crt https://app.team1.test/api/status | grep -iE "^HTTP|X-Backend:"

# Mac 1: Restart primary DNS
sudo brew services start dnsmasq
```

### Observed Output

```
# dig @10.7.9.96 app.team1.test +short
10.7.9.96

# curl result
HTTP/1.1 200 OK
X-Backend: B
```

### Evidence

- [`evidence/phase2-backup-dns/ext-a-mac1-dnsmasq-stop-start-dig-ttl30.png`](../evidence/phase2-backup-dns/ext-a-mac1-dnsmasq-stop-start-dig-ttl30.png)

### Analysis

Mac 2's `dnsmasq` maintains an independent, authoritative copy of the zone data for `app.team1.test` and `api.team1.test`. Because both `dnsmasq` instances are configured identically (same `host-record` entries), the backup seamlessly absorbs all DNS queries when Mac 1's service is offline. This eliminates DNS as a single point of failure.

---

## 4. Extension B — TTL Cache Control

**Objective:** Demonstrate that `local-ttl=30` and `host-record=…,30` cause the DNS resolver to cache entries for exactly 30 seconds, and show both stale-cache behaviour and manual flush.

### Configuration

Both `dnsmasq` instances set:
```ini
local-ttl=30
host-record=app.team1.test,10.7.9.96,30
```

### Commands Executed

```bash
# Step 1: Confirm TTL=30 in answer section
dig @10.7.22.71 app.team1.test
# Expected: app.team1.test.  30  IN  A  10.7.9.96

# Step 2: Change host-record to new IP (Extension E preview) and flush
sudo sed -i '' 's/host-record=app.team1.test,10.7.9.96,30/host-record=app.team1.test,10.7.22.71,30/' \
  $(brew --prefix)/etc/dnsmasq.conf
sudo brew services restart dnsmasq

# Step 3: Observe stale cache (before 30s expire)
dscacheutil -q host -a name app.team1.test   # still returns old IP

# Step 4: Wait for TTL expiry
sleep 35
dscacheutil -q host -a name app.team1.test   # returns new IP automatically

# Step 5: Manual flush (instant update without waiting)
sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder
dscacheutil -q host -a name app.team1.test   # returns new IP immediately

# Step 6: Restore original host-record
sudo sed -i '' 's/host-record=app.team1.test,10.7.22.71,30/host-record=app.team1.test,10.7.9.96,30/' \
  $(brew --prefix)/etc/dnsmasq.conf
sudo brew services restart dnsmasq
```

### Observed Output

```
# dig answer section
app.team1.test.    30    IN    A    10.7.9.96
;; SERVER: 10.7.22.71#53(10.7.22.71)

# After host-record swap, before sleep 35:
name: app.team1.test
ip_address: 10.7.9.96    ← stale cache

# After sleep 35:
name: app.team1.test
ip_address: 10.7.22.71   ← auto-updated

# After dscacheutil -flushcache:
name: app.team1.test
ip_address: 10.7.9.96    ← restored and instant
```

### Evidence

- [`evidence/phase2-ttl-cache/ext-b-dig-ttl30-dscacheutil-flush-ip-update.png`](../evidence/phase2-ttl-cache/ext-b-dig-ttl30-dscacheutil-flush-ip-update.png)

### Analysis

The 30-second TTL is a deliberate engineering trade-off: short enough to allow rapid DNS-based failover (see Extension E) while long enough to reduce DNS query load. The `dscacheutil -flushcache` + `killall -HUP mDNSResponder` combination instantly clears the macOS resolver cache, enabling zero-TTL-wait migrations in operational emergencies.

---

## 5. Extension C — `pf` Firewall Enforcement

**Objective:** Use macOS `pf` to block direct backend access from arbitrary clients, while preserving nginx's ability to proxy requests.

### Configuration Files

**Mac 1 — Backend A firewall** ([`config/firewall/pf_backend_a.conf`](../config/firewall/pf_backend_a.conf)):
```pf
pass in quick on lo0 proto tcp from any to any port 3001
pass in quick proto tcp from 10.7.9.96 to any port 3001
block drop in quick proto tcp from any to any port 3001
```

**Mac 2 — Backend B firewall** ([`config/firewall/pf_backend_b.conf`](../config/firewall/pf_backend_b.conf)):
```pf
pass in quick on lo0 proto tcp from any to any port 3002
block drop in quick proto tcp from any to any port 3002
```

### Commands Executed

```bash
# Apply firewall on Mac 1 (Backend A)
sudo cp /etc/pf.conf /etc/pf.conf.bak
cat << 'EOF' > /tmp/pf_backend_a.conf
pass in quick on lo0 proto tcp from any to any port 3001
pass in quick proto tcp from 10.7.9.96 to any port 3001
block drop in quick proto tcp from any to any port 3001
EOF
sudo pfctl -f /tmp/pf_backend_a.conf
sudo pfctl -e

# Apply firewall on Mac 2 (Backend B)
sudo cp /etc/pf.conf /etc/pf.conf.bak
cat << 'EOF' > /tmp/pf_backend_b.conf
pass in quick on lo0 proto tcp from any to any port 3002
block drop in quick proto tcp from any to any port 3002
EOF
sudo pfctl -f /tmp/pf_backend_b.conf
sudo pfctl -e

# Verify: direct curl MUST timeout (exit code 28)
curl --max-time 3 -i http://10.7.9.96:3002/api/status
# → curl: (28) Connection timed out after 3006 milliseconds

# Verify: HTTPS via nginx MUST succeed and load-balance
for i in 1 2 3 4 5 6; do
  curl -sSI --cacert "$HOME/certs/ca.crt" https://app.team1.test/api/status | grep -iE "^HTTP|X-Backend:"
done

# Restore original firewall
sudo pfctl -f /etc/pf.conf.bak
```

### Observed Output

```
# Direct curl (blocked):
curl: (28) Connection timed out after 3006 milliseconds

# HTTPS via nginx (passes through — nginx is whitelisted):
HTTP/1.1 200 OK
X-Backend: B
HTTP/1.1 200 OK
X-Backend: B
HTTP/1.1 200 OK
X-Backend: B
...
```

> **Note:** When Backend A firewall is active and Backend B is also blocked, all 6 requests route to `X-Backend: B` (Mac 2 nginx proxies locally to 127.0.0.1:3002, which is whitelisted on loopback).

### Evidence

- [`evidence/phase2-firewall/ext-c-mac2-pfctl-backend-b-block-curl-timeout-https-ok.png`](../evidence/phase2-firewall/ext-c-mac2-pfctl-backend-b-block-curl-timeout-https-ok.png)
- [`evidence/phase2-firewall/ext-c-mac1-pf-backend-a-curl-timeout-https-ab-roundrobin.png`](../evidence/phase2-firewall/ext-c-mac1-pf-backend-a-curl-timeout-https-ab-roundrobin.png)

### Analysis

The `quick` keyword ensures rules short-circuit: the first matching rule terminates evaluation. Loopback (`lo0`) traffic is whitelisted first, so nginx on Mac 2 proxying to `127.0.0.1:3002` always succeeds. The `block drop` (silent discard rather than RST) causes `curl` to block until `--max-time` expires, confirming the firewall is active. The HTTPS path remains unaffected because nginx connects from `10.7.9.96`, which is explicitly permitted in `pf_backend_a.conf`.

---

## 6. Extension D — HA Failover and Recovery

**Objective:** Prove nginx automatically reroutes all traffic to Backend B when Backend A is unavailable, and resumes round-robin load balancing when Backend A recovers — without any manual nginx reconfiguration.

### nginx Upstream Configuration (Phase 2)

```nginx
upstream backends {
    server 10.7.22.71:3001 max_fails=2 fail_timeout=5s;
    server 127.0.0.1:3002  max_fails=2 fail_timeout=5s;
}

location / {
    proxy_pass         http://backends;
    proxy_connect_timeout 2s;
    proxy_read_timeout    5s;
    proxy_next_upstream error timeout http_502 http_503 http_504;
}
```

**Key parameters:**
- `max_fails=2`: mark a backend unavailable after 2 consecutive failures
- `fail_timeout=5s`: retry the failed backend every 5 seconds
- `proxy_next_upstream`: immediately retry the next upstream on connection error or timeout

### Commands Executed

```bash
# Step 1: Stop Backend A on Mac 1
# (kill node server.js A 3001 process)

# Step 2: Send 6 requests — ALL should return X-Backend: B
for i in 1 2 3 4 5 6; do
  curl -sSI --cacert "$HOME/certs/ca.crt" https://app.team1.test/api/status | grep -iE "^HTTP|X-Backend:"
done

# Step 3: Restart Backend A on Mac 1
node server.js A 3001 &

# Step 4: Send 6 more requests — should alternate A and B
for i in 1 2 3 4 5 6; do
  curl -sSI --cacert "$HOME/certs/ca.crt" https://app.team1.test/api/status | grep -iE "^HTTP|X-Backend:"
done
```

### Observed Output

```
# Backend A stopped — 6 requests all go to B:
HTTP/1.1 200 OK
X-Backend: B
HTTP/1.1 200 OK
X-Backend: B
HTTP/1.1 200 OK
X-Backend: B
HTTP/1.1 200 OK
X-Backend: B
HTTP/1.1 200 OK
X-Backend: B
HTTP/1.1 200 OK
X-Backend: B

# After Backend A restart — alternating:
HTTP/1.1 200 OK
X-Backend: B
HTTP/1.1 200 OK
X-Backend: A
HTTP/1.1 200 OK
X-Backend: B
HTTP/1.1 200 OK
X-Backend: A
HTTP/1.1 200 OK
X-Backend: B
HTTP/1.1 200 OK
X-Backend: A
```

### Evidence

- [`evidence/phase2-ha-failover/ext-d-mac1-backend-a-firewall-6x-backend-b-then-ab-recovery.png`](../evidence/phase2-ha-failover/ext-d-mac1-backend-a-firewall-6x-backend-b-then-ab-recovery.png)

### Analysis

nginx's `max_fails=2 fail_timeout=5s` passive health check detects Backend A's failure after just 2 connection errors. The `proxy_next_upstream` directive ensures the failed connection attempt is transparently retried against Backend B within the same client request — zero client-visible errors. After `fail_timeout=5s`, nginx re-probes Backend A; if it succeeds, the backend re-enters the round-robin pool. No configuration reload or restart is required at any point.

> **Single Point of Failure note:** Mac 2 nginx itself remains a SPOF. If `nginx` on Mac 2 crashes, all HTTPS access is interrupted. This is addressed by Extension E (standby edge) and can be fully automated with **VRRP/keepalived** (virtual IP failover) or an upstream **L4 load balancer** (e.g., HAProxy or a dedicated hardware appliance) that health-checks both nginx instances and routes traffic to whichever is alive.

---

## 7. Extension E — Standby Edge DNS Cutover

**Objective:** Demonstrate zero-downtime traffic migration from Mac 2 edge (`nginx` on `10.7.9.96:443`) to Mac 1 standby edge (`nginx` on `10.7.22.71:443`) by updating the DNS `host-record` and waiting one TTL cycle (30 seconds).

### Standby nginx Config on Mac 1 ([`config/nginx/nginx-standby-mac1.conf`](../config/nginx/nginx-standby-mac1.conf))

```nginx
upstream backends {
    server 127.0.0.1:3001 max_fails=2 fail_timeout=5s;
    server 10.7.9.96:3002 max_fails=2 fail_timeout=5s;
}
server {
    listen 443 ssl;
    server_name app.team1.test api.team1.test;
    ssl_certificate     /Users/abhisheksharma/certs/app.crt;
    ssl_certificate_key /Users/abhisheksharma/certs/app.key;
    ...
}
```

### Commands Executed

```bash
# Step 1: Verify standby nginx is listening on Mac 1
sudo lsof -i :443
# → nginx  ...  TCP 10.7.22.71:https (LISTEN)

# Step 2: Confirm current connection is to Mac 2
sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder
curl -v --cacert "$HOME/certs/ca.crt" https://app.team1.test/api/status 2>&1 | grep "Connected to"
# → Connected to app.team1.test (10.7.9.96) port 443

# Step 3: Flip DNS to point at Mac 1
sudo sed -i '' 's/host-record=app.team1.test,10.7.9.96,30/host-record=app.team1.test,10.7.22.71,30/' \
  $(brew --prefix)/etc/dnsmasq.conf
sudo brew services restart dnsmasq

# Step 4: Wait for 30s TTL to expire
sleep 35

# Step 5: Confirm new connection is to Mac 1
curl -v --cacert "$HOME/certs/ca.crt" https://app.team1.test/api/status 2>&1 | grep "Connected to"
# → Connected to app.team1.test (10.7.22.71) port 443

# Step 6: Confirm load balancing still works (Mac 1 standby upstream hits both backends)
for i in 1 2 3 4; do
  curl -sSI --cacert "$HOME/certs/ca.crt" https://app.team1.test/api/status | grep -iE "^HTTP|X-Backend:"
done
```

### Observed Output

```
# Before cutover:
Connected to app.team1.test (10.7.9.96) port 443

# After sleep 35 (TTL expired):
Connected to app.team1.test (10.7.22.71) port 443

# Load balancing via standby edge:
HTTP/1.1 200 OK
X-Backend: A
HTTP/1.1 200 OK
X-Backend: B
HTTP/1.1 200 OK
X-Backend: A
HTTP/1.1 200 OK
X-Backend: B
```

### Evidence

- [`evidence/phase2-edge-cutover/ext-e-dns-swap-curl-connected-96-to-71-sleep35-ttl-expiry-ab-lb.png`](../evidence/phase2-edge-cutover/ext-e-dns-swap-curl-connected-96-to-71-sleep35-ttl-expiry-ab-lb.png)

### Analysis

The 30-second TTL is what makes this migration safe and fast. During the 30-second window between DNS update and cache expiry, some clients still connect to `10.7.9.96` (Mac 2 edge — perfectly fine, it is still running). After TTL expiry all new connections go to `10.7.22.71` (Mac 1 standby edge). Both edges share the same TLS certificate (issued for `app.team1.test`), so clients never see a certificate error. The standby edge's upstream pool references its own Backend A via loopback and Backend B via LAN, maintaining full load balancing post-cutover.

---

## 8. Extension F — Layer-by-Layer Troubleshooting Methodology

When a request to `https://app.team1.test/api/status` fails or behaves unexpectedly, diagnose by traversing the network stack from bottom to top:

### Step 1 — Layer 3: IP Reachability (`ping`)

```bash
# From client, verify basic IP reachability to both nodes
ping -c 4 10.7.22.71   # Mac 1 (DNS, Backend A)
ping -c 4 10.7.9.96    # Mac 2 (Edge, Backend B)
```

**Diagnosis:**
- `0% packet loss` → Layer 3 is healthy; proceed up.
- `100% packet loss` → ARP/routing failure. Check Wi-Fi association, subnet (`ifconfig en0`), and gateway.

---

### Step 2 — Layer 7 (DNS): Name Resolution (`dig` / `dscacheutil`)

```bash
# Query primary DNS directly (bypass OS cache)
dig @10.7.22.71 app.team1.test
dig @10.7.9.96  app.team1.test

# Check OS resolver cache
dscacheutil -q host -a name app.team1.test

# Flush cache and re-resolve
sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder
dscacheutil -q host -a name app.team1.test
```

**Diagnosis:**
- `NOERROR` + correct IP → DNS is healthy.
- `SERVFAIL` or `NXDOMAIN` → `dnsmasq` is not running or misconfigured. Check `sudo brew services list | grep dnsmasq`.
- Stale IP in cache → flush cache or wait for TTL.
- Wrong DNS server in OS → `networksetup -getdnsservers Wi-Fi`; fix with `sudo networksetup -setdnsservers Wi-Fi 10.7.22.71`.

---

### Step 3 — Layer 4: TCP + Firewall (`nc` / `pfctl`)

```bash
# Test TCP connectivity to each service port
nc -zv 10.7.22.71 53    # DNS
nc -zv 10.7.9.96  443   # nginx HTTPS
nc -zv 10.7.22.71 3001  # Backend A
nc -zv 10.7.9.96  3002  # Backend B

# Check active pf firewall rules
sudo pfctl -sr

# Temporarily disable pf to isolate firewall issues
sudo pfctl -d
```

**Diagnosis:**
- `Connection refused` → service is not listening on that port. Start the service.
- `Connection timed out` → `pf` is blocking (`block drop`). Check `sudo pfctl -sr` and compare against [`config/firewall/`](../config/firewall/).
- `succeeded` → TCP layer is healthy; proceed up.

---

### Step 4 — Layer 6/7: TLS + HTTP (`curl -v`)

```bash
# Full verbose TLS + HTTP trace
curl -v --cacert ~/certs/ca.crt https://app.team1.test/api/status

# Check which IP the connection resolves to
curl -v --cacert ~/certs/ca.crt https://app.team1.test/api/status 2>&1 | grep "Connected to"

# Verify TLS certificate
openssl s_client -connect 10.7.9.96:443 -servername app.team1.test -CAfile ~/certs/ca.crt

# Check nginx is running and config is valid
sudo nginx -t
sudo lsof -i :443
```

**Diagnosis:**
- `SSL certificate problem: unable to get local issuer certificate` → CA not trusted. Add via `sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain ca.crt`.
- `Connection refused` on :443 → nginx is not running. `sudo brew services start nginx`.
- `502 Bad Gateway` → Both backends are unreachable. Start `node server.js A 3001` and `node server.js B 3002`.
- `200 OK` with correct `X-Backend` header → System fully operational.

---

## 9. Architectural Analysis — Single Points of Failure

| Component | SPOF? | Impact | Mitigation |
|-----------|-------|--------|------------|
| Primary DNS (`dnsmasq` Mac 1) | Partial | New resolutions fail | ✅ **Fixed** — Backup DNS on Mac 2 (Extension A) |
| Backend A (Mac 1 :3001) | Partial | Load halved | ✅ **Fixed** — nginx reroutes to B automatically (Extension D) |
| Backend B (Mac 2 :3002) | Partial | Load halved | ✅ **Fixed** — nginx reroutes to A automatically (Extension D) |
| Primary Edge nginx (Mac 2) | **Yes** | Full HTTPS outage | ⚠️ **Partial** — Standby nginx on Mac 1 (Extension E, manual DNS flip) |
| Wi-Fi LAN | Yes | Total isolation | Out of scope (physical infrastructure) |

### Eliminating the Edge SPOF

The standby `nginx` on Mac 1 (Extension E) requires a manual DNS flip — there is a ~30 second window of potential disruption. Two approaches fully automate this:

1. **VRRP / keepalived:** Assigns a virtual IP (VIP, e.g., `10.7.22.100`) shared between Mac 1 and Mac 2. The `MASTER` node owns the VIP; on failure, the `BACKUP` takes ownership within seconds (no DNS change needed). DNS always points to the VIP.

2. **Upstream L4 Load Balancer (HAProxy / hardware):** A third node runs HAProxy, health-checking both Mac 2 and Mac 1 nginx instances and distributing TCP :443 connections. Single DNS entry points to the HAProxy node.

---

## 10. Summary of All Phase 2 Commands

```bash
# --- Backup DNS (Extension A) ---
sudo brew services stop dnsmasq                              # Mac 1: kill primary
dig @10.7.9.96 app.team1.test +short                        # Mac 2: verify backup resolves
sudo brew services start dnsmasq                             # Mac 1: restore primary

# --- TTL Cache (Extension B) ---
dig @10.7.22.71 app.team1.test                               # Confirm TTL=30
sleep 35                                                     # Wait for cache expiry
sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder# Manual flush

# --- Firewall (Extension C) ---
sudo pfctl -f /tmp/pf_backend_a.conf && sudo pfctl -e        # Mac 1: apply pf
curl --max-time 3 -i http://10.7.9.96:3002/api/status        # Expect timeout (28)
sudo pfctl -f /etc/pf.conf.bak                               # Restore

# --- HA Failover (Extension D) ---
# Kill Backend A, then:
for i in 1 2 3 4 5 6; do curl -sSI --cacert ~/certs/ca.crt https://app.team1.test/api/status | grep X-Backend; done
# Restart Backend A, then:
for i in 1 2 3 4 5 6; do curl -sSI --cacert ~/certs/ca.crt https://app.team1.test/api/status | grep X-Backend; done

# --- Edge Cutover (Extension E) ---
sudo lsof -i :443                                            # Mac 1: confirm standby listening
sudo sed -i '' 's/host-record=app.team1.test,10.7.9.96,30/host-record=app.team1.test,10.7.22.71,30/' \
  $(brew --prefix)/etc/dnsmasq.conf
sudo brew services restart dnsmasq                           # Flip DNS
sleep 35                                                     # Wait TTL
curl -v --cacert ~/certs/ca.crt https://app.team1.test/api/status 2>&1 | grep "Connected to"

# --- Troubleshooting (Extension F) ---
ping -c 4 10.7.9.96                                         # L3
dig @10.7.22.71 app.team1.test                               # L7 DNS
nc -zv 10.7.9.96 443                                        # L4 TCP
curl -v --cacert ~/certs/ca.crt https://app.team1.test/api/status # L6/7 TLS+HTTP
```

