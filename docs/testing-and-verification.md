# Testing and Verification Strategy

This document outlines the systematic approach used to verify that the Private Network Service Platform functions correctly at every OSI layer.

## Verification Hierarchy

Our testing strategy follows a bottom-up approach, ensuring the foundational network layers work before testing application features.

### Phase 1: Physical & Data Link (L1/L2)
**Goal:** Prove both Macs can communicate over the Wi-Fi LAN.
- **Method:** `ping 10.7.18.72` and `ping 10.7.10.115`
- **Result:** 0% packet loss. Basic connectivity established.

### Phase 2: Network & Transport (L3/L4)
**Goal:** Prove services are listening on correct TCP/UDP ports and accessible.
- **Method:** `lsof -i :53`, `lsof -i :443`, `lsof -i :3001`
- **Result:** Processes bound to correct ports. Firewall allows traffic.

### Phase 3: Application - DNS (L7)
**Goal:** Prove `dnsmasq` correctly resolves private domains.
- **Method:** `dig @10.7.18.72 app.team1.test`
- **Result:** Correctly returns `10.7.10.115`. Public DNS isolation verified via NXDOMAIN on `8.8.8.8`.

### Phase 4: Application - Backends (L7)
**Goal:** Prove Node.js servers process HTTP requests and return valid JSON.
- **Method:** `curl http://10.7.18.72:3001/api/status`
- **Result:** Returns `{"service":"running","backend":"A"}` and `X-Backend` header.

### Phase 5: Presentation - TLS (L6)
**Goal:** Prove nginx terminates TLS with a valid certificate.
- **Method:** `curl -v https://app.team1.test --cacert ca.crt`
- **Result:** TLSv1.3 handshake succeeds, certificate SAN matches.

### Phase 6: System Integration (Load Balancing & Failure)
**Goal:** Prove nginx distributes load and recovers from backend failure.
- **Method:** Scripted `curl` loops and manual process termination.
- **Result:** Round-robin verified. Graceful degradation verified.

## The Role of Wireshark

While CLI tools (`ping`, `dig`, `curl`) prove the *outcome* is correct, Wireshark proves the *mechanism* is correct.

We used Wireshark to:
1. See the actual UDP datagrams containing DNS queries.
2. See the TCP 3-way handshake before HTTP requests.
3. Prove that client-to-nginx traffic is encrypted (unreadable Application Data).
4. Prove that nginx-to-backend traffic is unencrypted (readable HTTP text).

By combining CLI functional testing with Wireshark packet inspection, we achieved 100% verification of the project requirements.
