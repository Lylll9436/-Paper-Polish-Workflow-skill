# Feature Research: v2.0 New Capabilities

**Domain:** AI-Assisted Academic Paper Writing Tool Suite (Claude Code Skills) -- v2.0 Extension
**Researched:** 2026-03-17
**Confidence:** MEDIUM-HIGH (repo-to-paper is novel; bilingual and AskUserQuestion patterns are well-grounded in existing codebase)

> **Scope note:** This document covers ONLY the three new v2.0 features. For v1.0 feature landscape (11 existing Skills), see the v1.0 FEATURES.md in git history. Existing Skills are referenced as dependencies, not re-researched.

---

## Feature Landscape

### Table Stakes (Users Expect These)

These are the minimum capabilities each v2.0 feature must deliver. Without them, the feature would feel half-baked relative to what the existing v1.0 Skills already provide.

#### TS-1: Repo Scan -- Full Codebase Comprehension

| Aspect | Detail |
|--------|--------|
| **Why Expected** | If the Skill claims "point to a repo, get a paper draft," it must actually read and understand the whole repo. A shallow scan that misses key files destroys trust immediately. |
| **Complexity** | MEDIUM |
| **Existing Dependency** | Glob, Grep, Read tools (Claude Code built-in) |
| **What "Good" Looks Like** | Recursively scan repo structure. Identify: (1) experiment scripts (training, evaluation, inference), (2) configuration files (hyperparameters, model architecture), (3) result files (CSVs, JSON logs, saved metrics), (4) README/documentation, (5) data loading/preprocessing code. Produce a structured inventory before generating anything. |
| **Notes** | Must handle large repos without context overflow. Strategy: scan directory tree first (Glob), then selectively Read key files. Never attempt to read every file -- prioritize by filename conventions (train.py, config.yaml, results/, etc.). |

#### TS-2: Top-Down Generation with User Checkpoints

| Aspect | Detail |
|--------|--------|
| **Why Expected** | This is the core UX promise of repo-to-paper. Generating a full paper in one shot without user input would produce garbage -- the user must shape the narrative at each structural level. Top-down (outline first, then body) is the established pattern for LLM-assisted long-form generation (Anthropic prompt chaining pattern). |
| **Complexity** | HIGH |
| **Existing Dependency** | AskUserQuestion tool for checkpoints; Write tool for output |
| **What "Good" Looks Like** | Three-level generation: (1) **H1 level** -- propose paper title + major section headings, user confirms/edits. (2) **H2 level** -- for each H1 section, propose subsection headings + 1-sentence summary per subsection, user confirms/edits. (3) **H3/body level** -- generate paragraph-level content per confirmed subsection. User confirms at each level before going deeper. |
| **Notes** | This mirrors the PaperCoder architecture (planning -> analysis -> generation) but in reverse direction (code->paper instead of paper->code). The three-stage pattern is well-validated in multi-agent writing frameworks. Must NOT skip H1 confirmation -- generating H2 for a rejected structure wastes everything. |

#### TS-3: Literature Integration at H2 Stage

| Aspect | Detail |
|--------|--------|
| **Why Expected** | A paper draft without related work references is unusable. By the H2 stage, the user has confirmed what subsections exist, so the Skill knows exactly what topics need literature support. This is the natural insertion point. |
| **Complexity** | MEDIUM |
| **Existing Dependency** | literature-skill (v1.0) for Semantic Scholar MCP calls; existing BibTeX generation logic |
| **What "Good" Looks Like** | For each H2 subsection in Introduction/Related Work, automatically search Semantic Scholar for relevant papers. Present top 3-5 results per subsection. Save selected references as structured ref files (metadata + abstract) alongside the draft. Generate `\cite{}` placeholders in the body text referencing these entries. |
| **Notes** | Reuse the literature-skill's MCP pre-flight check, search, and BibTeX generation logic. The new addition is: (a) automatic query derivation from subsection titles, (b) batch search across multiple subsections, (c) ref file persistence with metadata+abstract for later use. |

#### TS-4: Bilingual Paragraph-by-Paragraph Output

| Aspect | Detail |
|--------|--------|
| **Why Expected** | The user explicitly requested this. Chinese researchers need to read what each English paragraph says in Chinese to verify accuracy and intent. The v1.0 translation-skill already produces `_bilingual.tex` with Chinese in LaTeX comments. The v1.0 reviewer-simulation-skill and logic-skill produce inline `> **[Chinese]**` blockquotes. Users expect this pattern to extend across more Skills. |
| **Complexity** | LOW-MEDIUM |
| **Existing Dependency** | translation-skill bilingual pattern (LaTeX `%` comments); reviewer/logic-skill bilingual pattern (markdown blockquotes) |
| **What "Good" Looks Like** | For markdown output: each English paragraph followed by a Chinese translation in blockquote format (`> [Chinese translation]`). For LaTeX output: each English paragraph followed by Chinese translation as `%` comment lines. Paragraph pairing must be 1:1 -- no merging or splitting between languages. |
| **Notes** | Two distinct rendering formats needed: markdown (blockquote) and LaTeX (comment). The format choice depends on the output file type. For repo-to-paper (which outputs markdown or .tex), the bilingual mode should be opt-in, not default, since it doubles output length. |

#### TS-5: AskUserQuestion Correct Usage

| Aspect | Detail |
|--------|--------|
| **Why Expected** | The paper-polish-workflow currently references `mcp_question` -- a tool name that does not exist in Claude Code. The actual built-in tool is `AskUserQuestion`. When Claude cannot find `mcp_question`, it falls back to plain dialogue, which breaks the structured interaction UX. This is a bug, not a feature. |
| **Complexity** | LOW |
| **Existing Dependency** | AskUserQuestion tool (built-in to Claude Code) |
| **What "Good" Looks Like** | paper-polish-workflow SKILL.md updated to reference `AskUserQuestion` instead of `mcp_question`. Correct schema: `questions` array with `question`, `header` (max 12 chars), `multiSelect` boolean, `options` array (2-4 items, each with `label` and `description`). Users always get "Other" option automatically. |
| **Notes** | The fix is small (rename tool references, update schema examples) but the impact is large -- the entire paper-polish-workflow's interactive UX is broken without it. Other v1.0 Skills reference "Structured Interaction" in frontmatter as a capability category, which is correct per skill-conventions.md. The paper-polish-workflow is the outlier that hardcodes the wrong tool name. |

---

### Differentiators (Competitive Advantage)

These features go beyond table stakes and distinguish repo-to-paper from the landscape of AI paper writing tools (Gatsbi, Paperguide, SciSpace, etc.).

#### D-1: Code-Aware Paper Structure (Not Topic-Based)

| Aspect | Detail |
|--------|--------|
| **Value Proposition** | Existing AI paper drafters (Gatsbi, NoteGPT, Smodin) generate from a topic or title. They have zero knowledge of the actual code, experiments, or results. Repo-to-paper generates from the actual codebase -- the paper structure reflects what was actually built and measured, not what someone typed as a topic. |
| **Complexity** | HIGH |
| **What "Good" Looks Like** | The H1 structure is derived from what the code actually does: if the repo has `train.py` + `evaluate.py` + `data_loader.py`, the Methods section naturally has "Model Architecture," "Training Procedure," and "Data" subsections. If the repo has `ablation.py`, an Ablation Study subsection is proposed. Structure follows implementation, not template. |
| **Notes** | This is the key differentiator that no existing tool offers. The closest analog is PaperCoder's reverse direction (paper->code), which builds code structure from paper structure. We do the inverse: infer paper structure from code structure. |

#### D-2: Ref Files with Metadata + Abstracts

| Aspect | Detail |
|--------|--------|
| **Value Proposition** | v1.0 literature-skill generates BibTeX entries one at a time. v2.0 repo-to-paper collects references in bulk at H2 stage and saves them as persistent ref files that include not just BibTeX but also abstract text and relevance notes. This creates a reusable literature database for the paper, not just citation entries. |
| **Complexity** | MEDIUM |
| **What "Good" Looks Like** | Each ref file contains: BibTeX entry, full abstract (from Semantic Scholar), citation count, relevance annotation (which subsection it supports), and the Semantic Scholar paper ID for later retrieval. Files saved to a `refs/` directory alongside the draft. |
| **Notes** | Abstracts are critical for the body generation stage -- when writing Related Work paragraphs, the Skill can read ref file abstracts to accurately describe what prior work found, rather than hallucinating. This is the anti-hallucination strategy for literature discussion. |

#### D-3: Evidence-Before-Interpretation in Generated Body

| Aspect | Detail |
|--------|--------|
| **Value Proposition** | The v1.0 experiment-skill established a strict "evidence sentence first, interpretation sentence second" rule. Repo-to-paper must extend this to all generated Results/Discussion content. Most AI paper drafters produce interpretation-first text ("Our model significantly outperforms...") which reads as AI-generated and lacks academic rigor. |
| **Complexity** | LOW (pattern already exists in experiment-skill) |
| **What "Good" Looks Like** | Every results paragraph opens with a quantified evidence statement from actual result files in the repo, followed by calibrated interpretation using expression patterns from `references/expression-patterns/results-and-discussion.md` and `conclusions-and-claims.md`. |
| **Notes** | This is a writing quality differentiator, not a feature. But it must be explicitly enforced in the Skill design because LLMs default to interpretation-first generation. |

#### D-4: Bilingual Mode as Cross-Skill Capability

| Aspect | Detail |
|--------|--------|
| **Value Proposition** | Rather than implementing bilingual output per-Skill (which v1.0 did: translation-skill has its own bilingual format, reviewer-skill has its own, logic-skill has its own), v2.0 should define a shared bilingual output module that any Skill can invoke. This reduces inconsistency and makes adding bilingual support to future Skills trivial. |
| **Complexity** | MEDIUM |
| **What "Good" Looks Like** | A reference file (e.g., `references/bilingual-output.md`) that defines the two bilingual formats (markdown blockquote, LaTeX comment) with exact templates. Skills that support bilingual output load this reference and apply the appropriate format based on output file type. The user activates bilingual mode via a trigger keyword ("bilingual" / "双语对照"). |
| **Notes** | The v1.0 patterns are already consistent (blockquote for markdown, `%` comment for LaTeX) but they are independently implemented in each Skill's Output Contract section. Extracting this to a shared reference enables the repo-to-paper Skill to use the same format without re-inventing it. |

#### D-5: Anti-Hallucination Placeholders Throughout

| Aspect | Detail |
|--------|--------|
| **Value Proposition** | Following the experiment-skill's `[CONNECT TO: ...]` and `[CITATION NEEDED]` pattern, the repo-to-paper Skill should never fabricate specific claims, numbers, or citations. Any claim that cannot be grounded in actual repo contents gets a placeholder. This is the ethical guardrail that distinguishes assisted writing from generated writing. |
| **Complexity** | LOW (pattern already established in v1.0) |
| **What "Good" Looks Like** | Placeholder types: `[CITATION NEEDED: topic]` for missing references, `[EXACT VALUE: metric name]` for numbers not found in result files, `[CONNECT TO: prior finding]` for literature connections, `[VERIFY: claim]` for interpretive claims needing human validation. |
| **Notes** | Must be enforced in the Skill prompt, not left to the LLM's judgment. The prompt must explicitly say "if you cannot find this value in the repo files, write a placeholder." |

---

### Anti-Features (Commonly Requested, Often Problematic)

#### AF-1: One-Click Full Paper Generation

| Feature | Why Requested | Why Problematic | Alternative |
|---------|---------------|-----------------|-------------|
| "Just point to repo, get complete paper" | Seems maximally efficient | Produces hallucinated content that the user must completely rewrite. No user checkpoints means wrong structure propagates everywhere. Papers require human judgment about what to emphasize, what to omit, and how to frame contributions. | Three-level top-down generation with mandatory user confirmation at H1, H2, and body stages. |

#### AF-2: Automatic Figure/Table Extraction from Repo

| Feature | Why Requested | Why Problematic | Alternative |
|---------|---------------|-----------------|-------------|
| Scan repo for .png/.pdf figures and auto-include them | Saves time finding figures | Repos contain many intermediate/debug visualizations that should not go in the paper. Without human curation, the paper becomes a dump of every plot generated. | List figures found in repo as a sidebar inventory. User selects which to include at H2 stage. Caption-skill generates captions for selected figures. |

#### AF-3: Auto-Formatting for Specific Conference Templates

| Feature | Why Requested | Why Problematic | Alternative |
|---------|---------------|-----------------|-------------|
| Output directly in NeurIPS/ICML LaTeX template format | Saves manual formatting | Conference templates change yearly. Maintaining template compatibility is a moving target. The Skill's value is content, not formatting. | Output clean LaTeX or markdown. User applies their own conference template. The journal template system (`references/journals/`) handles style guidance, not layout. |

#### AF-4: Bidirectional Sync (Paper Edits Back to Code)

| Feature | Why Requested | Why Problematic | Alternative |
|---------|---------------|-----------------|-------------|
| When user edits paper, update code comments/docs to match | Keeps code and paper in sync | Enormously complex. Code changes for documentation updates risk breaking functionality. The paper and code have different lifecycles. | One-directional: code -> paper. User manually updates code documentation if needed. |

#### AF-5: Real-Time Bilingual Co-Editing

| Feature | Why Requested | Why Problematic | Alternative |
|---------|---------------|-----------------|-------------|
| Edit English paragraph, Chinese auto-updates; edit Chinese, English auto-updates | Seems useful for bilingual review | Creates infinite loops and version conflicts. The Chinese translation is a review aid, not a co-authored document. Editing the translation should not modify the English source. | Bilingual output is read-only for review. To change the English, user edits the English-only version and re-generates bilingual. |

#### AF-6: Expanding Bilingual to Non-Chinese Languages

| Feature | Why Requested | Why Problematic | Alternative |
|---------|---------------|-----------------|-------------|
| Support Japanese, Korean, or other L1 translations alongside English | Broader user base | Adds complexity for a niche use case. The entire reference library (expression patterns, anti-AI patterns) is designed for the Chinese-English bilingual researcher. Adding other languages requires new reference files, testing, and cultural calibration. | Scope to Chinese-English. If demand arises, the bilingual output module can be extended later with new reference files per language. |

---

## Feature Dependencies

```
[AskUserQuestion Fix (TS-5)]
    (independent, no dependencies -- pure SKILL.md text fix)

[Bilingual Output Module (D-4)]
    (new reference file, independent of other v2.0 features)
        |
        v
[Bilingual Paragraph-by-Paragraph (TS-4)]
    requires: Bilingual Output Module reference file
    enhances: Repo-to-Paper (bilingual mode for generated draft)
    enhances: Any existing v1.0 Skill that wants to add bilingual output

[Repo Scan (TS-1)]
    requires: Glob, Grep, Read tools (built-in)
        |
        v
[Top-Down Generation (TS-2)]
    requires: Repo Scan inventory
    requires: AskUserQuestion for checkpoints
        |
        +---> [Literature Integration at H2 (TS-3)]
        |         requires: Top-Down at H2 stage
        |         requires: literature-skill MCP logic (reuse)
        |         produces: Ref files (D-2)
        |
        +---> [Body Generation at H3]
                  requires: Top-Down at H3 stage
                  uses: Ref files for Related Work paragraphs
                  uses: Expression patterns (v1.0 references)
                  uses: Anti-AI patterns (v1.0 references)
                  uses: Bilingual Output Module (optional)

[Evidence-Before-Interpretation (D-3)]
    enhances: Body Generation
    requires: experiment-skill pattern (already established in v1.0)

[Anti-Hallucination Placeholders (D-5)]
    enhances: All generated body content
    requires: nothing new (pattern from v1.0)
```

### Dependency Notes

- **AskUserQuestion Fix (TS-5) is fully independent.** It can be done first with zero risk. It unblocks correct interactive UX for paper-polish-workflow and should be done before the repo-to-paper Skill (which also relies on AskUserQuestion checkpoints).
- **Bilingual Output Module (D-4) is independent** of repo-to-paper. It should be built as a reference file first, then adopted by repo-to-paper and offered as an upgrade path for v1.0 Skills.
- **Repo Scan (TS-1) must precede Top-Down Generation (TS-2)** -- you cannot propose paper structure without understanding what the repo contains.
- **Literature Integration (TS-3) depends on H2 completion** -- you need confirmed subsection titles to know what to search for.
- **Ref Files (D-2) feed into Body Generation** -- Related Work paragraphs use ref file abstracts to avoid hallucination.

---

## MVP Definition

### Launch With (v2.0 core)

Minimum viable v2.0 -- what's needed to validate the three new features.

- [ ] **AskUserQuestion fix in paper-polish-workflow** -- Rename `mcp_question` to `AskUserQuestion`, update schema to match actual tool interface. P0 bug fix. (TS-5)
- [ ] **Bilingual output reference file** -- Create `references/bilingual-output.md` defining the two bilingual formats (markdown blockquote, LaTeX comment) with templates and usage rules. (D-4)
- [ ] **Bilingual paragraph-by-paragraph mode** -- Implement in repo-to-paper output and offer as an option for translation-skill (which already has its own format). (TS-4)
- [ ] **Repo-to-paper Skill: repo scan phase** -- Recursive repo inventory using Glob/Read, structured output of discovered files by category. (TS-1)
- [ ] **Repo-to-paper Skill: H1 generation + checkpoint** -- Propose title + section headings from scan, present via AskUserQuestion, wait for confirmation. (TS-2 partial)
- [ ] **Repo-to-paper Skill: H2 generation + checkpoint** -- For each confirmed H1, propose subsections with 1-sentence summaries. (TS-2 partial)
- [ ] **Repo-to-paper Skill: Literature search at H2** -- Automatic Semantic Scholar search per H2 subsection, ref file generation. (TS-3)
- [ ] **Repo-to-paper Skill: Body generation** -- Generate paragraphs for confirmed H2 subsections using repo data, ref files, and expression patterns. (TS-2 complete)

### Add After Validation (v2.x)

Features to add once core repo-to-paper is working.

- [ ] **Retrofit bilingual mode to existing v1.0 Skills** -- Apply bilingual output reference to polish-skill, abstract-skill, experiment-skill via `bilingual` trigger keyword.
- [ ] **Repo-to-paper: figure inventory sidebar** -- During scan, list image files found in repo. User selects which to include at H2 stage. Caption-skill generates captions.
- [ ] **Repo-to-paper: iterative revision mode** -- After first complete draft, re-enter at any structural level (H1/H2/body) to revise specific sections without regenerating everything.

### Future Consideration (v3+)

Features to defer until v2.0 is validated.

- [ ] **Multi-repo paper** -- Support papers that draw on multiple experiment repos (e.g., ablation in one repo, main experiment in another). Requires cross-repo scan coordination.
- [ ] **Non-English language bilingual pairs** -- Japanese-English, Korean-English, etc. Requires new reference files and testing.
- [ ] **Paper-to-repo reverse direction** -- Given a paper draft, generate a skeleton code repo. PaperCoder does this; we could integrate but it is a fundamentally different workflow.

---

## Feature Prioritization Matrix

| Feature | User Value | Implementation Cost | Priority | Rationale |
|---------|------------|---------------------|----------|-----------|
| AskUserQuestion fix (TS-5) | HIGH | LOW | **P0** | Bug fix. Unblocks interactive UX for paper-polish-workflow. 30 min of work. |
| Bilingual output reference (D-4) | MEDIUM | LOW | **P1** | Shared module that benefits multiple Skills. 1-2 hours. |
| Bilingual paragraph-by-paragraph (TS-4) | HIGH | LOW-MEDIUM | **P1** | User-requested feature, patterns already exist in v1.0. |
| Repo scan (TS-1) | HIGH | MEDIUM | **P1** | Foundation for entire repo-to-paper feature. |
| Top-down H1 generation (TS-2a) | HIGH | MEDIUM | **P1** | Core repo-to-paper UX: structure first. |
| Top-down H2 generation (TS-2b) | HIGH | MEDIUM | **P1** | Must follow H1, unlocks literature search. |
| Literature at H2 (TS-3) | HIGH | MEDIUM | **P1** | Quality differentiator; reuses v1.0 literature-skill logic. |
| Body generation (TS-2c) | HIGH | HIGH | **P1** | Final stage; complexity is in evidence grounding + placeholder enforcement. |
| Anti-hallucination placeholders (D-5) | HIGH | LOW | **P1** | Pattern already exists; must be enforced in prompts. |
| Evidence-before-interpretation (D-3) | MEDIUM | LOW | **P2** | Quality differentiator; pattern already exists in experiment-skill. |
| Code-aware paper structure (D-1) | HIGH | HIGH | **P2** | The "magic" differentiator but requires sophisticated code analysis heuristics. |
| Ref files with metadata (D-2) | MEDIUM | MEDIUM | **P2** | Enhances literature integration; saves abstracts for body generation. |
| Retrofit bilingual to v1.0 Skills | LOW | LOW | **P3** | Nice-to-have; v1.0 Skills work fine without it. |

**Priority key:**
- P0: Must fix immediately (bug)
- P1: Must have for v2.0 launch
- P2: Should have, add during v2.0 development
- P3: Nice to have, future consideration

---

## Competitor Feature Analysis

### Repo-to-Paper Domain

No direct competitor exists for "code repository -> academic paper draft." The landscape has tools in adjacent spaces:

| Feature | Gatsbi | Paperguide | SciSpace | **Our Approach** |
|---------|--------|------------|----------|-----------------|
| Input source | Topic/title/notes | Topic/instructions | Topic/notes | **Actual code repository** |
| Structure generation | AI-generated outline | AI-generated outline | AI-generated outline | **Code-derived outline with user checkpoints** |
| Literature integration | Auto-citations from DB | Citation-backed drafts | 280M+ paper search | **Semantic Scholar MCP at H2 stage, ref files with abstracts** |
| User control | Low (one-click) | Medium (outline then expand) | Low (one-click) | **High (confirm at H1, H2, body levels)** |
| Evidence grounding | None (topic-based) | None (topic-based) | None (topic-based) | **Reads actual result files from repo** |
| Bilingual | No | No | No | **English + Chinese paragraph-by-paragraph** |
| Anti-hallucination | Varies | Varies | Claims real sources | **Explicit placeholders for ungrounded claims** |
| Platform | Web app | Web app | Web app | **Claude Code Skill (local, private, offline-capable)** |

### Bilingual Academic Output Domain

| Feature | Translation-Skill (v1.0) | Reviewer/Logic (v1.0) | DeepL | **v2.0 Approach** |
|---------|--------------------------|----------------------|-------|-------------------|
| Bilingual format | LaTeX `%` comments | Markdown `>` blockquotes | Side-by-side panes | **Both formats, selected by output type** |
| Paragraph pairing | 1:1 with `% --- Paragraph N ---` markers | Per-concern, not per-paragraph | Sentence-level | **1:1 paragraph pairing with explicit markers** |
| Academic register | Yes (expression patterns) | Yes (domain terminology) | Generic | **Full reference library integration** |
| Shared spec | No (per-Skill) | No (per-Skill) | N/A | **Shared `bilingual-output.md` reference** |

### AskUserQuestion Usage

| Aspect | paper-polish-workflow (current) | Other v1.0 Skills | **v2.0 Fix** |
|--------|-------------------------------|-------------------|--------------|
| Tool name | `mcp_question` (WRONG) | "Structured Interaction" category (correct per conventions) | **`AskUserQuestion` (correct built-in name)** |
| Schema | `{ questions: [...], options: [{ label, description }] }` | Skill-conventions fallback pattern | **Correct schema: `{ questions: [{ question, header, multiSelect, options: [{ label, description }] }] }`** |
| Fallback | Text-based table format | Plain-text 1-3 questions | **Keep text-based fallback; fix primary path** |

---

## Complexity Deep-Dive: Repo-to-Paper Skill

The repo-to-paper Skill is by far the most complex v2.0 feature. This section breaks down its sub-components for roadmap planning.

### Phase 1: Repo Scan (~100 lines of SKILL.md)

**What it does:** Recursively scan repo, categorize files, extract key metadata.

**File categorization heuristics:**
| Pattern | Category |
|---------|----------|
| `train*.py`, `run*.py`, `main.py` | Training scripts |
| `eval*.py`, `test*.py`, `benchmark*.py` | Evaluation scripts |
| `model*.py`, `network*.py`, `arch*.py` | Model architecture |
| `data*.py`, `dataset*.py`, `loader*.py` | Data pipeline |
| `config*.yaml`, `config*.json`, `*.cfg` | Configuration |
| `results/`, `output/`, `logs/`, `*.csv` | Result files |
| `ablation*.py` | Ablation studies |
| `*.png`, `*.pdf`, `*.jpg` in `figures/`, `plots/` | Visualizations |
| `README.md`, `docs/` | Documentation |
| `requirements.txt`, `environment.yml` | Environment |

**Output:** Structured inventory table presented to user before any generation begins.

**Risks:** Very large repos (>1000 files) may overwhelm context. Mitigate by reading only directory structure first, then selectively reading key files.

### Phase 2: Top-Down Structure Generation (~150 lines of SKILL.md)

**H1 level:** Map repo categories to standard paper sections. Propose ordering. Use AskUserQuestion with 2-4 structure variants.

**H2 level:** For each H1, derive subsections from actual code structure. Example: Methods H1 with `data_loader.py` + `model.py` + `train.py` suggests "Data Collection and Preprocessing" + "Model Architecture" + "Training Procedure" as H2 subsections.

**Risks:** The mapping from code structure to paper structure is heuristic and may not work for unconventional repos. Must include fallback: "I could not automatically derive structure. Please describe your paper's intended organization."

### Phase 3: Literature Integration (~80 lines of SKILL.md)

**Reuses:** literature-skill's MCP pre-flight check, search call, BibTeX generation.
**Adds:** Batch search (one query per H2 subsection), ref file persistence, automatic query derivation from subsection titles.

**Risks:** Semantic Scholar MCP may be unavailable. Fallback: skip literature integration, use `[CITATION NEEDED]` placeholders throughout.

### Phase 4: Body Generation (~100 lines of SKILL.md)

**For each confirmed H2 subsection:**
1. Read relevant repo files (code, results, configs).
2. Load appropriate expression pattern leaves based on section type.
3. Generate paragraphs following evidence-before-interpretation rule.
4. Screen against anti-AI vocabulary patterns.
5. Insert placeholders for anything not grounded in repo data.

**Risks:** Longest phase, highest chance of hallucination. Must enforce placeholder discipline aggressively. The prompt must explicitly list what constitutes "grounded" (value found in result file, function found in code) vs. "ungrounded" (interpretive claim, literature comparison).

### Total SKILL.md Budget

Estimated: ~430 lines, which exceeds the 300-line convention. Options:
1. **Accept the overage with justification** -- repo-to-paper is genuinely a multi-phase complex Skill.
2. **Extract scan heuristics to a reference file** -- Move file categorization table to `references/repo-scan-heuristics.md` (~80 lines saved).
3. **Extract bilingual output rules** -- Already planned as `references/bilingual-output.md` (~30 lines saved).

**Recommendation:** Option 2 + Option 3, targeting ~320 lines. Slight overage justified for this Skill given its multi-phase nature.

---

## Sources

### HIGH Confidence
- [PROJECT.md](../../PROJECT.md) -- v2.0 milestone definition, constraints, existing codebase structure
- [skill-conventions.md](../../references/skill-conventions.md) -- Skill design rules, 300-line budget, interaction modes, fallback patterns
- [translation-skill SKILL.md](../../.claude/skills/translation-skill/SKILL.md) -- Existing bilingual LaTeX comment pattern
- [logic-skill SKILL.md](../../.claude/skills/logic-skill/SKILL.md) -- Existing bilingual markdown blockquote pattern
- [reviewer-simulation-skill SKILL.md](../../.claude/skills/reviewer-simulation-skill/SKILL.md) -- Existing bilingual concern format
- [experiment-skill SKILL.md](../../.claude/skills/experiment-skill/SKILL.md) -- Evidence-before-interpretation pattern, placeholder pattern
- [literature-skill SKILL.md](../../.claude/skills/literature-skill/SKILL.md) -- Semantic Scholar MCP integration pattern
- [paper-polish-workflow SKILL.md](../../paper-polish-workflow/SKILL.md) -- Source of AskUserQuestion bug (references `mcp_question`)
- [Claude Code AskUserQuestion tool description](https://github.com/Piebald-AI/claude-code-system-prompts/blob/main/system-prompts/tool-description-askuserquestion.md) -- Exact tool schema (287 tokens)
- [Claude Code internal tools implementation](https://gist.github.com/bgauryy/0cdb9aa337d01ae5bd0c803943aa36bd) -- AskUserQuestion interface: questions[1-4], options[2-4], header max 12 chars

### MEDIUM Confidence
- [Anthropic Building Effective Agents](https://www.anthropic.com/research/building-effective-agents) -- Prompt chaining pattern: outline-then-body validated as standard LLM workflow
- [PaperCoder / Paper2Code (arXiv:2504.17192)](https://arxiv.org/abs/2504.17192) -- Multi-agent paper<->code framework; three-stage planning->analysis->generation pattern
- [Gatsbi AI Paper Writer](https://www.gatsbi.com/) -- Competitor: one-click paper drafts with citations and figures
- [Paperguide AI Research Assistant](https://paperguide.ai/) -- Competitor: generate full document from topic + methodology
- [Elicit AI](https://elicit.com/) -- Evidence synthesis, 138M+ papers, evaluated Claude Opus 4.5 for extraction
- [Overleaf multilingual LaTeX guide](https://www.overleaf.com/learn/latex/Multilingual_typesetting_on_Overleaf_using_polyglossia_and_fontspec) -- CTeX/XeCJK/babel for Chinese-English LaTeX
- [Claude Code Best Practices](https://code.claude.com/docs/en/best-practices) -- AskUserQuestion usage patterns

### LOW Confidence (needs validation)
- Repo file categorization heuristics -- Based on common Python ML project conventions. Non-Python repos or unconventional structures may require different heuristics. Needs testing against real user repos.
- 430-line SKILL.md estimate -- Based on section-by-section estimation. Actual line count may vary significantly during implementation. The 300-line budget may require more aggressive extraction to reference files.

---
*Feature research for: v2.0 Repo-to-Paper, Bilingual Enhancement, AskUserQuestion Fix*
*Researched: 2026-03-17*
