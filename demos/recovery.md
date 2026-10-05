# Demo: Recovery

This document demonstrates the automatic recovery capabilities of the nginx load balancer when a failed backend comes back online.

## Execution

1. **Start from Failure State**
   Ensure Backend A is stopped, and all traffic is routing to Backend B (as demonstrated in `backend-a-failure.md`).

2. **Restart Backend A**
   On Mac 1, restart the backend process:
   ```bash
   cd backend/backend-a
   node server.js A 3001
   ```

3. **Test Load Balancer**
   Run the load balancing test loop from either Mac:
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
X-Backend: A
---
Request 2:
X-Backend: B
---
Request 3:
X-Backend: A
---
Request 4:
X-Backend: B
---
```

## Key Takeaways
- As soon as Backend A starts listening on port 3001 again, nginx automatically detects its availability.
- nginx immediately re-includes Backend A in the active upstream pool.
- The round-robin load balancing pattern resumes.
- No manual intervention, nginx restart, or configuration reload is required to recover from the failure.
