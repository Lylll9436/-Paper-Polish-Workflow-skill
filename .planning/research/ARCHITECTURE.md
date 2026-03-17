# Architecture Research: v2.0 Integration

**Domain:** Multi-Skill Academic Writing Tool Suite for Claude Code (subsequent milestone)
**Researched:** 2026-03-17
**Confidence:** HIGH -- all integration points verified against existing codebase

## Executive Summary

v2.0 adds three features to an established 11-Skill architecture: (1) a new Repo-to-Paper Skill that scans code repositories and generates structured paper drafts in a top-down H1-H2-H3-body workflow, (2) bilingual paragraph-by-paragraph comparison output extended across translation, polish, and the new repo-to-paper Skill, and (3) a prompt engineering fix for AskUserQuestion misuse in the legacy paper-polish-workflow SKILL.md.

The existing architecture is well-suited for these additions. The flat multi-Skill pattern, shared reference library, and convention system all remain valid. No structural changes to v1.0 components are required. The primary new artifact is a single new Skill directory (`.claude/skills/repo-to-paper-skill/`), plus a new reference file category for literature metadata saved during repo-to-paper execution.

## System Overview: v2.0 Delta

```
EXISTING (unchanged)                    NEW / MODIFIED
============================            ============================

.claude/skills/                         .claude/skills/
  abstract-skill/SKILL.md                 repo-to-paper-skill/SKILL.md  [NEW]
  caption-skill/SKILL.md
  cover-letter-skill/SKILL.md
  de-ai-skill/SKILL.md
  experiment-skill/SKILL.md
  literature-skill/SKILL.md
  logic-skill/SKILL.md
  polish-skill/SKILL.md               >> polish-skill/SKILL.md      [MODIFIED: bilingual output]
  reviewer-simulation-skill/SKILL.md
  translation-skill/SKILL.md          >> translation-skill/SKILL.md  [MINOR: verify bilingual pattern]
  visualization-skill/SKILL.md

references/                             references/
  expression-patterns.md                  (unchanged)
  expression-patterns/*.md                (unchanged)
  anti-ai-patterns.md                     (unchanged)
  anti-ai-patterns/*.md                   (unchanged)
  journals/ceus.md                        (unchanged)
  skill-conventions.md                    (unchanged)
  skill-skeleton.md                       (unchanged)

paper-polish-workflow/                  paper-polish-workflow/
  SKILL.md                             >> SKILL.md                   [MODIFIED: AskUserQuestion fix]

                                        [USER PROJECT]/
                                          .paper-refs/               [NEW: output directory]
                                            {topic}-refs.md          [NEW: literature metadata files]
```

## Component Responsibilities

| Component | Type | Status | Responsibility |
|-----------|------|--------|---------------|
| `repo-to-paper-skill` | New Skill | NEW | Scan arbitrary repo, generate structured paper draft via top-down H1-H2-H3-body workflow with user checkpoints |
| `polish-skill` | Existing Skill | MODIFY | Add bilingual paragraph-by-paragraph comparison output option |
| `translation-skill` | Existing Skill | VERIFY | Already has bilingual `_bilingual.tex` output; verify pattern matches v2.0 standard |
| `paper-polish-workflow` | Legacy Skill | MODIFY | Fix AskUserQuestion prompt engineering to use structured tool instead of plain dialogue |
| `.paper-refs/` | Output convention | NEW | Directory convention for literature metadata files saved during repo-to-paper execution |
| `references/` | Shared library | UNCHANGED | No new reference files needed in the shared library |

## Feature 1: Repo-to-Paper Skill

### Architecture Decision: Single New Skill (Not Orchestrator)

The repo-to-paper Skill is a new standalone Skill, not an orchestrator that chains existing Skills. Rationale:

- **Literature search integration** within the Skill calls Semantic Scholar MCP directly (same pattern as `literature-skill`) rather than invoking `literature-skill` as a sub-Skill, because: (a) Claude Code does not reliably support Skill-calling-Skill, (b) the repo-to-paper context (H2-level sections) is needed for targeted queries, and (c) the output format (saved ref files with abstracts) differs from `literature-skill`'s interactive BibTeX workflow.
- **Expression patterns** are loaded the same way all other Skills load them -- via the stable-entrypoint-plus-leaf-module architecture.
- **The Skill is independently invokable.** Users can trigger it without having run any other Skill first.

### Data Flow: Repo Scan to Paper Draft

```
User provides: repo path (or current repo) + target journal + paper topic
    |
    v
Step 0: MCP Pre-flight (Semantic Scholar)
    |  Confirm MCP availability before starting multi-phase workflow.
    |  If unavailable: warn, proceed without literature (use placeholders).
    |
    v
Step 1: Repo Scan  [tools: Glob, Grep, Read]
    |  Glob for code files, configs, READMEs, notebooks
    |  Grep for key patterns (model names, dataset names, evaluation metrics)
    |  Read key files (README, main scripts, config files, result files)
    |  Output: Repo Understanding Document (internal, not shown to user)
    |
    v
Step 2: H1 Structure  [user checkpoint]
    |  Propose top-level paper sections based on repo analysis
    |  Default: Introduction / Related Work / Methods / Experiments / Discussion / Conclusion
    |  User confirms or adjusts H1 structure
    |
    v
Step 3: H2 Structure + Literature  [user checkpoint]
    |  For each H1 section, propose H2 subsections
    |  Call Semantic Scholar MCP for each H2 topic:
    |    - mcp__semantic-scholar__papers-search-basic for each subsection theme
    |    - mcp__semantic-scholar__get-paper-abstract for top hits
    |  Save literature metadata to .paper-refs/{section}-refs.md
    |  Present H2 structure with associated references to user
    |  User confirms or adjusts
    |
    v
Step 4: H3 Structure  [user checkpoint]
    |  For each H2 subsection, propose H3 paragraphs/points
    |  Map repo artifacts (code, results, configs) to specific H3 points
    |  User confirms or adjusts
    |
    v
Step 5: Body Generation  [per-section, with expression patterns]
    |  Load appropriate expression-patterns leaf for each section
    |  Load anti-ai-patterns for vocabulary screening
    |  Load journal template if specified
    |  Generate body text section by section
    |  Use [CITE: author2024keyword] placeholders linked to saved refs
    |  Use [TODO: ...] for content the Skill cannot determine from repo
    |  Bilingual output: English body + Chinese comment lines (same as translation-skill)
    |
    v
Step 6: Output  [tools: Write]
    |  Write draft to {topic}_draft.tex (English) and {topic}_draft_bilingual.tex
    |  Write literature ref files to .paper-refs/
    |  Present summary: section count, ref count, TODO count
```

### Repo Scan Strategy

The Skill must handle arbitrary repositories without assuming a specific language, framework, or structure. The scan strategy uses progressive discovery:

```
Priority 1: README.md, README.rst, README.txt
    -> Extract project description, key claims, usage examples

Priority 2: Configuration files
    -> Glob for: *.yaml, *.yml, *.toml, *.json, *.cfg, *.ini
    -> Extract: model names, hyperparameters, dataset paths, experiment configs

Priority 3: Main code files
    -> Glob for: main.py, train.py, run.py, app.py, index.*, src/**/*
    -> Grep for: class names, function signatures, import statements
    -> Extract: architecture names, library dependencies, pipeline structure

Priority 4: Results and evaluation
    -> Glob for: results/, outputs/, logs/, *.csv, *.log
    -> Grep for: accuracy, F1, loss, RMSE, R2, AUC, precision, recall
    -> Extract: metric names, values, comparison baselines

Priority 5: Documentation and notebooks
    -> Glob for: docs/**, *.ipynb, *.md (non-README)
    -> Extract: explanations, visualizations, experiment narratives
```

**Context budget discipline:** The Skill must not read entire large files. Strategy:
- README: read fully (typically < 500 lines)
- Config files: read fully (typically < 200 lines)
- Code files: read first 100 lines for imports/structure, then Grep for specific patterns
- Result files: read first 50 lines for headers and sample data
- Total context target: keep repo scan under ~3000 lines of loaded content

### Literature Reference File Format

Literature metadata files saved during H2 stage use a consistent format. These are NOT placed in the shared `references/` library (which is project-owned, not user-paper-owned). They go into a `.paper-refs/` directory within the user's working context.

```markdown
# Literature References: {Section Title}

**Generated:** {date}
**Query:** {Semantic Scholar search query}

## Ref 1: {Author et al., Year}

**Title:** {Full title from MCP}
**Authors:** {Author list from MCP}
**Year:** {Year}
**Citation Count:** {Count from MCP}
**Abstract:** {Abstract from MCP}
**Relevance:** {1-sentence explanation of why this paper relates to the section}

**BibTeX:**
```bibtex
@article{key,
  title = {...},
  author = {...},
  ...
}
```

## Ref 2: ...
```

**Design rationale for .paper-refs/ location:**
- `references/` is reserved for the project's shared reference library (expression patterns, anti-AI patterns, journal templates). It is version-controlled and maintained across all users.
- `.paper-refs/` contains per-paper, per-session literature artifacts. It is user-generated output, not a shared resource.
- The Skill instructs users to add `.paper-refs/` to `.gitignore` if they do not want to track these files.
- Other Skills (experiment-skill, abstract-skill) can Read these files if the user asks to connect literature.

### Integration with Existing References

| Reference | How Repo-to-Paper Uses It | When Loaded |
|-----------|---------------------------|-------------|
| `expression-patterns.md` (overview) | Orient which leaf to load per section | Step 5 start |
| `expression-patterns/introduction-and-gap.md` | Introduction body generation | Step 5 for Introduction |
| `expression-patterns/methods-and-data.md` | Methods body generation | Step 5 for Methods |
| `expression-patterns/results-and-discussion.md` | Results and Discussion body | Step 5 for Results/Discussion |
| `expression-patterns/conclusions-and-claims.md` | Conclusion body generation | Step 5 for Conclusion |
| `expression-patterns/geography-domain.md` | When repo involves spatial/urban topics | Step 5 conditional |
| `anti-ai-patterns.md` | Overview for vocabulary screening | Step 5 |
| `anti-ai-patterns/vocabulary.md` | Screen generated body text | Step 5 post-generation |
| `journals/ceus.md` | Journal style and structure constraints | Steps 2-5 when journal specified |

### Frontmatter Design

```yaml
---
name: repo-to-paper-skill
description: >-
  Generate structured paper draft from code repository.
  Top-down H1-H2-H3-body workflow with user checkpoints at each level.
  Integrates Semantic Scholar for literature at H2 stage.
  Triggers: "generate paper from repo", "write paper from code",
  "从代码仓库生成论文", "帮我从项目代码写论文草稿".
triggers:
  primary_intent: generate paper draft from code repository
  examples:
    - "Generate a paper draft from this repo"
    - "从这个代码仓库生成论文草稿"
    - "Write a paper based on my experiment code"
    - "帮我从项目代码写论文"
    - "Draft a CEUS paper from my urban analysis repo"
    - "把我的实验代码整理成论文"
tools:
  - Read
  - Write
  - Glob
  - Grep
  - External MCP
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
input_modes:
  - repo_path
  - current_directory
output_contract:
  - draft_tex
  - bilingual_draft_tex
  - literature_refs
---
```

**Notable frontmatter choices:**
- `tools` includes `Glob` and `Grep` -- first Skill in the suite to require these for repo scanning. All other Skills use only Read/Write/Edit.
- `tools` includes `External MCP` following the convention fix identified in v1.0 tech debt (not "Semantic Scholar MCP").
- `input_modes` introduces `repo_path` and `current_directory` -- new input modes not used by any v1.0 Skill.
- `references.required` includes only `expression-patterns.md` (same as translation, polish, experiment Skills). Literature references are generated at runtime, not pre-loaded.

## Feature 2: Bilingual Paragraph-by-Paragraph Output

### Existing Patterns Audit

The codebase has two distinct bilingual output patterns:

**Pattern A: Inline Blockquote (reviewer-simulation-skill, logic-skill)**
```markdown
**Problem:** [English description]
**Why this matters:** [English impact]
**Suggestion:** [English guidance]

> **[Chinese]** [Chinese translation of all three parts]
```
- Used for: structured analysis output (concerns, issues)
- Granularity: per-concern/per-issue
- Language: Chinese follows English immediately, as blockquote

**Pattern B: LaTeX Comment Comparison (translation-skill)**
```latex
% --- Paragraph N ---
% [Chinese original text line 1]
% [Chinese original text line 2]
English translated text for paragraph N.
```
- Used for: full body text output
- Granularity: per-paragraph
- Language: Chinese precedes English, as LaTeX comments

### Recommended v2.0 Bilingual Standard

**Use Pattern B (LaTeX comment comparison) for all body text output across Skills.** This is the correct pattern for paragraph-by-paragraph comparison because:

1. It produces compilable LaTeX -- Chinese text is comments, English text is body.
2. Paragraph markers (`% --- Paragraph N ---`) make alignment unambiguous.
3. The user can toggle bilingual visibility by searching for/removing comment lines.
4. Translation-skill already implements this pattern, so it becomes the canonical example.

**Use Pattern A (inline blockquote) for analysis/review output.** This remains correct for reviewer-simulation-skill and logic-skill because their output is structured Markdown reports, not LaTeX body text.

### Skills Requiring Changes

| Skill | Current Bilingual | v2.0 Change | Effort |
|-------|-------------------|-------------|--------|
| `translation-skill` | Pattern B in `_bilingual.tex` | None -- already canonical | NONE |
| `polish-skill` | None -- produces in-place edits only | Add optional bilingual comparison output when user requests it | LOW |
| `repo-to-paper-skill` | N/A (new) | Implement Pattern B from the start | MEDIUM (built into new Skill) |
| `reviewer-simulation-skill` | Pattern A (inline blockquote) | None -- Pattern A is correct for analysis output | NONE |
| `logic-skill` | Pattern A (inline blockquote) | None -- Pattern A is correct for analysis output | NONE |
| `experiment-skill` | None | Not in v2.0 scope | NONE |

### Polish-Skill Bilingual Extension

The polish-skill currently produces in-place edits with `% [Polish] Original:` annotations. For v2.0, add an optional bilingual comparison output mode:

**Trigger:** User explicitly requests bilingual output ("polish with bilingual comparison", "润色并生成中英对照").

**Implementation:** After polishing is complete, if bilingual was requested:
1. Read the original text (preserved in `% [Polish] Original:` annotations)
2. Generate a `_bilingual.tex` companion file using Pattern B:
   ```latex
   % --- Paragraph N ---
   % [Original English text before polishing]
   [Polished English text for paragraph N]
   ```
3. This is Original-vs-Polished comparison, not Chinese-vs-English.

**Note:** This is a different use case from translation-skill's bilingual (Chinese source vs. English translation). The polish-skill bilingual shows Before vs. After in English. The same LaTeX comment format works for both -- the pattern is structurally identical.

### Repo-to-Paper Bilingual Implementation

Since repo-to-paper generates English body text while the user likely thinks in Chinese, the bilingual output serves as a comprehension aid:

```latex
% --- Paragraph N ---
% [Chinese explanation of what this paragraph says]
% [This is generated Chinese, not source Chinese]
English body text for paragraph N.
```

**Key difference from translation-skill:** In translation-skill, the Chinese lines are the user's original input (preserved verbatim). In repo-to-paper-skill, the Chinese lines are generated alongside the English (parallel generation). The Skill must clearly label this distinction in the output header:

```latex
% Bilingual draft: English body + Chinese comprehension aid
% Note: Chinese text is AI-generated for comprehension, not user-authored source material
```

## Feature 3: AskUserQuestion Fix

### Problem Analysis

The legacy `paper-polish-workflow/SKILL.md` (located at `paper-polish-workflow/SKILL.md`, outside `.claude/skills/`) uses vendor-specific tool names (`mcp_question`, `mcp_read`, `mcp_write`, `mcp_edit`, `mcp_look_at`) that do not match Claude Code's actual tool names. The result: Claude falls back to plain dialogue instead of using AskUserQuestion for structured multi-option selection.

**Root cause:** The SKILL.md was written before the v1.0 convention system was established. It references `mcp_question` (an MCP tool name from a different context) instead of `AskUserQuestion` (Claude Code's built-in structured interaction tool).

### Fix Strategy

The fix is purely prompt engineering -- rewrite the paper-polish-workflow SKILL.md to:

1. Replace all `mcp_question` references with `AskUserQuestion` (Claude Code's actual tool name).
2. Replace `mcp_read`, `mcp_write`, `mcp_edit`, `mcp_look_at` with `Read`, `Write`, `Edit` (Claude Code tool names).
3. Add explicit instructions that match the v1.0 convention for Structured Interaction:
   - "Use AskUserQuestion to present multi-option selections"
   - "When AskUserQuestion is unavailable, fall back to plain-text numbered lists"
4. Optionally: bring the SKILL.md into compliance with v1.0 `skill-conventions.md` (YAML frontmatter, body structure, line budget).

### Scope Decision: Fix vs. Full Rewrite

The paper-polish-workflow SKILL.md (349 lines) is a legacy artifact that predates the v1.0 convention system. Two options:

**Option A: Minimal fix (recommended for v2.0).** Replace tool names, add AskUserQuestion instructions. Keep the existing workflow structure. This is a 30-minute task that directly solves the user's reported problem.

**Option B: Full convention rewrite.** Rewrite the entire SKILL.md to match `skill-conventions.md` format (YAML frontmatter, standard body sections, line budget). This is a 2-hour task that improves consistency but does not add functional value -- the polish-skill already exists as the convention-compliant replacement.

**Recommendation: Option A.** The paper-polish-workflow SKILL.md is a separate entry point (lives outside `.claude/skills/`) with a different interaction philosophy (sentence-by-sentence confirmation vs. polish-skill's quick-fix/guided modes). A full rewrite would risk losing its distinctive interactive workflow. Fix the tool names and add explicit AskUserQuestion instructions.

### Files Modified

| File | Change | Lines Affected |
|------|--------|---------------|
| `paper-polish-workflow/SKILL.md` | Replace `mcp_question` with `AskUserQuestion` | ~10 occurrences |
| `paper-polish-workflow/SKILL.md` | Replace `mcp_read/write/edit/look_at` with `Read/Write/Edit` | ~6 occurrences |
| `paper-polish-workflow/SKILL.md` | Add Structured Interaction fallback note | +5 lines |

## Architectural Patterns

### Pattern 1: Top-Down Generation with User Checkpoints

**What:** Generate structured output by progressively refining from H1 (coarse structure) to body text (fine detail), with explicit user confirmation gates between each level.

**When to use:** Any Skill that generates substantial structured text where the user needs to approve direction before details are committed. Used by: repo-to-paper-skill (new), experiment-skill (existing two-phase pattern).

**Trade-offs:**
- Pro: User catches structural mistakes early, before body text is wasted
- Pro: Each checkpoint is a natural save point
- Con: More interaction turns than a single-pass Skill
- Con: Multi-turn state must be maintained across checkpoints

**Pattern in SKILL.md:**
```markdown
### Step N: [Level] Structure  [user checkpoint]
- Generate [level] structure from [previous level + repo data]
- Present structure to user with numbered items
- Wait for user confirmation: "Confirm, adjust, or add items"
- Apply user adjustments before proceeding to next level
```

### Pattern 2: MCP-Driven Literature at Structure Stage

**What:** Call Semantic Scholar MCP during structure generation (H2 stage) rather than during body generation. Save results as ref files for later use during body writing.

**When to use:** Repo-to-paper-skill. Not applicable to existing Skills (literature-skill is interactive, not structure-driven).

**Trade-offs:**
- Pro: Literature informs structure decisions (user sees which subtopics have strong prior work)
- Pro: Ref files are reusable across body generation iterations
- Con: MCP calls add latency at H2 stage
- Con: Ref files may include irrelevant papers if H2 structure changes after literature fetch

**Implementation detail:** If MCP is unavailable (pre-flight fails), the Skill must NOT block. Proceed with `[LITERATURE NEEDED: topic]` placeholders in the structure and `[CITE: PLACEHOLDER]` in body text. The user can later invoke `literature-skill` independently to fill these.

### Pattern 3: Bilingual LaTeX Comment Comparison

**What:** Produce parallel bilingual output where one language is LaTeX comments and the other is body text, with paragraph markers for alignment.

**When to use:** Any Skill producing full body text that serves bilingual users. Already used by translation-skill; extended to polish-skill and repo-to-paper-skill in v2.0.

**Format specification:**
```latex
% ============================================================
% Bilingual output: [source language] in comments, [target language] as body
% Generated by: [skill-name]
% Date: [date]
% ============================================================

% --- Paragraph 1 ---
% [Source/comparison language line 1]
% [Source/comparison language line 2]
[Target language paragraph 1 body text.]

% --- Paragraph 2 ---
% [Source/comparison language line 1]
[Target language paragraph 2 body text.]
```

### Pattern 4: Progressive Repo Discovery

**What:** Scan an arbitrary repository using a priority-ordered file discovery strategy (README > configs > code > results > docs) with context budget limits per category.

**When to use:** Repo-to-paper-skill only.

**Trade-offs:**
- Pro: Works on any repository regardless of language or framework
- Pro: Context budget prevents overloading on large monorepos
- Con: May miss important files in unconventional repo structures
- Con: Limited understanding of code semantics (reads structure, not logic)

**Mitigation:** At the H1 checkpoint, the Skill presents its understanding of the repo and asks the user to correct any misunderstandings. This catches scan failures early.

## Anti-Patterns to Avoid

### Anti-Pattern 1: Skill-Calling-Skill for Literature

**What people might do:** Have repo-to-paper-skill invoke literature-skill as a sub-Skill for reference searching.

**Why it's wrong:** Claude Code does not reliably support Skill chaining. The sub-Skill invocation would lose the repo-to-paper context (which H2 section needs references), and the interactive selection flow of literature-skill (numbered list, user picks one) does not fit the batch search pattern needed during H2 structure generation.

**Do this instead:** Call `mcp__semantic-scholar__papers-search-basic` directly within repo-to-paper-skill. Implement a non-interactive batch search pattern (top 3 results per H2 topic, auto-selected by relevance, no user selection required). Save results to .paper-refs/ files.

### Anti-Pattern 2: Putting Generated Literature Refs in references/

**What people might do:** Save literature metadata files into `references/` alongside expression-patterns and anti-ai-patterns.

**Why it's wrong:** The `references/` directory is the project's shared, version-controlled reference library. Literature refs are user-generated, paper-specific, session-specific output. Mixing them would pollute the shared library and create merge conflicts for users who track `references/` in git.

**Do this instead:** Use a separate `.paper-refs/` directory convention. Add it to `.gitignore` by default.

### Anti-Pattern 3: Generating Chinese by Translating English

**What people might do:** In repo-to-paper bilingual mode, generate English body first, then translate to Chinese for the comment lines.

**Why it's wrong:** Two-pass generation (English then translate) doubles the generation cost and introduces translation artifacts. The Chinese comprehension aid does not need to be a faithful translation -- it needs to be a clear Chinese explanation of the paragraph's purpose.

**Do this instead:** Generate English and Chinese in parallel (same generation pass). The Chinese lines are comprehension summaries, not word-for-word translations. This is cheaper and more useful.

### Anti-Pattern 4: Full Convention Rewrite of Legacy SKILL.md

**What people might do:** Rewrite paper-polish-workflow/SKILL.md from scratch to match skill-conventions.md.

**Why it's wrong:** The legacy SKILL.md has a distinct interaction philosophy (sentence-by-sentence confirmation with mcp_question batches) that differs from polish-skill's quick-fix/guided modes. A full rewrite risks losing this workflow in an attempt to achieve format compliance. The user's reported problem is specifically about AskUserQuestion not being called -- not about the workflow design.

**Do this instead:** Targeted tool name replacement + AskUserQuestion instruction addition. Preserve the existing workflow structure.

## Integration Points

### External Services

| Service | Integration Pattern | Skills Using It | Notes |
|---------|---------------------|-----------------|-------|
| Semantic Scholar MCP | `mcp__semantic-scholar__papers-search-basic` and `mcp__semantic-scholar__get-paper-abstract` | literature-skill (v1.0), repo-to-paper-skill (v2.0) | Pre-flight check required; graceful degradation with placeholders |

### Internal Boundaries

| Boundary | Communication | Notes |
|----------|---------------|-------|
| repo-to-paper-skill <-> expression-patterns | Read tool at runtime | Same pattern as all writing Skills |
| repo-to-paper-skill <-> .paper-refs/ | Write tool (create), Read tool (consume during body gen) | Skill creates files that it later reads within the same session |
| repo-to-paper-skill <-> journal templates | Read tool at runtime | Same pattern as translation/polish/abstract Skills |
| polish-skill <-> bilingual output | New optional output path | Triggered by user request; uses same format as translation-skill |
| .paper-refs/ <-> other Skills | Manual user action | User can ask experiment-skill or abstract-skill to read .paper-refs/ for literature context |

### Cross-Skill Data Flow (User-Driven)

```
repo-to-paper-skill
    |
    |  generates: {topic}_draft.tex, {topic}_draft_bilingual.tex, .paper-refs/*.md
    |
    v
User decides next step:
    |
    +---> polish-skill        (polish the generated draft)
    +---> de-ai-skill         (screen for AI patterns)
    +---> logic-skill         (verify argument chains)
    +---> reviewer-simulation (self-review before refining)
    +---> experiment-skill    (strengthen results/discussion using .paper-refs/)
    +---> abstract-skill      (generate abstract from draft)
```

This is the same "user-driven chaining" pattern from v1.0. No technical coupling between Skills.

## Suggested Build Order

```
Phase 1: AskUserQuestion Fix
    paper-polish-workflow/SKILL.md  (tool name replacement)
    Rationale: Smallest scope, immediate user value, no dependencies,
    can be completed and tested in isolation.

Phase 2: Bilingual Pattern Standardization
    Verify translation-skill bilingual output matches the v2.0 standard format.
    Add bilingual comparison output option to polish-skill.
    Rationale: Establishes the bilingual pattern before repo-to-paper-skill
    needs it. polish-skill modification is low-risk (additive, not changing
    existing behavior).

Phase 3: Repo-to-Paper Skill (Core Structure)
    Create .claude/skills/repo-to-paper-skill/SKILL.md
    Implement: frontmatter, repo scan strategy, H1/H2/H3 structure workflow
    Implement: user checkpoints at each level
    Rationale: Core Skill structure before literature integration.
    Can be tested with structure generation only (no literature).

Phase 4: Repo-to-Paper Literature Integration
    Add Semantic Scholar MCP integration at H2 stage
    Implement .paper-refs/ output convention
    Add literature metadata file format
    Rationale: Literature integration depends on H2 structure being stable.
    Separated from Phase 3 to allow testing structure independently.

Phase 5: Repo-to-Paper Body Generation + Bilingual Output
    Implement body text generation with expression patterns
    Implement bilingual output (Pattern B) for generated text
    Implement anti-AI vocabulary screening
    Add journal template integration
    Rationale: Body generation depends on all previous phases --
    structure (Phase 3), literature refs (Phase 4), bilingual pattern (Phase 2).
```

### Build Order Rationale

1. **AskUserQuestion fix first** because it is zero-risk, zero-dependency, and immediately fixes a user-reported problem. Getting it done early removes it from the critical path.

2. **Bilingual pattern before repo-to-paper** because repo-to-paper will use the bilingual pattern. Establishing and testing the pattern on the existing polish-skill (which is simpler) validates the approach before implementing it in a more complex new Skill.

3. **Repo-to-paper structure before literature** because the H2 section structure determines what literature queries to make. If you build literature integration first, you have nothing to attach it to. Structure can be tested independently by verifying H1/H2/H3 output quality.

4. **Literature as separate phase** because MCP integration has its own failure modes (MCP unavailable, poor search results, rate limits). Isolating it allows the Skill to be functional without literature (using placeholders) while literature integration is refined.

5. **Body generation last** because it consumes outputs from all previous phases (structure, literature, bilingual pattern, expression patterns, anti-AI patterns). It is the final assembly step.

## New vs. Modified Components Summary

| Component | Action | Estimated Effort | Phase |
|-----------|--------|-----------------|-------|
| `paper-polish-workflow/SKILL.md` | MODIFY (tool name fix) | ~30 min | 1 |
| `translation-skill/SKILL.md` | VERIFY (bilingual pattern check) | ~15 min | 2 |
| `polish-skill/SKILL.md` | MODIFY (add bilingual output option) | ~1 hour | 2 |
| `.claude/skills/repo-to-paper-skill/SKILL.md` | NEW (core structure) | ~3 hours | 3 |
| `.claude/skills/repo-to-paper-skill/SKILL.md` | EXTEND (literature integration) | ~2 hours | 4 |
| `.claude/skills/repo-to-paper-skill/SKILL.md` | EXTEND (body gen + bilingual) | ~3 hours | 5 |
| `.paper-refs/` convention | NEW (documentation in Skill) | Included in Phase 4 | 4 |

**Total new files:** 1 (repo-to-paper-skill/SKILL.md)
**Total modified files:** 2 (polish-skill/SKILL.md, paper-polish-workflow/SKILL.md)
**Total verified files:** 1 (translation-skill/SKILL.md)
**Reference library changes:** 0

## Skill Conventions Compliance

The new repo-to-paper-skill must comply with all existing conventions from `references/skill-conventions.md`:

| Convention | Compliance Plan |
|------------|----------------|
| YAML frontmatter with all required fields | See Frontmatter Design section above |
| ~300 line budget | Target 280 lines; repo scan strategy goes in Workflow section, not separate reference |
| 4 interaction modes declared | `guided` (default, H1-H2-H3 checkpoints), `direct` (skip checkpoints, generate all at once) |
| Body structure (Purpose, Trigger, Modes, References, Ask Strategy, Workflow, Output Contract, Edge Cases, Fallbacks) | Follow skeleton template |
| References via stable-entrypoint-plus-leaf-module | Load expression-patterns.md overview, then section-specific leaves |
| Tool names use capability categories | `External MCP` not `Semantic Scholar MCP` |
| Bilingual triggers in examples | Include both English and Chinese trigger phrases |

### Convention Gaps to Address

The v1.0 conventions do not cover two patterns introduced by repo-to-paper-skill:

1. **`input_modes: repo_path`** -- No existing convention for Skill that takes a directory path as input rather than a file or pasted text. The Ask Strategy must handle: "Provide the path to your repository, or I'll scan the current working directory."

2. **Multi-file output** -- No existing convention for Skills that produce 3+ output files (draft tex, bilingual tex, multiple .paper-refs/ files). The Output Contract section must enumerate all outputs clearly.

These gaps do not require changes to `skill-conventions.md` -- they are documented within the Skill's own body sections as extensions of the existing patterns.

## Sources

- Existing codebase analysis (HIGH confidence -- direct reading of all 11 SKILL.md files, skill-conventions.md, skill-skeleton.md, and all reference files)
- v1.0 architecture research at `.planning/research/ARCHITECTURE.md` (HIGH confidence -- verified against actual implementation)
- Claude Code Skill documentation patterns from v1.0 research sources (HIGH confidence -- already validated by v1.0 implementation)

---
*Architecture research for: v2.0 Repo-to-Paper, Bilingual Enhancement, AskUserQuestion Fix*
*Researched: 2026-03-17*
