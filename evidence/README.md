# Project Evidence

This directory contains the required evidence demonstrating that the network behaved exactly as specified in the project requirements.

> **Note:** The actual `.pcap` files and image screenshots are omitted from this Git repository to keep the repository lightweight. Below is a description of what was captured and submitted for the project evaluation.

## Directory Structure

### `dns/`
Contains Wireshark captures of DNS traffic over UDP port 53.
- Shows Mac 2 sending a DNS query for `app.team1.test` to Mac 1.
- Shows Mac 1 (`dnsmasq`) responding with the IP `10.7.10.115`.

### `tcp/`
Contains Wireshark captures of backend TCP traffic over port 3001.
- Shows the TCP 3-way handshake (`SYN`, `SYN-ACK`, `ACK`).
- Shows the unencrypted HTTP request and response between nginx and Backend A.

### `tls/`
Contains Wireshark captures of encrypted HTTPS traffic over port 443.
- Shows the TLS 1.3 handshake (`Client Hello`, `Server Hello`, `Certificate`).
- Shows encrypted `Application Data` payloads, proving that client communication is secure.

### `load-balancing/`
Contains terminal screenshots of the `curl` loop demonstrating round-robin load balancing.
- Shows alternating `X-Backend: A` and `X-Backend: B` headers.

### `caching/`
Contains terminal screenshots of `curl` requests with caching headers.
- Shows the initial HTTP 200 response with `Cache-Control: max-age=60` and `ETag`.
- Shows the subsequent conditional request (`If-None-Match`) returning HTTP 304 Not Modified.

### `failure/`
Contains terminal screenshots demonstrating failure and recovery scenarios.
- Shows requests succeeding with `X-Backend: B` after Backend A is stopped.
- Shows nginx returning `502 Bad Gateway` when both backends are down.
- Shows load balancing resuming automatically upon backend recovery.
