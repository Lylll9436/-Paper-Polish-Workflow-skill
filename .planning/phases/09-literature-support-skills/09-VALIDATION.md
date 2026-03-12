---
phase: 9
slug: literature-support-skills
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-12
---

# Phase 9 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | None — markdown-only Skills, no executable code |
| **Config file** | None — no test infrastructure in this project |
| **Quick run command** | Manual: convention compliance self-check against skill-conventions.md |
| **Full suite command** | Manual review of all three SKILL.md files against skill-conventions.md checklist |
| **Estimated runtime** | ~5 minutes (manual review) |

---

## Sampling Rate

- **After every task commit:** Convention compliance self-check: verify SKILL.md against skill-conventions.md checklist (line count, required sections, frontmatter fields)
- **After every plan wave:** N/A — all three plans are in a single wave
- **Before `/gsd:verify-work`:** Manual invocation of each Skill to verify output structure
- **Max feedback latency:** N/A (manual-only)

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 09-01-01 | 01 | 1 | SUPP-02 | manual | Convention compliance self-check | ❌ N/A | ⬜ pending |
| 09-01-02 | 01 | 1 | SUPP-02 | manual | MCP pre-flight: invoke Skill, confirm refusal on MCP unavailable | ❌ N/A | ⬜ pending |
| 09-01-03 | 01 | 1 | SUPP-02 | manual | Result card format: Title/Authors/Year/Citations/Excerpt present | ❌ N/A | ⬜ pending |
| 09-01-04 | 01 | 1 | SUPP-02 | manual | BibTeX output from MCP data only, not prior knowledge | ❌ N/A | ⬜ pending |
| 09-02-01 | 02 | 1 | SUPP-01 | manual | Convention compliance self-check | ❌ N/A | ⬜ pending |
| 09-02-02 | 02 | 1 | SUPP-01 | manual | All four content blocks present in cover letter output | ❌ N/A | ⬜ pending |
| 09-02-03 | 02 | 1 | SUPP-01 | manual | Missing journal template → refuse (not fallback to generic) | ❌ N/A | ⬜ pending |
| 09-03-01 | 03 | 1 | SUPP-03 | manual | Convention compliance self-check | ❌ N/A | ⬜ pending |
| 09-03-02 | 03 | 1 | SUPP-03 | manual | 2–3 recommendations each with type/reason/tool hints | ❌ N/A | ⬜ pending |
| 09-03-03 | 03 | 1 | SUPP-03 | manual | Spatial data signals trigger choropleth/spatial scatter candidates | ❌ N/A | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

None — no test infrastructure needed for markdown Skill authoring. Validation is structural (convention compliance) and performed during PLAN execution self-check.

*Existing infrastructure covers all phase requirements.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| MCP pre-flight refuses when Semantic Scholar unavailable | SUPP-02 | Requires live MCP context; cannot be automated without running Claude Code session | Disconnect MCP server in settings → invoke Literature Skill → confirm error message with setup guidance |
| BibTeX entries sourced from MCP data, not training knowledge | SUPP-02 | Requires human review of BibTeX against live paper record | Search a known paper → compare returned BibTeX fields against Semantic Scholar web record |
| Result cards show Title/Authors/Year/Citations/Abstract excerpt | SUPP-02 | Output validation in live session | Search any topic → verify each result card has all five fields |
| All four cover letter content blocks present (contribution, data, CoI, contact) | SUPP-01 | Markdown output, human review | Invoke Cover Letter Skill with CEUS journal → verify output contains all four blocks |
| Missing journal template → refuse (not generic fallback) | SUPP-01 | Requires invocation test | Invoke Cover Letter Skill with unknown journal name → confirm refusal message |
| 2–3 recommendations each with type/reason/tool hints | SUPP-03 | Output validation in session | Invoke Visualization Skill with tabular data description → verify recommendation card structure |
| Spatial data triggers choropleth/spatial scatter candidates | SUPP-03 | Requires invocation test with spatial data signals | Invoke with "administrative unit polygons + outcome variable" → confirm choropleth appears in recommendations |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < N/A (manual-only phase)
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
