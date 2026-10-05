# Task C — Backend Servers

> Create two Node.js HTTP API backends for the upstream pool.

---

## Objective

Build two Express.js backend servers — Backend A on Mac 1 and Backend B on Mac 2 — that serve as the upstream pool for nginx load balancing.

---

## Implementation

### Backend A (Mac 1 — Abhishek)

```bash
mkdir backend-a && cd backend-a
npm init -y
npm install express
```

Start:
```bash
node server.js A 3001
```

Accessible at: `http://10.7.18.72:3001`

### Backend B (Mac 2 — Mohit)

```bash
mkdir backend-b && cd backend-b
npm init -y
npm install express
```

Start:
```bash
node server.js B 3002
```

Accessible at: `http://127.0.0.1:3002`

---

## API Endpoints

| Endpoint | Response | Special Headers |
|----------|----------|-----------------|
| `GET /api/status` | `{"service":"running","backend":"<ID>"}` | `X-Backend: <ID>` |
| `GET /api/cached` | `{"service":"running","backend":"<ID>"}` | `X-Backend: <ID>`, `Cache-Control: max-age=60`, `ETag` |

---

## Design Decisions

1. **Shared codebase**: Both backends use identical `server.js` — identity and port are CLI arguments
2. **X-Backend header**: Added to every response via Express middleware for load balancing verification
3. **ETag support**: Express automatically generates ETags for JSON responses
4. **Cache-Control**: The `/api/cached` endpoint explicitly sets `max-age=60`

---

## Verification

```bash
curl http://10.7.18.72:3001/api/status    # Backend A
curl http://127.0.0.1:3002/api/status     # Backend B
```

Both return HTTP 200 with their respective backend identity.

---

## Status: ✅ Complete
