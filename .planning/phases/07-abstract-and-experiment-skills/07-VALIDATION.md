---
phase: 7
slug: abstract-and-experiment-skills
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-12
---

# Phase 7 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | None (markdown-only Skills; no runtime executable) |
| **Config file** | None |
| **Quick run command** | `wc -l .claude/skills/abstract-skill/SKILL.md .claude/skills/experiment-skill/SKILL.md` |
| **Full suite command** | `grep -n "^## " .claude/skills/abstract-skill/SKILL.md .claude/skills/experiment-skill/SKILL.md` |
| **Estimated runtime** | ~5 seconds (structural inspection only) |

---

## Sampling Rate

- **After every task commit:** Run `wc -l .claude/skills/abstract-skill/SKILL.md .claude/skills/experiment-skill/SKILL.md` (line budget check)
- **After every plan wave:** Run `grep -n "^## " .claude/skills/abstract-skill/SKILL.md .claude/skills/experiment-skill/SKILL.md` (required sections present)
- **Before `/gsd:verify-work`:** Both SKILL.md files pass structural audit and at least one manual invocation test
- **Max feedback latency:** 5 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 7-01-01 | 01 | 1 | SECT-01 | structural | `wc -l .claude/skills/abstract-skill/SKILL.md` | ❌ Wave 0 | ⬜ pending |
| 7-01-02 | 01 | 1 | SECT-01 | structural | `grep -c "generate\|restructure" .claude/skills/abstract-skill/SKILL.md` | ❌ Wave 0 | ⬜ pending |
| 7-02-01 | 02 | 1 | SECT-03 | structural | `wc -l .claude/skills/experiment-skill/SKILL.md` | ❌ Wave 0 | ⬜ pending |
| 7-02-02 | 02 | 1 | SECT-03 | structural | `grep -c "Phase 1\|Phase 2\|Finding" .claude/skills/experiment-skill/SKILL.md` | ❌ Wave 0 | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- [ ] `.claude/skills/abstract-skill/SKILL.md` — covers SECT-01 (generate and restructure paths)
- [ ] `.claude/skills/experiment-skill/SKILL.md` — covers SECT-03 (analyze and generate phases)
- [ ] Manual invocation test for each Skill before `/gsd:verify-work`

*Existing infrastructure: No test framework exists — this phase is markdown-only. Structural commands above serve as automated checks.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Abstract Skill generates labeled 5-sentence abstract | SECT-01 | Skill is a Claude prompt, not executable code — output requires human invocation | Invoke skill with sample paper content, verify output has `[1: Contribution]` through `[5: Key Result]` labels |
| Abstract Skill restructures existing abstract preserving content | SECT-01 | Same — behavioral output requires human invocation | Paste an existing abstract, verify restructure path runs and preserves all original sentences (may flag missing positions) |
| Experiment Skill identifies patterns from table/stats input | SECT-03 | Same — pattern extraction requires human invocation | Provide a sample results table, verify Phase 1 produces "Finding N:" structured output |
| Experiment Skill generates discussion paragraphs connected to research questions | SECT-03 | Same — discussion quality requires human judgment | Confirm findings in Phase 1, verify Phase 2 generates discussion with grounded evidence + interpretation and uses `[CONNECT TO: ...]` placeholder for literature |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 5s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
