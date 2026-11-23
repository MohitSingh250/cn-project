# Sambhav — Private Network Service Platform

> **Course:** Computer Networks
> **Phases:** Phase 1 ✅ + Phase 2 ✅
> **Team:** Sambhav

---

## 👥 Team Members

| Name | Enrollment No. | Role | Machine | IP Address (Phase 2) |
|------|----------------|------|---------|----------------------|
| Abhishek Sharma | 2401020081 | Primary DNS (`dnsmasq` :53), Backend A (:3001), Standby Edge (`nginx` :443) | Mac 1 | `10.7.22.71` |
| Mohit Singh | 2401020098 | Primary Edge (`nginx` :443), Backup DNS (`dnsmasq` :53), Backend B (:3002) | Mac 2 | `10.7.9.96` |

> **Phase 1 IPs** (historical reference): Mac 1 was `10.7.18.72`, Mac 2 was `10.7.10.115`. Both machines received new DHCP leases for Phase 2.

---

## 📖 Project Overview

This project implements a **Private Network Service Platform** — a fully functional, enterprise-style microservice infrastructure running on two MacBooks connected over a shared Wi-Fi LAN.

**Phase 1** established the core stack. **Phase 2** hardens it with:
- Backup DNS (Extension A)
- 30-second TTL cache control (Extension B)
- `pf` firewall enforcement (Extension C)
- HA load-balancer failover + recovery (Extension D)
- Standby edge (`nginx`) cutover with DNS-based migration (Extension E)
- Layer-by-layer troubleshooting methodology (Extension F)

No cloud services, no public DNS, no third-party infrastructure. Everything runs on the local network.

---

## 🌐 Network Architecture (Phase 1 + Phase 2)

```
┌───────────────────────────────────────────────────────────────────────┐
│                         Wi-Fi LAN (Same Network)                      │
│                      Subnet: 10.7.16.0/21                             │
│                                                                       │
│  ┌────────────────────────────┐   ┌──────────────────────────────┐    │
│  │      Mac 1 — Abhishek      │   │       Mac 2 — Mohit           │    │
│  │       10.7.22.71           │   │       10.7.9.96              │    │
│  │                            │   │                               │    │
│  │  ┌──────────────────────┐  │   │  ┌─────────────────────────┐ │    │
│  │  │  dnsmasq (PRIMARY)   │  │   │  │   nginx (PRIMARY EDGE)  │ │    │
│  │  │  UDP :53             │  │   │  │   TCP :443 TLS          │ │    │
│  │  │  local-ttl=30        │  │   │  │   Reverse Proxy + LB   │ │    │
│  │  │  app.team1.test      │──│───│──│►  max_fails=2 5s        │ │    │
│  │  │   → 10.7.9.96        │  │   │  │                         │ │    │
│  │  │  api.team1.test      │  │   │  │  Upstream Pool:         │ │    │
│  │  │   → 10.7.9.96        │  │   │  │  10.7.22.71:3001 (A)   │ │    │
│  │  └──────────────────────┘  │   │  │  127.0.0.1:3002  (B)   │ │    │
│  │                            │   │  └─────────────────────────┘ │    │
│  │  ┌──────────────────────┐  │   │                               │    │
│  │  │  Backend A (Node.js) │  │   │  ┌─────────────────────────┐ │    │
│  │  │  TCP :3001           │  │   │  │  dnsmasq (BACKUP DNS)   │ │    │
│  │  │  X-Backend: A        │  │   │  │  UDP :53                │ │    │
│  │  └──────────────────────┘  │   │  │  local-ttl=30           │ │    │
│  │                            │   │  └─────────────────────────┘ │    │
│  │  ┌──────────────────────┐  │   │                               │    │
│  │  │  nginx (STANDBY)     │  │   │  ┌─────────────────────────┐ │    │
│  │  │  TCP :443 (Phase 2E) │  │   │  │  Backend B (Node.js)    │ │    │
│  │  │  Upstream:           │  │   │  │  TCP :3002              │ │    │
│  │  │  127.0.0.1:3001      │  │   │  │  X-Backend: B          │ │    │
│  │  │  10.7.9.96:3002      │  │   │  └─────────────────────────┘ │    │
│  │  └──────────────────────┘  │   │                               │    │
│  └────────────────────────────┘   └──────────────────────────────┘    │
│                                                                       │
│  Upstream DNS Forwarders:  10.5.7.1 (campus)  |  8.8.8.8 (Google)    │
│  Private Domains:          app.team1.test  |  api.team1.test          │
└───────────────────────────────────────────────────────────────────────┘
```

---

## 🧱 Technology Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| DNS | `dnsmasq` | Private domain resolution on LAN + backup DNS |
| Firewall | `pf` (macOS packet filter) | Block direct backend access, force proxy path |
| TLS | OpenSSL + Local CA | Self-signed certificates for HTTPS |
| Edge / Proxy | `nginx` | TLS termination, reverse proxy, HA load balancer |
| Backend | Node.js + Express | HTTP API servers (Backend A & B) |
| Analysis | Wireshark | Packet capture and protocol verification |

---

## ✅ Verified Capabilities

### Phase 1
| Capability | Status | Evidence |
|------------|--------|----------|
| Private DNS resolution | ✅ | `dig @10.7.22.71 app.team1.test` → `10.7.9.96` |
| Public DNS isolation | ✅ | `dig @8.8.8.8 app.team1.test` → `NXDOMAIN` |
| LAN connectivity (bidirectional) | ✅ | 0% packet loss both directions |
| TLS 1.3 HTTPS | ✅ | `TLSv1.3`, `SSL certificate verify ok` |
| Round-robin load balancing | ✅ | Alternating `X-Backend: A` / `X-Backend: B` |
| HTTP caching (Cache-Control) | ✅ | `Cache-Control: max-age=60`, `ETag` present |
| Single backend failure + recovery | ✅ | Traffic re-routed, then balanced on restart |

### Phase 2
| Extension | Capability | Status | Evidence |
|-----------|------------|--------|----------|
| A | Backup DNS on Mac 2 | ✅ | [`evidence/phase2-backup-dns/`](evidence/phase2-backup-dns/) |
| B | 30s TTL cache & flush | ✅ | [`evidence/phase2-ttl-cache/`](evidence/phase2-ttl-cache/) |
| C | `pf` firewall enforcement | ✅ | [`evidence/phase2-firewall/`](evidence/phase2-firewall/) |
| D | HA failover + recovery | ✅ | [`evidence/phase2-ha-failover/`](evidence/phase2-ha-failover/) |
| E | Standby edge DNS cutover | ✅ | [`evidence/phase2-edge-cutover/`](evidence/phase2-edge-cutover/) |
| F | Layer-by-layer troubleshooting | ✅ | [`docs/phase2-report.md`](docs/phase2-report.md) |

---

## 📂 Repository Structure

```
sambhav-private-network-service-platform/
│
├── README.md                        ← You are here
├── .gitignore
├── LICENSE
│
├── backend/                         ← Backend A & B source code (Phase 1 ✅)
│   ├── backend-a/
│   ├── backend-b/
│   └── README.md
│
├── config/                          ← Configuration files
│   ├── dnsmasq/
│   │   ├── dnsmasq-mac1.conf        ← Phase 2: Primary DNS (Mac 1 10.7.22.71)
│   │   ├── dnsmasq-mac2.conf        ← Phase 2: Backup DNS (Mac 2 10.7.9.96)
│   │   └── dnsmasq.conf.example     ← Phase 1 example
│   ├── nginx/
│   │   ├── nginx-primary-mac2.conf  ← Phase 2: Primary edge (Mac 2)
│   │   ├── nginx-standby-mac1.conf  ← Phase 2: Standby edge (Mac 1 Extension E)
│   │   └── nginx.conf.example       ← Phase 1 example
│   ├── firewall/
│   │   ├── pf_backend_a.conf        ← Phase 2: pf rules for Mac 1 (Extension C)
│   │   └── pf_backend_b.conf        ← Phase 2: pf rules for Mac 2 (Extension C)
│   └── tls/
│
├── evidence/                        ← Classified screenshots per extension
│   ├── phase2-backup-dns/           ← Extension A screenshots
│   ├── phase2-ttl-cache/            ← Extension B screenshots
│   ├── phase2-firewall/             ← Extension C screenshots
│   ├── phase2-ha-failover/          ← Extension D screenshots
│   ├── phase2-edge-cutover/         ← Extension E screenshots
│   └── README.md
│
├── docs/                            ← Architecture, commands, reports
│   ├── architecture.md              ← Updated for Phase 2
│   ├── network-topology.md          ← Updated for Phase 2
│   ├── complete-command-reference.md← Updated for Phase 2
│   ├── phase2-report.md             ← NEW: Phase 2 Final Report
│   ├── troubleshooting.md
│   └── testing-and-verification.md
│
├── scripts/                         ← Step-by-step setup & test guides
├── tasks/                           ← Task-by-task implementation story
├── demos/                           ← Demo walkthroughs & expected output
└── forms/                           ← Evaluation form answers
```

---

## 🚀 Quick Start

This repository is a **documentation and evidence archive**.

### Phase 2 Setup
1. Apply DNS configs: [`config/dnsmasq/dnsmasq-mac1.conf`](config/dnsmasq/dnsmasq-mac1.conf) and [`config/dnsmasq/dnsmasq-mac2.conf`](config/dnsmasq/dnsmasq-mac2.conf)
2. Apply nginx configs: [`config/nginx/nginx-primary-mac2.conf`](config/nginx/nginx-primary-mac2.conf) and [`config/nginx/nginx-standby-mac1.conf`](config/nginx/nginx-standby-mac1.conf)
3. Apply firewall rules: [`config/firewall/pf_backend_a.conf`](config/firewall/pf_backend_a.conf) and [`config/firewall/pf_backend_b.conf`](config/firewall/pf_backend_b.conf)
4. Read the full Phase 2 report: [`docs/phase2-report.md`](docs/phase2-report.md)

### Phase 1 Setup (reference)
1. Read [`docs/architecture.md`](docs/architecture.md) for the full system design.
2. Follow [`scripts/setup/mac1-setup.md`](scripts/setup/mac1-setup.md) to configure Mac 1.
3. Follow [`scripts/setup/mac2-setup.md`](scripts/setup/mac2-setup.md) to configure Mac 2.
4. Start backends per [`scripts/backend/start-backends.md`](scripts/backend/start-backends.md).
5. Verify DNS per [`scripts/testing/dns-tests.md`](scripts/testing/dns-tests.md).
6. Verify HTTPS per [`scripts/testing/https-tests.md`](scripts/testing/https-tests.md).

---

## 📜 License

This project is licensed under the MIT License. See [`LICENSE`](LICENSE) for details.

---

> **Sambhav** — Built from scratch on a local network. No cloud. No shortcuts. Every packet verified.
