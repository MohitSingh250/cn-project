# Recovery After Failure

---

## Scenario

After demonstrating Backend A failure, restore the system to full operation with both backends serving traffic.

---

## State Before Recovery

```
nginx (10.7.10.115:443)
    ├── Backend A (10.7.18.72:3001) ❌ Down
    └── Backend B (127.0.0.1:3002)  ✅ Running
```

All traffic is currently routed to Backend B only.

---

## Step 1: Restart Backend A

On Mac 1 (Abhishek):

```bash
cd backend/backend-a
node server.js A 3001
```

Expected output:
```
Backend A listening on port 3001
```

---

## Step 2: Verify Backend A Directly

```bash
curl http://10.7.18.72:3001/api/status
```

### Verified Result

```
HTTP/1.1 200 OK
```

```json
{"service":"running","backend":"A"}
```

**✅ Backend A is back online.**

---

## Step 3: Verify Backend B Still Running

```bash
curl http://127.0.0.1:3002/api/status
```

### Verified Result

```json
{"service":"running","backend":"B"}
```

**✅ Backend B continues running.**

---

## Step 4: Verify Load Balancing Resumes

```bash
for i in {1..4}; do
  curl -s https://app.team1.test/api/status --cacert /path/to/ca.crt \
    -D - | grep "X-Backend"
done
```

### Verified Result

```
X-Backend: B
X-Backend: A
X-Backend: B
X-Backend: A
```

**✅ Round-robin load balancing has resumed.** nginx automatically detects that Backend A is available again.

---

## State After Recovery

```
nginx (10.7.10.115:443)
    ├── Backend A (10.7.18.72:3001) ✅ Running
    └── Backend B (127.0.0.1:3002)  ✅ Running
```

---

## Complete Failure/Recovery Timeline

| Phase | Backend A | Backend B | nginx Behavior |
|-------|-----------|-----------|----------------|
| Normal | ✅ Running | ✅ Running | Round-robin A ↔ B |
| Failure | ❌ Down | ✅ Running | All traffic → B |
| Recovery | ✅ Restarted | ✅ Running | Round-robin A ↔ B |

---

## What This Proves

| Concept | Evidence |
|---------|----------|
| Auto-recovery | nginx re-includes Backend A without restart |
| No configuration change | Recovery is fully automatic |
| Zero downtime | No service interruption during recovery |
| Load balancing resumes | Alternating A/B pattern returns |
