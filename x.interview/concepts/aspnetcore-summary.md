# 🚀 ASP.NET (incl. ASP.NET Core) — What they’re likely to test + rapid-fire Qs

15 questions in 10 minutes → \~40s per question. Expect short, conceptual multiple-choice with tiny code snippets.

## 🎯 High-yield topics (skim once, then jump to Qs)

- **Hosting & Startup**: minimal hosting model (`var app = WebApplication.CreateBuilder();`), middleware order, `app.MapControllers()`.
- **Routing**: attribute routing vs endpoint routing, route constraints.
- **Controllers vs Minimal APIs**; `[ApiController]` behavior (auto 400, binding inference, ProblemDetails).
- **Model binding & validation**: `[FromBody]`, `[FromRoute]`, `[FromQuery]`, data annotations, one-body rule.
- **Filters**: MVC filters (auth/resource/action/exception) vs **Endpoint Filters** (minimal APIs).
- **DI & lifetimes**: `Singleton/Scoped/Transient`, captive-dependency pitfalls.
- **HttpClientFactory**: typed clients, prevent socket exhaustion, Polly integration.
- **AuthN/Z**: JWT vs cookies, `[Authorize]`, policies/roles/claims, CORS vs CSRF.
- **Config & Options**: `appsettings.*`, env vars override, `IOptions<T> / IOptionsMonitor<T>`.
- **Caching & Perf**: **Output Caching** (server cache), **Response Caching** (HTTP cache headers), compression, rate limiting.
- **Health & Observability**: health checks, logging, ProblemDetails, OpenAPI.
- **Background work**: `IHostedService`, scoped services in background.
- **Kestrel & hosting**: HTTPS redirection/HSTS, forwarded headers behind reverse proxy.
- **EF Core basics** in web context: async, `AsNoTracking`, `rowversion` for concurrency.

---

## 🧪 Rapid-fire practice (typical assessment style)

(Each has a short, crisp answer so you can memorize the gist.)

1. **What does `[ApiController]` add?**
   Auto 400 on invalid model state, binding inference (complex types from body, simple from route/query), ProblemDetails responses.

2. **Correct middleware order (classic MVC):**
   `UseRouting` → `UseAuthentication` → `UseAuthorization` → `MapControllers`.
   (Minimal hosting often: `UseAuthentication` → `UseAuthorization` → `MapControllers`.)

3. **Model binding: how many `[FromBody]` per action?**
   **One**. Wrap multiple values into a DTO.

4. **Return 201 with Location (MVC):**
   `return CreatedAtAction(nameof(Get), new { id = x.Id }, x);`
   (Minimal APIs: `Results.Created($"/resource/{id}", obj)`)

5. **Content negotiation uses…**
   `Accept` header + registered formatters (System.Text.Json by default).

6. **Difference: Output Caching vs Response Caching?**
   Output Caching = **server-side** cache in ASP.NET Core (fast, .NET 7/8).
   Response Caching = sets **HTTP cache headers** (client/proxy cache).

7. **Enable CORS?**
   `builder.Services.AddCors();` → `app.UseCors("Policy");` (before `MapControllers`).

8. **CORS vs CSRF?**
   CORS = cross-origin **permission**. CSRF = browser cookie attack → use anti-forgery for cookie auth; JWT APIs usually disable anti-forgery.

9. **DI lifetimes:**
   `Singleton` (one per app), `Scoped` (per request), `Transient` (every inject). Avoid **scoped inside singleton** (captive dependency).

10. **HttpClientFactory advantages:**
    Pooled sockets, DNS refresh, central config, Polly policies. Use **typed clients**.

11. **Bind options from config:**
    `builder.Services.Configure<MyOpt>(config.GetSection("MyOpt"));` then inject `IOptions<MyOpt>`.

12. **Auto-validation failure result under `[ApiController]`?**
    400 with **ProblemDetails**.

13. **Add Swagger (OpenAPI):**
    `AddEndpointsApiExplorer(); AddSwaggerGen();` + `UseSwagger(); UseSwaggerUI();`

14. **Authorize by policy:**
    `[Authorize(Policy = "CanShip")]` + `AddAuthorization(o => o.AddPolicy("CanShip", p => p.RequireClaim("scope","ship")));`

15. **Rate limiting built-in?**
    Yes. Add `AddRateLimiter()` → `UseRateLimiter()`; policies like fixed window/token bucket.

16. **Health checks:**
    `AddHealthChecks()` then `MapHealthChecks("/health")` (use readiness/liveness tags).

17. **Exception to ProblemDetails globally:**
    `app.UseExceptionHandler("/error")` + an endpoint returning `Results.Problem()`.

18. **Minimal API: add validation?**
    Manual or endpoint filters; or bind to DTO with data annotations and check `ValidationProblem`.

19. **Get route value in controller:**
    `[HttpGet("{id:int}")]` parameter `int id` or `RouteData.Values["id"]`.

20. **File upload (simple):**
    `IFormFile` (small files). For large, stream via `Request.Body`/`PipeReader`.

21. **Forwarded headers behind Nginx/IIS?**
    `app.UseForwardedHeaders(new ForwardedHeadersOptions { ForwardedHeaders = ... });` and configure trusted networks.

22. **HTTP logging & correlation:**
    Add logging scopes/middleware that inject `X-Correlation-Id`; log it with each request.

23. **EF Core in web apps—two musts:**
    Always **async** (no blocking), use `AsNoTracking()` for reads; concurrency with `rowversion`.

24. **Default JSON lib & naming:**
    System.Text.Json; `JsonSerializerOptions.PropertyNamingPolicy = CamelCase` by default.

25. **gRPC in ASP.NET Core needs…**
    HTTP/2 (and gRPC-Web for browsers), `Grpc.AspNetCore`.

26. **SignalR basics:**
    `app.MapHub<ChatHub>("/chat")`; real-time websockets with fallback.

27. **Filters vs Middleware?**
    Middleware = **outside** MVC (cross-cutting for all requests).
    Filters = **inside** MVC/endpoint pipeline (per controller/action/endpoint).

28. **Minimal API: inject services into handler?**
    Add parameter with `[FromServices]` or use DI in lambda parameters (by type).

29. **HSTS/HTTPS:**
    `UseHttpsRedirection()` and `UseHsts()` (prod).

30. **Testing web APIs:**
    `WebApplicationFactory<T>` with TestServer for integration tests.

---

## 🧷 Tiny code bites you might see

**HttpClientFactory + Polly sketch:**

```csharp
builder.Services.AddHttpClient<IPricingClient, PricingClient>(c =>
    c.BaseAddress = new Uri(cfg.PricingUrl))
  .SetHandlerLifetime(TimeSpan.FromMinutes(5));
```

**Endpoint filter (minimal API):**

```csharp
app.MapPost("/orders", (OrderDto dto, IOrderSvc svc) => svc.Create(dto))
   .AddEndpointFilter(new ValidationFilter());
```

**Rate limiter (fixed window):**

```csharp
builder.Services.AddRateLimiter(_ => _.AddFixedWindowLimiter("api", o => {
  o.PermitLimit = 100; o.Window = TimeSpan.FromMinutes(1);
}));
app.UseRateLimiter();
```

**Output caching:**

```csharp
builder.Services.AddOutputCache();
app.UseOutputCache();
app.MapGet("/products", GetProducts).CacheOutput(p => p.Expire(TimeSpan.FromSeconds(30)));
```

---

## ⚠️ Classic gotchas they love to test

- Trying to bind **two** body parameters → invalid; use a DTO.
- Putting **scoped** services into **singletons** → bug.
- Forgetting **authentication before authorization** in the pipeline.
- Confusing **Output** vs **Response** caching.
- Using one `HttpClient` manually → socket exhaustion (use factory).
- Not enabling **CORS** but expecting browser to call from another origin.
- Returning 200 on create instead of **201 + Location**.

---

## ⏱️ Test-taking tips

- 10 minutes / 15 Qs: skim choices, answer the obvious ones first, **guess and move** if stuck (don’t burn >45s on any Q).
- If a snippet looks off, look for **pipeline order**, **binding source**, **DI lifetime**, or **HTTP status** mistakes—those are the common traps.
