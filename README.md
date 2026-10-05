# Sambhav — Private Network Service Platform

> **Course:** Computer Networks  
> **Phase:** Phase 1  
> **Team:** Sambhav  

---

## 👥 Team Members

| Name | Enrollment No. | Role | Machine | IP Address |
|------|----------------|------|---------|------------|
| Mohit Singh | 2401020098 | nginx HTTPS Edge / Reverse Proxy / Load Balancer, Backend B | Mac 2 | `10.7.10.115` |
| Abhishek | 2401020081 | Private DNS Server (dnsmasq), Backend A | Mac 1 | `10.7.18.72` |

---

## 📖 Project Overview

This project implements a **Private Network Service Platform** — a fully functional, enterprise-style microservice infrastructure running on two MacBooks connected over a shared Wi-Fi LAN.

The platform demonstrates core computer networking concepts end-to-end:

- **Private DNS** resolution using `dnsmasq`
- **TLS/HTTPS** termination with a self-signed local CA
- **Reverse proxying** via `nginx`
- **Round-robin load balancing** across two backend servers
- **HTTP caching** with `Cache-Control` and `ETag` headers
- **Failure resilience** and graceful degradation
- **Wireshark-verified** protocol behavior at every layer

No cloud services, no public DNS, no third-party infrastructure. Everything runs on the local network.

---

## 🌐 Network Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Wi-Fi LAN (Same Network)                     │
│                                                                     │
│  ┌──────────────────────────┐    ┌──────────────────────────────┐   │
│  │     Mac 1 — Abhishek     │    │       Mac 2 — Mohit          │   │
│  │      10.7.18.72          │    │       10.7.10.115             │   │
│  │                          │    │                               │   │
│  │  ┌────────────────────┐  │    │  ┌─────────────────────────┐ │   │
│  │  │   dnsmasq (DNS)    │  │    │  │   nginx (HTTPS :443)    │ │   │
│  │  │   :53              │  │    │  │   TLS Termination       │ │   │
│  │  │                    │  │    │  │   Reverse Proxy          │ │   │
│  │  │   app.team1.test   │──│────│──│   Load Balancer          │ │   │
│  │  │   → 10.7.10.115   │  │    │  │                           │ │   │
│  │  │                    │  │    │  │   ┌───────────────────┐   │ │   │
│  │  │   api.team1.test   │  │    │  │   │ Upstream Pool     │   │ │   │
│  │  │   → 10.7.10.115   │  │    │  │   │                   │   │ │   │
│  │  └────────────────────┘  │    │  │   │  10.7.18.72:3001  │   │ │   │
│  │                          │    │  │   │  (Backend A)       │   │ │   │
│  │  ┌────────────────────┐  │    │  │   │                   │   │ │   │
│  │  │  Backend A (Node)  │  │    │  │   │  127.0.0.1:3002   │   │ │   │
│  │  │  :3001             │  │    │  │   │  (Backend B)       │   │ │   │
│  │  │                    │  │    │  │   └───────────────────┘   │ │   │
│  │  │  /api/status       │  │    │  └─────────────────────────┘ │   │
│  │  │  /api/cached       │  │    │                               │   │
│  │  └────────────────────┘  │    │  ┌─────────────────────────┐ │   │
│  │                          │    │  │  Backend B (Node)       │ │   │
│  └──────────────────────────┘    │  │  :3002                  │ │   │
│                                  │  │                         │ │   │
│                                  │  │  /api/status            │ │   │
│                                  │  │  /api/cached            │ │   │
│                                  │  └─────────────────────────┘ │   │
│                                  └──────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔑 Key Domains

| Domain | Resolves To | Purpose |
|--------|-------------|---------|
| `app.team1.test` | `10.7.10.115` | Main application endpoint (via nginx) |
| `api.team1.test` | `10.7.10.115` | API endpoint (via nginx) |

> Both domains are **private** — they do not exist in public DNS. Querying `dig @8.8.8.8 app.team1.test` returns `NXDOMAIN`.

---

## 🧱 Technology Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| DNS | `dnsmasq` | Private domain resolution on LAN |
| TLS | OpenSSL + Local CA | Self-signed certificates for HTTPS |
| Edge / Proxy | `nginx 1.31.6` | TLS termination, reverse proxy, load balancer |
| Backend | Node.js + Express | HTTP API servers (Backend A & B) |
| Analysis | Wireshark | Packet capture and protocol verification |

---

## ✅ Verified Capabilities

| Capability | Status | Evidence |
|------------|--------|----------|
| Private DNS resolution | ✅ Verified | `dig @10.7.18.72 app.team1.test` → `10.7.10.115` |
| Public DNS isolation | ✅ Verified | `dig @8.8.8.8 app.team1.test` → `NXDOMAIN` |
| LAN connectivity (bidirectional) | ✅ Verified | 0% packet loss both directions |
| TLS 1.3 HTTPS | ✅ Verified | `TLSv1.3`, `SSL certificate verify ok` |
| Certificate chain | ✅ Verified | CN=app.team1.test, Issuer=Team1 Local CA |
| Round-robin load balancing | ✅ Verified | Alternating `X-Backend: A` / `X-Backend: B` |
| HTTP caching (Cache-Control) | ✅ Verified | `Cache-Control: max-age=60`, `ETag` present |
| Conditional requests (304) | ✅ Verified | `If-None-Match` → `304 Not Modified` |
| Single backend failure | ✅ Verified | Traffic continues via remaining backend |
| Total backend failure | ✅ Verified | nginx returns `502 Bad Gateway` |
| Recovery after failure | ✅ Verified | Load balancing resumes after restart |
| Wireshark DNS capture | ✅ Verified | Query/response packets captured |
| Wireshark TCP capture | ✅ Verified | Backend TCP/HTTP communication captured |
| Wireshark TLS capture | ✅ Verified | TLS handshake + encrypted Application Data |

---

## 📂 Repository Structure

```
sambhav-private-network-service-platform/
│
├── README.md                        ← You are here
├── .gitignore
├── LICENSE
│
├── backend/                         ← Backend A & B source code
│   ├── backend-a/
│   ├── backend-b/
│   └── README.md
│
├── config/                          ← Configuration files (examples)
│   ├── dnsmasq/
│   ├── nginx/
│   └── tls/
│
├── scripts/                         ← Step-by-step setup & test guides
│   ├── setup/
│   ├── backend/
│   ├── dns/
│   ├── tls/
│   ├── nginx/
│   ├── testing/
│   └── failure-demo/
│
├── tasks/                           ← Task-by-task implementation story
│   ├── task-a-network-identity.md
│   ├── task-b-private-dns.md
│   ├── task-c-backends.md
│   ├── task-d-tls.md
│   ├── task-e-nginx.md
│   ├── task-f-load-balancing.md
│   ├── task-g-caching.md
│   ├── task-h-wireshark.md
│   └── task-i-failure-demo.md
│
├── demos/                           ← Demo walkthroughs & expected output
│   ├── normal-operation.md
│   ├── load-balancing.md
│   ├── caching.md
│   ├── backend-a-failure.md
│   ├── both-backends-down.md
│   └── recovery.md
│
├── evidence/                        ← Wireshark captures & screenshots
│   ├── dns/
│   ├── tcp/
│   ├── tls/
│   ├── load-balancing/
│   ├── caching/
│   └── failure/
│
├── docs/                            ← Architecture, commands, report
│   ├── architecture.md
│   ├── network-topology.md
│   ├── complete-command-reference.md
│   ├── troubleshooting.md
│   └── testing-and-verification.md
│
└── forms/                           ← Evaluation form answers
    ├── A1-A5.md
    ├── B1-B3.md
    ├── C1-C3.md
    └── D1-D3.md
```

---

## 🚀 Quick Start (Documentation Only)

This repository is a **documentation and evidence archive**. To reproduce the project:

1. Read [`docs/architecture.md`](docs/architecture.md) for the full system design.
2. Follow [`scripts/setup/mac1-setup.md`](scripts/setup/mac1-setup.md) to configure Mac 1.
3. Follow [`scripts/setup/mac2-setup.md`](scripts/setup/mac2-setup.md) to configure Mac 2.
4. Start backends per [`scripts/backend/start-backends.md`](scripts/backend/start-backends.md).
5. Verify DNS per [`scripts/testing/dns-tests.md`](scripts/testing/dns-tests.md).
6. Verify HTTPS per [`scripts/testing/https-tests.md`](scripts/testing/https-tests.md).
7. Run failure demos per [`demos/backend-a-failure.md`](demos/backend-a-failure.md).

---

## 📜 License

This project is licensed under the MIT License. See [`LICENSE`](LICENSE) for details.

---

> **Sambhav** — Built from scratch on a local network. No cloud. No shortcuts. Every packet verified.
