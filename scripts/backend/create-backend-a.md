# Creating Backend A

> **Machine:** Mac 1 — Abhishek (`10.7.18.72`)

---

## Step 1: Create the Project Directory

```bash
mkdir -p backend-a
cd backend-a
```

## Step 2: Initialize Node.js Project

```bash
npm init -y
```

This creates `package.json` with default settings.

## Step 3: Install Express

```bash
npm install express
```

Express is a minimal HTTP framework for Node.js. It provides:
- Route handling (`app.get()`)
- Middleware pipeline (`app.use()`)
- Automatic `ETag` generation for cacheable responses

## Step 4: Create `server.js`

The server accepts two CLI arguments:
- **Backend identity** (`A`) — used in the `X-Backend` response header and JSON body
- **Port** (`3001`) — the TCP port to listen on

### Endpoints

| Endpoint | Purpose | Special Headers |
|----------|---------|-----------------|
| `GET /api/status` | Service health check | `X-Backend: A` |
| `GET /api/cached` | Cacheable response | `X-Backend: A`, `Cache-Control: max-age=60`, `ETag` |

### X-Backend Header

Every response includes an `X-Backend` header set via middleware. This header is critical for:
- **Load balancing verification**: Confirms which backend handled the request
- **Failure demo**: Proves traffic shifts when a backend goes down

## Step 5: Verify

```bash
node server.js A 3001
```

In another terminal:
```bash
curl http://10.7.18.72:3001/api/status
```

Expected:
```json
{"service":"running","backend":"A"}
```

---

## Network Layer Analysis

| Layer | What Happens |
|-------|-------------|
| Application (L7) | Express processes HTTP GET, returns JSON |
| Transport (L4) | TCP connection on port 3001 |
| Network (L3) | IP packet from client to `10.7.18.72` |
| Data Link (L2) | Wi-Fi frame over `en0` |
