# Complete Command Reference

This is a cheat sheet of all critical commands used to build, configure, and test the platform.

> **DO NOT blindly copy-paste these commands.** They are for reference only and assume specific paths and IP addresses.

## Network Discovery

```bash
# Find IP address on Wi-Fi interface
ifconfig en0 | grep "inet "

# Test LAN connectivity
ping -c 4 10.7.10.115
```

## Service Management (Homebrew)

```bash
# Start/Restart/Stop dnsmasq
sudo brew services start dnsmasq
sudo brew services restart dnsmasq

# Start/Reload/Stop nginx
sudo brew services start nginx
sudo nginx -s reload
sudo nginx -s stop
```

## DNS Testing

```bash
# Query specific DNS server
dig @10.7.18.72 app.team1.test

# Test public isolation
dig @8.8.8.8 app.team1.test

# Change macOS DNS server via CLI
sudo networksetup -setdnsservers Wi-Fi 10.7.18.72
```

## TLS & OpenSSL

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
```

## HTTP & Testing

```bash
# Basic HTTP GET
curl http://10.7.18.72:3001/api/status

# Verbose HTTPS GET with custom CA
curl -v https://app.team1.test/api/status --cacert ca.crt

# View specific headers
curl -s -D - https://app.team1.test/api/status | grep "X-Backend"

# Test Conditional Request (Caching)
curl -v https://app.team1.test/api/cached -H 'If-None-Match: W/"etag-value"'

# Load Balancing Loop
for i in {1..6}; do curl -s -D - https://app.team1.test/api/status | grep "X-Backend"; done
```

## Diagnostics

```bash
# Check if a port is in use
lsof -i :443
lsof -i :3001

# Test nginx configuration syntax
sudo nginx -t

# View active processes
ps aux | grep nginx
```
