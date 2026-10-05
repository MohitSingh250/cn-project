# Demo: Caching

This document demonstrates HTTP caching headers and conditional requests.

## Part 1: Initial Request

```bash
curl -v https://app.team1.test/api/cached --cacert config/tls/ca.crt
```

**Expected Headers:**
```
Cache-Control: max-age=60
ETag: W/"19-3Xq2iFjXnMbnAKEagMsqMcjEGaE"
```
The server sends the full response body along with instructions to cache it for 60 seconds and an ETag identifier.

## Part 2: Conditional Request

Simulate a client re-requesting the resource while it has the ETag in its cache. Replace the ETag value below with the one received in Part 1.

```bash
curl -v https://app.team1.test/api/cached \
  --cacert config/tls/ca.crt \
  -H 'If-None-Match: W/"19-3Xq2iFjXnMbnAKEagMsqMcjEGaE"'
```

**Expected Response:**
```
> GET /api/cached HTTP/2
> Host: app.team1.test
> User-Agent: curl/8.4.0
> Accept: */*
> If-None-Match: W/"19-3Xq2iFjXnMbnAKEagMsqMcjEGaE"
>
< HTTP/2 304 
< server: nginx/1.31.6
< x-backend: B
< cache-control: max-age=60
< etag: W/"19-3Xq2iFjXnMbnAKEagMsqMcjEGaE"
```

## Key Takeaways
- The server responds with HTTP 304 Not Modified.
- No response body is transmitted, saving bandwidth and processing time.
- nginx correctly proxies the caching headers back and forth between the client and the backend.
