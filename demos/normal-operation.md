# Demo: Normal Operation

This document demonstrates the expected behavior of the system under normal conditions, where both backends are healthy.

## Prerequisites
- `dnsmasq` running on Mac 1
- `nginx` running on Mac 2
- Backend A running on Mac 1 (`node server.js A 3001`)
- Backend B running on Mac 2 (`node server.js B 3002`)

## Execution

```bash
curl -i https://app.team1.test/api/status --cacert config/tls/ca.crt
```

## Expected Output

```
HTTP/1.1 200 OK
Server: nginx/1.31.6
Date: Tue, 05 Oct 2026 12:00:00 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 35
Connection: keep-alive
X-Backend: A
X-Powered-By: Express
ETag: W/"23-abcd1234efgh5678"

{"service":"running","backend":"A"}
```

(The `X-Backend` and `backend` value in the JSON may be `B` depending on the load balancer state).

## Key Takeaways
- The DNS correctly routed to nginx.
- TLS was successfully negotiated and verified by curl.
- nginx successfully proxied the request to a healthy backend.
- The backend successfully returned the expected JSON payload.
