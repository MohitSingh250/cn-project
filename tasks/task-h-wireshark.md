# Task H — Wireshark Analysis

> Capture and analyze network traffic to prove the system works exactly as designed at the packet level.

---

## Objective

Use Wireshark to verify:
1. DNS queries are sent to `10.7.18.72` and return `10.7.10.115`.
2. Backend communication happens over plaintext HTTP.
3. Client communication happens over encrypted TLS.

> **Note:** Actual `.pcap` files are excluded from this repo due to size, but the findings are documented below.

---

## Capture 1: DNS Resolution

**Filter:** `udp.port == 53`

**Observations:**
- **Source:** `10.7.10.115` (Mac 2)
- **Destination:** `10.7.18.72` (Mac 1)
- **Protocol:** DNS
- **Info (Query):** `Standard query 0x1234 A app.team1.test`
- **Info (Response):** `Standard query response 0x1234 A app.team1.test A 10.7.10.115`

**Proof:** Mac 2 successfully asked Mac 1's dnsmasq server to resolve the domain.

---

## Capture 2: Backend Communication (Mac 1 to Mac 2)

**Filter:** `tcp.port == 3001`

**Observations:**
- We see the standard TCP 3-way handshake (`SYN`, `SYN-ACK`, `ACK`) between `10.7.10.115` and `10.7.18.72:3001`.
- Following the handshake, we see an `HTTP GET /api/status` packet.
- The response from Backend A contains the JSON payload in **plaintext**, visible directly in the Wireshark packet bytes pane.

**Proof:** nginx proxies requests to Backend A over the LAN in plaintext HTTP.

---

## Capture 3: TLS Encryption (Client to nginx)

**Filter:** `tcp.port == 443`

**Observations:**
- `Client Hello` shows the client supporting TLS 1.3 and offering modern ciphers.
- `Server Hello` shows nginx agreeing to TLS 1.3.
- `Certificate` shows the server sending `app.team1.test` signed by `Team1 Local CA`.
- After the handshake, all data packets show as `Application Data`.
- The actual HTTP GET request and JSON response are **completely invisible/encrypted**.

**Proof:** nginx successfully terminates TLS, encrypting all client-facing traffic.

---

## Status: ✅ Complete
