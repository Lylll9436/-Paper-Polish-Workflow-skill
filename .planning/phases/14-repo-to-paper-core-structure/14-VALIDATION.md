---
phase: 14
slug: repo-to-paper-core-structure
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-18
---

# Phase 14 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Manual verification (pure markdown project — no test framework) |
| **Config file** | none |
| **Quick run command** | `grep -c "##" .claude/skills/repo-to-paper-skill/SKILL.md` |
| **Full suite command** | `wc -l .claude/skills/repo-to-paper-skill/SKILL.md references/repo-patterns.md` |
| **Estimated runtime** | ~1 second |

---

## Sampling Rate

- **After every task commit:** Run quick run command to verify file exists and has expected structure
- **After every plan wave:** Run full suite command to verify line counts
- **Before `/gsd:verify-work`:** All manual checks must pass
- **Max feedback latency:** 2 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 14-01-01 | 01 | 1 | REPO-08 | file check | `test -f references/repo-patterns.md && echo PASS` | ❌ W0 | ⬜ pending |
| 14-01-02 | 01 | 1 | REPO-08 | content check | `grep -c "Pattern\|Category\|Priority" references/repo-patterns.md` | ❌ W0 | ⬜ pending |
| 14-02-01 | 02 | 1 | REPO-01 | file check | `test -f .claude/skills/repo-to-paper-skill/SKILL.md && echo PASS` | ❌ W0 | ⬜ pending |
| 14-02-02 | 02 | 1 | REPO-01 | content check | `grep -c "Scan\|scan" .claude/skills/repo-to-paper-skill/SKILL.md` | ❌ W0 | ⬜ pending |
| 14-02-03 | 02 | 1 | REPO-02 | content check | `grep -c "H1\|heading" .claude/skills/repo-to-paper-skill/SKILL.md` | ❌ W0 | ⬜ pending |
| 14-02-04 | 02 | 1 | REPO-03 | content check | `grep -c "H2\|subsection" .claude/skills/repo-to-paper-skill/SKILL.md` | ❌ W0 | ⬜ pending |
| 14-02-05 | 02 | 1 | REPO-05 | content check | `grep -c "H3" .claude/skills/repo-to-paper-skill/SKILL.md` | ❌ W0 | ⬜ pending |
| 14-02-06 | 02 | 1 | REPO-01-05 | line budget | `wc -l < .claude/skills/repo-to-paper-skill/SKILL.md` | ❌ W0 | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- [ ] `.claude/skills/repo-to-paper-skill/` directory created
- [ ] `references/repo-patterns.md` created

*Existing infrastructure: Pure markdown project with no build/test framework. Validation is file-existence and content-grep based.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Skill follows skeleton structure | All | Structure compliance requires reading full file | Compare SKILL.md sections against `references/skill-skeleton.md` |
| Scan summary is categorized table | REPO-01 | Output format is runtime behavior | Read Workflow Step 1 and verify table format described |
| H1/H2/H3 checkpoints exist | REPO-02,03,05 | Checkpoint flow is Skill workflow logic | Read Workflow Steps 2-4 and verify confirmation loops |
| Source annotations on H2/H3 | REPO-03,05 | Annotation format is workflow instruction | Verify Workflow mentions `← from:` annotation pattern |
| repo-patterns.md has two tables | REPO-08 | Table structure requires human review | Read file and verify file-pattern table + section-mapping table |
| Line budget compliance | All | Budget is ~300 lines target | Run `wc -l` and verify ≤ 320 |

*All phase behaviors require manual verification due to pure-markdown nature of the project.*

---

## Validation Sign-Off

- [ ] All tasks have automated verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 2s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
