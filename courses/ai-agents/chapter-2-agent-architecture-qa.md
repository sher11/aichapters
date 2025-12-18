---
layout: chapter
title: Agent Architecture Q&A
course_id: ai-agents
chapter_number: 2
---

**Quick Revision:** Test your understanding of AI agent fundamentals and patterns.

## Core Components

**Q1:** What are the four core components of an AI agent?
**A:** 1) Perception Layer (input processing), 2) Reasoning Engine (LLM decision-making), 3) Action Layer (tool execution), 4) Memory Systems (context management). Missing any significantly reduces autonomy.

**Q2:** What is the agent execution loop?
**A:** **Observe → Think → Act → Repeat** until task complete. Mnemonic: OTAR.

**Q3:** Three types of agent memory?
**A:** 1) Short-term (conversation context), 2) Long-term (vector DB for experiences), 3) Working memory (current task state). Mnemonic: SLW.

## Agent Patterns

**Q4:** Compare ReAct vs Tool-calling patterns.
**A:** **ReAct:** Interpretable (explicit reasoning), slower, token-heavy. Good for: complex reasoning, debugging. **Tool-calling:** Fast, native support, less interpretable. Good for: production APIs, simple operations.

**Q5:** When to use multi-agent vs monolithic agent?
**A:** **Multi-agent** when: Specialized tasks, parallel work needed, scale requirements. **Monolithic** when: Simple tasks, tight integration, lower overhead. Multi-agent scales better for complex systems.

**Q6:** What is the ReAct pattern structure?
**A:** Thought → Action → Observation → Thought → ... repeating. Agent explicitly states reasoning before each action. More interpretable but slower than direct tool calling.

## Tool Design

**Q7:** Three principles of good tool design?
**A:** 1) **Atomic operations** (single responsibility), 2) **Idempotent** when possible (safe to retry), 3) **Structured I/O** (JSON schemas, clear errors).

**Q8:** Why structured error messages matter?
**A:** LLM needs actionable guidance. Bad: "Failed". Good: "FileNotFoundError: config.json. Try: list_files() to see available files". Include error + suggestion + next steps.

**Q9:** What makes a tool idempotent?
**A:** Same input → same output, no side effects. Example: read_file() is idempotent, write_file() is not. Safe to retry on failure without unintended consequences.

## Common Problems

**Q10:** What causes agent infinite loops?
**A:** 1) Doesn't recognize completion, 2) Tool failures not handled, 3) Ambiguous stopping conditions, 4) Context overflow (forgets attempts). **Fix:** Step limits, timeouts, duplicate action detection, circuit breakers.

**Q11:** How to prevent infinite loops?
**A:** 1) MAX_STEPS limit (e.g., 10 steps), 2) Timeout budget (e.g., 300s), 3) Duplicate action detection, 4) Circuit breaker (3 failures → stop). Use multiple layers.

**Q12:** Agent keeps retrying same failing tool. Solution?
**A:** **Circuit breaker:** After 3-5 failures, disable tool temporarily. Suggest alternatives. Track failure history. Inject context: "Tool X failed 3x, try different approach". Example: Read fails → suggest Grep/Glob.

## Design Scenarios

**Q13:** Design agent to debug production outages.
**A:** **Triage Agent** (analyze alerts) → **Investigation Agent** (read-only queries on logs/metrics) → **Root Cause Agent** (correlate findings) → **Human approval** before remediation. Key: All queries read-only, 5min timeout, audit trail.

**Q14:** Scale agent system to 10K concurrent users - bottlenecks?
**A:** 1) **LLM API rate limits** (use multi-provider), 2) **Sandbox creation** (pre-warm pool), 3) **Memory/context** (compression), 4) **Tool latency** (caching, parallel execution). Need ~100 LLM QPS, 200 sandboxes.

**Q15:** User asks to "optimize codebase" - too vague. Strategy?
**A:** 1) **Clarify:** What type (speed/memory/size)? Which part? Trade-offs? 2) **Provide options** if unclear 3) **Profile first** - don't guess. Get user approval on plan before implementing.

## Latency & Performance

**Q16:** Typical latency per agent step?
**A:** **5-15 seconds total:** LLM inference (1-3s), tool execution (0.5-2s), sandbox (2-10s if code). 90% of users tolerate 5s, drop-off increases 10% per additional second.

**Q17:** How to manage agent context window?
**A:** **Summarization** (compress old messages), **Pruning** (remove irrelevant), **Extraction** (key facts to long-term memory). Reserve: 10% system prompt, 10% tools, 60% conversation, 20% working memory.

**Q18:** What's a reasonable token budget per request?
**A:** **10K-50K tokens** for most tasks. Example: 3K prompt + 2K completion per step × 10 steps = 50K tokens = $1.50 (GPT-4). Set hard limits to control cost.

## Memory & Context

**Q19:** How does vector memory work for agents?
**A:** Agent experiences → embeddings → vector DB. On new task: similarity search retrieves relevant past context. Use cases: similar task retrieval, documentation RAG, error pattern matching.

**Q20:** When to use each memory type?
**A:** **Short-term:** Current conversation (always). **Long-term:** Patterns, documentation, past solutions (when experiences reusable). **Working memory:** Current task state, intermediate results (always).

## Key Insights

- Agent loop = Observe → Think → Act → Repeat
- ReAct (interpretable) vs Tool-calling (fast) - pick based on needs
- Sandboxing mandatory for code execution
- Multi-agent systems scale better than monolithic
- Error handling is 50% of production code
- Circuit breakers prevent infinite retry loops
- Latency compounds: optimize each step
- Context management critical for long conversations

## Self-Test Checklist

Can you:
- [ ] Explain agent execution loop (OTAR)?
- [ ] Compare ReAct vs Tool-calling?
- [ ] List three tool design principles?
- [ ] Prevent infinite loops (3 strategies)?
- [ ] Design multi-agent system?
- [ ] Size infrastructure for 10K users?
- [ ] Handle vague user requests?
- [ ] Manage context window effectively?
