# 📚 Microservices Study Roadmap (TOC)

## 1️⃣ Foundations

**1. What Are Microservices?** – Core definition, principles, monolith vs microservices.  
**2. Service Boundaries & DDD** – How to split services using bounded context.  
**3. 12-Factor & Cloud-Native Basics** – The rules that microservices follow.

## 2️⃣ Communication

**4. Sync Communication** – REST, gRPC, pros/cons.  
**5. Async Communication** – Events, messaging brokers, decoupling.  
**6. API Gateway** – One entry point for routing, auth, and throttling.  
**7. Service Discovery** – How services find each other (DNS, Consul, Kubernetes).

## 3️⃣ Data & Consistency

**8. Database Per Service** – Why each service needs its own DB.  
**9. Eventual Consistency** – Why “strong” consistency across services is rare.  
**10. Saga Pattern** – Handling distributed transactions (orchestration vs choreography).  
**11. Outbox & Inbox Patterns** – Reliable event publishing & deduplication.  
**12. CQRS & Event Sourcing** – Separating reads/writes and storing events.

## 4️⃣ Reliability & Resilience

**13. Timeouts & Retries** – Never wait forever, retry safely.  
**14. Circuit Breaker** – Stop cascading failures.  
**15. Bulkheads & Isolation** – Limit blast radius of failures.  
**16. Idempotency Keys** – Safe retries for payments and POST requests.

## 5️⃣ Security

**17. Auth at the Edge** – OAuth2/OIDC via API Gateway.  
**18. Service-to-Service Security** – JWTs, mTLS, zero trust.  
**19. Secrets Management** – Key Vault, Secrets Manager, Vault.

## 6️⃣ Observability

**20. Structured Logging** – Centralized JSON logs + correlation IDs.  
**21. Metrics & Health Checks** – RED/USE metrics, /health endpoints.  
**22. Distributed Tracing** – OpenTelemetry, Jaeger, App Insights.

## 7️⃣ Deployment & Scaling

**23. Containers & Docker** – Why every service runs in a container.  
**24. Kubernetes Basics** – Deployments, services, ingresses, probes.  
**25. Scaling Patterns** – Horizontal scaling, HPA, KEDA for queue-based scaling.  
**26. Service Mesh** – Istio/Linkerd for advanced routing, retries, mTLS.

## 8️⃣ Testing & Delivery

**27. Unit & Integration Testing** – Test per service, with real DB/broker (Testcontainers).  
**28. Consumer-Driven Contracts** – Pact to avoid breaking clients.  
**29. End-to-End & Chaos Testing** – Full journey + resilience checks.  
**30. Release Strategies** – Blue/Green, Canary, Feature Flags.

## 9️⃣ Performance & Caching

**31. Cache Patterns** – Cache-aside, write-through, write-behind.  
**32. Load Leveling** – Queue buffering for traffic spikes.  
**33. DB Migration Patterns** – Expand/contract to avoid downtime.

## 🔟 Advanced & Anti-Patterns

**34. Polyglot Persistence** – Mixing SQL, NoSQL, search engines.  
**35. Microservices Anti-Patterns** – Nano-services, shared DBs, chatty sync calls.  
**36. Future Trends** – Dapr, serverless microservices, event-driven everything.

---

👉 If you study these topics in this order, you’ll **go from zero → exam-ready pro**.
