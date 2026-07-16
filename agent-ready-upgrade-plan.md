# Agent-Ready Upgrade Plan

**Date:** 2026-05-18  
**Previous passes completed:** 2 (created #250-260, enriched #31/#50/#130)

---

## Phase 1: Quick Wins — Label Already-Ready Issues

These issues already have sufficient detail for an agent. They just need the `agent-ready` label.

### maplespike (10 issues)

| # | Title | Prereq | Notes |
|---|-------|--------|-------|
| **130** | Email notifications | Enriched: full architecture spec added | Body now has file paths, schema, templates, acceptance criteria, implementation order. Just needs `agent-ready` label. |
| **17-36** | Phase 9 OWASP issues (17 issues) | All have Context + Task + Estimate + OWASP Category | Already well-structured per the Co-Authored template. #26 (4h microVM) is borderline — label the rest. |

Strategy: Label all OWASP issues **except** #26 (microVM, 4h) and #37 (penetration testing, 3d). That's 17 issues labelled, skipped 2.

### nixos-config (11 issues)

| # | Title | Why Ready |
|---|-------|-----------|
| **14** | Sentry replicas=0 in Nix | Single-line fix, exact file + line given |
| **15** | Gateway image drift | Single-line version bump, exact file given |
| **16** | Hermes config mismatch | 3 clearly defined fix options |
| **27** | SearXNG→Brain pipeline | 3 clear steps, referenced doc |
| **28** | Replace colmena DNS | 4 clear steps, well-scoped |
| **29** | Gitea TLS cert mounting | 4 clear steps |
| **30** | Gitea Casdoor OAuth TLS | 4 clear steps |
| **33** | Block DNS tunneling | 4 clear deliverables |
| **20** | Hermes↔Knowledge Fabric | 3 clear steps |
| **19** | Deploy Knowledge Fabric | 3 clear steps |
| **24** | nixkube CSI verify | Simple verification, single deliverable |

Strategy: #14, #15 are risk-free single-line Nix changes. #28-30, #33 are p2 infra tasks. #20, #19, #24, #27 are p1 tasks with clear steps.

### ai-inference-gateway (3 issues)

| # | Title | Why Ready |
|---|-------|-----------|
| **3** | Cost comparison | File + line + logic described |
| **4** | Fix security filter | File + line + bug described |
| **5** | Fix scorer async | File + line + issue described |

All have exact file paths and line numbers. Just need `agent-ready` label.

---

## Phase 2: Enrich — Upgrade Issue Bodies for Agent Actionability

These issues need body enrichment before labelling. Follow the pattern used on #130, #31, and #50.

### Pattern for Enrichment

For each issue, add:
1. **Files section** — exact paths to create/modify
2. **Acceptance Criteria** — checkboxes for each deliverable
3. **Implementation Order** — numbered steps
4. **Existing Patterns to Follow** — links to similar code
5. **Verification** — how to test (build, test commands)

### maplespike (4 issues)

| # | Title | Current State | Enrichment Needed |
|---|-------|--------------|-------------------|
| **120** | Phase 4.3: Graph Builder | Terse: "Build cross-reference graph. Implement briefs/signals." | Add file paths (pipeline-core/, api-server/), AC for graph nodes/edges, RAG brief template format |
| **60** | Remaining: vault + verification | #250 extracted; vault access remains | Agent can't write to vault. Mark remaining items as manual-only. Close or defer. |
| **117** | Phase 7 umbrella | Now has sub-issues #254-257, #260 | Already decomposed. Close umbrella or keep as tracker only. |
| **115** | Phase 6 umbrella | Now has sub-issues #258-259 | Already decomposed. Close umbrella or keep as tracker only. |
| **122** | Sprint-to-Live v1 | Now has sub-issues #251-253 | Already decomposed. Close umbrella or keep as tracker only. |

### nixos-config (5 issues)

| # | Title | Enrichment Needed |
|---|-------|-------------------|
| **8** | Monitoring Egress Restrictions | Has files + 4 policies. Add example NetworkPolicy YAML, kubectl verify commands |
| **11** | PSA Enforcement Labels | Has namespaces + risk notes. Add kubectl label commands, verify commands, rollback steps |
| **32** | nftables/iptables rules | Terse: 4 bullets. Agent can't SSH — this needs host access. Flag as NOT agent-ready. |
| **25** | easykubenix build fix | Unknown root cause. Add debug steps + investigation template. |
| **23** | Casdoor Native MCP Auth | 4 steps, has URL ref. Add Casdoor API call examples, test verification. |

---

## Phase 3: Decompose — Split Epics Into Agent-Sized Issues

Issues that are too large for a single agent session. Split into sub-issues, keeping the originals as trackers.

### Candidate Epic Breakdown

| Original | Size | Sub-Issues Proposed |
|----------|------|---------------------|
| nixos-config #21: MCP Operational Tools (5 tools) | 5 deliverables | Split into 5 issues (one per tool): rollback_host, config_diff, check_storage, check_network, check_secrets. **Already running as a single task** — let it complete, then split remaining PRs. |
| nixos-config #22: MCP Registry C1-C6 | 6 deliverables | Currently running. Split into 6 issues after completion. |
| maplespike #37: Penetration testing framework | 3d | Too large. Split: #37a (test case templates), #37b (automated suite), #37c (report generation) |

---

## Phase 4: Prune — Issues That Cannot Be Agent-Ready

These issues either require host access the agent doesn't have, need human investigation, or are meta-trackers.

| Repo | # | Issue | Reason |
|------|---|-------|--------|
| nixos-config | 4 | 422 evicted pods | Needs `kubectl delete` + CronJob creation. Agent has K8s access so borderline — could work. |
| nixos-config | 6 | Zephyr node Unknown | Requires SSH to zephyr node. Agent can't do this. |
| nixos-config | 32 | nftables/iptables rules | Requires SSH + `nixos-rebuild` on all hosts. Agent can't do this. |
| nixos-config | 34 | Docs-to-Issues Migration | Meta issue, no code changes. Close as completed. |
| maplespike | 117 | Phase 7: CI/CD Polish | Decomposed into #254-257, #260. Keep as tracker only. |
| maplespike | 115 | Phase 6: SaaS/Public API | Decomposed into #258-259. Keep as tracker only. |
| maplespike | 122 | Sprint-to-Live v1 | Decomposed into #251-253. Keep as tracker only. |
| maplespike | 60 | Deploy Security Hardening | #250 extracted; remaining vault + nixos-rebuild steps need human. Close or mark remaining manual. |
| ai-inf-gwy | 6 | Docs-to-Issues Migration | Meta, no code changes. Close as completed. |

---

## Execution Plan

### Phase 1 — Label (immediate, 30s per issue)
Apply `agent-ready` label to the 24 issues identified above.

### Phase 2 — Enrich (medium-term, 2-3h total)
Update issue bodies for #120, #8, #11, #25, #23 following the established pattern (file paths + AC + verification).

### Phase 3 — Decompose (ongoing)
Split #21, #22 into focused sub-issues after their current tasks complete.
Split #37 into 3 smaller issues (1-2h each).

### Phase 4 — Prune (cleanup)
Close #34, #6 (ai-inference-gateway) as completed.
Tag #6 (nixos-config), #32 as manual-only.
Mark #60, #117, #115, #122 as tracker-only.

---

## Summary

| Phase | Action | Count |
|-------|--------|-------|
| 1 ✅ | Label `agent-ready` | 27 issues — **DONE** |
| 2 | Enrich body | 5 issues |
| 3 | Decompose | 3 epics → ~14 sub-issues |
| 4 | Prune/close | 8 issues |

**Agent-ready now:** 42 (from 15 baseline)
**Remaining not labelled:** ~6 (meta issues, SSH-only tasks, or already tracks work of sub-issues)
**Phase 2-4 ready to execute when you are.**
