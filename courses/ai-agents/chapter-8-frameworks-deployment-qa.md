---
layout: chapter
title: Frameworks & Deployment Q&A
course_id: ai-agents
chapter_number: 8
---

**Quick Revision:** Test your understanding of agent frameworks, deployment, and production operations.

## Framework Choice

**Q1:** When to use LangChain vs custom framework?
**A:** **LangChain:** Prototyping, standard use cases (RAG/chatbots), need ecosystem integrations, fast time-to-market. **Custom:** Performance critical, specific requirements, long-term maintenance, scale (1M+ requests/day). Hybrid: Use LangChain components with custom orchestration.

**Q2:** Why build custom framework for production?
**A:** Full control over: performance optimization, exact token counting, custom retry logic, dependency management (avoid breaking changes). LangChain good for prototyping, custom for scale/control.

**Q3:** What's a hybrid approach to frameworks?
**A:** Use LangChain components (LLM wrappers, embeddings) but custom agent loop. Gets ecosystem benefits without abstraction overhead. Optimize hot paths as you scale.

## Deployment Patterns

**Q4:** Compare serverless vs container deployment.
**A:** **Serverless:** 1-3s cold start, 15min max, auto-scale, $0.20/1M requests, zero ops. **Containers:** No cold start, unlimited timeout, manual scale, ~$200/mo baseline, more control. Use serverless for spiky traffic, containers for steady high load.

**Q5:** When is serverless better than containers?
**A:** Spiky/unpredictable traffic, short tasks (<15min), want zero ops, low baseline cost. Example: Chatbot with 1000 requests/day = $5/mo serverless vs $200/mo containers (95% savings).

**Q6:** When are containers better than serverless?
**A:** Consistent high traffic, long-running tasks (>15min), need stateful connections, require full control. Example: Code analysis at 50 RPS with WebSockets - containers better for features, similar cost.

## Monitoring

**Q7:** Five critical metrics for agent systems?
**A:** 1) **Request metrics** (RPS, latency p50/p95/p99, success rate), 2) **Agent metrics** (steps/task, tokens/request, cost), 3) **Tool metrics** (call frequency, success rate, latency), 4) **Error metrics** (by type/tool/user), 5) **Cost metrics** (LLM cost, cache hit rate).

**Q8:** Why focus on P95 latency vs P50?
**A:** P50 is median (half users), P95 is tail (5% worst case). Agent systems have variable latency (1-10+ steps). Tail latency matters more for UX. Target: P50 <2s, P95 <5s, P99 <10s.

**Q9:** What alerts to configure for production?
**A:** **Critical:** error_rate >5%, p95_latency >30s, LLM_API down (PagerDuty). **Warning:** success_rate <98%, daily_cost >budget, cache_hit_rate <30% (Slack/email). Always alert on money and uptime.

## Scaling

**Q10:** Design system for 10K concurrent users - capacity calculation?
**A:** 10K users × 2 requests/min / 60 = 333 RPS. Each request: 10 steps × 5s = 50s avg. Concurrent tasks: 333 × 5s / 30s = 55 tasks. Need: 100 LLM QPS, 200 sandboxes (if 20% use code), 3 API gateways, 200 workers.

**Q11:** Bottlenecks when scaling agents?
**A:** 1) **LLM API rate limits** (multi-provider fallback), 2) **Sandbox creation** (pre-warm pool), 3) **Memory/context** (compression, summarization), 4) **Tool latency** (caching, parallel execution). Plan for each.

**Q12:** Cost for 10K concurrent user system?
**A:** ~$405K/mo initially (mostly LLM at $390K). Optimizations: aggressive caching (40% hit = $156K savings), smaller models for 90% of tasks ($300K → $40K), prompt compression (30% reduction). After optimization: ~$80K/mo ($8/user).

## Error Handling

**Q13:** What is circuit breaker pattern?
**A:** **States:** Closed (normal) → Open (failing after 5 errors) → Half-open (testing after 60s timeout). Open circuit rejects calls to prevent cascade. Half-open tries recovery. Used for LLM APIs, tools, databases.

**Q14:** Retry strategy with exponential backoff?
**A:** Retry delays: 1s, 2s, 4s, 8s (max 3 retries). Base delay × 2^attempt, max 60s. Only retry transient errors (timeout, network, rate limit). Don't retry permanent failures (validation, not found, permissions).

**Q15:** Debug agent timeout - systematic approach?
**A:** 1) **Metrics:** Check timeout rate, which users/steps. 2) **Traces:** Find slow spans (LLM? tool? sandbox?). 3) **Logs:** Pattern in failing tasks. 4) **Reproduce:** Run locally with debugging. 5) **Fix:** Add timeout + fallback or optimize slow component. 6) **Verify:** Deploy canary, monitor improvement.

## Cost Optimization

**Q16:** Token budget implementation?
**A:** Track tokens used per request. Check: `if used + estimated > budget: return error`. Budget: 10K-50K tokens typical (GPT-4: $0.30-$1.50). Prevents runaway costs. Set per-user and per-request limits.

**Q17:** Cost optimization strategies?
**A:** 1) **Caching** (20-40% savings), 2) **Smaller models** (GPT-3.5 for 90% of tasks: 90% cheaper), 3) **Prompt compression** (30% token reduction), 4) **Spot instances** (70% compute savings), 5) **Request deduplication**. Caching + smaller models biggest wins.

**Q18:** Typical cache hit rate and savings?
**A:** 20-40% hit rate in practice. Each hit saves full LLM call. Example: 40% hits on $10K/day LLM costs = $4K/day savings = $120K/year. Use Redis with TTL, hash(prompt + model) as key.

## Failure Scenarios

**Q19:** LLM provider (OpenAI) goes down - response strategy?
**A:** **Immediate:** Auto-fallback to Anthropic (circuit breaker opens after 5 failures). **Short-term:** Update status page, alert team, monitor fallback capacity. **Long-term:** Scale backup providers, analyze cost (+50%), communicate with users. Test quarterly with chaos engineering.

**Q20:** Multi-provider fallback implementation?
**A:** Priority order: OpenAI (1) → Anthropic (2) → Self-hosted (3). On failure: circuit breaker opens, try next provider. Track: which provider active, error rates, fallback success rate. Need pre-negotiated rate limits with backups.

**Q21:** Graceful degradation when all providers down?
**A:** 1) Use cached responses (if available), 2) Use smaller local model (Llama), 3) Return degraded mode message, 4) Queue requests for later. Don't just fail - provide best-effort response.

## Production Patterns

**Q22:** Hybrid deployment pattern (API + workers)?
**A:** **API:** FastAPI sync handler (<5s tasks) + job queue (long tasks). **Workers:** Celery processes pull from Redis queue, execute long-running agents. **Benefit:** Fast API response + handles long tasks + worker auto-scaling + retry logic.

**Q23:** Observability stack for production?
**A:** **Metrics:** Prometheus (counters, histograms). **Logs:** Structured logging (JSON) to ELK/Splunk. **Traces:** OpenTelemetry → Jaeger/Datadog. **Dashboards:** Grafana. **Alerts:** Prometheus Alertmanager → PagerDuty. All three required.

**Q24:** Container vs serverless cost example?
**A:** Chatbot: 1000 req/day, 3s avg. Serverless: $5/mo. Containers: $200/mo baseline. **Winner:** Serverless (95% savings). Code analysis: 50 RPS steady, 30s avg. Both ~$500/mo, but containers have better features. **Winner:** Containers.

## Capacity Planning

**Q25:** Calculate workers needed for 10K users?
**A:** Concurrent users: 10K. Avg step duration: 5s. Steps every: 30s. Concurrent tasks: (10K × 5s) / 30s = 1667. Target 80% utilization: 1667 / 0.8 = 2084 workers. If 10 workers per m5.2xlarge: 209 machines.

**Q26:** LLM API capacity for 10K users?
**A:** Steps/sec: 333. Tokens/step: 5K. Total: 1.67M tokens/sec. Cost at $0.03/1K: $50/sec = $130K/day. Optimization: Use GPT-3.5 for 90% → $13K/day (10x reduction). Critical to use smaller models.

**Q27:** Sandbox pool sizing?
**A:** 20% of steps use code. Code steps/sec: 333 × 0.2 = 66. Avg exec time: 3s. Concurrent: 66 × 3 = 200 sandboxes. Pre-warm 300 (headroom). Resource: 300 × 512MB = 150GB RAM = 5 × m5.2xlarge machines.

## Debugging

**Q28:** High error rate suddenly. Debug steps?
**A:** 1) **Dashboard:** When started? Which errors? Which users? 2) **Logs:** Error patterns, stack traces. 3) **Traces:** Find failing component (LLM? tool? DB?). 4) **Dependencies:** Check provider status pages. 5) **Recent changes:** Deployments? Config? 6) **Rollback** if recent deploy caused it.

**Q29:** P95 latency degraded. Investigation?
**A:** 1) **Metrics:** Which component slow (LLM, tools, sandbox)? 2) **Traces:** Analyze slow requests. 3) **Profiling:** CPU/memory/network usage. 4) **External:** Provider latency increased? 5) **Fix:** Optimize slow path, add timeout, or scale resources. 6) **Verify:** Monitor P95 after changes.

## Key Insights

- Framework: Custom = control, LangChain = speed, hybrid = best of both
- Deployment: Serverless (spiky, <15min) vs Containers (steady, >15min)
- Critical metrics: P95 latency, error rate, cost/request, cache hit rate
- Multi-provider fallback essential: OpenAI → Anthropic → self-hosted
- Circuit breakers prevent cascade failures, auto-recover after timeout
- Monitoring: Metrics + Logs + Traces = complete observability
- Cost optimization: Caching (40%), smaller models (90%), compression
- Capacity planning: Calculate concurrent tasks, size infrastructure appropriately

## Self-Test Checklist

Can you:
- [ ] Compare custom vs LangChain trade-offs?
- [ ] Choose serverless vs container deployment?
- [ ] List 5 critical agent system metrics?
- [ ] Size infrastructure for 10K concurrent users?
- [ ] Debug production timeout systematically?
- [ ] Implement multi-provider LLM fallback?
- [ ] Calculate cost per request?
- [ ] Design circuit breaker pattern?
- [ ] Set up distributed tracing?
- [ ] Handle provider outages gracefully?
