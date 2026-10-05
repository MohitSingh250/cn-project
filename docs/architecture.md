# System Architecture

The Private Network Service Platform is designed as a miniature representation of a modern enterprise microservices architecture. It demonstrates how various network layers interact to provide a reliable, secure, and performant service.

## Architectural Components

The architecture is composed of four main layers:

1. **DNS Layer (dnsmasq on Mac 1)**
   - Acts as the address book for the private network.
   - Translates human-readable domains (`app.team1.test`) into IP addresses (`10.7.10.115`).
   - Ensures the service remains entirely private and unreachable from the public internet.

2. **Edge / Reverse Proxy Layer (nginx on Mac 2)**
   - Serves as the single entry point for all client requests.
   - Provides **TLS Termination**, decrypting incoming HTTPS traffic so backends can communicate in plain HTTP.
   - Shields the internal network topology from clients.

3. **Load Balancing Layer (nginx on Mac 2)**
   - Distributes incoming traffic across multiple backend servers using a round-robin algorithm.
   - Monitors backend health and automatically routes around failures.

4. **Application / Backend Layer (Node.js on Mac 1 and Mac 2)**
   - The actual service logic.
   - Two instances (Backend A and Backend B) run on separate network nodes to provide redundancy and prove cross-machine communication.

## Request Flow

When a client requests `https://app.team1.test/api/status`:

1. **DNS Resolution (UDP: 53)**: The client asks the DNS server (`10.7.18.72`) for the IP of `app.team1.test`. The DNS server responds with `10.7.10.115`.
2. **TCP Handshake (TCP: 443)**: The client initiates a TCP connection to `10.7.10.115` on port 443.
3. **TLS Handshake (TLSv1.3)**: The client and nginx negotiate encryption. nginx presents the server certificate.
4. **HTTP Request (L7)**: The client sends the encrypted HTTP GET request.
5. **Reverse Proxy (TCP: 3001/3002)**: nginx decrypts the request and forwards it in plaintext to the next available backend in the upstream pool (e.g., `10.7.18.72:3001`).
6. **Application Logic (L7)**: The backend processes the request and generates a JSON response.
7. **Response Path**: The response travels back through nginx, gets encrypted, and is delivered to the client.

## Design Philosophy

- **Decentralization**: The DNS server and Backend A run on one machine, while the Edge server and Backend B run on another. This proves the system is truly network-distributed.
- **Security at the Edge**: Only nginx handles TLS certificates. The internal network (between nginx and backends) is assumed secure, reducing processing overhead on the backends.
- **Resilience**: The system is designed to survive the failure of any single backend component without client-facing errors.
