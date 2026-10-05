# Backend B

> **Machine:** Mac 2 — Mohit (`10.7.10.115`)  
> **Port:** `3002`  
> **Identity:** `B`

---

## Overview

Backend B is a lightweight Node.js HTTP API server running on **Mac 2 (Mohit's machine)**. It is the second upstream backend behind the nginx reverse proxy / load balancer, also running on Mac 2.

Backend B listens on port `3002` at `127.0.0.1:3002` (localhost). Since nginx also runs on Mac 2, it accesses Backend B over the loopback interface.

---

## How It Was Built

### 1. Initialize the project

```bash
mkdir backend-b && cd backend-b
npm init -y
```

### 2. Install Express

```bash
npm install express
```

### 3. Create `server.js`

The server is **functionally identical** to Backend A. The only differences are:

| Property | Backend A | Backend B |
|----------|-----------|-----------|
| Identity | `A` | `B` |
| Default port | `3001` | `3002` |
| Machine | Mac 1 (`10.7.18.72`) | Mac 2 (`10.7.10.115`) |
| nginx access | Cross-LAN (`10.7.18.72:3001`) | Localhost (`127.0.0.1:3002`) |

Both backends share the same codebase structure. The identity and port are passed as CLI arguments.

### 4. Start the server

```bash
node server.js B 3002
```

Output:
```
Backend B listening on port 3002
```

---

## API Reference

### `GET /api/status`

**Response:**
```json
{
  "service": "running",
  "backend": "B"
}
```

**Headers:**
```
X-Backend: B
```

### `GET /api/cached`

**Response:**
```json
{
  "service": "running",
  "backend": "B"
}
```

**Headers:**
```
X-Backend: B
Cache-Control: max-age=60
ETag: W/"19-3Xq2iFjXnMbnAKEagMsqMcjEGaE"
```

---

## Verification

```bash
curl http://127.0.0.1:3002/api/status
```

Expected:
```json
{"service":"running","backend":"B"}
```

---

## Role in the System

```
Client → nginx (10.7.10.115:443)
              ├── Backend A (10.7.18.72:3001)
              └── Backend B (127.0.0.1:3002)   ← This server
```

Backend B runs on the same machine as nginx but communicates over the loopback interface (`127.0.0.1`), demonstrating that upstream backends can be local or remote. Combined with Backend A on a separate physical machine, this shows true distributed load balancing.
