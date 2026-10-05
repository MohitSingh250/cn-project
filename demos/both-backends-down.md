# Demo: Total Backend Failure

This document demonstrates the system's behavior when no upstream backends are available.

## Execution

1. **Stop Both Backends**
   Ensure Backend A is stopped on Mac 1.
   Stop Backend B on Mac 2 by terminating the `node server.js B 3002` process.

2. **Verify Both Are Down**
   ```bash
   curl --max-time 2 http://10.7.18.72:3001/api/status
   curl --max-time 2 http://127.0.0.1:3002/api/status
   ```
   *Expected: Connection refused for both.*

3. **Test Load Balancer**
   Send a request to the main application endpoint:
   ```bash
   curl -i https://app.team1.test/api/status --cacert config/tls/ca.crt
   ```

## Expected Output

```
HTTP/1.1 502 Bad Gateway
Server: nginx/1.31.6
Date: Tue, 05 Oct 2026 12:05:00 GMT
Content-Type: text/html
Content-Length: 157
Connection: keep-alive

<html>
<head><title>502 Bad Gateway</title></head>
<body>
<center><h1>502 Bad Gateway</h1></center>
<hr><center>nginx/1.31.6</center>
</body>
</html>
```

## Key Takeaways
- When no backends are available, nginx returns a standard HTTP 502 Bad Gateway error.
- nginx itself remains responsive and continues to terminate TLS and serve error pages.
