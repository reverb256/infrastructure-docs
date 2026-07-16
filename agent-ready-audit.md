# Agent-Ready Audit Report

**Generated:** 2026-05-18
**Scope:** All 12 repos monitored by Kelos TaskSpawners
**Total Open Issues:** 64
**Currently agent-ready:** 15

---

## What "Agent-Ready" Means

The `agent-ready` label is the trigger for Kelos to spawn an autonomous agent pod (opencode) that clones the repo, implements the issue, pushes a branch, and opens a PR. For this to work reliably, an issue must satisfy these criteria:

### Criteria Matrix

| Criterion | Pass | Fail |
|-----------|------|------|
| **Self-contained** | One session, no hand-offs | Needs human mid-task |
| **Clear AC** | Checkboxes or enumerated deliverables | Vague "make it better" |
| **Actionable** | Tells WHAT + WHERE (files, patterns) | Needs architectural decisions |
| **Sized right** | 30m - 4h | <5m trivial or >1d epic |
| **Dependencies met** | Credentials injected, no SSH/physical | Needs `nixos-rebuild`, `kubectl exec` on host |
| **Low risk** | PR reviewable, rollback easy | Could break prod silently |

---

## Repo-by-Repo Analysis

### 1. reverb256/maplespike — 28 open issues

**Already agent-ready (4):**
| # | Title | Priority | Status |
|---|-------|----------|--------|
| 250 | Apply network policies + resource quotas | p1 | **Running** |
| 138 | Journalism mode with editorial overrides | p2 | **Queued** (label re-added) |
| 119 | Phase 4.2: Entity Resolution Layer on SQLite | p1 | **Running** |
| 16 | Trace context propagation through MCP tools | p1 | **Running** |

**Recommend labelling agent-ready (17):**
| # | Title | Pri | Est | Why |
|---|-------|-----|-----|-----|
| **130** | Email notifications & alerting | p2 | 2-3h | Well-defined feature, clear deliverables list, p2 so low risk |
| **36** | Compliance dashboard | p1 | 2h | Specific deliverable (matrix, tracker, alerts, export) |
| **35** | OWASP category mapping | p1 | 1h | Documentation task, clear inputs/outputs |
| **34** | Model weight verification | p1 | 2h | Config + validation logic, specific files implied |
| **33** | MCP server integrity checks | p1 | 1h | Binary signing + verification, scoped |
| **32** | Container image digest pinning | p1 | 2h | Find-replace across config, well-defined |
| **29** | Rate limiting per client | p1 | 1h | Token bucket implementation, scoped |
| **27** | Restricted securityContext | p1 | 2h | Apply YAML / Nix config changes |
| **25** | Egress whitelist | p1 | 2h | Network policy + config |
| **24** | Agent network namespace | p1 | 2h | Namespace isolation implementation |
| **23** | Redaction logging | p1 | 1h | Specific: detect, redact, store, audit |
| **22** | Panic mode toggle | p1 | 30m | Simple: global kill switch |
| **21** | Per-skill action gate overrides | p1 | 1h | Config/metadata changes |
| **20** | Confirmation UI | p1 | 2h | UI + logging, well-scoped |
| **19** | Session-start memory diff | p1 | 1h | Snapshot + compare logic |
| **18** | TTL enforcement | p1 | 1h | Schema + cleanup job |
| **17** | Provenance tagging on memory | p1 | 2h | Tag schema + queries |

**NOT agent-ready (7):**
| # | Title | Reason |
|---|-------|--------|
| 122 | Sprint-to-Live v1 | Meta/epic umbrella, not executable |
| 120 | Phase 4.3: Graph Builder | Too terse — needs detailed AC, file paths |
| 117 | Phase 7: CI/CD Polish + White-Label | Multi-day epic, cross-cutting |
| 115 | Phase 6: SaaS / Public API + Billing | Multiple sub-systems (Stripe, console, quotas, SDKs) |
| 60 | Deploy Security Hardening | Cross-repo (#250 already extracted), needs vault + nixos-rebuild |
| 37 | Penetration testing framework | 3d estimate, complex framework |
| 26 | MicroVM for subagents (#26) | 4h + Firecracker integration, needs investigation first |

---

### 2. reverb256/nixos-config — 30 open issues

**Already agent-ready (8):**
| # | Title | Priority |
|---|-------|----------|
| 22 | MCP Registry Generate Configs | p1 |
| 21 | MCP Operational Tools (5 missing) | p1 |
| 17 | vLLM missing CUDA_VISIBLE_DEVICES | medium |
| 13 | CUDA_VISIBLE_DEVICES=0 on 3090-moe | **high** |
| 12 | Phase 8: Resource Limits | p1 |
| 10 | Phase 6: Image Tag Pinning | p1 |
| 9 | Phase 5: SA Token Cleanup | p1 |
| 7 | 3090-moe Service selector host=sentry→zephyr | **high** |

**Recommend labelling agent-ready (13):**
| # | Title | Pri | Why |
|---|-------|-----|-----|
| **50** | Context bridge directory | — | Well-defined: create dirs + README, update skills. Was agent-ready, needs re-label |
| **31** | Add oom-protect.nix to cluster | p2 | 30m module addition to Nix config |
| **30** | Configure Gitea Casdoor OAuth TLS | p2 | 30m config change to Nix |
| **29** | Configure Gitea TLS cert mounting | p2 | 1h, well-scoped |
| **28** | Replace colmena apply with permanent DNS | p2 | 1h, well-scoped Nix change |
| **33** | Block DNS tunneling for agents | p2 | 2h, security policy change |
| **27** | SearXNG→Brain Ingestion Pipeline | p1 | 3 steps, well-defined, no human needed |
| **24** | nixkube CSI verify pods start | p1 | Simple: verify DaemonSet pods on all nodes |
| **20** | Hermes↔Knowledge Fabric integration | p1 | 3 clear steps |
| **19** | Deploy Knowledge Fabric to K8s | p1 | Well-scoped deployment task |
| **16** | Hermes config model/provider mismatch | medium | Defined fix: rename provider, fix model key |
| **15** | Gateway image drift | medium | Single-line version bump in Nix config |
| **14** | Sentry replicas=0 in Nix but running | medium | Single-line `replicas = 1` change |

**NOT agent-ready or borderline (9):**
| # | Title | Reason |
|---|-------|--------|
| 55 | Test verification comment | Trivial (<1m), already completed pattern |
| 34 | Docs-to-Issues Migration | Meta/documentation, no code changes |
| 32 | nftables/iptables rules (#32) | Agent can't SSH to apply nftables |
| 23 | Casdoor Native MCP Auth (#23) | Needs Casdoor admin panel access |
| 25 | easykubenix manifest build fix | Root cause unknown, needs investigation |
| 11 | PSA Enforcement Labels (#11) | Well-scoped but could break pods; test in warn mode first |
| 8 | Monitoring Egress Restrictions (#8) | Well-scoped but multi-policy, could break monitoring |
| 6 | Zephyr node Unknown (#6) | Requires SSH to node, agent can't fix |
| 4 | 422 evicted pods (#4) | Kubectl delete — simple but tedious; better as CronJob |

---

### 3. reverb256/ai-inference-gateway — 5 open issues

**Recommend labelling agent-ready (3):**
| # | Title | Pri | Est | Why |
|---|-------|-----|-----|-----|
| **3** | Add cost comparison to model benchmark | p2 | 2h | File path given, clear logic (score diff <10 → pick cheaper) |
| **4** | Fix security filter event loop | p2 | 1h | File + line number, known bug pattern |
| **5** | Fix scorer async and re-enable | p2 | 2h | File + line number, fix + re-enable |

**NOT agent-ready (2):**
| # | Title | Reason |
|---|-------|--------|
| 7 | Kelos pipeline verification | Already completed, just needs cleanup |
| 6 | Docs-to-Issues Migration | Meta, no code change |

---

### 4. reverb256/compute-market — 1 open issue

| # | Title | Status | Action |
|---|-------|--------|--------|
| 1 | Mining exporter metrics loop only runs once | **Already agent-ready** | No change needed |

---

### 5-12. Remaining repos — 0 open issues

| Repo | Issues | Notes |
|------|--------|-------|
| caddy-ingress | 0 | — |
| gpu-proxy | 0 | — |
| knowledge-fabric | 0 | Issues may exist in code but not migrated yet |
| llama-cpp-turboquant | 0 | — |
| mcp-registry | 0 | — |
| searxng-cluster | 0 | — |
| Vane | 0 | — |
| vllm-turboquant | 0 | — |

---

## Summary

| Category | Count |
|----------|-------|
| **Already agent-ready** | 15 |
| **Recommend agent-ready** | 33 |
| **NOT agent-ready** | 16 |

### Top Priority Recommendations (label ASAP)

These are P1/high bugs with clear fixes that an agent can execute immediately:

1. **nixos-config #7** — 3090-moe Service selector wrong (already agent-ready) — **Running**
2. **nixos-config #13** — CUDA_VISIBLE_DEVICES=0 on wrong GPU (already agent-ready)
3. **nixos-config #17** — vLLM missing CUDA_VISIBLE_DEVICES (already agent-ready)
4. **nixos-config #14** — Sentry replicas mismatch — quick Nix fix
5. **nixos-config #15** — Gateway image drift — quick version bump
6. **nixos-config #16** — Hermes config model/provider mismatch
7. **ai-inference-gateway #4** — Security filter event loop — file + line given
8. **ai-inference-gateway #5** — Scorer async fix — file + line given
9. **maplespike #130** — Email notifications — well-scoped feature

### Quick Wins (under 1h, easy):
- maplespike #22 (Panic mode toggle, 30m)
- maplespike #35 (Category mapping, 1h)
- maplespike #18 (TTL enforcement, 1h)
- maplespike #19 (Memory diff, 1h)
- nixos-config #28 (Colmena DNS, 1h)
- nixos-config #29 (Gitea TLS mount, 1h)
- nixos-config #30 (Gitea Casdoor TLS, 30m)
- nixos-config #31 (oom-protect, 30m)

### Issues to Defer or Split:
- maplespike #120 (Graph Builder) — needs more detail before agent can run
- maplespike #60 (Security Hardening) — #250 was extracted, remaining bits need vault
- nixos-config #25 (easykubenix build) — unknown cause, needs human debugging first

### Resource Consideration

With 12 TaskSpawners all at maxConcurrency=2, up to 24 pods can run simultaneously. Each with 2Gi/2CPU limits (post-fix), that's 48Gi memory and 48 CPUs total — well within cluster capacity (sentry 31GB, nexus 46GB, forge 16GB, zephyr 31GB = 124GB). No bottleneck concern.
