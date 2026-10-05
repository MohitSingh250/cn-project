# Creating Backend B

> **Machine:** Mac 2 — Mohit (`10.7.10.115`)

---

## Step 1: Create the Project Directory

```bash
mkdir -p backend-b
cd backend-b
```

## Step 2: Initialize Node.js Project

```bash
npm init -y
```

## Step 3: Install Express

```bash
npm install express
```

## Step 4: Create `server.js`

Backend B uses the **same codebase** as Backend A. The only differences are the CLI arguments:

| Argument | Backend A | Backend B |
|----------|-----------|-----------|
| Identity | `A` | `B` |
| Port | `3001` | `3002` |

This design means a single `server.js` file can power any number of backend instances — a common microservices pattern.

## Step 5: Verify

```bash
node server.js B 3002
```

In another terminal:
```bash
curl http://127.0.0.1:3002/api/status
```

Expected:
```json
{"service":"running","backend":"B"}
```

---

## Why Backend B Is on Localhost

Backend B runs on the **same machine** as nginx (Mac 2). nginx accesses it via `127.0.0.1:3002` (loopback interface).

This demonstrates that upstream backends can be:
- **Remote** (Backend A on Mac 1, accessed cross-LAN)
- **Local** (Backend B on Mac 2, accessed via loopback)

Both patterns are common in production architectures.

---

## Network Layer Analysis

| Layer | What Happens |
|-------|-------------|
| Application (L7) | Express processes HTTP GET, returns JSON |
| Transport (L4) | TCP connection on port 3002 |
| Network (L3) | IP packet via loopback (`127.0.0.1`) |
| Data Link (L2) | Loopback interface (no physical network) |
