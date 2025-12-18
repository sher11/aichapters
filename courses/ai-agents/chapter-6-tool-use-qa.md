---
layout: chapter
title: Tool Use & Function Calling Q&A
course_id: ai-agents
chapter_number: 6
---

**Quick Revision:** Test your understanding of tool design, validation, and execution patterns.

## Tool Design Principles

**Q1:** Why is single responsibility important for tools?

**A:** LLM can compose simple tools but struggles with complex multi-mode functions. `read_file()` + `write_file()` better than `file_operation(path, mode, ...)`. Clearer parameters, better errors, easier validation, more composable.

**Q2:** Three principles of good tool design?

**A:** 1) **Single responsibility** (one tool = one operation), 2) **Explicit parameters** (no magic strings), 3) **Structured responses** (JSON with success/error/data). Makes LLM usage predictable and composable.

**Q3:** What makes a good tool error message?

**A:** **Actionable guidance:** Include error + suggestion + next steps. Bad: "Failed". Good: "FileNotFoundError: config.json. Suggestion: Use list_files() to see available files". LLM needs to know what to try next.

## Validation

**Q4:** JSON Schema vs Pydantic - when to use each?

**A:** **JSON Schema:** Simple validation, native LLM support (OpenAI/Anthropic), faster. **Pydantic:** Complex validation, custom validators, type safety. Use JSON Schema for API-first, Pydantic for Python-first with complex rules.

**Q5:** Why is parameter validation mandatory?

**A:** Prevent bad inputs: hallucinated parameters, wrong types, missing required fields. Use JSON Schema or Pydantic. Validation catches errors before execution, provides clear feedback to LLM.

**Q6:** Example of Pydantic custom validation?

**A:**
```python
@validator('message')
def validate_conventional_commits(cls, v):
    if not re.match(r'^(feat|fix|docs):', v):
        raise ValueError("Must follow conventional commits")
    return v
```
Use for: business rules, cross-field validation, external checks.

## Execution Patterns

**Q7:** When can tools execute in parallel?

**A:** When: 1) **Independent** (no data dependencies), 2) **I/O-bound** (network/file operations), 3) **Idempotent** (order doesn't matter), 4) **No shared state**. Example: Reading 3 different files. Speedup: 3x.

**Q8:** When must tools be sequential?

**A:** When: 1) **Dependent** (Tool B needs Tool A output), 2) **Stateful** (order matters: create before write), 3) **Shared resources** (both modify same data), 4) **Rate-limited** (API quotas). Can't parallelize.

**Q9:** How much speedup from parallel execution?

**A:** 3 sequential 1s I/O operations = 3s. Parallel = 1s (3x speedup). Only works for independent I/O-bound operations. CPU-bound tasks don't benefit as much.

## Caching

**Q10:** How to implement tool result caching?

**A:** Cache key: `hash(tool_name + JSON.stringify(params))`. Check cache before execution. Store with TTL. Cacheable: deterministic operations (read_file, web_search). Non-cacheable: side effects (write_file, send_email).

**Q11:** What's cacheable vs non-cacheable?

**A:** **Cacheable:** read_file (1hr TTL), web_search (24hr), api_get (10min). **Non-cacheable:** write_file, send_email, execute_code, current_time. If result changes or has side effects, don't cache.

**Q12:** Typical cache hit rate and savings?

**A:** 40-60% hit rate typical. Savings: web_search 500x faster (1000ms → 2ms), api_call 250x faster. Can save 40% of costs. Use Redis for distributed caching.

## Error Handling

**Q13:** LLM keeps calling same failing tool. Solution?

**A:** **Circuit breaker:** Track recent failures. After 3 failures in 60s, return error + suggestion. "Tool X failed 3x, try alternative". Inject context to LLM: "Stop calling X, it's failing". Real example: Read fails → suggest Grep/Glob.

**Q14:** What is a circuit breaker pattern?

**A:** **States:** Closed (normal) → Open (failing, reject calls) → Half-open (testing recovery). After N failures, open circuit. After timeout, try half-open. Success → close. Prevents cascade failures.

**Q15:** Retry strategy for tools?

**A:** Exponential backoff: 1s, 2s, 4s, 8s (max 3 retries). Only retry transient errors (timeout, network, rate limit). Don't retry: validation errors, not found, permissions. Check `is_retryable()`.

## Rate Limiting

**Q16:** Rate limiting dimensions?

**A:** 1) **Per tool** (web_search: 10/min), 2) **Per user** (total: 100/hr), 3) **Per session** (max 500 calls total), 4) **Expensive tools** (image_gen: 5/min). Multi-dimensional limits.

**Q17:** Rate limiting per tool type - examples?

**A:** **Expensive:** web_search (10/60s), image_gen (5/60s), email (20/hr). **Moderate:** code_exec (50/60s), api_call (100/60s). **Cheap:** read_file (1000/60s), list (500/60s). Based on cost/load.

**Q18:** What happens when rate limit hit?

**A:** Return error with retry-after time. Try fallback tool if available (Google → Bing search). Or queue for later execution. Don't just fail - provide alternative path or delay.

## Permissions & Safety

**Q19:** Tool permission levels?

**A:** **READ** (safe, always allow), **WRITE** (modifies data, needs approval), **EXECUTE** (runs code, dangerous), **NETWORK** (external calls), **ADMIN** (destructive, requires confirmation). Check permissions before execution.

**Q20:** When to require human-in-loop?

**A:** WRITE/EXECUTE/ADMIN permissions AND not auto-approved. Show: "Agent wants to delete_file(important.txt). Approve? [Yes/No/Yes to all]". Let user control destructive actions.

**Q21:** How to handle expensive tool usage?

**A:** 1) Rate limiting, 2) Budget tracking (token/cost limits), 3) User tier limits (free: 10/day, pro: 1000/day), 4) Approval for expensive operations, 5) Fallback to cheaper alternatives.

## Tool Discovery

**Q22:** How to select relevant tools from 100+ options?

**A:** **Semantic search:** Embed tool descriptions, embed task, find top-K similar tools. Example: "analyze sales data" → [read_csv, analyze_data, plot_chart, generate_report]. Reduces LLM context, improves accuracy.

**Q23:** Good tool description format?

**A:** Clear, concise, includes: What it does, required parameters, example use case. "Execute Python code in sandboxed environment. Returns stdout, stderr, exit code. Use for: calculations, data processing."

## Design Scenarios

**Q24:** Design tools for AWS infrastructure agent.

**A:** **Read-only** (15 tools): list_instances, get_metrics, estimate_cost. **Write** (10 tools): create_instance, update_config (requires approval). **Destructive** (5 tools): terminate_instance (requires approval + confirmation code). More tools = better safety.

**Q25:** Tool result cache implementation?

**A:** Redis with TTL. Key: `SHA256(tool_name + params)`. Before execution: check cache. After: store if cacheable and successful. TTL based on volatility: read_file (1hr), web_search (24hr), stock_price (1min).

**Q26:** Implement circuit breaker for tools?

**A:** Track failures per tool. Threshold: 5 failures. After threshold: **Open** (reject calls for 60s). After timeout: **Half-open** (try one call). Success → **Closed**. Failure → **Open** again.

## Patterns & Anti-patterns

**Q27:** Good tool design pattern?

**A:**
```python
# ✅ Good
def read_file(path: str) -> str
def write_file(path: str, content: str) -> None
# Clear, single purpose, composable
```

**Q28:** Bad tool design anti-pattern?

**A:**
```python
# ❌ Bad
def file_operation(path: str, operation: str, content: str = None)
# Too many modes, ambiguous parameters, hard to validate
```

**Q29:** Three common tool mistakes?

**A:** 1) **Overly complex** (too many parameters/modes), 2) **Poor errors** ("Failed" with no guidance), 3) **No timeouts** (can hang forever). Keep tools simple, errors actionable, always timeout.

## Key Insights

- Single responsibility: one tool = one operation, LLMs compose better
- Validation mandatory: JSON Schema or Pydantic, always validate
- Structured responses: success/error + actionable messages
- Parallel execution: 3-10x speedup for independent I/O-bound tools
- Caching essential: 500x speedup for repeated operations
- Circuit breakers: stop infinite retry loops after 3-5 failures
- Rate limiting: per-tool, per-user, per-session - multi-dimensional
- Permissions: READ/WRITE/EXECUTE/ADMIN - human approval for destructive

## Self-Test Checklist

Can you:
- [ ] Design tool schema with validation?
- [ ] Explain JSON Schema vs Pydantic trade-offs?
- [ ] Implement tool result caching with TTL?
- [ ] Handle repeated tool failures with circuit breaker?
- [ ] Design rate limiting for multiple dimensions?
- [ ] Determine when tools can execute in parallel?
- [ ] Create actionable error messages?
- [ ] Implement permission system for tools?
