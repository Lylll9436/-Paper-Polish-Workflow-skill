# Phase 4: Polish Skill - Research

**Researched:** 2026-03-11
**Domain:** English academic paper polishing as a Claude Code Skill
**Confidence:** HIGH

## Summary

Phase 4 builds the English polishing Skill that supports two modes: Quick-fix (default, intelligent single-pass) and Guided (fixed three-step: structure, logic, expression). Unlike the Translation Skill which writes new output files, the Polish Skill edits files in-place using the Edit tool and preserves original text as LaTeX comment annotations for traceability. After user acceptance, annotations are cleaned up automatically.

This is a Skill authoring task producing a single `SKILL.md` file at `.claude/skills/polish-skill/SKILL.md`. The complexity lies in encoding two distinct modes within the 300-line budget, designing the LaTeX annotation format, implementing the guided mode checklist workflow, handling both file-based and pasted-text input, detecting translation artifacts automatically, and integrating anti-AI patterns proactively. The established Translation Skill (Phase 3) provides a strong structural precedent.

**Primary recommendation:** Copy the skeleton from `references/skill-skeleton.md`, follow the Translation Skill's structure as a model, and focus the Workflow section on clearly distinguishing Quick-fix (single workflow path) from Guided (three-step workflow with checklist confirmation at each step).

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions
- Default mode is Quick-fix: user says "polish this" and Skill runs directly
- Quick-fix intelligently judges what to focus on based on input text quality -- expression issues get rewritten, structural issues get adjusted, all in one pass
- Guided mode activated only when user explicitly requests it (e.g., "guided polish", "guided mode")
- Guided mode follows fixed three steps: structure -> logic -> expression, each step completes before next begins
- Each guided step lists discovered problems as a checklist; user selects which problems to fix; selected fixes applied in batch, then next step begins
- Logic step includes cross-section coherence check (argument chains, terminology consistency) when input covers multiple sections
- In-place modification: Skill edits the original file directly using Edit tool
- Before each modification, original text preserved as LaTeX comment annotation (e.g., `% Original: ...` followed by new text)
- After user confirms acceptance, annotations automatically cleaned up (original text comments removed)
- Concise summary report generated after modifications: number of changes, main types, notes/caveats
- Smart adaptation: short text polished entirely, long text split by section and processed in batches
- Both file path and pasted text supported as input
- Pasted text results output directly in conversation (no file writing)
- Automatic translationese detection: when input appears to come from translation, pay special attention to translationese patterns, Chinese-English calques, overly literal structures
- Anti-AI patterns library proactively loaded during polishing
- Summary report includes "Recommend running De-AI Skill for further detection check" when applicable

### Claude's Discretion
- Exact threshold for "short text" vs "long text" smart adaptation split
- How to detect translationese vs native English issues
- Internal strategy for Quick-fix intelligent judgment (which dimensions to prioritize)
- Exact LaTeX comment annotation format

### Deferred Ideas (OUT OF SCOPE)
None -- discussion stayed within phase scope.
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| CORE-02 | English polishing Skill with flexible flow supporting both quick-fix mode (direct polish) and guided mode (structure -> logic -> expression) | Entire research supports this: dual-mode workflow design, LaTeX annotation format, reference loading, guided mode checklist pattern, in-place editing strategy, translationese detection, anti-AI integration |
</phase_requirements>

## Standard Stack

### Core

This is a Skill authoring phase. The "stack" is the project's own conventions and references.

| Asset | Path | Purpose | Why Standard |
|-------|------|---------|--------------|
| Skill Conventions | `references/skill-conventions.md` | Defines frontmatter contract, body structure, line budget | Phase 2 mandatory for all Skills |
| Skill Skeleton | `references/skill-skeleton.md` | Copyable starting point | Phase 2 deliverable |
| Expression Patterns | `references/expression-patterns.md` + leaf files | Academic writing patterns by section type | Phase 1 deliverable; Polish Skill loads on demand |
| Anti-AI Patterns | `references/anti-ai-patterns.md` + leaf files | AI vocabulary blacklist and alternatives | Phase 1 deliverable; loaded proactively during polishing |
| CEUS Template | `references/journals/ceus.md` | Journal-specific style rules | Phase 1 deliverable; first supported journal |
| Translation Skill | `.claude/skills/translation-skill/SKILL.md` | Structural precedent for Skill authoring | Phase 3 deliverable; Polish Skill follows same structure |

### Supporting

| Asset | Purpose | When to Use |
|-------|---------|-------------|
| `references/expression-patterns/introduction-and-gap.md` | Introduction section patterns | When polishing introduction content |
| `references/expression-patterns/methods-and-data.md` | Methods section patterns | When polishing methods content |
| `references/expression-patterns/results-and-discussion.md` | Results/discussion patterns | When polishing results or discussion |
| `references/expression-patterns/conclusions-and-claims.md` | Conclusion patterns | When polishing conclusion content |
| `references/expression-patterns/geography-domain.md` | Geography-specific phrasing | When content involves spatial/urban topics |
| `references/anti-ai-patterns/vocabulary.md` | High-frequency AI vocabulary | Loaded proactively for vocabulary screening |
| `references/anti-ai-patterns/sentence-patterns.md` | AI sentence pattern detection | Loaded when sentence-level issues detected |
| `references/anti-ai-patterns/transitions-and-tone.md` | Over-smoothed transitions | Loaded when transition overuse detected |

## Architecture Patterns

### Project Structure

```
.claude/skills/polish-skill/
  SKILL.md              # The Skill file (~250-300 lines)
```

Single deliverable. References loaded at runtime from `references/`.

### Pattern 1: Dual-Mode Workflow

**What:** The Skill supports two distinct modes with different workflow paths.

**Quick-fix mode (default):**
- User says "polish this" or similar
- Skill runs a single intelligent pass, deciding what to focus on based on text quality
- No intermediate confirmation -- edits applied directly
- Summary report after completion

**Guided mode (explicit request):**
- User says "guided polish" or "引导模式润色"
- Three sequential steps: structure -> logic -> expression
- Each step: analyze -> list problems as checklist -> user selects -> apply fixes -> next step
- Logic step adds cross-section coherence check when multi-section input

**Mode mapping to Phase 2 conventions:**
| Polish Mode | Convention Mode | Rationale |
|-------------|----------------|-----------|
| Quick-fix | `direct` | Single-pass, minimal interaction |
| Guided | `guided` | Multi-pass with confirmation at key checkpoints |
| (batch variant) | `batch` | Quick-fix across multiple files/sections |

The `interactive` mode from conventions is not a primary mode for this Skill but can be inferred from "step by step" phrasing as a more granular variant of guided.

### Pattern 2: In-Place Editing with LaTeX Annotations

**What:** Unlike Translation Skill (which writes new files), Polish Skill edits the original file using the Edit tool and preserves originals as LaTeX comments.

**Recommended LaTeX annotation format (Claude's Discretion):**
```latex
% [Polish] Original: The urban heat island effect is a very important phenomenon.
The urban heat island effect is a critical environmental concern in rapidly urbanizing regions.
```

**Design considerations:**
- Prefix `% [Polish] Original:` distinguishes polish annotations from other LaTeX comments
- Single-line annotations keep the format parseable for cleanup
- For multi-line originals, each line gets its own `% [Polish] Original:` prefix
- Cleanup step: remove all lines matching `^% \[Polish\] Original:` pattern

**For pasted text:** No file editing. Present polished text directly in conversation with before/after comparison inline.

### Pattern 3: Guided Mode Checklist Pattern

**What:** Each guided step produces a problem list that the user selects from before fixes are applied.

**Workflow per step:**
```
1. Analyze text for [structure/logic/expression] issues
2. Present numbered checklist:
   - [ ] 1. Paragraph 3: weak topic sentence, does not connect to previous argument
   - [ ] 2. Section transition: abrupt shift from methods to results
   - [ ] 3. ...
3. User replies with selection (e.g., "fix 1 and 3" or "all")
4. Apply selected fixes in batch using Edit tool
5. Present brief summary of changes made
6. Proceed to next step
```

**Three guided steps defined:**
| Step | Focus | What to Check |
|------|-------|---------------|
| Structure | Paragraph organization, topic sentences, section flow, redundancy | Does each paragraph serve a clear purpose? Are sections in logical order? |
| Logic | Argument coherence, claim support, terminology consistency, cross-section coherence | Do arguments chain logically? Are claims supported? Are terms used consistently? |
| Expression | Word choice, sentence clarity, academic tone, conciseness, anti-AI patterns | Is phrasing precise? Are sentences clear? Does it sound human and academic? |

### Pattern 4: Translationese Detection

**What:** The Skill automatically detects when input appears to come from translation and adjusts polishing focus accordingly.

**Detection heuristics (Claude's Discretion):**
- Overly literal structures that mirror Chinese syntax (e.g., "regarding X, it has the characteristic of...")
- Chinese-English calques (e.g., "put forward" for "propose", "carry on" for "conduct")
- Excessive use of "the" or "of" constructions typical of literal translation from Chinese
- Unnaturally long noun phrases from Chinese modifier chains
- Topic-comment sentence structures (Chinese topicalization pattern)

**When detected:** The Skill flags translationese as a priority in Quick-fix mode and adds specific translationese fixes in the expression step of Guided mode.

### Pattern 5: Smart Adaptation for Input Length

**What:** Short text is polished entirely; long text is split by section and processed in batches.

**Recommended threshold (Claude's Discretion):**
- Short: up to ~2-3 paragraphs or ~500 words -- polish entirely in one pass
- Long: 4+ paragraphs or multiple sections -- split by `\section{}` or `\subsection{}` markers, process each section sequentially
- When splitting: maintain cross-section awareness for terminology consistency

### Anti-Patterns to Avoid

- **Duplicating reference content into SKILL.md:** Expression patterns, anti-AI vocabulary, and journal rules must be loaded at runtime, never inlined.
- **Making Quick-fix mode ask too many questions:** Quick-fix should feel like handing a draft to a skilled colleague. Minimal pre-questions.
- **Treating guided steps as independent passes:** Each guided step should be aware of changes made in previous steps. Structure fixes in step 1 affect what logic step 2 sees.
- **Confusing Edit and Write tools:** Polish Skill uses Edit for in-place modification of existing files, not Write (which overwrites entirely). Write is only for pasted-text output scenarios.
- **Annotation format that breaks LaTeX compilation:** Annotations must be valid LaTeX comments (`%` prefix) so the file still compiles.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Expression patterns | Inline pattern tables | Load from `references/expression-patterns/*.md` | Central maintenance, avoid drift |
| Journal style rules | Hardcode rules | Load from `references/journals/[journal].md` | Single source of truth |
| Anti-AI vocabulary | Inline avoidance list | Load from `references/anti-ai-patterns.md` + leaves | Centralized, shared with Translation and De-AI Skills |
| Translationese patterns | Hardcode detection rules | Use Claude's natural language understanding + anti-AI patterns | Claude already recognizes translation artifacts; brief heuristic hints suffice |

**Key insight:** This Skill is a prompt document. "Don't hand-roll" means "don't duplicate reference content." The Skill orchestrates what Claude does, not how to polish (Claude already knows academic writing).

## Common Pitfalls

### Pitfall 1: Exceeding the 300-Line Budget

**What goes wrong:** The Polish Skill has more concerns than the Translation Skill (two modes, three guided steps, annotation format, cleanup, translationese detection, smart adaptation) which tempt verbosity.
**Why it happens:** Trying to specify every detail of both Quick-fix and Guided workflows.
**How to avoid:** Quick-fix workflow should be very brief (3-4 bullets). Guided steps share structure (analyze, checklist, apply) -- describe the pattern once and list step-specific focus areas in a compact table. Trust Claude to execute the pattern.
**Warning signs:** Draft exceeds 250 lines before Edge Cases section.

### Pitfall 2: Quick-Fix Mode Becoming Too Rigid

**What goes wrong:** Quick-fix mode tries to enumerate every possible issue type and priority, turning it into a hidden multi-step process.
**Why it happens:** Attempting to codify "intelligent judgment" into explicit rules.
**How to avoid:** State the principle ("judge what to focus on based on input quality") and trust Claude's language understanding. Provide a brief priority hint: expression > logic > structure (most polishing requests are about expression). Do not enumerate every possible issue type.
**Warning signs:** Quick-fix section is longer than Guided section.

### Pitfall 3: LaTeX Annotation Cleanup Incompleteness

**What goes wrong:** Cleanup step misses annotations or removes non-annotation comments.
**Why it happens:** Annotation format is too similar to regular LaTeX comments.
**How to avoid:** Use a distinctive prefix like `% [Polish] Original:` that is unlikely to appear in normal LaTeX. Cleanup uses this exact prefix pattern. Document the format so users know what to expect.
**Warning signs:** Cleanup regex is vague (e.g., just `% Original`).

### Pitfall 4: Guided Mode Step Isolation

**What goes wrong:** Each guided step is treated as independent, so fixes conflict or undo each other.
**Why it happens:** Steps are described without cross-step awareness.
**How to avoid:** State that each step operates on the current version of the text (after previous step's edits). The logic step should re-read the file after structure edits. The expression step should re-read after logic edits.
**Warning signs:** Steps described as "analyze the original text" instead of "analyze the current text."

### Pitfall 5: Pasted Text vs File Path Confusion

**What goes wrong:** The Skill tries to use Edit tool on pasted text or tries to write a new file when editing in-place.
**Why it happens:** Two input modes require different output strategies.
**How to avoid:** Clear branching at the start of Workflow: if input is a file path, use Edit for in-place modification with annotations; if input is pasted text, output polished text directly in conversation with before/after comparison.
**Warning signs:** Single workflow path tries to handle both input modes without branching.

## Code Examples

These are Skill content examples showing what specific SKILL.md sections should look like.

### Frontmatter

```yaml
---
name: polish-skill
description: >-
  Polish English academic text through quick-fix or guided multi-pass workflow.
  Adapts to journal style with in-place editing and change tracking. 英文学术论文润色。
triggers:
  primary_intent: polish English academic text for journal submission
  examples:
    - "Polish this paragraph"
    - "润色这段英文"
    - "Help me polish my introduction for CEUS"
    - "Guided polish my methods section"
    - "引导模式润色这篇论文"
    - "Quick fix this draft"
tools:
  - Read
  - Edit
  - Structured Interaction
references:
  required:
    - references/expression-patterns.md
  leaf_hints:
    - references/expression-patterns/introduction-and-gap.md
    - references/expression-patterns/methods-and-data.md
    - references/expression-patterns/results-and-discussion.md
    - references/expression-patterns/conclusions-and-claims.md
    - references/expression-patterns/geography-domain.md
    - references/anti-ai-patterns.md
    - references/anti-ai-patterns/vocabulary.md
    - references/anti-ai-patterns/sentence-patterns.md
    - references/anti-ai-patterns/transitions-and-tone.md
input_modes:
  - file
  - pasted_text
output_contract:
  - polished_text
  - change_annotations
  - summary_report
---
```

**Key differences from Translation Skill frontmatter:**
- tools: `Edit` instead of `Write` (in-place editing)
- leaf_hints: includes all three anti-AI pattern leaves (loaded proactively)
- output_contract: polished_text, change_annotations, summary_report (not dual file output)

### Mode Table

```markdown
## Modes

| Mode | Default | Behavior |
|------|---------|----------|
| `direct` (Quick-fix) | Yes | Intelligent single-pass polish, minimal interaction |
| `guided` | | Three-step flow: structure -> logic -> expression, checklist at each step |
| `batch` | | Quick-fix across multiple sections/files with same settings |

**Default mode:** `direct` (Quick-fix). User says "polish this" and gets polished output.

**Mode inference:** "guided polish" or "step by step" switches to `guided`. "Polish all sections" switches to `batch`.
```

### Guided Workflow Snippet

```markdown
### Guided Mode Steps

Each step follows the same pattern: analyze -> present checklist -> user selects -> apply fixes.

| Step | Focus | Key Checks |
|------|-------|------------|
| 1. Structure | Paragraph organization, topic sentences, section flow | Redundancy, ordering, paragraph purpose |
| 2. Logic | Argument coherence, claim support, cross-section consistency | Argument chains, terminology, evidence links |
| 3. Expression | Word choice, sentence clarity, tone, anti-AI patterns | Translationese, conciseness, academic register |

After each step:
- Apply selected fixes using Edit tool with annotation comments
- Report changes made
- Re-read updated text before proceeding to next step
```

### LaTeX Annotation Example

```latex
% [Polish] Original: The results demonstrate that the proposed method is very effective.
The results indicate that the proposed method achieves competitive accuracy on both benchmarks.
```

### Summary Report Example

```markdown
## Polish Summary

**Changes:** 12 modifications (5 expression, 4 logic, 3 structure)
**Main types:** Hedging calibration, sentence consolidation, transition improvement
**Notes:** Detected translationese patterns in paragraphs 2 and 5 (likely from translation)

> Recommend running De-AI Skill for further detection check.
```

## State of the Art

| Aspect | Approach | Rationale |
|--------|----------|-----------|
| Default mode | Quick-fix (direct) single-pass | User decision; most polish requests want fast results |
| Guided workflow | Fixed 3-step with checklist | User decision; avoids old sentence-by-sentence tedium |
| Change tracking | LaTeX comment annotations | User decision; works in any editor, valid LaTeX |
| Anti-AI integration | Proactive loading during polishing | Established in Phase 3; consistent across Skills |
| Translation detection | Automatic heuristic-based | User decision; not a flag the user sets |
| Journal adaptation | Template-driven style | Phase 1 architecture; journal contract is source of truth |

## Validation Architecture

### Test Framework

| Property | Value |
|----------|-------|
| Framework | Manual validation (Skill is a markdown prompt, not executable code) |
| Config file | None |
| Quick run command | Manual: invoke Skill with test input and verify behavior |
| Full suite command | Manual: run all test scenarios below |

### Phase Requirements to Test Map

| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| CORE-02a | Quick-fix mode polishes text in a single pass when user says "polish this" | manual | Invoke Skill with a .tex file; verify single-pass edit with annotations | N/A |
| CORE-02b | Guided mode activates on explicit request and follows structure->logic->expression | manual | Say "guided polish"; verify three-step flow with checklists | N/A |
| CORE-02c | LaTeX comment annotations preserve original text before each modification | manual | Check edited file for `% [Polish] Original:` annotations | N/A |
| CORE-02d | Annotation cleanup removes all polish annotations after user acceptance | manual | Confirm acceptance; verify all `% [Polish] Original:` lines removed | N/A |
| CORE-02e | Summary report generated with change count, types, and notes | manual | Verify summary appears after polishing completes | N/A |
| CORE-02f | Pasted text polished directly in conversation without file writing | manual | Paste text instead of file path; verify output in session | N/A |
| CORE-02g | Journal-specific style applied when journal specified | manual | Specify CEUS; verify cautious verbs, no promotional language | N/A |
| CORE-02h | Missing journal template triggers refusal | manual | Specify nonexistent journal; verify refusal | N/A |
| CORE-02i | Anti-AI patterns loaded proactively and avoided in output | manual | Check polished text avoids high-risk AI vocabulary | N/A |
| CORE-02j | Translationese detected and addressed when present | manual | Provide text with obvious translation artifacts; verify they are flagged/fixed | N/A |
| CORE-02k | Skill follows Phase 2 conventions (frontmatter, sections, line budget) | manual | Check YAML, required sections, line count <= 300 | N/A |
| CORE-02l | De-AI recommendation appears in summary when applicable | manual | Verify summary includes "Recommend running De-AI Skill" prompt | N/A |

### Sampling Rate

- **Per task commit:** Manual smoke test with one English paragraph (Quick-fix mode)
- **Per wave merge:** Run all 12 test scenarios above
- **Phase gate:** All scenarios pass before `/gsd:verify-work`

### Wave 0 Gaps

- [ ] `.claude/skills/polish-skill/` directory needs to be created
- [ ] Test input files needed: a short English academic paragraph in .tex format with some expression and structural issues, a text with translationese patterns, a multi-section file for guided mode testing
- [ ] No automated testing possible (Skill is a prompt document consumed by Claude at runtime)

## Open Questions

1. **LaTeX annotation format for multi-line originals**
   - What we know: Single-line originals map to `% [Polish] Original: <text>` cleanly
   - What's unclear: How to handle multi-line original text (e.g., a full paragraph rewrite)
   - Recommendation: Each original line gets its own `% [Polish] Original:` prefix. This keeps cleanup simple (grep for prefix, delete those lines). Note this in the Skill as Claude's Discretion.

2. **Cross-step state in Guided mode**
   - What we know: Each step should see the result of previous steps
   - What's unclear: Whether the Skill should re-read the file between steps or maintain state in memory
   - Recommendation: Re-read the file after each step's edits are applied. This is more reliable than tracking in-memory state and ensures the Skill sees the actual file state.

3. **Smart adaptation threshold**
   - What we know: Short text polished entirely, long text split by section
   - What's unclear: Exact threshold
   - Recommendation: ~500 words or 3+ paragraphs as the split point (Claude's Discretion). When splitting, process sections sequentially and maintain terminology awareness across sections.

## Sources

### Primary (HIGH confidence)
- `references/skill-conventions.md` -- full Skill authoring contract
- `references/skill-skeleton.md` -- copyable skeleton structure
- `references/expression-patterns.md` + leaf files -- expression pattern library
- `references/anti-ai-patterns.md` + leaf files -- anti-AI vocabulary and patterns
- `references/journals/ceus.md` -- CEUS journal contract
- `.claude/skills/translation-skill/SKILL.md` -- structural precedent (Phase 3 output)
- `.planning/phases/04-polish-skill/04-CONTEXT.md` -- locked user decisions

### Secondary (MEDIUM confidence)
- `.planning/phases/03-translation-skill/03-RESEARCH.md` -- Phase 3 research approach (parallel structure)
- `.planning/phases/03-translation-skill/03-01-PLAN.md` -- Phase 3 plan format precedent

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH -- all assets are project-internal and built in Phases 1-3
- Architecture: HIGH -- Skill conventions fully defined; Translation Skill provides precedent; user decisions are detailed
- Pitfalls: HIGH -- pitfalls derived from locked decisions and concrete design constraints

**Research date:** 2026-03-11
**Valid until:** 2026-04-11 (stable; no external dependencies that change frequently)
