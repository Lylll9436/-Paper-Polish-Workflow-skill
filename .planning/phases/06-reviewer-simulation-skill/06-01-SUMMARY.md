---
phase: 06-reviewer-simulation-skill
plan: 01
subsystem: skill
tags: [peer-review, reviewer-simulation, bilingual, scoring, academic-writing]

# Dependency graph
requires:
  - phase: 01-reference-authoring
    provides: expression-patterns library and journal templates for writing quality assessment
  - phase: 02-skill-conventions
    provides: skill conventions, skeleton template, and line budget rules
provides:
  - "Reviewer Simulation Skill (SKILL.md) for structured peer review of academic papers"
  - "Five-dimension scoring framework (Novelty, Methodology, Writing Quality, Presentation, Significance)"
  - "Three-part concern format (problem, why, suggestion) with bilingual output"
affects: [07-structure-checker-skill, 08-figure-table-caption-skill, 10-qa-and-hardening]

# Tech tracking
tech-stack:
  added: []
  patterns: [read-analyze-report single-phase workflow, concerns-first-then-scores assessment, inline bilingual blockquote translation]

key-files:
  created:
    - .claude/skills/reviewer-simulation-skill/SKILL.md
  modified: []

key-decisions:
  - "Chinese translation uses inline blockquote format after each concern, not appended section at end"
  - "Anti-AI patterns NOT loaded by default -- this is a review skill, not a detection skill"
  - "Batch mode explicitly listed as unsupported since review requires full-paper context"
  - "Report filename convention: {input_filename_without_ext}_review.md for file input"

patterns-established:
  - "Read-analyze-report: single-phase analytical workflow producing informational output without modifying input"
  - "Concerns-first-then-scores: generate all concerns before assigning dimension scores to prevent inconsistency"
  - "Section-level concern location: concerns reference paper sections, not paragraphs or sentences"

requirements-completed: [CORE-04]

# Metrics
duration: 5min
completed: 2026-03-12
---

# Phase 6 Plan 1: Reviewer Simulation Skill Summary

**Peer review simulation Skill with five-dimension scoring, three-part bilingual concerns, and journal-aware review using read-analyze-report workflow**

## Performance

- **Duration:** 5 min
- **Started:** 2026-03-12T02:54:06Z
- **Completed:** 2026-03-12T02:59:46Z
- **Tasks:** 2
- **Files modified:** 1

## Accomplishments
- Authored complete Reviewer Simulation Skill at 227 lines (under 300-line budget)
- Encoded all 15 locked decisions from CONTEXT.md including five dimensions, three-part concerns, bilingual output, verdict options, and journal refusal behavior
- Validated against five categories: frontmatter contract, body section ordering, line count, locked decisions, and anti-pattern avoidance

## Task Commits

Each task was committed atomically:

1. **Task 1: Author the Reviewer Simulation Skill SKILL.md** - `174ac06` (feat)
2. **Task 2: Validate Skill against conventions and locked decisions** - `32d2470` (refactor)

## Files Created/Modified
- `.claude/skills/reviewer-simulation-skill/SKILL.md` - Complete Reviewer Simulation Skill with read-analyze-report workflow, five-dimension scoring, three-part concern format, bilingual output, and journal-aware review

## Decisions Made
- Chinese translation uses inline blockquote format (`> **[Chinese]** ...`) after each concern rather than an appended section -- keeps translation co-located with English for quick comprehension
- Anti-AI patterns are NOT loaded by default; this is a review skill, not a detection skill -- keeps context lean
- Batch mode listed as unsupported since review requires full-paper context; only `direct` mode is supported
- Report filename convention: `{input_filename_without_ext}_review.md` for file input, fallback to `review-report.md`

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 1 - Bug] Fixed section ordering to match conventions**
- **Found during:** Task 2 (Validation)
- **Issue:** Chinese Translation Guidelines was a standalone `##` section between Workflow and Output Contract, breaking the required section ordering from skill-conventions.md
- **Fix:** Moved Chinese translation rules into the Workflow section as a subsection of Step 3 (Generate Review Report)
- **Files modified:** .claude/skills/reviewer-simulation-skill/SKILL.md
- **Verification:** All 9 required sections now appear in correct convention order
- **Committed in:** 32d2470 (Task 2 commit)

---

**Total deviations:** 1 auto-fixed (1 bug)
**Impact on plan:** Minor structural fix for conventions compliance. No scope creep.

## Issues Encountered
None

## User Setup Required
None - no external service configuration required.

## Next Phase Readiness
- Reviewer Simulation Skill is complete and ready for use
- Expression patterns library and CEUS journal template are available as runtime references
- No downstream blockers for Phase 7 (Structure Checker Skill) or later phases

## Self-Check: PASSED

- [x] `.claude/skills/reviewer-simulation-skill/SKILL.md` exists (227 lines)
- [x] Commit `174ac06` (Task 1) exists
- [x] Commit `32d2470` (Task 2) exists
- [x] SKILL.md under 300-line budget

---
*Phase: 06-reviewer-simulation-skill*
*Completed: 2026-03-12*
