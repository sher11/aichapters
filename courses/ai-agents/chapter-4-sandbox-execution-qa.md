---
layout: chapter
title: Sandbox Execution Q&A
course_id: ai-agents
chapter_number: 4
---

**Quick Revision:** Test your understanding of code sandboxing and isolation techniques.

## Sandboxing Basics

**Q1:** Why can't you use Python's `exec()` without sandboxing?

**A:** Zero isolation. Can delete files (`os.system('rm -rf /')`), access network, read sensitive files, infinite loops without timeout. Same process as your app with full system access. Only use with trusted code.

**Q2:** Five essential Docker security settings?

**A:** 1) `network_mode="none"` (no network), 2) Resource limits (CPU/memory/PIDs), 3) `read_only=True` (immutable FS), 4) `cap_drop=["ALL"]` (no capabilities), 5) Non-root user. Miss any = vulnerability.

**Q3:** What are Docker namespaces used for?

**A:** Process isolation. **PID** (can't see host processes), **NET** (isolated network), **MNT** (isolated filesystem), **USER** (root in container ≠ host root), **IPC** (inter-process isolation).

## Technology Comparison

**Q4:** Compare Docker vs Firecracker vs gVisor.

**A:** **Docker:** 1-3s boot, process isolation, good security, production default. **Firecracker:** 125ms boot, full VM, excellent security, AWS Lambda's choice. **gVisor:** 0.5s boot, syscall filtering, excellent security, high-security needs.

**Q5:** When to use Firecracker over Docker?

**A:** Cold-start critical applications (serverless, Lambda). 125ms boot vs 1-3s. Trade-off: Smaller ecosystem, AWS-focused. Docker still better for most production use.

**Q6:** What does gVisor provide that Docker doesn't?

**A:** Kernel-level syscall interception. Blocks dangerous syscalls before they reach host kernel. Better security than Docker's namespace isolation. Used by: Google Cloud Run, Replit.

## Container Pooling

**Q7:** Why is container pooling essential?

**A:** Speed. Cold start: 1-3s. Pool hit: 10-50ms (60x faster!). Pre-warm 10-20 containers, reuse between executions. Must reset state (clear /tmp, kill processes) between uses.

**Q8:** How does container pooling work?

**A:** 1) Pre-create N idle containers at startup, 2) Get from pool (10ms), 3) Execute code, 4) Reset state, 5) Return to pool. On miss: create new container (fallback to 1-3s).

**Q9:** What's the trade-off of container pooling?

**A:** **Pro:** 60x faster execution. **Con:** Memory overhead (20 containers × 512MB = 10GB). Must properly reset state or risk leakage between users. Worth it for production.

## Security

**Q10:** How to prevent network data exfiltration?

**A:** `network_mode="none"` (no interface). If network needed: HTTP proxy with allowlist, inspect all traffic, block sensitive data patterns (DLP), only allow specific domains.

**Q11:** What's the principle of least privilege for containers?

**A:** Run as non-root user, read-only filesystem, drop all capabilities, minimal syscall access. Bad: root with full permissions. Good: user:1000, read-only, cap_drop=["ALL"].

**Q12:** How to prevent fork bombs?

**A:** `--pids-limit=50`. Limits total processes. Prevents exponential process creation (`while True: os.fork()`). Without limit: can crash host system.

## Resource Limits

**Q13:** Multi-layer timeout strategy?

**A:** **Layer 1:** Container timeout (30s), **Layer 2:** Python signal.alarm (30s), **Layer 3:** External monitor (35s safety net). If one fails, others catch it. Defense in depth.

**Q14:** How to prevent disk space exhaustion?

**A:** 1) `--storage-opt size=1G` (limit writable layer), 2) `tmpfs` with size limit, 3) Inode limits (max files), 4) Monitor disk usage during execution. Prevents: writing 10K files, zip bombs.

**Q15:** Resource limits to always set?

**A:** `--memory=512m` (RAM), `--cpu-quota=100000` (1 core), `--pids-limit=50` (processes), `--storage-opt size=1G` (disk), timeout=30 (time). All mandatory for production.

## Performance

**Q16:** Container execution latency breakdown?

**A:** Cold: Container creation (1200ms) + code exec (500ms) + cleanup (30ms) = 1800ms. Warm: Pool fetch (50ms) + exec (500ms) = 600ms. Pooling gives 3x speedup.

**Q17:** How to optimize container performance?

**A:** 1) **Pool containers** (60x faster), 2) **Snapshot/restore** (Firecracker: 125ms), 3) **Copy-on-write FS** (share base image), 4) **Parallel tool execution**, 5) **Result caching**. Biggest win: pooling.

**Q18:** What's cacheable vs non-cacheable for sandboxes?

**A:** **Cacheable:** Deterministic operations (same input → same output). **Non-cacheable:** Code with side effects, time-dependent, random operations, network calls. Hash(code + language) → result for cache key.

## Debugging

**Q19:** Container is stuck/hanging. How to debug?

**A:** 1) `docker ps` (is it running?), 2) `docker stats` (CPU/memory usage), 3) `docker logs` (stdout/stderr), 4) `docker exec -it sh` (attach shell), 5) Check for: infinite loop (CPU 100%), deadlock (CPU 0%), waiting for input.

**Q20:** Common causes of container hangs?

**A:** 1) Infinite loop (CPU pegged), 2) Deadlock (CPU idle), 3) Waiting for input (`input()` in Python), 4) Long sleep (`time.sleep(1000000)`), 5) Fork bomb (PID limit should prevent).

## Real-World Examples

**Q21:** How does GitHub Copilot Workspace sandbox code?

**A:** Docker containers, no network, 30s timeout, read-only filesystem, 512MB memory, non-root user. Millions of executions/day. Industry-standard approach.

**Q22:** What makes AWS Lambda fast (125ms cold start)?

**A:** **Firecracker microVMs:** Full VM isolation with snapshot restore. Pre-warm VMs, restore from snapshot in 125ms. Memory overhead: ~5MB per VM. Enables serverless scale.

**Q23:** Why does Replit use gVisor?

**A:** User projects need network access (unlike pure code execution). gVisor provides better security than Docker while allowing network. Syscall filtering blocks dangerous operations.

## Scenarios

**Q24:** User code creates 10K files and fills disk. Prevention?

**A:** 1) `tmpfs` size limit (100MB), 2) Inode limit (1000 files), 3) Storage quota (1GB total), 4) Monitor disk usage (kill if >100MB), 5) FD limit (100 open files). Multi-layer defense.

**Q25:** Container execution takes 5s but users expect <1s. Optimize?

**A:** 1) **Pool containers** (1500ms → 50ms savings), 2) **Stream output** (perceived speed), 3) **Async execution** (return job_id immediately), 4) **Cache results** (identical code → 5ms), 5) **Parallel operations**.

**Q26:** Need to allow network access safely. How?

**A:** **Option 1:** Allowlist specific domains via HTTP proxy. **Option 2:** DLP proxy (inspect for secrets). **Option 3:** No network (safest). Start with none, add allowlist only when needed.

## Security Checklist

**Q27:** Must-have security settings for production?

**A:** ✅ `network_mode="none"` ✅ Resource limits (CPU/RAM/PIDs) ✅ `read_only=True` ✅ `cap_drop=["ALL"]` ✅ Non-root user ✅ Timeout ✅ Seccomp profile ✅ Storage quota.

**Q28:** What capabilities should NEVER be allowed?

**A:** `NET_ADMIN` (modify network), `SYS_ADMIN` (mount filesystems, load kernel modules), `SYS_PTRACE` (debug other processes). Drop all capabilities by default, add back minimal set if needed.

**Q29:** How to handle malicious code attempts?

**A:** Plan for: fork bombs (PID limit), disk exhaustion (storage quota), infinite loops (timeout), zip bombs (file size limits), network exfiltration (no network). Assume code is malicious.

## Key Insights

- Never use `exec()` without sandboxing - zero isolation
- Docker is production default - good security, mature ecosystem
- Five essentials: no network, resource limits, read-only, drop caps, non-root
- Container pooling mandatory for production - 60x faster
- Defense in depth: multiple timeout layers, resource monitors
- Firecracker for cold-start critical (AWS Lambda)
- gVisor for high security (syscall filtering)
- Plan for malicious code: fork bombs, disk exhaustion, infinite loops

## Self-Test Checklist

Can you:
- [ ] Explain why `exec()` is dangerous?
- [ ] List five critical Docker security settings?
- [ ] Describe container pooling benefits?
- [ ] Compare Docker vs Firecracker vs gVisor?
- [ ] Prevent disk exhaustion attacks?
- [ ] Debug a stuck container?
- [ ] Optimize container execution latency?
- [ ] Design network isolation strategy?
