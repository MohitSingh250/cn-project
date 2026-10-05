# Starting Both Backends

---

## On Mac 1 — Abhishek (`10.7.18.72`)

### Start Backend A

```bash
cd backend/backend-a
npm install
node server.js A 3001
```

Expected output:
```
Backend A listening on port 3001
```

### Verify Backend A is accessible from Mac 2

From Mac 2:
```bash
curl http://10.7.18.72:3001/api/status
```

Expected:
```json
{"service":"running","backend":"A"}
```

---

## On Mac 2 — Mohit (`10.7.10.115`)

### Start Backend B

```bash
cd backend/backend-b
npm install
node server.js B 3002
```

Expected output:
```
Backend B listening on port 3002
```

### Verify Backend B locally

```bash
curl http://127.0.0.1:3002/api/status
```

Expected:
```json
{"service":"running","backend":"B"}
```

---

## Both Running — System Ready

Once both backends are running, nginx can distribute requests between them.

```
nginx (10.7.10.115:443)
    ├── Backend A (10.7.18.72:3001) ✅ Running
    └── Backend B (127.0.0.1:3002)  ✅ Running
```

---

## Stopping Backends

To stop a backend, press `Ctrl+C` in its terminal, or:

```bash
# Find the process
lsof -i :3001   # Backend A
lsof -i :3002   # Backend B

# Kill it
kill <PID>
```
