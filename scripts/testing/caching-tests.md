# Caching Tests

---

## Test 1: Cache-Control Header

```bash
curl -v https://app.team1.test/api/cached --cacert /path/to/ca.crt
```

### Verified Response Headers

```
HTTP/1.1 200 OK
Server: nginx/1.31.6
X-Backend: B
Cache-Control: max-age=60
ETag: W/"19-3Xq2iFjXnMbnAKEagMsqMcjEGaE"
```

### Verified Response Body

```json
{"service":"running","backend":"B"}
```

**✅ PASS** — Response includes `Cache-Control: max-age=60` and an `ETag`.

---

## Test 2: Conditional Request (If-None-Match → 304)

Using the ETag from the previous response:

```bash
curl -v https://app.team1.test/api/cached \
  --cacert /path/to/ca.crt \
  -H 'If-None-Match: W/"19-3Xq2iFjXnMbnAKEagMsqMcjEGaE"'
```

### Verified Result

```
HTTP/1.1 304 Not Modified
```

**✅ PASS** — Server returns `304 Not Modified` when the ETag matches, saving bandwidth.

---

## How HTTP Caching Works Here

### Cache-Control: max-age=60

Tells clients (and intermediate proxies) they can cache this response for **60 seconds** without revalidating.

```
Client ──→ Cache (has fresh copy?) ──→ YES → Return cached response
                                   └──→ NO  → Forward to backend
```

### ETag (Entity Tag)

A unique fingerprint of the response content. Express auto-generates ETags.

```
Client sends:  If-None-Match: W/"19-3Xq2iFjXnMbnAKEagMsqMcjEGaE"
Server checks: Does current response match this ETag?
  YES → 304 Not Modified (no body sent, saves bandwidth)
  NO  → 200 OK (full response with new ETag)
```

---

## What These Tests Prove

| Concept | Evidence |
|---------|----------|
| Cache-Control works | `max-age=60` header present |
| ETag generation | Express generates `W/"19-..."` automatically |
| Conditional requests | `If-None-Match` → `304 Not Modified` |
| Bandwidth savings | 304 response has no body |
| Backend passthrough | Cache headers pass through nginx correctly |

---

## Network Layers Involved

| Layer | Caching Role |
|-------|-------------|
| Application (L7) | HTTP cache headers (`Cache-Control`, `ETag`, `If-None-Match`) |
| Transport (L4) | TCP carries the HTTP request/response |
| Presentation (L6) | TLS encrypts the cached/uncached response identically |
