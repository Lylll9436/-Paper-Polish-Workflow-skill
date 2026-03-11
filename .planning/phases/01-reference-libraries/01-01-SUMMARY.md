---
phase: 01-reference-libraries
plan: "01"
subsystem: content
tags:
  - references
  - expression-patterns
  - academic-writing
requires: []
provides:
  - Stable overview entrypoint for the expression reference library
  - Five scenario-based leaf modules with Chinese guidance and bilingual examples
  - Independent-loading rules for downstream Skill retrieval
affects:
  - translation
  - polish
  - abstract
  - experiment
  - review
tech-stack:
  added: []
  patterns:
    - Stable entrypoint plus modular leaf files under references/
    - Scenario-based rhetoric modules with bilingual example rows
key-files:
  created:
    - references/expression-patterns/introduction-and-gap.md
    - references/expression-patterns/methods-and-data.md
    - references/expression-patterns/results-and-discussion.md
    - references/expression-patterns/conclusions-and-claims.md
    - references/expression-patterns/geography-domain.md
  modified:
    - references/expression-patterns.md
key-decisions:
  - "Keep references/expression-patterns.md as the stable entrypoint while moving detailed content into leaf modules."
  - "Organize expression guidance by writing scenario instead of grammar category so downstream Skills can load narrower context."
patterns-established:
  - "Overview-first retrieval: Skills start from the stable overview, then load one leaf module as needed."
  - "Each leaf module must stay independently readable and include recommended expressions, avoid expressions, usage scenarios, and bilingual examples."
requirements-completed:
  - FNDN-01
duration: 5 min
completed: 2026-03-11
---

# Phase 1 Plan 01: Reference Libraries Summary

**Modular academic expression references with a stable overview entrypoint, five scenario-based leaf modules, and bilingual example patterns**

## Performance

- **Duration:** 5 min
- **Started:** 2026-03-11T14:50:01+08:00
- **Completed:** 2026-03-11T14:54:41+08:00
- **Tasks:** 3
- **Files modified:** 6

## Accomplishments
- Replaced the monolithic expression reference file with a stable overview contract for on-demand loading.
- Added five scenario-based leaf modules covering introduction, methods, results, conclusions, and geography-specific phrasing.
- Hardened standalone-reading guarantees so future Skills can load any leaf file directly without hidden sibling context.

## Task Commits

Each task was committed atomically:

1. **Task 1: Convert the top-level expression file into a stable overview contract** - `ac465eb` (feat)
2. **Task 2: Author scenario-based expression modules with Chinese guidance** - `429dcf3` (feat)
3. **Task 3: Validate independent loadability and naming stability** - `bb96f3a` (refactor)

**Plan metadata:** pending current commit

## Files Created/Modified
- `references/expression-patterns.md` - Stable overview contract and module map for downstream Skill loading
- `references/expression-patterns/introduction-and-gap.md` - Introduction, gap, and contribution framing expressions
- `references/expression-patterns/methods-and-data.md` - Dataset, study area, and methodology language patterns
- `references/expression-patterns/results-and-discussion.md` - Results reporting and interpretation expressions
- `references/expression-patterns/conclusions-and-claims.md` - Conclusion, contribution, and claim-calibration expressions
- `references/expression-patterns/geography-domain.md` - Geography and urban-science-specific phrasing layer

## Decisions Made
- Kept the original overview file path stable so current and future Skills do not need path migrations.
- Split the reusable language library by rhetorical scenario, not by narrow grammar labels, to better match downstream Skill loading patterns.

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered
None.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness
- Expression reference foundation is complete and ready for CEUS/anti-AI work in 01-02.
- Wave 2 can now align public docs to the modular reference structure without guessing the final expression layout.

---
*Phase: 01-reference-libraries*
*Completed: 2026-03-11*
