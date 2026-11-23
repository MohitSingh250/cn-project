# System Architecture

The Private Network Service Platform is designed as a miniature representation of a modern enterprise microservices architecture. It demonstrates how various network layers interact to provide a reliable, secure, and performant service.

> **Phase 2 Update:** IPs updated to live Phase 2 LAN addresses. Mac 1 is now `10.7.22.71`, Mac 2 is now `10.7.9.96`. Phase 2 added backup DNS, 30s TTL, `pf` firewall, HA failover, and standby edge.

## Architectural Components

The architecture is composed of five main layers (Phase 1 + Phase 2):

1. **DNS Layer — Primary (dnsmasq on Mac 1 `10.7.22.71`)**
   - Acts as the address book for the private network.
   - Translates human-readable domains (`app.team1.test`, `api.team1.test`) into IP addresses (`10.7.9.96`).
   - `local-ttl=30` and `host-record` TTL=30 enable rapid DNS-based failover (Phase 2).
   - Upstream forwarders: `10.5.7.1` (campus) and `8.8.8.8`.

2. **DNS Layer — Backup (dnsmasq on Mac 2 `10.7.9.96`) [Phase 2]**
   - Identical zone data to Primary DNS.
   - Serves as hot standby: if Mac 1 `dnsmasq` stops, clients using Mac 2's resolver continue resolving without interruption (Extension A).

3. **Firewall Layer (`pf` on Mac 1 and Mac 2) [Phase 2]**
   - `pf_backend_a.conf`: blocks direct TCP access to port 3001 from arbitrary clients; allows loopback and nginx IP (`10.7.9.96`) only.
   - `pf_backend_b.conf`: blocks direct TCP access to port 3002; allows loopback only.
   - Forces all application traffic through the nginx reverse proxy.

4. **Edge / Reverse Proxy Layer**
   - **Primary (nginx on Mac 2 `10.7.9.96:443`)**: TLS termination, reverse proxy, HA round-robin load balancer with `max_fails=2 fail_timeout=5s` and `proxy_next_upstream`.
   - **Standby (nginx on Mac 1 `10.7.22.71:443`) [Phase 2 Extension E]**: mirrors primary config; activated by flipping DNS `host-record` from `10.7.9.96` to `10.7.22.71`; full load balancing resumes within one 30s TTL cycle.

5. **Application / Backend Layer (Node.js on Mac 1 and Mac 2)**
   - Backend A: `10.7.22.71:3001` — `X-Backend: A`
   - Backend B: `127.0.0.1:3002` (Mac 2 loopback) — `X-Backend: B`
   - Both run Node.js + Express with `/api/status` and `/api/cached` endpoints.

## Request Flow (Phase 2 Normal Operation)

When a client requests `https://app.team1.test/api/status`:

1. **DNS Resolution (UDP: 53)**: The client asks `10.7.22.71:53` for `app.team1.test`. Primary `dnsmasq` responds with `10.7.9.96`, TTL=30. If Mac 1 is down, Mac 2 backup `dnsmasq` answers identically.
2. **TCP Handshake (TCP: 443)**: The client initiates a TCP connection to `10.7.9.96` on port 443 (primary edge) — or `10.7.22.71:443` after a DNS cutover (Extension E).
3. **TLS Handshake (TLSv1.3)**: The client and nginx negotiate encryption. nginx presents the `app.team1.test` certificate.
4. **HTTP Request (L7)**: The client sends the encrypted HTTP GET request.
5. **Firewall check**: `pf` validates that the nginx IP (`10.7.9.96`) is whitelisted for port 3001.
6. **Reverse Proxy (TCP: 3001/3002)**: nginx decrypts and round-robins to `10.7.22.71:3001` (Backend A) or `127.0.0.1:3002` (Backend B). If Backend A fails, `proxy_next_upstream` routes transparently to B within the same request.
7. **Application Logic (L7)**: The backend processes the request and returns JSON with `X-Backend: A` or `X-Backend: B`.
8. **Response Path**: The response travels back through nginx (re-encrypted) to the client.

## Design Philosophy

- **Decentralization**: DNS server and Backend A run on Mac 1; Edge, Backup DNS, and Backend B run on Mac 2. Truly network-distributed.
- **Defense in Depth**: `pf` firewall ensures backends are only reachable through the nginx proxy layer, preventing direct bypassing of TLS and authentication.
- **Resilience**: The system survives the failure of any single backend without client-visible errors (Extension D). DNS-based edge migration (Extension E) enables planned maintenance without downtime.
- **Short TTL for agility**: 30-second TTL balances cache efficiency against failover speed. Manual `dscacheutil -flushcache` enables instant migration in emergencies.
