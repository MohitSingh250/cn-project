# Task I — Failure Demonstration

> Demonstrate system resilience when backend components fail.

---

## Objective

Show that the nginx load balancer detects backend failures and gracefully reroutes traffic, preventing complete service outage.

---

## Scenario 1: Single Backend Failure

1. **State:** Both Backend A and B are running. Load is balanced A ↔ B.
2. **Action:** We manually kill Backend A (press `Ctrl+C`).
3. **Test:** We send multiple requests to `https://app.team1.test/api/status`.
4. **Observation:**
   - Client requests still succeed (HTTP 200).
   - `X-Backend` header exclusively returns `B`.
   - nginx detects that A is unreachable and temporarily removes it from the active pool.
5. **Conclusion:** Graceful degradation achieved. Zero client-facing errors.

---

## Scenario 2: Total Backend Failure

1. **State:** Backend A is down. Backend B is running.
2. **Action:** We manually kill Backend B.
3. **Test:** We send a request to `https://app.team1.test/api/status`.
4. **Observation:**
   - Client receives `HTTP 502 Bad Gateway`.
   - nginx cannot find any healthy upstreams.
5. **Conclusion:** Correct failure behavior. nginx handles the total outage gracefully by returning a standard 5xx error.

---

## Scenario 3: Recovery

1. **State:** Both backends are down.
2. **Action:** We restart both Backend A and Backend B.
3. **Test:** We send multiple requests.
4. **Observation:**
   - Client requests succeed (HTTP 200).
   - Load balancing resumes (A ↔ B).
5. **Conclusion:** nginx automatically recovers the upstreams without requiring a restart or config reload.

---

## Status: ✅ Complete
