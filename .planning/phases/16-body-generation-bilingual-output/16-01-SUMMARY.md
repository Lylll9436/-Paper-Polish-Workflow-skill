---
phase: 16-body-generation-bilingual-output
plan: 01
subsystem: skill
tags: [body-generation, bilingual, latex, anti-hallucination, ceus, citations]

# Dependency graph
requires:
  - phase: 14-repo-to-paper-core-structure
    provides: Steps 1-4 + Step 2.5 workflow in repo-to-paper-skill SKILL.md
  - phase: 15-literature-integration
    provides: .paper-refs/ directory format and Step 2.5 literature collection
provides:
  - Step 5 body generation workflow in repo-to-paper-skill SKILL.md
  - references/body-generation-rules.md with anti-hallucination, citation, bilingual, and output rules
  - body_text output contract (per-H1 .tex files + references.bib)
affects: [17-bilingual-migration]

# Tech tracking
tech-stack:
  added: []
  patterns: [reference-file-extraction-for-line-budget, section-selection-via-multiSelect]

key-files:
  created:
    - references/body-generation-rules.md
  modified:
    - .claude/skills/repo-to-paper-skill/SKILL.md

key-decisions:
  - "Extracted all generation rules to references/body-generation-rules.md to keep SKILL.md under 450 lines"
  - "Moved journals/ceus.md from leaf_hints to required because Step 5 always needs CEUS formatting"
  - "SKILL.md at 448 lines justified by 5-step + 1 decimal step complexity with rules already extracted"

patterns-established:
  - "Reference file extraction pattern: complex step rules go to references/, SKILL.md keeps lean workflow skeleton"
  - "AskUserQuestion multiSelect for section-level selection; Confirm/Modify/Skip for per-section review"

requirements-completed: [REPO-06, REPO-07, BILN-01]

# Metrics
duration: 5min
completed: 2026-03-18
---

# Phase 16 Plan 01: Body Generation & Bilingual Output Summary

**Step 5 body generation added to repo-to-paper-skill with anti-hallucination annotations, strict .paper-refs/ citation boundary, CEUS formatting, bilingual LaTeX paragraph comparison, and references.bib auto-generation**

## Performance

- **Duration:** 5 min
- **Started:** 2026-03-18T10:20:53Z
- **Completed:** 2026-03-18T10:26:01Z
- **Tasks:** 2
- **Files modified:** 2

## Accomplishments
- Created `references/body-generation-rules.md` (121 lines) with 6 sections covering claim classification, citation integration, bilingual LaTeX format, CEUS writing style, references.bib generation algorithm, and output file structure
- Added Step 5: Body Generation to `repo-to-paper-skill/SKILL.md` with AskUserQuestion multiSelect for section selection and Confirm/Modify/Skip per-section review
- Updated SKILL.md frontmatter: `body_text` in output_contract, `body-generation-rules.md` and `journals/ceus.md` in required references, `leaf_hints` emptied
- Added 3 edge case rows and 1 fallback row for Step 5 scenarios

## Task Commits

Each task was committed atomically:

1. **Task 1: Create references/body-generation-rules.md** - `3234ef5` (feat)
2. **Task 2: Add Step 5 to repo-to-paper-skill SKILL.md** - `4ed83af` (feat)

## Files Created/Modified
- `references/body-generation-rules.md` - Anti-hallucination rules, citation integration, bilingual LaTeX format, CEUS style, references.bib algorithm, output file structure (121 lines)
- `.claude/skills/repo-to-paper-skill/SKILL.md` - Step 5 body generation workflow added; frontmatter updated; edge cases and fallbacks extended (448 lines, +67 net)

## Decisions Made
- Extracted all generation rules to `references/body-generation-rules.md` to keep SKILL.md lean (~40-50 lines for Step 5 skeleton instead of ~120 inline)
- Moved `journals/ceus.md` from `leaf_hints` to `required` because Step 5 always needs CEUS formatting rules
- SKILL.md at 448 lines exceeds 300-line convention but is justified: most complex Skill in project (5 steps + 1 decimal step) with generation rules already extracted to reference file

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness
- Complete repo-to-paper pipeline: Steps 1 -> 2 -> 3 -> 2.5 -> 4 -> 5
- Phase 17 (bilingual migration for existing Skills) can proceed independently -- it does not modify repo-to-paper-skill
- All locked decisions from 16-CONTEXT.md implemented: section selection via multiSelect, per-section Confirm/Modify/Skip, auto-continue after H3, .paper-output/ directory, LaTeX comment bilingual format, references.bib at end, strict .paper-refs/ citation boundary, anti-hallucination placeholders

## Self-Check: PASSED

- All created files verified present on disk
- All task commits verified in git log (3234ef5, 4ed83af)

---
*Phase: 16-body-generation-bilingual-output*
*Completed: 2026-03-18*
