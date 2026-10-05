# Task G — Caching

> Implement and verify HTTP caching mechanisms using Cache-Control and ETag headers.

---

## Objective

Demonstrate how standard HTTP cache headers instruct clients to cache responses and allow conditional requests to save bandwidth.

---

## Implementation

In the Express backends, the `/api/cached` endpoint explicitly sets headers:

```javascript
app.get('/api/cached', (req, res) => {
  res.set('Cache-Control', 'max-age=60');
  res.json({
    service: 'running',
    backend: BACKEND_ID,
  });
});
```

Express also automatically computes an `ETag` (hash) for the JSON body.

---

## Verification

### 1. Initial Request

```bash
curl -v https://app.team1.test/api/cached --cacert ca.crt
```

**Result Headers:**
```
HTTP/1.1 200 OK
Cache-Control: max-age=60
ETag: W/"19-3Xq2iFjXnMbnAKEagMsqMcjEGaE"
```

- `max-age=60` tells the client it can reuse this response for 60 seconds.
- `ETag` is the unique fingerprint of the response.

### 2. Conditional Request

We simulate a client that already has the cached response but wants to verify if it's still fresh after 60 seconds.

```bash
curl -v https://app.team1.test/api/cached \
  --cacert ca.crt \
  -H 'If-None-Match: W/"19-3Xq2iFjXnMbnAKEagMsqMcjEGaE"'
```

**Result:**
```
HTTP/1.1 304 Not Modified
```

### Analysis

- **304 Not Modified**: The server confirmed the content hasn't changed.
- **No Body**: The response contains headers only, saving bandwidth.
- **nginx Passthrough**: nginx correctly passes the caching headers from the backend to the client.

---

## Status: ✅ Complete
