# Load Balancing Tests

---

## Test: Round-Robin Distribution

Six consecutive requests were sent to verify nginx distributes traffic evenly:

```bash
for i in {1..6}; do
  curl -s -o /dev/null -w "Request $i: %{http_code}\n" \
    -D - https://app.team1.test/api/status --cacert /path/to/ca.crt \
    | grep "X-Backend"
done
```

### Verified Result

```
X-Backend: B
X-Backend: A
X-Backend: B
X-Backend: A
X-Backend: B
X-Backend: A
```

**✅ PASS** — Requests alternate perfectly between Backend A and Backend B.

---

## How Round-Robin Works

```
Request 1 → nginx → Backend B (127.0.0.1:3002)
Request 2 → nginx → Backend A (10.7.18.72:3001)
Request 3 → nginx → Backend B (127.0.0.1:3002)
Request 4 → nginx → Backend A (10.7.18.72:3001)
Request 5 → nginx → Backend B (127.0.0.1:3002)
Request 6 → nginx → Backend A (10.7.18.72:3001)
```

nginx uses **round-robin** by default when multiple servers are defined in an `upstream` block. No additional configuration is needed.

---

## Why This Matters

| Property | Benefit |
|----------|---------|
| Even distribution | No single backend is overloaded |
| Automatic failover | If one backend dies, nginx routes to the other |
| Cross-machine | Backend A is on a separate physical machine |
| Transparent to clients | Clients see a single endpoint (`app.team1.test`) |

---

## Verifying Both Backends Are Different Machines

| Backend | IP | Machine | Network Path |
|---------|----|---------|-------------|
| A | `10.7.18.72:3001` | Mac 1 (Abhishek) | Cross-LAN (Wi-Fi) |
| B | `127.0.0.1:3002` | Mac 2 (Mohit) | Loopback |

This proves the load balancer distributes traffic across physically separate machines on the network.
