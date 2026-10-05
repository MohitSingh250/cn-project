# Failure Demo — Backend A Failure

---

## Scenario

Demonstrate what happens when **Backend A** (on Mac 1) becomes unavailable while Backend B continues running.

---

## Before (Normal Operation)

Both backends running:

```
nginx (10.7.10.115:443)
    ├── Backend A (10.7.18.72:3001) ✅ Running
    └── Backend B (127.0.0.1:3002)  ✅ Running
```

Requests alternate: `A → B → A → B`

---

## Step 1: Stop Backend A

On Mac 1 (Abhishek), stop the Backend A process:

```bash
# Press Ctrl+C in the Backend A terminal
# Or find and kill the process:
lsof -i :3001
kill <PID>
```

---

## Step 2: Verify Backend A is Down

```bash
curl http://10.7.18.72:3001/api/status
```

**Expected:** Connection timeout / refused. No response.

---

## Step 3: Test Through nginx

```bash
curl -s https://app.team1.test/api/status --cacert /path/to/ca.crt \
  -D - | grep -E "(HTTP|X-Backend)"
```

### Verified Result

```
HTTP/1.1 200 OK
X-Backend: B
```

**✅ All requests succeed through Backend B.**

nginx automatically detects that Backend A is unreachable and routes all traffic to Backend B.

---

## After (Backend A Down)

```
nginx (10.7.10.115:443)
    ├── Backend A (10.7.18.72:3001) ❌ Down
    └── Backend B (127.0.0.1:3002)  ✅ Running ← All traffic here
```

---

## What This Proves

| Concept | Evidence |
|---------|----------|
| Graceful degradation | Service continues with one backend down |
| Automatic failover | nginx reroutes without manual intervention |
| No client errors | Clients still get HTTP 200 |
| Backend identity | `X-Backend: B` confirms only B is serving |
| Cross-machine failure | Physical machine failure handled transparently |

---

## Network Layer Analysis

| Layer | What Happens |
|-------|-------------|
| Transport (L4) | TCP SYN to `10.7.18.72:3001` fails (no response or RST) |
| Application (L7) | nginx marks Backend A as temporarily unavailable |
| Application (L7) | nginx forwards the request to Backend B instead |
| Application (L7) | Client receives a normal HTTP 200 response |
