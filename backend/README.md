# Backend Services

This directory contains the two backend API servers that form the upstream pool for the nginx load balancer.

---

## Architecture

```
nginx (10.7.10.115:443)
    │
    ├── round-robin ──→ Backend A (10.7.18.72:3001)   [Mac 1 — Abhishek]
    │
    └── round-robin ──→ Backend B (127.0.0.1:3002)     [Mac 2 — Mohit]
```

---

## Backend Comparison

| Property | Backend A | Backend B |
|----------|-----------|-----------|
| Machine | Mac 1 — Abhishek | Mac 2 — Mohit |
| IP | `10.7.18.72` | `127.0.0.1` |
| Port | `3001` | `3002` |
| Identity | `A` | `B` |
| Network path from nginx | Cross-LAN | Loopback |
| Technology | Node.js + Express | Node.js + Express |

---

## Endpoints (both backends)

| Method | Path | Response | Headers |
|--------|------|----------|---------|
| GET | `/api/status` | `{"service":"running","backend":"<ID>"}` | `X-Backend: <ID>` |
| GET | `/api/cached` | `{"service":"running","backend":"<ID>"}` | `X-Backend: <ID>`, `Cache-Control: max-age=60`, `ETag` |

---

## Why Two Backends?

1. **Load Balancing**: nginx distributes requests evenly across both backends using round-robin.
2. **Fault Tolerance**: If one backend fails, nginx routes all traffic to the surviving backend.
3. **Cross-Machine Networking**: Backend A runs on a physically separate machine, proving real LAN-based service distribution.
4. **Protocol Diversity**: Backend A is accessed cross-LAN (TCP over Wi-Fi), Backend B over loopback.

---

## Starting the Backends

**On Mac 1 (Backend A):**
```bash
cd backend/backend-a
npm install
node server.js A 3001
```

**On Mac 2 (Backend B):**
```bash
cd backend/backend-b
npm install
node server.js B 3002
```

---

## Subdirectories

- [`backend-a/`](backend-a/) — Backend A source and documentation
- [`backend-b/`](backend-b/) — Backend B source and documentation
