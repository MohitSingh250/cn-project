# Complete Command Reference

This is a cheat sheet of all critical commands used to build, configure, and test the platform.

> **Phase 2 Update:** All IP addresses reflect live Phase 2 LAN addresses: Mac 1 = `10.7.22.71`, Mac 2 = `10.7.9.96`.
> **DO NOT blindly copy-paste these commands.** They are for reference only and assume specific paths and IP addresses.

---

## Phase 1 Commands

### Network Discovery

```bash
# Find IP address on Wi-Fi interface
ifconfig en0 | grep "inet "

# Test LAN connectivity
ping -c 4 10.7.9.96    # Mac 2 (Phase 2)
ping -c 4 10.7.22.71   # Mac 1 (Phase 2)
```

### Service Management (Homebrew)

```bash
# Start/Restart/Stop dnsmasq
sudo brew services start dnsmasq
sudo brew services restart dnsmasq
sudo brew services stop dnsmasq

# Start/Reload/Stop nginx
sudo brew services start nginx
sudo nginx -s reload
sudo nginx -s stop
```

### DNS Testing

```bash
# Query primary DNS server (Phase 2 IP)
dig @10.7.22.71 app.team1.test

# Query backup DNS server (Phase 2 — Extension A)
dig @10.7.9.96 app.team1.test

# Test public isolation
dig @8.8.8.8 app.team1.test

# Change macOS DNS server via CLI
sudo networksetup -setdnsservers Wi-Fi 10.7.22.71

# Check current DNS servers
networksetup -getdnsservers Wi-Fi
```

### TLS & OpenSSL

```bash
# Generate private key
openssl genrsa -out key.pem 2048

# Create self-signed Root CA
openssl req -x509 -new -nodes -key ca.key -sha256 -days 365 -out ca.crt

# Create CSR (Certificate Signing Request)
openssl req -new -key server.key -out server.csr -config san.cnf

# Sign Certificate with CA
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 365 -sha256 -extensions v3_req -extfile san.cnf

# View Certificate Contents
openssl x509 -in server.crt -text -noout

# Verify Certificate Chain
openssl verify -CAfile ca.crt server.crt

# Trust CA in macOS Keychain
sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain ca.crt

# Inspect live TLS certificate
openssl s_client -connect 10.7.9.96:443 -servername app.team1.test -CAfile ~/certs/ca.crt
```

### HTTP & Testing

```bash
# Basic HTTP GET to Backend A (Phase 2 IP)
curl http://10.7.22.71:3001/api/status

# Basic HTTP GET to Backend B (Phase 2 IP)
curl http://10.7.9.96:3002/api/status

# Verbose HTTPS GET with custom CA
curl -v https://app.team1.test/api/status --cacert ~/certs/ca.crt

# View specific headers
curl -s -D - https://app.team1.test/api/status | grep "X-Backend"

# Test Conditional Request (Caching)
curl -v https://app.team1.test/api/cached -H 'If-None-Match: W/"etag-value"'

# Load Balancing Loop (6 requests)
for i in 1 2 3 4 5 6; do curl -sSI --cacert ~/certs/ca.crt https://app.team1.test/api/status | grep -iE "^HTTP|X-Backend:"; done
```

### Diagnostics

```bash
# Check if a port is in use
lsof -i :443
lsof -i :3001
lsof -i :53

# Test nginx configuration syntax
sudo nginx -t

# View active processes
ps aux | grep nginx
ps aux | grep node
ps aux | grep dnsmasq
```

---

## Phase 2 Commands

### Extension A — Backup DNS

```bash
# Stop primary DNS on Mac 1
sudo brew services stop dnsmasq

# From Mac 2: verify backup DNS resolves
dscacheutil -flushcache; sudo killall -HUP mDNSResponder
dig @10.7.9.96 app.team1.test +short
curl -sSI --cacert ~/certs/ca.crt https://app.team1.test/api/status | grep -iE "^HTTP|X-Backend:"

# Restart primary DNS on Mac 1
sudo brew services start dnsmasq
```

### Extension B — TTL Cache Control

```bash
# Verify 30s TTL in dig answer
dig @10.7.22.71 app.team1.test
# Expected: app.team1.test.  30  IN  A  10.7.9.96

# Flush macOS DNS cache manually (instant update)
sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder

# Check resolver cache
dscacheutil -q host -a name app.team1.test

# Wait for TTL expiry (auto-update)
sleep 35
dscacheutil -q host -a name app.team1.test

# Update host-record in dnsmasq (example: change to Mac 1 IP for Extension E)
sudo sed -i '' 's/host-record=app.team1.test,10.7.9.96,30/host-record=app.team1.test,10.7.22.71,30/' \
  $(brew --prefix)/etc/dnsmasq.conf
sudo brew services restart dnsmasq
```

### Extension C — `pf` Firewall

```bash
# Backup existing pf config
sudo cp /etc/pf.conf /etc/pf.conf.bak

# Apply Backend A firewall (Mac 1)
sudo pfctl -f /tmp/pf_backend_a.conf
sudo pfctl -e

# Apply Backend B firewall (Mac 2)
sudo pfctl -f /tmp/pf_backend_b.conf
sudo pfctl -e

# View active firewall rules
sudo pfctl -sr

# Test that direct access is blocked (expect exit 28 / timeout)
curl --max-time 3 -i http://10.7.9.96:3002/api/status

# Test that HTTPS via nginx still works
curl -sSI --cacert ~/certs/ca.crt https://app.team1.test/api/status

# Restore original firewall
sudo pfctl -f /etc/pf.conf.bak

# Disable pf entirely (for testing isolation)
sudo pfctl -d
```

### Extension D — HA Failover & Recovery

```bash
# Send 6 requests while Backend A is down (all should return X-Backend: B)
for i in 1 2 3 4 5 6; do
  curl -sSI --cacert ~/certs/ca.crt https://app.team1.test/api/status | grep -iE "^HTTP|X-Backend:"
done

# After restarting Backend A, send 6 requests (should alternate A/B)
for i in 1 2 3 4 5 6; do
  curl -sSI --cacert ~/certs/ca.crt https://app.team1.test/api/status | grep -iE "^HTTP|X-Backend:"
done
```

### Extension E — Standby Edge DNS Cutover

```bash
# Verify standby nginx is listening on Mac 1
sudo lsof -i :443   # Should show nginx bound to 10.7.22.71:443

# Confirm current connection goes to Mac 2
curl -v --cacert ~/certs/ca.crt https://app.team1.test/api/status 2>&1 | grep "Connected to"
# → Connected to app.team1.test (10.7.9.96) port 443

# Flip DNS to point at Mac 1 standby nginx
sudo sed -i '' 's/host-record=app.team1.test,10.7.9.96,30/host-record=app.team1.test,10.7.22.71,30/' \
  $(brew --prefix)/etc/dnsmasq.conf
sudo brew services restart dnsmasq

# Wait for 30s TTL expiry
sleep 35

# Confirm new connection goes to Mac 1
curl -v --cacert ~/certs/ca.crt https://app.team1.test/api/status 2>&1 | grep "Connected to"
# → Connected to app.team1.test (10.7.22.71) port 443

# Verify load balancing still works via standby edge
for i in 1 2 3 4; do
  curl -sSI --cacert ~/certs/ca.crt https://app.team1.test/api/status | grep -iE "^HTTP|X-Backend:"
done
```

### Extension F — Layer-by-Layer Troubleshooting

```bash
# Step 1 — L3: Ping both nodes
ping -c 4 10.7.22.71
ping -c 4 10.7.9.96

# Step 2 — L7 DNS: Verify resolution
dig @10.7.22.71 app.team1.test
dig @10.7.9.96  app.team1.test
dscacheutil -q host -a name app.team1.test
sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder

# Step 3 — L4 TCP/Firewall: Test port connectivity
nc -zv 10.7.22.71 53
nc -zv 10.7.9.96  443
nc -zv 10.7.22.71 3001
nc -zv 10.7.9.96  3002
sudo pfctl -sr   # View active firewall rules

# Step 4 — L6/7 TLS+HTTP: Full request trace
curl -v --cacert ~/certs/ca.crt https://app.team1.test/api/status
openssl s_client -connect 10.7.9.96:443 -servername app.team1.test -CAfile ~/certs/ca.crt
sudo nginx -t
```
