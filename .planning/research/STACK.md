# Stack Research: v2.0 Additions

**Domain:** Claude Code Skill suite for academic paper writing -- v2.0 new capabilities
**Researched:** 2026-03-17
**Confidence:** HIGH

> The v1.0 stack (pure markdown Skills, shared reference library, Semantic Scholar MCP, AskUserQuestion, ~300-line budget, 4 interaction modes) is validated and unchanged. This document covers ONLY the stack additions and patterns needed for three v2.0 features: Repo-to-Paper generation, bilingual paragraph-by-paragraph comparison, and AskUserQuestion enforcement.

---

## Validated v1.0 Stack (DO NOT re-research)

| Component | Status | Notes |
|-----------|--------|-------|
| SKILL.md format with YAML frontmatter | Validated | 11 Skills shipped, all convention-compliant |
| Shared reference library (`references/`) | Validated | 5 expression pattern leaves, 3 anti-AI leaves, 1 journal template |
| Semantic Scholar MCP | Validated | Pre-flight check pattern, BibTeX-from-MCP-only rule |
| AskUserQuestion tool | Validated | Available and working; all 11 Skills list `Structured Interaction` |
| Claude Code built-in tools | Validated | Read, Write, Edit, Glob, Grep, Bash |
| ~300-line Skill budget | Validated | All 11 Skills within budget |

---

## New Stack Components for v2.0

### 1. Repo Scanning Tool Strategy

**What:** The repo-to-paper Skill must scan an experiment repository to extract structure, code intent, results, and configuration before drafting a paper.

**Recommended tools:** Use the existing Claude Code built-in tools -- Glob, Grep, Read -- in a specific scanning sequence. No new tools needed.

| Tool | Purpose in Repo Scan | Why This Tool |
|------|---------------------|---------------|
| `Glob` | Discover repo structure: find all `.py`, `.ipynb`, `.yaml/.yml`, `.json`, `.md`, `.csv`, `.tex` files | Fast pattern matching, returns sorted file paths, handles any codebase size. Prefer over `Bash(find)` per Claude Code conventions. |
| `Grep` | Extract key patterns: function signatures, class definitions, model names, metric values, configuration keys | Content search with regex support, file type filtering. Use `output_mode: content` with context lines for surrounding code. |
| `Read` | Deep-read critical files: README, main entry points, config files, result files, existing drafts | Full file content with line numbers. Use `limit` parameter for large files. |

**Scanning sequence (ordered by information density):**

```
Phase 1 -- Discover structure:
  Glob("**/README*")           -> repo purpose, high-level description
  Glob("**/*.{py,ipynb}")      -> code files
  Glob("**/*.{yaml,yml,json}") -> config files
  Glob("**/*.{csv,xlsx,txt}")  -> data/result files
  Glob("**/*.{tex,md}")        -> existing writing

Phase 2 -- Extract skeleton:
  Read(README)                          -> project overview
  Read(main entry point or __init__.py) -> code architecture
  Grep("class |def ", type: "py")       -> API surface
  Read(config files)                    -> hyperparameters, model configs

Phase 3 -- Extract results:
  Grep("accuracy|loss|f1|auc|rmse|mae|r2|precision|recall", *.py) -> metric names
  Read(result files: *.csv, logs)       -> actual values
  Grep("Table|Figure|\\\\begin\{table\}", *.tex) -> existing tables/figures
```

**Why NOT use Bash for scanning:** Claude Code conventions explicitly prefer Glob/Grep/Read over `find`/`grep`/`cat` Bash commands. The built-in tools provide better permission handling, output formatting, and user experience.

**Context budget concern:** A full repo scan could consume significant context. The Skill must implement progressive disclosure:
1. Scan structure first (Glob only, minimal context)
2. Present repo overview to user for confirmation
3. Deep-read only files the user confirms are relevant
4. Never read binary files, `node_modules`, `.git`, `__pycache__`, or virtual environments

**Confidence:** HIGH -- these are standard Claude Code tools, documented and validated in v1.0 Skills.

### 2. Repo Scan File Pattern Registry

**What:** A reference file listing common experiment repo file patterns by category, so the repo-to-paper Skill knows what to look for.

**Recommended location:** `references/repo-patterns.md`

**Purpose:** Maps file patterns to paper sections. This keeps domain knowledge out of SKILL.md (line budget) and makes patterns maintainable.

| File Pattern | Maps To Paper Section | Priority |
|-------------|----------------------|----------|
| `README.md`, `README.rst` | Introduction (project overview) | High |
| `*.py` (main/train/run entry) | Methods (pipeline architecture) | High |
| `config/*.yaml`, `*.yml`, `hparams.yaml` | Methods (hyperparameters) | High |
| `models/*.py`, `model.py` | Methods (model architecture) | High |
| `data/*.py`, `dataset.py` | Data Description | Medium |
| `results/*.csv`, `logs/`, `metrics/` | Results (quantitative) | High |
| `notebooks/*.ipynb` | Results + Discussion (analysis) | Medium |
| `figures/`, `plots/`, `*.png`, `*.pdf` | Figures section | Medium |
| `requirements.txt`, `environment.yml` | Methods (reproducibility) | Low |
| `tests/` | Skip (not paper content) | None |

**File structure:**
```markdown
# Repo File Patterns

## High Priority (always scan)
[patterns that almost always contain paper-relevant content]

## Medium Priority (scan if present)
[patterns for supplementary content]

## Skip Patterns (never scan)
[patterns to exclude: .git, __pycache__, node_modules, .venv, *.pyc, etc.]

## Section Mapping Rules
[which file categories map to which paper sections]
```

**Why a reference file instead of inline in SKILL.md:** The pattern registry will be 40-80 lines. Inlining it would consume 15-25% of the 300-line budget. Moving it to a reference file follows the v1.0 pattern of `references/expression-patterns.md` serving as a loadable knowledge base.

**Confidence:** HIGH -- follows established reference file pattern from v1.0.

### 3. Literature Reference File Format

**What:** When the repo-to-paper Skill collects Semantic Scholar references at H2 outline stage, it needs a structured file format to store them for later use during body generation.

**Recommended format:** Markdown file at `{output_dir}/references-collected.md`

**Specification:**

```markdown
# Collected References

## [H2 Section Title]

### ref-001: {citationKey}
- **Title:** {title}
- **Authors:** {authors}
- **Year:** {year}
- **Citations:** {citationCount}
- **Abstract:** {abstract excerpt, first 2-3 sentences}
- **Relevance:** {one-sentence note on why this ref is relevant to this H2}
- **BibTeX:**
  ```bibtex
  @article{citationKey, ...}
  ```

### ref-002: {citationKey}
[same structure]

## [Next H2 Section Title]
[same structure]
```

**Why Markdown and not JSON/YAML:**
- Claude parses markdown tables and structured markdown natively and efficiently (established in v1.0 STACK.md)
- The file serves dual purpose: machine-readable for body generation AND human-readable for user review
- Follows the project convention that all reference files are `.md`
- JSON would add parsing complexity without benefit for this content type

**Why abstracts are included:**
- The body generation phase needs abstract content to write grounded connection sentences
- Without abstracts, the Skill would need to re-query Semantic Scholar during body generation (wasteful, may hit rate limits)
- Abstracts enable the Skill to write `[CITE: citationKey]` with contextual accuracy

**Why per-H2 grouping:**
- Literature relevance is section-specific
- During body generation, the Skill loads only the refs for the current H2, keeping context narrow
- Users can review and prune refs per section before generation starts

**Anti-hallucination rule:** Same as v1.0 literature-skill -- all fields must come from MCP-returned data. Abstract field uses `mcp__semantic-scholar__get-paper-abstract` if not returned in search. Never fill fields from prior knowledge.

**Confidence:** HIGH -- extends validated patterns from literature-skill.

### 4. Top-Down Generation Checkpoint Format

**What:** The repo-to-paper Skill generates in top-down order (H1 outline -> H2 outline -> H3 outline -> body). User approval is required at each level before proceeding deeper. A standardized checkpoint format enables clean user interaction.

**Checkpoint presentation format:**

```markdown
## H1 Outline (Level 1 Checkpoint)

1. Introduction
2. Related Work
3. Methods
4. Experiments
5. Results and Discussion
6. Conclusion

**Approve this structure?** [Modify / Approve / Restart]
```

```markdown
## H2 Outline (Level 2 Checkpoint)

### 1. Introduction
  1.1 Background and Motivation
  1.2 Research Gap
  1.3 Contributions

### 2. Related Work
  2.1 [Topic area 1]
  2.2 [Topic area 2]

[...continued...]

**Approve this structure?** [Modify / Approve]
```

**Why use AskUserQuestion for checkpoints:** Each checkpoint is a multi-option decision point (Approve / Modify / Restart). This is exactly the use case AskUserQuestion was designed for. The Skill should present the outline in conversation, then use AskUserQuestion with options:
- "Approve and continue to next level" (Recommended)
- "Modify -- I'll provide corrections"
- "Restart this level"

**Why NOT skip checkpoints even in direct mode:** Paper structure decisions are high-impact and irreversible within a generation session. An incorrect H1 outline cascades into wrong H2, H3, and body. The two-phase confirmation pattern from experiment-skill (Phase 1 findings -> user confirm -> Phase 2 discussion) is the established project precedent.

**Confidence:** HIGH -- extends the two-phase confirmation pattern already validated in experiment-skill.

### 5. Bilingual Paragraph-by-Paragraph Comparison Format

**What:** v2.0 adds bilingual paragraph-by-paragraph comparison output across Skills. This requires a standardized format specification.

**Recommended format:** Two variants depending on output context.

#### Variant A: LaTeX Output (for translation-skill, polish-skill, de-ai-skill)

Already partially implemented in translation-skill (`_bilingual.tex`). Standardize across all text-producing Skills:

```latex
% --- Paragraph 1 ---
% [ZH] 城市热岛效应是城市化进程中最显著的环境问题之一。
% [ZH] 随着全球城市人口的增长，理解和缓解这一现象变得越来越重要。
Urban heat island effect is one of the most prominent environmental issues
associated with urbanization. As global urban populations grow,
understanding and mitigating this phenomenon becomes increasingly important.

% --- Paragraph 2 ---
% [ZH] 本研究提出了一种基于遥感数据的城市热岛强度评估框架。
This study proposes a framework for assessing urban heat island intensity
based on remote sensing data.
```

**Format rules:**
- `% --- Paragraph N ---` markers for paragraph boundaries
- `% [ZH]` prefix for Chinese source lines (LaTeX comments, file still compiles)
- English body text follows immediately after Chinese comment block
- One blank line between paragraph blocks for readability
- Paragraph numbering is sequential and continuous

#### Variant B: Markdown Output (for reviewer-skill, logic-skill, experiment-skill, abstract-skill)

For Skills that output Markdown (not LaTeX), use blockquote format:

```markdown
### Paragraph 1

Urban heat island effect is one of the most prominent environmental issues
associated with urbanization. As global urban populations grow,
understanding and mitigating this phenomenon becomes increasingly important.

> **[ZH]** 城市热岛效应是城市化进程中最显著的环境问题之一。随着全球城市人口的增长，理解和缓解这一现象变得越来越重要。

### Paragraph 2

This study proposes a framework for assessing urban heat island intensity
based on remote sensing data.

> **[ZH]** 本研究提出了一种基于遥感数据的城市热岛强度评估框架。
```

**Format rules:**
- English paragraph first (primary language, matches existing output contracts)
- Chinese blockquote immediately after each English paragraph
- `> **[ZH]**` prefix consistent with reviewer-skill and logic-skill bilingual patterns
- `### Paragraph N` headers for clear section boundaries

#### When to Use Which Variant

| Skill | Output Format | Bilingual Variant | Rationale |
|-------|--------------|-------------------|-----------|
| translation-skill | LaTeX `.tex` | Variant A | Already uses this; standardize `% [ZH]` prefix |
| polish-skill | LaTeX in-place edit | Variant A | Edits `.tex` files; Chinese in comments |
| de-ai-skill | LaTeX in-place edit | Variant A | Same as polish-skill |
| reviewer-skill | Markdown report | Variant B | Already uses `> **[Chinese]**` blockquote |
| logic-skill | Markdown report | Variant B | Already uses `> **[中文]**` blockquote |
| experiment-skill | Markdown paragraphs | Variant B | Discussion output is Markdown |
| abstract-skill | Plain text block | Variant B | Abstract output is not LaTeX |
| repo-to-paper-skill | LaTeX `.tex` | Variant A | Generates paper body in LaTeX |

**Why two variants (not one unified format):**
- LaTeX Skills already output `.tex` files where Chinese must be in comments to preserve compilation
- Markdown Skills already use blockquote Chinese per reviewer-skill and logic-skill conventions
- Forcing LaTeX-style comments into Markdown output would be unnatural and less readable
- Forcing blockquotes into LaTeX output would break compilation

**Change from v1.0:** The reviewer-skill and logic-skill already implement Variant B partially. The v2.0 change is:
1. Standardize the `> **[ZH]**` prefix (currently varies: `> **[Chinese]**`, `> **[中文]**`)
2. Add paragraph-level markers (`### Paragraph N` or `% --- Paragraph N ---`)
3. Make bilingual output an explicit mode flag (not always-on)

**Confidence:** HIGH -- extends existing patterns from translation-skill and reviewer-skill.

### 6. Bilingual Output Mode Flag

**What:** A convention for Skills to support an optional bilingual comparison mode, triggered by user request.

**Recommended approach:** Add `bilingual` as a mode modifier, not a standalone mode.

```yaml
# In SKILL.md frontmatter
output_contract:
  - polished_text
  - bilingual_comparison  # NEW: added for v2.0
```

**Trigger phrases:**
- "Polish with bilingual comparison" / "润色并输出双语对照"
- "Translate with paragraph-by-paragraph comparison" / "逐段对照翻译"
- "Show me the Chinese alongside" / "中英对照输出"

**Activation logic in Skill workflow:**
```
IF user requests bilingual comparison OR trigger contains bilingual keywords:
  SET bilingual_mode = true
  After generating primary output, also generate comparison version
ELSE:
  Standard output only (no comparison overhead)
```

**Why a mode modifier, not a standalone mode:** Bilingual comparison is an output format enhancement, not a different workflow. A polish in bilingual mode still follows the same structure -> logic -> expression pipeline; it just adds a comparison layer to the output. Making it a modifier keeps the 4-mode taxonomy (interactive, guided, direct, batch) clean.

**Confidence:** MEDIUM -- this is a design decision, not an ecosystem standard. The modifier approach is simpler but the skill-conventions.md may need updating to document this pattern.

### 7. AskUserQuestion Enforcement Pattern

**What:** v1.0 Skills list `Structured Interaction` in their `tools` field and document fallback to plain-text questions. However, the observed behavior is that Claude sometimes uses plain conversational dialogue instead of calling the `AskUserQuestion` tool, even when the tool is available. This produces unstructured free-text questions rather than multiple-choice selection interfaces.

**Root cause analysis:**

The issue is NOT that AskUserQuestion is unavailable. It is confirmed working (STATE.md). The issue is that Skills say "Use Structured Interaction when available" without providing explicit instructions on HOW to use AskUserQuestion. Claude defaults to conversational questions because:

1. SKILL.md files describe WHAT to ask but not the tool call format
2. The convention says "Structured Interaction" (an abstraction) but never maps it to the actual tool name `AskUserQuestion`
3. No Skill provides an explicit AskUserQuestion invocation example
4. Claude optimizes for natural conversation flow, which means plain-text questions feel more natural than tool calls

**Recommended fix -- two-pronged approach:**

#### Fix A: Skill-level enforcement (per SKILL.md)

Add an explicit AskUserQuestion invocation pattern to the Ask Strategy section of each Skill that uses structured questions:

```markdown
## Ask Strategy

**Structured questions (use AskUserQuestion tool):**

When asking the user to choose between options, ALWAYS use the AskUserQuestion tool
with labeled options. Do NOT ask options as plain conversational text.

Example -- target journal selection:
  AskUserQuestion: "Which journal is this paper targeting?"
  Options:
    - "CEUS (Computers, Environment and Urban Systems)" (Recommended)
    - "IJGIS (International Journal of Geographical Information Science)"
    - "General academic style (no specific journal)"

Example -- mode selection:
  AskUserQuestion: "How would you like to proceed?"
  Options:
    - "Quick polish (single pass)" (Recommended)
    - "Guided polish (step by step: structure, logic, expression)"
    - "Batch polish (all sections with same settings)"
```

**Why explicit examples work:** Claude follows demonstrated patterns more reliably than abstract instructions. Showing the exact tool usage in the Skill body makes it the path of least resistance.

#### Fix B: Convention-level enforcement (in skill-conventions.md)

Add a new section to `references/skill-conventions.md`:

```markdown
### AskUserQuestion Usage

When a Skill's Ask Strategy includes multi-option questions:

1. Use the `AskUserQuestion` tool for every question that offers 2+ discrete options.
2. Do NOT present options as numbered lists in plain conversation text.
3. Place the recommended option first in the options list and append "(Recommended)".
4. Reserve plain-text questions for open-ended inputs only (e.g., "What is the file path?").
5. If AskUserQuestion is unavailable, fall back to 1-3 plain-text questions as documented in Fallbacks.
```

**Why both levels (A + B):**
- Convention-level (B) establishes the rule for all future Skills
- Skill-level (A) is needed because Claude reads SKILL.md during execution and may not load skill-conventions.md unless explicitly told to
- The explicit examples in individual Skills (A) are more effective than the abstract rule in conventions (B) because they appear in the active execution context

**What NOT to do:**
- Do NOT add CLAUDE.md-level instructions saying "always use AskUserQuestion" -- this would affect all Claude Code behavior, not just paper-polish Skills
- Do NOT make AskUserQuestion a hard requirement with no fallback -- environments without the tool should still work
- Do NOT use `multiSelect: true` for Ask Strategy questions -- these are typically single-choice (journal, mode, scope)

**Confidence:** HIGH for the root cause analysis (confirmed by AskUserQuestion system prompt documentation and v1.0 Skill patterns). MEDIUM for the fix effectiveness (untested; may need iteration).

---

## Reference File Additions Summary

| New File | Purpose | Size Estimate | Loaded By |
|----------|---------|---------------|-----------|
| `references/repo-patterns.md` | File pattern registry for repo scanning | 40-80 lines | repo-to-paper-skill |
| `references/bilingual-format.md` | Bilingual output format specification (Variant A + B) | 30-50 lines | Any Skill producing bilingual output |

**Why only 2 new reference files:**
- The literature ref file format (`references-collected.md`) is output, not a reference -- it lives in the user's working directory
- The checkpoint format is simple enough to inline in SKILL.md
- The AskUserQuestion enforcement goes into existing files (skill-conventions.md + individual SKILL.md files)

---

## Skill Additions Summary

| New Skill | Directory | Tools | References | Estimated Lines |
|-----------|-----------|-------|------------|-----------------|
| `repo-to-paper-skill` | `.claude/skills/repo-to-paper-skill/` | Read, Write, Glob, Grep, Structured Interaction, External MCP | `references/expression-patterns.md`, `references/repo-patterns.md`, `references/anti-ai-patterns.md` | ~280-300 (at budget limit due to multi-phase workflow) |

**Why only 1 new Skill:**
- Bilingual comparison is a mode addition to existing Skills, not a new Skill
- AskUserQuestion fix is a convention + Skill update, not a new Skill

---

## Semantic Scholar Integration for Repo-to-Paper

**What changes from v1.0 literature-skill:** The literature-skill is a standalone search-and-cite workflow. The repo-to-paper-skill needs integrated literature search at H2 outline stage.

**Integration approach:**

```
After H2 outline is approved:
  FOR each H2 section:
    1. Extract 2-3 search keywords from the H2 title and description
    2. Call mcp__semantic-scholar__papers-search-basic with keywords
    3. For each result: call mcp__semantic-scholar__get-paper-abstract
    4. Present results to user (numbered cards, same format as literature-skill)
    5. User selects relevant papers for this section
    6. Write selected papers to references-collected.md under this H2 heading
  END FOR

  User reviews complete references-collected.md
  Proceed to H3 outline generation
```

**Why search at H2 level (not H1 or H3):**
- H1 is too coarse (searching "Introduction" returns nothing useful)
- H3 is too fine-grained (too many searches, rate limit risk)
- H2 maps to meaningful research topics ("Urban Heat Island Measurement Methods", "Remote Sensing Approaches")

**Rate limit awareness:** Semantic Scholar API allows 1 request/second with API key, 100 requests per 5 minutes without. A paper with 6 H2 sections, 5 search results each, with abstract fetches = ~36 API calls. Well within limits with an API key. Without a key, the Skill should batch requests with brief pauses.

**Reuse from literature-skill:** The result card format, BibTeX generation logic, and anti-hallucination verification prompt should be consistent with literature-skill. Do NOT duplicate these patterns -- reference the same conventions.

**Confidence:** HIGH -- extends validated Semantic Scholar MCP integration from v1.0.

---

## What NOT to Add

| Temptation | Why Resist |
|-----------|------------|
| npm packages for repo analysis (tree-sitter, etc.) | Pure markdown Skill constraint. Claude's built-in Glob/Grep/Read handles repo scanning adequately for the paper-generation use case. |
| External APIs beyond Semantic Scholar | Adds installation complexity, API key management, and failure modes. Semantic Scholar covers the citation need. |
| Custom MCP server for repo scanning | Overkill. Built-in tools suffice. An MCP server would add a Python dependency and installation step. |
| AST parsing tools | Claude reads code well enough for paper-level understanding. AST-level precision is unnecessary for generating "we implemented X using Y" prose. |
| PDF generation from LaTeX | Out of scope. The Skill produces `.tex` files. Users compile with their own LaTeX toolchain. |
| Separate bilingual Skill | Bilingual output is a mode, not a standalone capability. Adding a 12th Skill for "make this bilingual" adds unnecessary indirection. |
| Database for storing references | Markdown files are sufficient for the per-session, per-paper reference collection use case. No persistent state needed between sessions. |
| Template engine (Jinja, Handlebars) | SKILL.md is the template. Claude fills it in. Adding a template engine adds a build dependency for no gain. |

---

## Alternatives Considered

| Decision | Recommended | Alternative | Why Not Alternative |
|----------|-------------|-------------|---------------------|
| Repo scanning tools | Glob + Grep + Read (built-in) | Custom MCP server with tree-sitter | Adds Python dependency; Claude's code reading is sufficient for paper-level prose |
| Literature ref storage | Per-session Markdown file | SQLite database | No persistence needed; Markdown is human-readable and consistent with project conventions |
| Bilingual format | Two variants (LaTeX + Markdown) | Single unified format | LaTeX files must compile; Markdown blockquotes are more natural in Markdown output |
| AskUserQuestion fix | Convention + Skill-level examples | CLAUDE.md global instruction | CLAUDE.md affects all behavior; fix should be scoped to paper-polish Skills only |
| Checkpoint interaction | AskUserQuestion with options | Plain text "type approve or modify" | AskUserQuestion provides clickable UI; plain text is the fallback, not the primary path |
| Top-down generation | H1 -> H2 -> H3 -> body with checkpoints | Single-pass full draft | Single pass produces unrecoverable structure errors; checkpoints are cheaper than regeneration |

---

## Compatibility with v1.0 Stack

| v1.0 Component | v2.0 Impact | Action |
|----------------|-------------|--------|
| skill-conventions.md | Needs AskUserQuestion section added | Update existing file |
| skill-skeleton.md | Needs bilingual output_contract example | Update existing file |
| 11 existing SKILL.md files | Ask Strategy sections need AskUserQuestion examples | Update incrementally |
| expression-patterns library | No changes needed | None |
| anti-ai-patterns library | No changes needed | None |
| CEUS journal template | No changes needed | None |
| Semantic Scholar MCP config | No changes needed | None |

**Breaking changes:** None. All v2.0 additions are backward-compatible. Existing Skills continue to work without modification; AskUserQuestion enforcement and bilingual mode are additive.

---

## Sources

### Official Documentation (HIGH confidence)
- [AskUserQuestion tool description](https://github.com/Piebald-AI/claude-code-system-prompts/blob/main/system-prompts/tool-description-askuserquestion.md) -- Exact tool interface, parameters, plan mode notes
- [Claude Code Skills Documentation](https://code.claude.com/docs/en/skills) -- Skill format, progressive disclosure

### Community Sources (MEDIUM confidence)
- [AskUserQuestion usage guide](https://www.atcyrus.com/stories/claude-code-ask-user-question-tool-guide) -- Usage patterns, timeout behavior, best practices
- [AskUserQuestion first impressions](https://torqsoftware.com/blog/2026/2026-01-14-claude-ask-user-question/) -- Real-world usage observations
- [Interactive tools with AskUserQuestion](https://egghead.io/create-interactive-ai-tools-with-claude-codes-ask-user-question~b47wn) -- Skill-based interview patterns
- [SmartScope AskUserQuestion guide](https://smartscope.blog/en/generative-ai/claude/claude-code-askuserquestion-tool-guide/) -- Structured question patterns

### Project Sources (HIGH confidence)
- v1.0 STACK.md -- Validated stack decisions
- v1.0 MILESTONE-AUDIT.md -- Tech debt and known issues
- v1.0 literature-skill SKILL.md -- Semantic Scholar MCP integration pattern
- v1.0 experiment-skill SKILL.md -- Two-phase confirmation pattern
- v1.0 reviewer-simulation-skill SKILL.md -- Bilingual blockquote pattern
- v1.0 translation-skill SKILL.md -- LaTeX bilingual comparison pattern
- v1.0 skill-conventions.md -- Authoring rules and tool conventions

---
*Stack research for: v2.0 Repo-to-Paper, Bilingual Enhancement, AskUserQuestion Fix*
*Researched: 2026-03-17*
