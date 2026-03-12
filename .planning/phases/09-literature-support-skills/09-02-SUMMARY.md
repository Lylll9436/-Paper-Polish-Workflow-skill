---
phase: 09-literature-support-skills
plan: 02
subsystem: skills
tags: [cover-letter, journal-aware, skill-authoring, academic-writing]

# Dependency graph
requires:
  - phase: 02-skill-conventions
    provides: skill-conventions.md frontmatter contract and section requirements
  - phase: 01-reference-foundation
    provides: references/journals/ceus.md journal template for scope alignment
provides:
  - Cover letter generation Skill with journal-aware contribution statement
  - SUPP-01 requirement fulfilled
  - journal-missing → refuse pattern applied to cover letter generation
affects:
  - 09-03-visualization-skill (pattern reference for direct-mode, no-reference-load Skills)

# Tech tracking
tech-stack:
  added: []
  patterns:
    - journal-aware-generation-with-refuse-on-missing applied to cover letter
    - ask-strategy-4-inputs pattern (journal, paper, author details, data availability)
    - file-input-write / pasted-text-conversation dual output mode

key-files:
  created:
    - .claude/skills/cover-letter-skill/SKILL.md
  modified: []

key-decisions:
  - "Cover letter contribution statement must explicitly reference loaded journal scope — never generic framing; enforced by workflow instruction"
  - "File input produces {input}_cover_letter.md via Write tool; pasted text outputs in conversation — matches reviewer-simulation-skill convention"
  - "Ask Strategy collects max 3 questions: journal (required), paper (required), author details (grouped, required), data availability (optional)"

patterns-established:
  - "Journal template missing → refuse immediately, no generic fallback — consistent across Phases 3–9"
  - "Correspondence author details collected as single grouped question (name, email, institution) to stay within 3-question Ask Strategy limit"

requirements-completed: [SUPP-01]

# Metrics
duration: 3min
completed: 2026-03-12
---

# Phase 9 Plan 02: Cover Letter Skill Summary

**Journal-aware cover letter generation Skill with four mandatory content blocks and refuse-on-missing journal template enforcement**

## Performance

- **Duration:** 3 min
- **Started:** 2026-03-12T14:33:12Z
- **Completed:** 2026-03-12T14:35:38Z
- **Tasks:** 2 (Task 1: author SKILL.md, Task 2: structural validation)
- **Files modified:** 1

## Accomplishments

- Authored `.claude/skills/cover-letter-skill/SKILL.md` at 219 lines (under 300-line budget)
- All 10 required sections present per skill-conventions.md: Purpose, Trigger, Modes, References, Ask Strategy, Workflow, Output Contract, Edge Cases, Fallbacks, Examples
- Four mandatory content blocks enforced: contribution statement (journal-scope-aligned), data/code availability, conflict of interest, correspondence author contact
- Journal-missing → refuse pattern implemented consistently with Phases 3–8
- Bilingual triggers (English + Chinese) per Phase 2 convention
- All 9 structural validation checks passed on first attempt; no fixes required

## Task Commits

Each task was committed atomically:

1. **Task 1: Author cover-letter-skill SKILL.md** - `beca120` (feat)
2. **Task 2: Validate cover-letter-skill conventions compliance** - no file changes; all checks passed without modifications

## Files Created/Modified

- `.claude/skills/cover-letter-skill/SKILL.md` - Complete Cover Letter Skill: journal-aware generation, four required content blocks, Ask Strategy for author details and data availability, file-write and conversation output modes

## Decisions Made

- Contribution statement rule: write 1–2 sentences explicitly referencing journal's stated scope and aims from the loaded template; use journal-specific framing not generic academic praise
- Correspondence author details collected as a single grouped question (name, email, institution) to stay within the 3-question Ask Strategy maximum
- Data availability defaults to "not publicly available" if user declines to specify — safe default that can be updated at revision stage
- file input → `{input_filename_without_ext}_cover_letter.md`; pasted text → conversation output; matches reviewer-simulation-skill naming convention

## Deviations from Plan

None — plan executed exactly as written. All 9 structural validation checks passed on first attempt without requiring any fixes.

## Issues Encountered

None.

## User Setup Required

None — no external service configuration required.

## Next Phase Readiness

- Cover Letter Skill complete and ready for use
- Pattern for direct-mode, journal-aware generation with refuse-on-missing now established for three consecutive phases (3, 4, 8, 9)
- Phase 9 Plan 03 (Visualization Skill) can proceed — simpler than this plan (no reference loads, no file writes)

---
*Phase: 09-literature-support-skills*
*Completed: 2026-03-12*
