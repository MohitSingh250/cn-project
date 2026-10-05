# Backend A

> **Machine:** Mac 1 — Abhishek (`10.7.18.72`)  
> **Port:** `3001`  
> **Identity:** `A`

---

## Overview

Backend A is a lightweight Node.js HTTP API server running on **Mac 1 (Abhishek's machine)**. It is one of two upstream backends behind the nginx reverse proxy / load balancer on Mac 2.

Backend A listens on port `3001` and is reachable across the LAN at `10.7.18.72:3001`.

---

## How It Was Built

### 1. Initialize the project

```bash
mkdir backend-a && cd backend-a
npm init -y
```

### 2. Install Express

```bash
npm install express
```

### 3. Create `server.js`

The server exposes two endpoints and attaches an `X-Backend` header to every response:

| Endpoint | Purpose |
|----------|---------|
| `GET /api/status` | Returns `{"service":"running","backend":"A"}` |
| `GET /api/cached` | Same response, with `Cache-Control: max-age=60` and `ETag` |

The `X-Backend` header lets us verify which backend handled each request during load balancing tests.

### 4. Start the server

```bash
node server.js A 3001
```

Output:
```
Backend A listening on port 3001
```

---

## API Reference

### `GET /api/status`

**Response:**
```json
{
  "service": "running",
  "backend": "A"
}
```

**Headers:**
```
X-Backend: A
```

### `GET /api/cached`

**Response:**
```json
{
  "service": "running",
  "backend": "A"
}
```

**Headers:**
```
X-Backend: A
Cache-Control: max-age=60
ETag: W/"19-3Xq2iFjXnMbnAKEagMsqMcjEGaE"
```

---

## Verification

```bash
curl http://10.7.18.72:3001/api/status
```

Expected:
```json
{"service":"running","backend":"A"}
```

---

## Role in the System

```
Client → nginx (10.7.10.115:443)
              ├── Backend A (10.7.18.72:3001)   ← This server
              └── Backend B (127.0.0.1:3002)
```

nginx distributes requests between Backend A and Backend B using round-robin load balancing. Backend A runs on a separate physical machine (Mac 1), demonstrating true cross-machine networking over the LAN.
