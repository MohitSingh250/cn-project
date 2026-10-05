# Demo: Backend A Failure

This document demonstrates graceful degradation when a backend server fails.

## Execution

1. **Stop Backend A**
   On Mac 1, terminate the `node server.js A 3001` process (e.g., using `Ctrl+C`).

2. **Verify Backend A is Down**
   ```bash
   curl --max-time 2 http://10.7.18.72:3001/api/status
   ```
   *Expected: Connection refused or timeout.*

3. **Test Load Balancer**
   From either Mac, run the load balancing test loop:
   ```bash
   for i in {1..4}; do
     echo "Request $i:"
     curl -s -D - https://app.team1.test/api/status --cacert config/tls/ca.crt | grep "X-Backend"
     echo "---"
   done
   ```

## Expected Output

```
Request 1:
X-Backend: B
---
Request 2:
X-Backend: B
---
Request 3:
X-Backend: B
---
Request 4:
X-Backend: B
---
```

## Key Takeaways
- Client requests continue to succeed (HTTP 200) despite a backend failure.
- nginx automatically detects that Backend A is unresponsive and routes all incoming traffic to the surviving Backend B.
- The failure is completely transparent to the client.
