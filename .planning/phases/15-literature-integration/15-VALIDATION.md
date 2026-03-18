---
phase: 15
slug: literature-integration
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-18
---

# Phase 15 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Manual verification (Claude Code Skill project — no executable test framework) |
| **Config file** | N/A — Skills are prompt documents, not executable code |
| **Quick run command** | Manual: review SKILL.md changes for completeness |
| **Full suite command** | Manual: invoke repo-to-paper-skill on a test repo, full guided workflow |
| **Estimated runtime** | ~5 minutes (manual review) |

---

## Sampling Rate

- **After every task commit:** Manual review of SKILL.md changes for correctness and completeness
- **After every plan wave:** Invoke the full repo-to-paper-skill workflow on a test repo to verify Step 2.5 integration
- **Before `/gsd:verify-work`:** Full manual walkthrough must confirm all success criteria
- **Max feedback latency:** ~5 minutes

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 15-01-01 | 01 | 1 | REPO-04 | manual | Review SKILL.md contains Step 2.5 with pre-flight check | N/A — Skill | ⬜ pending |
| 15-01-02 | 01 | 1 | REPO-04 | manual | Review SKILL.md contains batch search loop with progress display | N/A — Skill | ⬜ pending |
| 15-01-03 | 01 | 1 | REPO-04 | manual | Review SKILL.md contains ref card format and file write pattern | N/A — Skill | ⬜ pending |
| 15-01-04 | 01 | 1 | REPO-04 | manual | Review SKILL.md contains MCP fallback with `[CITATION NEEDED]` | N/A — Skill | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

*Existing infrastructure covers all phase requirements. No test framework to install — this phase modifies an existing Skill (SKILL.md), which is a prompt document.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Step 2.5 triggers after H2 confirmation | REPO-04 | Skill is a prompt document, not executable code | Invoke repo-to-paper-skill on test repo; confirm H2; verify Step 2.5 runs |
| MCP pre-flight check succeeds and batch search runs | REPO-04 | Requires live MCP connection | Run Skill with Semantic Scholar MCP available; observe search calls |
| Ref files created in `.paper-refs/` with correct format | REPO-04 | Requires Skill execution on real repo | Check `.paper-refs/` directory for per-H1 files with Markdown card format |
| MCP failure produces `[CITATION NEEDED]` placeholders | REPO-04 | Requires simulating MCP unavailability | Disconnect MCP; run Skill; verify placeholders and graceful continuation |

---

## Validation Sign-Off

- [ ] All tasks have manual verify instructions documented
- [ ] Sampling continuity: manual review after each task commit
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 300s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
