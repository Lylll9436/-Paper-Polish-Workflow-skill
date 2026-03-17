# Phase 12: AskUserQuestion Fix - Research

**Researched:** 2026-03-17
**Domain:** Claude Code Skill authoring -- tool name correction and structural alignment
**Confidence:** HIGH

## Summary

Phase 12 is a focused bug fix and structural alignment task for `paper-polish-workflow/SKILL.md`. The current file (348 lines) uses incorrect tool names inherited from an older MCP-based naming convention (`mcp_question`, `mcp_read`, `mcp_write`, `mcp_edit`, `mcp_look_at`) and does not follow the standard Skill skeleton structure established in `references/skill-skeleton.md`. The fix requires both renaming all tool references to correct Claude Code names (`AskUserQuestion`, `Read`, `Write`, `Edit`) and restructuring the file to match the skeleton's required sections (Purpose, Trigger, Modes, References, Ask Strategy, Workflow, Output Contract, Edge Cases, Fallbacks).

The CONTEXT.md decisions are unusually clear and complete: merge the current 6-step workflow into 4 steps, add standard frontmatter and missing body sections, compress from 348 lines to within the 300-line budget, and deliver in one shot. Two well-structured sibling Skills (`polish-skill` at 197 lines, `translation-skill` at 210 lines) serve as direct structural references.

**Primary recommendation:** Treat this as a single-file rewrite guided by the skeleton template, not an incremental edit. The structural changes are too pervasive for patch-style editing.

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions
- **Full restructure**: Fix all tool names AND restructure to match skill-skeleton.md structure
- Add standard frontmatter (triggers, tools, references, input_modes, output_contract)
- Add missing convention sections: Modes, Ask Strategy, Edge Cases, Fallbacks
- Compress from 347 lines to within 300-line budget
- One-shot delivery -- no phased approach
- Merge current 6 steps into 4 steps:
  1. Collect Context (pre-polish baseline + reference loading)
  2. Structure & Logic Confirmation (macro structure + per-sentence logic)
  3. Expression Polish & Consistency (expression selection via AskUserQuestion + repetition/coherence check + cross-section consistency)
  4. Output (final review + write to file)
- Step 4.5 (journal style check) and Step 5.7 (highlights) fold into relevant merged steps
- Tool name corrections: `mcp_question` -> `AskUserQuestion`, `mcp_read` -> `Read`, `mcp_write` -> `Write`, `mcp_edit` -> `Edit`, `mcp_look_at` -> `Read`
- Use pseudocode format: `AskUserQuestion({ question:..., options:[...] })`
- Use generic placeholders (`[Expression A]`, `[Expression B]`) instead of hardcoded content
- One concise example in the workflow is sufficient
- Replace `mcp_look_at` with `Read` -- Claude Code's Read tool supports PDF files
- Follow skill-conventions.md Fallback Rules pattern
- Structured interaction unavailable -> fall back to 1-3 plain-text questions, don't block workflow
- Remove the old "If mcp_question is unavailable, use text-based table format" pattern

### Claude's Discretion
- Exact wording of the 4 merged workflow steps
- How to distribute sub-steps (journal check, highlights, cross-section) across the 4 main steps
- Frontmatter field values (triggers examples, tools list, references paths)
- Whether to keep the "Common Issue Handling" table or fold it into Edge Cases

### Deferred Ideas (OUT OF SCOPE)
None -- discussion stayed within phase scope.
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| UXFIX-01 | paper-polish-workflow uses AskUserQuestion for all structured questions instead of plain dialogue | Tool name inventory (17 occurrences), skeleton template structure, AskUserQuestion enforcement rules from skill-conventions.md lines 190-224, sibling Skill examples |
</phase_requirements>

## Standard Stack

This phase does not involve code libraries. It is a Markdown file rewrite within an established project convention.

### Core Assets

| Asset | Path | Purpose | Authority |
|-------|------|---------|-----------|
| Skill conventions | `references/skill-conventions.md` | Authoritative rules for Skill structure, AskUserQuestion enforcement, line budget | PRIMARY -- all decisions must comply |
| Skill skeleton | `references/skill-skeleton.md` | Copyable template that the restructured file must match | PRIMARY -- structural blueprint |
| Target file | `paper-polish-workflow/SKILL.md` | The file being rewritten (current: 348 lines, wrong tool names) | INPUT -- current state |
| CEUS journal contract | `references/journals/ceus.md` | Referenced by the Skill's workflow for journal-specific formatting | DEPENDENCY -- must remain referenced |
| Expression patterns overview | `references/expression-patterns.md` | Referenced by the Skill's workflow for academic expressions | DEPENDENCY -- must remain referenced |

### Sibling Skills (structural references)

| Skill | Lines | Relevant Pattern |
|-------|-------|-----------------|
| `polish-skill` | 197 | Best example of correct tool names, modes, ask strategy, edge cases, fallbacks |
| `translation-skill` | 210 | Best example of complete frontmatter, reference loading rules, workflow steps |

## Architecture Patterns

### Required Skeleton Structure

The rewritten `paper-polish-workflow/SKILL.md` must follow this exact section order (from `skill-skeleton.md`):

```
---
[YAML frontmatter]
---

## Purpose
## Trigger
## Modes
## References
  ### Required (always loaded)
  ### Leaf Hints (loaded when needed)
  ### Loading Rules
## Ask Strategy
## Workflow
  ### Step 1: Collect Context
  ### Step 2: Structure & Logic Confirmation
  ### Step 3: Expression Polish & Consistency
  ### Step 4: Output
## Output Contract
## Edge Cases
## Fallbacks
```

### Frontmatter Contract

Must include all required fields per `skill-conventions.md`:

```yaml
---
name: paper-polish-workflow
description: >-
  [bilingual description with trigger keywords]
triggers:
  primary_intent: [one phrase]
  examples:
    - [English and Chinese trigger phrases]
tools:
  - Read
  - Edit
  - Structured Interaction
references:
  required:
    - references/expression-patterns.md
  leaf_hints:
    - [expression pattern leaves]
    - references/anti-ai-patterns.md
    - references/journals/ceus.md
input_modes:
  - file
  - pasted_text
output_contract:
  - polished_text
  - [other outputs]
---
```

Key rules from conventions:
- `tools` uses capability categories (`Structured Interaction`), not vendor tool names
- `name` must match parent directory name: `paper-polish-workflow`
- `triggers.examples` must include both English and Chinese phrases

### AskUserQuestion Pseudocode Format

Per CONTEXT.md and skill-conventions.md line 201-212, use this format:

```
AskUserQuestion({
  question: "Which expression do you prefer for [sentence function]?",
  options: [
    { label: "[Expression A]", description: "[full sentence option]" },
    { label: "[Expression B]", description: "[full sentence option]" },
    { label: "[Expression C]", description: "[full sentence option]" }
  ]
})
```

Rules:
- Use generic placeholders, not hardcoded content
- One concise example in the workflow is sufficient (per CONTEXT.md)
- No `questions` array wrapping -- single question per call
- No `multiple: false` field (not part of the convention example format)

### Four-Step Workflow Mapping

Current steps map to the new 4-step structure as follows:

| New Step | Current Steps Merged | Key Sub-Operations |
|----------|---------------------|--------------------|
| **Step 1: Collect Context** | Phase 0 (Pre-Polish Baseline) | Load references, identify journal, read input, extract key numbers, locate example papers |
| **Step 2: Structure & Logic Confirmation** | Step 1 (Macro Structure) + Step 2 (Per-Sentence Logic) | Present structure table, confirm logic chain, checkpoint before expression work |
| **Step 3: Expression Polish & Consistency** | Step 3 (Expressions via AskUserQuestion) + Step 4 (Reference Consultation) + Step 4.5 (Journal Style Check) + Step 5 (Repetition & Coherence) + Step 5.5 (Cross-Section Consistency) + Step 5.7 (Highlights) | Expression options via AskUserQuestion, reference consultation on demand, journal style check, repetition/coherence pass, cross-section consistency, highlights if journal requires |
| **Step 4: Output** | Step 6 (Read-Aloud Check) + Write to File | Final review suggestion, compile confirmed content, write to `*_polished.md`, report word count |

### Mode Support

Per the skeleton pattern, `paper-polish-workflow` should declare modes. Based on the current file's behavior:

| Mode | Default | Behavior |
|------|---------|----------|
| `interactive` | Yes | Full 4-step flow with user confirmation at each decision |
| `guided` | | Multi-pass with confirmation at key checkpoints, not every sentence |
| `direct` | | Single-pass polish, skip AskUserQuestion, use defaults |
| `batch` | | Same operation across multiple sections |

Default should be `interactive` since the current workflow is deeply interactive (sentence-by-sentence confirmation). This differs from `polish-skill` which defaults to `direct`.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Skeleton structure | Invent a new layout | Copy `skill-skeleton.md` section order exactly | Convention compliance is mandatory |
| AskUserQuestion syntax | Create a new invocation format | Use the pseudocode format from `skill-conventions.md` lines 201-212 | Consistency across all Skills |
| Fallback behavior | Write custom fallback logic | Follow `skill-conventions.md > Fallback Rules` pattern | Three standard fallback scenarios already defined |
| Edge cases | Invent new scenarios | Combine existing `Common Issue Handling` table with skeleton Edge Cases format | Current table has useful content, just needs reformatting |

## Common Pitfalls

### Pitfall 1: Exceeding the 300-line budget
**What goes wrong:** The current file is 348 lines. Restructuring adds new sections (Modes, Ask Strategy, Edge Cases, Fallbacks, frontmatter) which could inflate the count further.
**Why it happens:** The current file has extensive code examples with hardcoded content, a full "Tools Used" section that is redundant with frontmatter, and verbose markdown formatting.
**How to avoid:**
- Remove the "Tools Used" section entirely (tools are declared in frontmatter)
- Remove "Key Features" section (covered by Purpose)
- Remove "Core Principles" section (distributed into workflow steps)
- Use one concise AskUserQuestion example with generic placeholders
- Remove all hardcoded expression content from examples
- Fold "Journal-Specific Requirements" into References section
- Fold "Common Issue Handling" into Edge Cases
**Warning signs:** If the draft exceeds 280 lines before adding the workflow section, it is too verbose.

### Pitfall 2: Inconsistent tool name replacement
**What goes wrong:** Missing one or more occurrences of `mcp_*` tool names.
**Why it happens:** Tool names appear in prose text, code blocks, section headers, and table cells -- 17 total occurrences across different contexts.
**How to avoid:** Since this is a full rewrite (not search-and-replace), the implementer writes fresh content referencing correct names. Post-write verification must grep for any remaining `mcp_` strings.
**Warning signs:** Any occurrence of `mcp_` in the final file.

### Pitfall 3: Losing workflow functionality during consolidation
**What goes wrong:** Important sub-steps (journal style check, highlights generation, cross-section consistency) get dropped during the merge from 6 steps to 4.
**Why it happens:** These are currently in half-steps (4.5, 5.5, 5.7) that are easy to overlook.
**How to avoid:** Use the mapping table above. Every current step must appear somewhere in the new 4-step structure. Key items to verify:
- Journal style check (Step 4.5) -> Step 3 or Step 4
- Cross-section consistency (Step 5.5) -> Step 3
- Highlights generation (Step 5.7) -> Step 4
- Reference paper consultation (Step 4) -> Step 3 (on-demand trigger)
- Read-aloud suggestion (Step 6) -> Step 4

### Pitfall 4: Wrong `name` field in frontmatter
**What goes wrong:** Using `paper-polish` or `paper-polish-skill` instead of `paper-polish-workflow`.
**Why it happens:** Other skills use the `-skill` suffix, but this one is `-workflow`.
**How to avoid:** The `name` field must exactly match the parent directory name: `paper-polish-workflow`.

### Pitfall 5: Omitting bilingual trigger examples
**What goes wrong:** The triggers.examples list contains only English phrases.
**Why it happens:** The current file has Chinese triggers but they are scattered, not in frontmatter.
**How to avoid:** Per `skill-conventions.md`, triggers.examples should include both English and Chinese phrases. Current Chinese triggers to preserve: "润色论文", "精修论文".

## Code Examples

### Correct Frontmatter (verified against skill-conventions.md and sibling Skills)

```yaml
---
name: paper-polish-workflow
description: >-
  Systematic top-down workflow for polishing academic papers.
  Structure to logic to expression with user confirmation at each step.
  润色论文、精修论文、学术写作改进。
triggers:
  primary_intent: polish academic paper through structured top-down workflow
  examples:
    - "Polish my paper section by section"
    - "润色论文"
    - "Help me revise my introduction step by step"
    - "精修论文"
    - "Guide me through polishing this draft"
    - "帮我逐步润色这篇论文"
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
    - references/journals/ceus.md
input_modes:
  - file
  - pasted_text
output_contract:
  - polished_text
  - change_summary
---
```

### Correct AskUserQuestion Example (per conventions)

```
AskUserQuestion({
  question: "Which expression do you prefer for [sentence function]?",
  options: [
    { label: "[Expression A]", description: "[full sentence with expression A]" },
    { label: "[Expression B]", description: "[full sentence with expression B]" },
    { label: "[Expression C]", description: "[full sentence with expression C]" }
  ]
})
```

### Correct Fallback Pattern (per skill-conventions.md and sibling Skills)

```markdown
## Fallbacks

| Scenario | Fallback |
|----------|----------|
| Structured Interaction unavailable | Ask 1-3 plain-text questions covering highest-impact gaps; do not block workflow |
| Reference file missing | Log the missing file, proceed with reduced capability, warn the user |
| Target journal not specified | Ask once; if declined, use general academic style |
| PDF reference paper unreadable | Ask user to paste relevant excerpts instead |
```

## Inventory of Changes

### Tool Name Occurrences (17 total in current file)

| Line | Current | Replacement | Context |
|------|---------|-------------|---------|
| 14 | `mcp_question` | `AskUserQuestion` | Key Features prose |
| 32 | `mcp_question` | `AskUserQuestion` | Core Principles prose |
| 118 | `mcp_question` | `AskUserQuestion` | Step 3 heading |
| 122 | `mcp_question` | `AskUserQuestion` | Step 3 instruction |
| 127 | `mcp_question` | `AskUserQuestion` | Step 3 action item |
| 132 | `mcp_question` | `AskUserQuestion` | Code example function call |
| 176 | `mcp_question` | `AskUserQuestion` | Benefits heading |
| 182 | `mcp_question` | `AskUserQuestion` | Fallback text |
| 200 | `mcp_look_at` | `Read` | Step 4 instruction (PDF analysis) |
| 237 | `mcp_question` | `AskUserQuestion` | Step 5 heading |
| 240 | `mcp_question` | `AskUserQuestion` | Step 5 code example |
| 330 | `mcp_look_at` | `Read` | Common Issue Handling table |
| 341 | `mcp_read` | `Read` | Tools Used table |
| 342 | `mcp_look_at` | `Read` | Tools Used table |
| 343 | `mcp_question` | `AskUserQuestion` | Tools Used table |
| 344 | `mcp_write` | `Write` | Tools Used table |
| 345 | `mcp_edit` | `Edit` | Tools Used table |

### Sections to Remove (absorbed into skeleton sections)

| Current Section | Disposition |
|----------------|-------------|
| `## Key Features` | Absorbed into `## Purpose` |
| `## Core Principles` | Distributed into workflow step descriptions |
| `## Tools Used` | Removed entirely (frontmatter `tools` field replaces it) |
| `## Journal-Specific Requirements` | Absorbed into `## References` loading rules |
| `## Common Issue Handling` | Fold into `## Edge Cases` table |

### Sections to Add (from skeleton template)

| New Section | Content Source |
|-------------|---------------|
| `## Purpose` | Synthesize from Key Features + first paragraph |
| `## Trigger` | Restructure from Trigger Conditions |
| `## Modes` | New -- define interactive/guided/direct/batch |
| `## References` | Synthesize from reference loading in workflow |
| `## Ask Strategy` | Synthesize from Phase 0 pre-questions + conventions |
| `## Edge Cases` | Combine from Common Issue Handling + new scenarios |
| `## Fallbacks` | New -- follow conventions pattern |

## Validation Architecture

### Test Framework

| Property | Value |
|----------|-------|
| Framework | Manual validation (Markdown file, no automated test runner) |
| Config file | N/A |
| Quick run command | `grep -c 'mcp_' paper-polish-workflow/SKILL.md` (must return 0) |
| Full suite command | See Req-Test Map below |

### Phase Requirements -> Test Map

| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| UXFIX-01 | No `mcp_*` tool names remain | smoke | `grep -c 'mcp_' paper-polish-workflow/SKILL.md` -- must return 0 | N/A (inline) |
| UXFIX-01 | `AskUserQuestion` appears in file | smoke | `grep -c 'AskUserQuestion' paper-polish-workflow/SKILL.md` -- must return >= 1 | N/A (inline) |
| UXFIX-01 | Correct Read/Write/Edit tool names | smoke | `grep -cE '\bRead\b|\bWrite\b|\bEdit\b' paper-polish-workflow/SKILL.md` -- must return >= 1 | N/A (inline) |
| UXFIX-01 | File within 300-line budget | smoke | `wc -l paper-polish-workflow/SKILL.md` -- must be <= 300 | N/A (inline) |
| UXFIX-01 | All skeleton sections present | manual | Verify section headers match skeleton order | N/A |
| UXFIX-01 | YAML frontmatter valid | smoke | `head -50 paper-polish-workflow/SKILL.md` contains `---` delimiters and required fields | N/A |
| UXFIX-01 | 4-step workflow (not 6) | manual | Count `### Step` headers -- must be exactly 4 | N/A |
| UXFIX-01 | No lost sub-steps | manual | Verify journal style check, highlights, cross-section consistency, reference consultation, read-aloud all present | N/A |

### Sampling Rate
- **Per task commit:** `grep -c 'mcp_' paper-polish-workflow/SKILL.md && wc -l paper-polish-workflow/SKILL.md`
- **Phase gate:** All automated commands return expected values; manual section review passes

### Wave 0 Gaps
None -- no test infrastructure needed for a Markdown file rewrite. Validation is inline grep/wc commands plus manual review.

## Open Questions

1. **`output_contract` values for paper-polish-workflow**
   - What we know: The Skill produces polished text and writes to `*_polished.md`. It also generates highlights when targeting CEUS.
   - What's unclear: Whether to list `highlights` as a separate output type or fold it into the main polished output.
   - Recommendation: Keep it simple -- `polished_text` and `change_summary` (matching the pattern from `polish-skill`). Highlights are conditional on journal, not a core output.

2. **Whether to keep "Common Issue Handling" table or fold into Edge Cases**
   - What we know: CONTEXT.md lists this as Claude's discretion.
   - What's unclear: The current table has 5 entries with useful guidance.
   - Recommendation: Fold into Edge Cases table. The skeleton requires Edge Cases; having both would waste lines. The content maps naturally: "This word isn't professional" -> edge case with handling.

## Sources

### Primary (HIGH confidence)
- `references/skill-conventions.md` -- Full authoring rules, AskUserQuestion enforcement (lines 190-224), fallback rules, line budget, frontmatter contract
- `references/skill-skeleton.md` -- Copyable template defining required section order and content shape
- `paper-polish-workflow/SKILL.md` -- Current file (348 lines, 17 wrong tool name occurrences)
- `.claude/skills/polish-skill/SKILL.md` -- Sibling Skill exemplar (197 lines, correct structure)
- `.claude/skills/translation-skill/SKILL.md` -- Sibling Skill exemplar (210 lines, correct structure)
- `.planning/phases/12-askuserquestion-fix/12-CONTEXT.md` -- User decisions locking scope

### Secondary (MEDIUM confidence)
- `references/expression-patterns.md` -- Referenced by the Skill; confirmed path structure
- `references/journals/ceus.md` -- Referenced by the Skill; confirmed content and path

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH -- all assets are local project files, fully readable and verified
- Architecture: HIGH -- skeleton template and two sibling Skills provide unambiguous structural guidance
- Pitfalls: HIGH -- derived from direct analysis of the 348-line current file vs. conventions

**Research date:** 2026-03-17
**Valid until:** Indefinite (project-internal conventions, not external dependencies)

---
*Phase: 12-askuserquestion-fix*
*Research: .planning/phases/12-askuserquestion-fix/12-RESEARCH.md*
