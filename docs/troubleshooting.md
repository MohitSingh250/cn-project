# Troubleshooting Guide

Common issues encountered during the project and how to resolve them.

---

## 1. dnsmasq Fails to Start

**Error:** `dnsmasq: failed to create listening socket for port 53: Permission denied`

**Cause:** Port 53 is a privileged port. You cannot bind to it without root access.

**Solution:** Always use `sudo` when starting dnsmasq.
```bash
sudo brew services start dnsmasq
```

---

## 2. DNS Queries Return Nothing or Timeout

**Error:** `dig @10.7.18.72 app.team1.test` hangs.

**Cause 1:** Mac 1 firewall is blocking UDP port 53.
**Solution 1:** Temporarily disable the macOS firewall in System Preferences, or add an explicit allow rule for dnsmasq.

**Cause 2:** dnsmasq is only listening on `127.0.0.1`.
**Solution 2:** Ensure `listen-address=127.0.0.1,10.7.18.72` is in `dnsmasq.conf`.

---

## 3. curl SSL Certificate Verify Failed

**Error:** `curl: (60) SSL certificate problem: self signed certificate in certificate chain`

**Cause:** curl does not trust our local CA by default.

**Solution 1:** Pass the CA certificate directly to curl:
```bash
curl --cacert /path/to/ca.crt https://app.team1.test
```

**Solution 2:** Trust the CA system-wide in macOS Keychain Access.

---

## 4. curl No Alternative Certificate Subject Name

**Error:** `curl: (60) SSL: no alternative certificate subject name matches target host name 'app.team1.test'`

**Cause:** The server certificate was generated with only a Common Name (CN) and no Subject Alternative Name (SAN). Modern TLS requires SAN.

**Solution:** Regenerate the server CSR and CRT using the `san.cnf` file as documented in the TLS setup guide.

---

## 5. nginx 502 Bad Gateway

**Error:** Client receives HTTP 502.

**Cause 1:** Both Backend A and Backend B are down.
**Solution 1:** Check if `node server.js` is running on both machines.

**Cause 2:** nginx cannot reach Backend A across the LAN.
**Solution 2:** Ensure Mac 1's firewall allows incoming TCP on port 3001. Test with `ping` and `curl http://10.7.18.72:3001/api/status` from Mac 2.

---

## 6. nginx Address Already in Use

**Error:** `nginx: [emerg] bind() to 0.0.0.0:443 failed (48: Address already in use)`

**Cause:** Another process (or another instance of nginx) is already listening on port 443.

**Solution:** Find the process and kill it.
```bash
sudo lsof -i :443
sudo kill <PID>
sudo nginx
```
