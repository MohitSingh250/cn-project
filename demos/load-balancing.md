# Demo: Load Balancing

This document demonstrates the round-robin load balancing configuration of nginx.

## Execution

Execute the following bash loop from either Mac:

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
- The output alternates strictly between Backend A and Backend B.
- This confirms that nginx is using the default round-robin algorithm to distribute traffic evenly across the upstream pool.
