# 🛡️ Reliability & Resilience — quick overview (then we’ll dive deep)

> Goal: keep your system **responsive, stable, and correct** when dependencies are slow, flaky, or down.

We’ll use four building blocks that work together:

---

## 13) ⏱️ Timeouts & 🔁 Retries — “Fail fast, retry smart”

- **Timeouts:** every external call gets a **deadline** (no waiting forever). Budget them so total fits your SLA (e.g., 250–800 ms internal).
- **Retries:** only for **transient** errors (timeouts, 5xx, `UNAVAILABLE`), with **exponential backoff + jitter**. Small, bounded attempts.
- **Never** retry non-idempotent work unless you have **idempotency** in place.

---

## 14) 🧯 Circuit Breaker — “Trip early, recover safely”

- Watches error rate/latency; when bad, it **opens** and **short-circuits** new calls (fast failures) to stop cascades.
- After a cool-off, it half-opens to probe if the downstream is healthy again.
- Protects your upstream threads and queues from being swamped.

---

## 15) 🧱 Bulkheads & Isolation — “Limit the blast radius”

- Cap **concurrency** per downstream (own thread/connection pools).
- Separate pools/clients by dependency (Catalog vs Payments) so one slow service doesn’t drown everything.
- Combine with **rate limits** and short queues (or no queue) to avoid request pile-ups.

---

## 16) 🔑 Idempotency Keys — “Safe to retry writes”

- Client sends a unique **Idempotency-Key** (or business key like `PaymentIntentId`) with POST/commands.
- The server stores `(key → result)` and **returns the same result** on duplicate attempts.
- Use **unique constraints** and **inbox/outbox** to make handlers idempotent end-to-end.

---

## How they fit together (mental flow)

1. **Timeout** on each call →
2. **Retry** transient failures (few times, jitter) →
3. If still bad, **Circuit Breaker opens** →
4. **Bulkhead** ensures only N concurrent calls per dependency →
5. For write/command retries, rely on **Idempotency** so you don’t double-charge/double-create.

---

## .NET (Polly + HttpClientFactory) one-shot setup

```csharp
using Polly;
using Polly.CircuitBreaker;
using Polly.Retry;
using Polly.Timeout;

var timeout = Policy.TimeoutAsync<HttpResponseMessage>(TimeSpan.FromMilliseconds(300));

var retry = Policy<HttpResponseMessage>
  .Handle<HttpRequestException>()
  .OrResult(r => (int)r.StatusCode >= 500)
  .WaitAndRetryAsync(3, i => TimeSpan.FromMilliseconds(50 * Math.Pow(2, i)) +
                             TimeSpan.FromMilliseconds(Random.Shared.Next(0, 30))); // jitter

var breaker = Policy<HttpResponseMessage>
  .Handle<HttpRequestException>()
  .OrResult(r => (int)r.StatusCode >= 500)
  .CircuitBreakerAsync(handledEventsAllowedBeforeBreaking: 5, durationOfBreak: TimeSpan.FromSeconds(30));

var bulkhead = Policy.BulkheadAsync<HttpResponseMessage>(maxParallelization: 50, maxQueuingActions: 0);

builder.Services.AddHttpClient("catalog", c => c.BaseAddress = new Uri(cfg.CatalogUrl))
  .AddPolicyHandler(bulkhead)
  .AddPolicyHandler(breaker)
  .AddPolicyHandler(retry)
  .AddPolicyHandler(timeout);
```

**Server-side idempotency (sketch):**

```csharp
// On POST /payments with header Idempotency-Key
var key = Request.Headers["Idempotency-Key"];
if (await store.ExistsAsync(key)) return store.GetResult(key); // repeatable response
var result = await handler.ProcessAsync(command);
await store.SaveAsync(key, result); // with unique index on key
return result;
```

---

## Common pitfalls (and fixes)

- **Infinite/default timeouts** → hung threads. ➜ Set explicit, small **timeouts**.
- **Retry storms** → amplify outages. ➜ **Backoff + jitter**, small attempts, **breaker**.
- **Retrying non-idempotent ops** → duplicates. ➜ Use **idempotency keys** + unique constraints.
- **Unbounded concurrency** → DB/socket exhaustion. ➜ **Bulkheads**, pool sizing.
- **One pool for all deps** → one bad service sinks all. ➜ Separate clients/pools per dependency.

---

## Assessment sound bites (crisp)

- “**Timeouts** bound latency; **retries** handle transient faults; **circuit breaker** stops cascades; **bulkheads** contain failures; **idempotency** makes retries safe.”
- “We budget timeouts so total fits SLA, retry with **exponential backoff + jitter**, and only on transient errors.”
- “For POST/commands we require an **Idempotency-Key** and store results so duplicates return the same response.”
- “Each critical downstream gets its own **HttpClient** + **Polly** policies and a **bulkhead**.”

---

## Pocket checklist

- [ ] Timeouts on **every** external call
- [ ] Retries: **few**, **backoff + jitter**, **transient only**
- [ ] Circuit breaker: tuned thresholds + health telemetry
- [ ] Bulkheads: per-dependency concurrency caps, separate pools
- [ ] Idempotency: key/header + server store, unique constraints
- [ ] Observability: log correlation ID, retry count, breaker state
