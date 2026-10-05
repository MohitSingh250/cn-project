# Task F — Load Balancing

> Verify that nginx evenly distributes traffic across the backend pool.

---

## Objective

Demonstrate that requests to `app.team1.test` are distributed across Backend A (`10.7.18.72:3001`) and Backend B (`127.0.0.1:3002`) using the default round-robin algorithm.

---

## Configuration

In `nginx.conf`:

```nginx
upstream backends {
    server 10.7.18.72:3001;
    server 127.0.0.1:3002;
}
```

By default, nginx uses round-robin, passing requests sequentially to each server in the upstream block.

---

## Test Execution

We issue 6 rapid requests to the `/api/status` endpoint and extract the `X-Backend` header:

```bash
for i in {1..6}; do
  curl -s -D - https://app.team1.test/api/status --cacert ca.crt \
    | grep "X-Backend"
done
```

---

## Result Analysis

**Actual Output:**
```
X-Backend: A
X-Backend: B
X-Backend: A
X-Backend: B
X-Backend: A
X-Backend: B
```

This perfectly alternating sequence confirms:
1. nginx is load balancing successfully.
2. Both backends are healthy and receiving traffic.
3. Requests traverse different network paths (A = Cross-LAN, B = Loopback) transparently to the client.

---

## Status: ✅ Complete
