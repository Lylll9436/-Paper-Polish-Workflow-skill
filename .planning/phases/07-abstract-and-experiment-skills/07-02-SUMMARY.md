---
phase: 07-abstract-and-experiment-skills
plan: "02"
subsystem: skill-authoring
tags: [experiment-skill, two-phase-workflow, discussion-generation, findings-analysis, anti-hallucination]

# Dependency graph
requires:
  - phase: 01-reference-library
    provides: expression-patterns reference library (results-and-discussion.md, conclusions-and-claims.md, anti-ai-patterns/vocabulary.md)
  - phase: 02-skill-conventions
    provides: skill-conventions.md and skill-skeleton.md authoring contract
  - phase: 05-de-ai-skill
    provides: two-phase detect-then-rewrite workflow pattern (Phase 1 confirm → Phase 2 generate)
provides:
  - Experiment Skill SKILL.md implementing SECT-03
  - Phase 1 structured Finding list output (Finding N: format with direction, magnitude, comparison group)
  - Phase 2 discussion paragraph generation with evidence-before-interpretation rule
  - Anti-hallucination literature connection pattern ([CONNECT TO: ...] placeholders)
affects: [09-literature-skill, any future skill using two-phase analysis workflow]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Two-phase analysis-then-generation workflow with mandatory user confirmation between phases"
    - "Evidence-before-interpretation sentence ordering for discussion paragraphs"
    - "CONNECT TO placeholder pattern for anti-hallucination literature connections"
    - "Measurable-data guard: refuse vague input before analysis phase"

key-files:
  created:
    - .claude/skills/experiment-skill/SKILL.md
  modified: []

key-decisions:
  - "Phase 2 requires Phase 1 confirmation — user cannot skip to discussion generation"
  - "Literature connections use [CONNECT TO: ...] placeholders rather than AI-inferred citations, matching CLAUDE.md anti-hallucination principle"
  - "Research questions are mandatory Ask Strategy input; skill writes [RESEARCH QUESTION: ...] placeholders if declined rather than blocking"
  - "Anti-AI vocabulary.md loaded in Phase 2 generation phase for output screening, consistent with De-AI and Polish Skill patterns"
  - "Batch mode unsupported — experiment analysis requires full result set context"

patterns-established:
  - "Evidence sentence must precede interpretation sentence in every discussion paragraph"
  - "Finding N: format locked: [subject] [comparison/trend] [value] on [metric/condition]"
  - "Journal-template-missing triggers refusal, not fallback to general style (consistent with all prior Skills)"

requirements-completed: [SECT-03]

# Metrics
duration: 3min
completed: 2026-03-12
---

# Phase 7 Plan 02: Experiment Skill Summary

**Two-phase experiment analysis Skill: structured Finding list (Phase 1) followed by evidence-grounded discussion paragraphs with [CONNECT TO: ...] anti-hallucination placeholders (Phase 2)**

## Performance

- **Duration:** 3 min
- **Started:** 2026-03-12T04:52:07Z
- **Completed:** 2026-03-12T04:55:25Z
- **Tasks:** 2 (Task 1: author SKILL.md, Task 2: validate conventions compliance)
- **Files modified:** 1

## Accomplishments

- Authored `.claude/skills/experiment-skill/SKILL.md` (230 lines, under 300 line budget) implementing SECT-03
- Implemented two-phase workflow: Phase 1 extracts measurable findings with locked "Finding N:" format; Phase 2 generates one discussion paragraph per confirmed finding
- Enforced evidence-before-interpretation ordering rule with CRITICAL annotation in workflow
- Built anti-hallucination literature connection pattern: user provides prior work or Skill writes [CONNECT TO: ...] placeholders; never invents references
- Measurable-data guard in Phase 1 refuses vague input without values, comparisons, or metrics
- All 7 structural validation checks passed: line count, 9 required sections, Phase 1/2 occurrences, Finding format, anti-hallucination rule (11 occurrences), frontmatter fields, bilingual triggers

## Task Commits

Each task was committed atomically:

1. **Task 1: Author experiment-skill SKILL.md** - `0d3f8c1` (feat)
2. **Task 2: Validate conventions compliance** - no file changes (all checks passed without fixes)

**Plan metadata:** (docs commit — see below)

## Files Created/Modified

- `.claude/skills/experiment-skill/SKILL.md` — Experiment analysis and discussion generation Skill, 230 lines, all 9 required sections, two-phase workflow

## Decisions Made

- Phase 2 cannot run without Phase 1 confirmation — enforces user validation of findings before generation
- Research questions mandatory for Phase 2 coherence; Skill uses [RESEARCH QUESTION: ...] placeholder if user declines rather than blocking entirely
- Anti-AI vocabulary.md loaded in Phase 2 for output screening, consistent with De-AI and Polish Skill patterns
- Batch mode unsupported — experiment analysis requires full result set context to identify patterns correctly
- [CONNECT TO: ...] placeholder pattern matches CLAUDE.md citation anti-hallucination principle (~40% AI citation error rate)

## Deviations from Plan

None — plan executed exactly as written. All 7 Task 2 validation checks passed on the first authored version without requiring any fixes.

## Issues Encountered

None.

## User Setup Required

None — no external service configuration required. Skill is markdown-only.

## Next Phase Readiness

- Experiment Skill (SECT-03) complete and validated
- Phase 7 second plan (07-02) complete; Phase 7 now fully delivered (07-01 Abstract Skill + 07-02 Experiment Skill)
- No blockers for Phase 8 (Caption Skill)

---
*Phase: 07-abstract-and-experiment-skills*
*Completed: 2026-03-12*
