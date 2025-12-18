---
layout: chapter
title: Tool Use & Function Calling Q&A
course_id: ai-agents
chapter_number: 6
---

**One-liner:** Test your understanding of tool design, validation, execution patterns, and error handling.

## Conceptual Questions

### Q1: Why is single responsibility important for tools?
<details>
<summary>Answer</summary>

**Problem with multi-mode tools:**
```python
# ❌ Bad - LLM struggles to use this correctly
def file_operation(path: str, operation: str, content: str = None):
    """Read, write, delete, or append files"""
    if operation == "read":
        return open(path).read()
    elif operation == "write":
        open(path, 'w').write(content)
    # etc...
```

**Why it fails:**
1. **Ambiguous parameters:** Does `content` apply to "delete"? LLM makes mistakes
2. **Poor error messages:** "Invalid parameters" - which parameter?
3. **Harder to validate:** Complex conditional logic
4. **Difficult to test:** Many code paths
5. **Worse composability:** Can't easily chain operations

**Better approach:**
```python
# ✅ Good - Clear, focused tools
def read_file(path: str) -> str:
    """Read file contents"""

def write_file(path: str, content: str) -> None:
    """Write content to file"""

def delete_file(path: str) -> None:
    """Delete a file"""
```

**Benefits:**
- LLM can combine: `write_file(path, read_file(path) + new_content)` for append
- Clear validation: Each tool has obvious required parameters
- Better errors: "read_file() failed: FileNotFound" vs "file_operation() failed"
- Easier to test and maintain

**Real-world:** Claude Code has 20+ file tools (read, write, edit, glob, grep) instead of one "file_manager" tool
</details>

### Q2: Compare JSON Schema validation vs Pydantic. When to use each?
<details>
<summary>Answer</summary>

| Aspect | JSON Schema | Pydantic |
|--------|-------------|----------|
| **Performance** | Faster (pure JSON) | Slower (Python objects) |
| **Type safety** | Runtime only | Compile-time + runtime |
| **Complexity** | Simple validation | Complex custom validators |
| **LLM integration** | Native (OpenAI, Anthropic) | Requires conversion |
| **Developer experience** | JSON objects | Python classes |

**JSON Schema - Use when:**
```python
# Simple validation, LLM-native
tool_schema = {
    "type": "object",
    "properties": {
        "code": {"type": "string", "maxLength": 10000},
        "timeout": {"type": "integer", "minimum": 1, "maximum": 300}
    },
    "required": ["code"]
}

# Fast, works directly with OpenAI/Anthropic
```

**Pydantic - Use when:**
```python
# Complex validation, Python-first
class GitCommitParams(BaseModel):
    message: str = Field(..., min_length=10, max_length=500)
    files: list[str] = Field(..., min_items=1)
    branch: str = Field(default="main")

    @validator('message')
    def validate_conventional_commits(cls, v):
        # Custom validation: "feat: ", "fix: ", etc.
        if not re.match(r'^(feat|fix|docs|style|refactor|test|chore):', v):
            raise ValueError("Must follow conventional commits format")
        return v

    @validator('files')
    def files_must_exist(cls, v):
        for f in v:
            if not os.path.exists(f):
                raise ValueError(f"File not found: {f}")
        return v
```

**Recommendation:**
- **API-first (LLM tools):** JSON Schema (simpler, faster, native support)
- **Python-first (complex validation):** Pydantic (type safety, custom validators)
- **Hybrid:** Use Pydantic, convert to JSON Schema for LLM

```python
# Best of both worlds
class ToolParams(BaseModel):
    # ... pydantic model ...

    @classmethod
    def to_json_schema(cls):
        return cls.schema()  # Auto-convert for LLM
```
</details>

### Q3: When should tools execute in parallel vs sequentially?
<details>
<summary>Answer</summary>

**Parallel execution criteria:**
1. **Independent:** No data dependencies between tools
2. **I/O-bound:** Network calls, file operations (not CPU-bound)
3. **Idempotent:** Order doesn't matter
4. **Safe:** No race conditions

**Sequential execution criteria:**
1. **Dependent:** Tool B needs output from Tool A
2. **Stateful:** Order matters (e.g., create file before writing)
3. **Shared resources:** Both modify same data
4. **Rate-limited:** API quotas, connection limits

**Examples:**

**✅ Parallelizable:**
```python
# Three independent reads - 3x speedup
results = await asyncio.gather(
    read_file("file1.txt"),
    read_file("file2.txt"),
    read_file("file3.txt")
)
```

**❌ Must be sequential:**
```python
# Tool 2 depends on Tool 1's output
search_results = web_search("Python tutorials")
summary = summarize_text(search_results)  # Needs search_results
```

**Mixed (partial parallelization):**
```python
# Step 1: Parallel reads
file1, file2, file3 = await asyncio.gather(
    read_file("a.txt"),
    read_file("b.txt"),
    read_file("c.txt")
)

# Step 2: Sequential processing (depends on reads)
combined = merge_content(file1, file2, file3)
result = analyze(combined)
```

**Performance impact:**
- 3 sequential I/O operations (1s each) = 3s total
- 3 parallel I/O operations = 1s total (3x speedup)

**Gotcha:** LLMs don't automatically parallelize - you must implement this in your agent framework
</details>

## Design Questions

### Q4: Design tools for an agent that manages AWS infrastructure
<details>
<summary>Answer</summary>

**Requirements:**
- Create/manage EC2 instances, S3 buckets, RDS databases
- Read-only queries for monitoring
- Safety: Prevent accidental deletions

**Tool design:**

**1. Read-only monitoring tools (safe, no approval needed):**
```python
tools = [
    {
        "name": "list_ec2_instances",
        "description": "List all EC2 instances with status",
        "parameters": {
            "region": {"type": "string", "default": "us-east-1"},
            "filters": {"type": "object", "description": "Tags to filter by"}
        },
        "permissions": ["READ"]
    },
    {
        "name": "get_instance_metrics",
        "description": "Get CPU, memory, network metrics for instance",
        "parameters": {
            "instance_id": {"type": "string"},
            "period": {"type": "string", "enum": ["1h", "24h", "7d"]}
        },
        "permissions": ["READ"]
    },
    {
        "name": "estimate_cost",
        "description": "Calculate estimated monthly cost",
        "parameters": {
            "instance_type": {"type": "string"},
            "storage_gb": {"type": "integer"}
        },
        "permissions": ["READ"]
    }
]
```

**2. Write tools (require approval):**
```python
{
    "name": "create_ec2_instance",
    "description": "Launch a new EC2 instance",
    "parameters": {
        "instance_type": {"type": "string", "enum": ["t2.micro", "t2.small", ...]},
        "ami_id": {"type": "string"},
        "tags": {"type": "object"}
    },
    "permissions": ["WRITE"],
    "requires_approval": True,
    "cost_estimate": True  # Show cost before approval
}
```

**3. Dangerous tools (extra confirmation):**
```python
{
    "name": "terminate_instance",
    "description": "Terminate an EC2 instance (DESTRUCTIVE)",
    "parameters": {
        "instance_id": {"type": "string"},
        "confirmation": {"type": "string", "pattern": "^delete-[a-z0-9]+$"}
    },
    "permissions": ["ADMIN"],
    "requires_approval": True,
    "require_confirmation_code": True  # User must type "delete-{id}"
}
```

**Safety layers:**

```python
class InfrastructureTool:
    def execute(self, tool_name: str, params: dict):
        # Layer 1: Validate parameters
        validate_schema(params, tool_schema)

        # Layer 2: Check permissions
        if tool.permissions in [ToolPermission.WRITE, ToolPermission.ADMIN]:
            if not self.request_approval(tool_name, params):
                return {"error": "User denied approval"}

        # Layer 3: Dry run for destructive actions
        if tool_name == "terminate_instance":
            dry_run_result = aws.terminate_instance(params, dry_run=True)
            if not confirm(f"This will {dry_run_result}. Continue?"):
                return {"error": "Operation cancelled"}

        # Layer 4: Rate limiting
        if not self.rate_limiter.check(tool_name, max_calls=10, window=60):
            return {"error": "Rate limit exceeded"}

        # Execute
        return tool.execute(params)
```

**Tool organization:**
- Total tools: ~20-30
- Read-only: 15 (list, get, describe, estimate)
- Write: 10 (create, update, modify)
- Destructive: 5 (delete, terminate, destroy)

**Trade-offs:**
- More tools = better safety (fine-grained control)
- Fewer tools = simpler for LLM to use
- **Recommendation:** Err on side of more tools for infrastructure
</details>

### Q5: How would you implement tool result caching?
<details>
<summary>Answer</summary>

**Cache key strategy:**
```python
import hashlib
import json

def cache_key(tool_name: str, params: dict) -> str:
    # Deterministic serialization
    key_string = f"{tool_name}:{json.dumps(params, sort_keys=True)}"
    return hashlib.sha256(key_string.encode()).hexdigest()

# Examples:
# read_file({"path": "/tmp/data.txt"}) → "a3f5..."
# web_search({"query": "Python"}) → "b8e2..."
```

**Cacheable vs non-cacheable:**
```python
CACHEABLE_TOOLS = {
    # Read operations - always cacheable
    "read_file": 3600,           # 1 hour TTL
    "list_files": 300,            # 5 min TTL (directory might change)
    "web_search": 86400,          # 24 hours
    "get_user_profile": 3600,

    # API calls (GET) - cacheable with short TTL
    "github_get_repo": 600,       # 10 min
    "fetch_stock_price": 60,      # 1 min (volatile data)
}

NON_CACHEABLE_TOOLS = [
    "write_file",         # Side effects
    "send_email",         # Must execute each time
    "execute_code",       # Results may vary
    "delete_file",        # Destructive
    "current_time",       # Changes constantly
]
```

**Implementation:**
```python
import redis
import time

class ToolCache:
    def __init__(self):
        self.redis = redis.Redis()

    def get(self, tool_name: str, params: dict):
        if tool_name in NON_CACHEABLE_TOOLS:
            return None

        key = cache_key(tool_name, params)
        cached = self.redis.get(key)

        if cached:
            data = json.loads(cached)
            if data["expires_at"] > time.time():
                return data["result"]
            else:
                self.redis.delete(key)  # Expired

        return None

    def set(self, tool_name: str, params: dict, result):
        if tool_name not in CACHEABLE_TOOLS:
            return

        ttl = CACHEABLE_TOOLS[tool_name]
        key = cache_key(tool_name, params)

        data = {
            "result": result,
            "expires_at": time.time() + ttl,
            "cached_at": time.time()
        }

        self.redis.setex(key, ttl, json.dumps(data))

# Usage
cache = ToolCache()

def execute_tool(tool_name: str, params: dict):
    # Check cache
    cached_result = cache.get(tool_name, params)
    if cached_result:
        return cached_result

    # Execute
    result = tools[tool_name].execute(params)

    # Cache if successful
    if result.get("success"):
        cache.set(tool_name, params, result)

    return result
```

**Cache invalidation strategies:**
```python
# 1. Time-based (TTL)
cache.setex(key, ttl=3600, value=result)

# 2. Event-based
def write_file(path: str, content: str):
    result = _write_file_impl(path, content)

    # Invalidate read_file cache for this path
    cache.delete(cache_key("read_file", {"path": path}))

    return result

# 3. Manual invalidation
@tool
def clear_cache(tool_name: str = None):
    """Clear tool cache (all or specific tool)"""
    if tool_name:
        pattern = f"cache:{tool_name}:*"
        keys = redis.keys(pattern)
        redis.delete(*keys)
    else:
        redis.flushdb()  # Clear all
```

**Performance impact:**
```
Without cache:
- read_file: 50ms
- web_search: 1000ms
- api_call: 500ms

With cache (hit):
- read_file: 2ms (25x faster)
- web_search: 2ms (500x faster!)
- api_call: 2ms (250x faster)

Cache hit rate: 60-80% in practice
```

**Edge cases:**
1. **Large results:** Don't cache if result > 1MB
2. **User-specific:** Include user_id in cache key
3. **Non-deterministic tools:** Never cache (e.g., random_number)
</details>

## Scenario Questions

### Q6: LLM keeps calling the same failing tool repeatedly. How to handle?
<details>
<summary>Answer</summary>

**Problem:** Tool fails → LLM retries → Fails again → Infinite loop

**Solution layers:**

**1. Track recent failures:**
```python
class ToolExecutor:
    def __init__(self):
        self.failure_history = defaultdict(list)

    def execute(self, tool_name: str, params: dict):
        # Check recent failures
        recent_failures = self.failure_history[tool_name]
        recent_failures = [f for f in recent_failures if time.time() - f < 60]

        if len(recent_failures) >= 3:
            return {
                "error": f"Tool {tool_name} has failed 3 times in the last minute. "
                         "Please try a different approach.",
                "suggestion": self.suggest_alternative(tool_name)
            }

        # Execute tool
        result = self._execute_impl(tool_name, params)

        # Track failure
        if not result.get("success"):
            self.failure_history[tool_name].append(time.time())

        return result
```

**2. Suggest alternatives:**
```python
def suggest_alternative(self, tool_name: str) -> str:
    alternatives = {
        "read_file": "Try list_files() to see available files, or use grep to search",
        "web_search": "Try asking human for information, or use cached documentation",
        "execute_code": "Try breaking into smaller steps, or ask human to debug"
    }
    return alternatives.get(tool_name, "Try a different approach")
```

**3. Circuit breaker:**
```python
class CircuitBreaker:
    def __init__(self, failure_threshold=5, timeout=60):
        self.failures = defaultdict(int)
        self.last_failure = {}
        self.threshold = failure_threshold
        self.timeout = timeout

    def call(self, tool_name: str, func):
        # Check if circuit is open
        if self.failures[tool_name] >= self.threshold:
            time_since_failure = time.time() - self.last_failure[tool_name]
            if time_since_failure < self.timeout:
                return {
                    "error": f"Circuit breaker open for {tool_name}. "
                             f"Tool disabled for {self.timeout}s due to repeated failures."
                }

            # Reset after timeout
            self.failures[tool_name] = 0

        # Execute
        try:
            result = func()
            if result.get("success"):
                self.failures[tool_name] = 0  # Reset on success
            else:
                self.failures[tool_name] += 1
                self.last_failure[tool_name] = time.time()
            return result
        except Exception as e:
            self.failures[tool_name] += 1
            self.last_failure[tool_name] = time.time()
            raise
```

**4. Inject context into LLM:**
```python
# After 2 failures, add to system prompt:
if failure_count >= 2:
    inject_message = f"""
    IMPORTANT: The {tool_name} tool has failed {failure_count} times.
    DO NOT call it again. Instead:
    1. Try an alternative approach
    2. Ask the human for help
    3. Skip this step if not critical
    """
    context.add_system_message(inject_message)
```

**Real-world example (Claude Code):**
If Read tool fails 3x → Suggests using Grep or Glob instead
</details>

### Q7: Design rate limiting for tools. What granularity and limits?
<details>
<summary>Answer</summary>

**Rate limit dimensions:**

**1. Per tool type:**
```python
RATE_LIMITS = {
    # Expensive API calls
    "web_search": (10, 60),        # 10 calls per 60 seconds
    "image_generation": (5, 60),    # 5 per minute
    "email_send": (20, 3600),       # 20 per hour

    # Moderate cost
    "code_execution": (50, 60),     # 50 per minute
    "api_call": (100, 60),

    # Cheap operations
    "read_file": (1000, 60),        # Essentially unlimited
    "list_files": (500, 60),
}
```

**2. Per user:**
```python
USER_LIMITS = {
    "free_tier": {
        "total_calls": (100, 3600),      # 100 calls/hour total
        "expensive_tools": (10, 3600)     # 10 expensive/hour
    },
    "pro_tier": {
        "total_calls": (1000, 3600),
        "expensive_tools": (100, 3600)
    }
}
```

**3. Per session:**
```python
SESSION_LIMITS = {
    "max_total_calls": 500,          # Prevent runaway agents
    "max_same_tool_calls": 50,       # Same tool repeatedly
}
```

**Implementation:**
```python
from collections import defaultdict
import time

class RateLimiter:
    def __init__(self):
        self.calls = defaultdict(lambda: defaultdict(list))

    def check_limit(
        self,
        key: str,         # "tool:web_search:user:123"
        max_calls: int,
        window: int
    ) -> tuple[bool, str]:
        now = time.time()

        # Clean old calls outside window
        self.calls[key] = [
            ts for ts in self.calls[key]
            if now - ts < window
        ]

        # Check limit
        if len(self.calls[key]) >= max_calls:
            retry_after = window - (now - self.calls[key][0])
            return False, f"Rate limit exceeded. Retry after {retry_after:.0f}s"

        # Allow and record
        self.calls[key].append(now)
        return True, None

# Usage
rate_limiter = RateLimiter()

def execute_tool(tool_name: str, user_id: str, params: dict):
    # Check multiple dimensions
    checks = [
        # Tool-specific limit
        (f"tool:{tool_name}", *RATE_LIMITS[tool_name]),

        # User total limit
        (f"user:{user_id}", *USER_LIMITS[user_tier]["total_calls"]),

        # Expensive tools for this user
        (f"user:{user_id}:expensive", *USER_LIMITS[user_tier]["expensive_tools"])
        if tool_name in EXPENSIVE_TOOLS else (None, None, None)
    ]

    for key, max_calls, window in checks:
        if key is None:
            continue

        allowed, error = rate_limiter.check_limit(key, max_calls, window)
        if not allowed:
            return {"error": error, "retry_after": window}

    # Execute tool
    return tool.execute(params)
```

**Graceful degradation:**
```python
def execute_with_fallback(tool_name: str, params: dict):
    # Try primary tool
    allowed, error = rate_limiter.check_limit(tool_name, max_calls, window)

    if not allowed:
        # Try fallback
        fallback = get_fallback_tool(tool_name)
        if fallback:
            return execute_tool(fallback, params)

        # No fallback - queue for later
        return {
            "status": "queued",
            "message": "Rate limited, queued for execution",
            "retry_after": extract_retry_time(error)
        }

    return execute_tool(tool_name, params)

FALLBACKS = {
    "web_search_google": "web_search_bing",  # If Google rate limited
    "gpt4": "gpt3.5",                         # If GPT-4 rate limited
}
```

**Monitoring & alerts:**
```python
# Alert if user hitting limits frequently
if rate_limit_hit_count > 10:
    alert("User {} hitting rate limits: {}", user_id, tool_name)
    consider_tier_upgrade(user_id)
```
</details>

## Key Takeaways

- **Single responsibility:** One tool = one operation, LLMs compose better
- **Validation is mandatory:** JSON Schema or Pydantic, always validate inputs
- **Structured responses:** Always return success/error + actionable messages
- **Parallel execution:** 3-10x speedup for independent I/O-bound tools
- **Caching essential:** 500x speedup for repeated expensive operations (web_search)
- **Circuit breakers:** Stop infinite retry loops after 3-5 failures
- **Rate limiting:** Per-tool, per-user, per-session - multi-dimensional
- **Permission system:** READ/WRITE/EXECUTE/ADMIN - human approval for destructive

## Self-Test Checklist

Can you:
- [ ] Design a tool schema with proper validation?
- [ ] Explain when to use JSON Schema vs Pydantic?
- [ ] Implement tool result caching with TTL?
- [ ] Handle repeated tool failures gracefully?
- [ ] Design rate limiting for multiple dimensions?
- [ ] Determine when tools can execute in parallel?
- [ ] Create actionable error messages?
- [ ] Implement a permission system for tools?
