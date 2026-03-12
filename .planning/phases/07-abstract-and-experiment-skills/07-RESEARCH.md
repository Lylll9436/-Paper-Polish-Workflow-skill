# Phase 7: Abstract and Experiment Skills - Research

**Researched:** 2026-03-12
**Domain:** Claude Code Skill authoring — abstract generation and experiment analysis/discussion for academic papers
**Confidence:** HIGH

---

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| SECT-01 | Abstract generation/optimization Skill produces abstracts following the 5-sentence formula (contribution, difficulty, method, evidence, result) | Abstract Skill workflow maps directly to the 5-sentence Farquhar formula; both generate-from-scratch and restructure-existing paths are documented in Architecture Patterns |
| SECT-03 | Experiment analysis Skill helps analyze results, identify patterns, and generate discussion paragraphs | Experiment Skill workflow covers pattern identification from structured data (tables, statistics) and discussion paragraph generation connecting findings to research questions and literature |
</phase_requirements>

---

## Summary

Phase 7 creates two new Skills following the conventions established in Phase 2. Both Skills are markdown-only, depend on existing reference libraries, and must stay under the ~300 line budget. No new libraries or tools are required; all needed reference content already exists in `references/expression-patterns/`.

The Abstract Skill (SECT-01) is a direct-mode-default Skill with two execution paths: generate a new abstract from user-provided content, or restructure an existing abstract using the 5-sentence Farquhar formula. The formula aligns exactly with the project's CLAUDE.md writing philosophy: contribution, difficulty, method, evidence, key result. The relevant reference modules for this Skill are `introduction-and-gap.md` (contribution/gap framing) and `conclusions-and-claims.md` (result framing and claim calibration).

The Experiment Analysis Skill (SECT-03) accepts structured result data (tables, statistics, or pasted result blocks) and produces two deliverables: a pattern analysis identifying significant findings, and discussion paragraphs that connect findings to research questions and existing literature. The relevant reference modules are `results-and-discussion.md` (result reporting language and pattern interpretation) and `conclusions-and-claims.md` (calibrated claim language when summarizing findings). Both Skills must follow the anti-AI vocabulary awareness convention established in Phases 3, 4, and 5.

**Primary recommendation:** Author two separate SKILL.md files — one per requirement ID — keeping both under 230 lines (well within budget) and using the existing expression-patterns reference library as their only external dependency beyond the conventions contract.

---

## Standard Stack

### Core

| Component | Version | Purpose | Why Standard |
|-----------|---------|---------|--------------|
| `references/skill-conventions.md` | Current | Authoring contract all Skills must follow | Phase 2 locked decision; all phases since have followed it |
| `references/skill-skeleton.md` | Current | Copyable starting point for new SKILL.md files | Companion to conventions; Phase 3-6 all used it |
| `references/expression-patterns.md` | Current | Expression patterns overview; entrypoint for leaf loading | Required by all expression-heavy Skills |
| `references/expression-patterns/introduction-and-gap.md` | Current | Contribution framing, gap statements — abstract sentences 1-2 | Used by Translation and Polish Skills for same rhetorical moves |
| `references/expression-patterns/results-and-discussion.md` | Current | Result reporting, pattern interpretation, discussion language | Core module for Experiment Skill; also supports Abstract Skill sentence 4-5 |
| `references/expression-patterns/conclusions-and-claims.md` | Current | Calibrated claim language, contribution statements | Supports both abstract sentence 1 (contribution) and discussion closing moves |

### Supporting

| Component | Version | Purpose | When to Use |
|-----------|---------|---------|-------------|
| `references/expression-patterns/methods-and-data.md` | Current | Method description patterns — abstract sentence 3 | Load when user's method description is weak or needs improving |
| `references/anti-ai-patterns.md` | Current | Vocabulary blacklist overview | Load as leaf hints to prevent AI-sounding output in generated text |
| `references/journals/ceus.md` | Current | CEUS journal style contract | Load when user specifies CEUS as target journal |

### Alternatives Considered

| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| Reusing expression-patterns references | Inlining abstract formula guidance in SKILL.md | Inlining is faster but violates the lean-Skill contract and duplicates content already in references |
| Two separate Skills | One combined abstract-and-experiment Skill | Combined would exceed the 300 line budget and blur the Skill's `primary_intent`, making discovery harder |

**Installation:**

```bash
# None — this phase is markdown-only. No packages required.
```

---

## Architecture Patterns

### Recommended Project Structure

```
.claude/skills/
├── abstract-skill/
│   └── SKILL.md              # SECT-01
└── experiment-skill/
    └── SKILL.md              # SECT-03
```

Both Skills follow the same flat structure all prior Skills use: one directory, one SKILL.md, no subdirectories.

### Pattern 1: Dual-Path Direct Skill (Abstract Skill)

**What:** A direct-mode Skill that handles two mutually exclusive execution paths within the same SKILL.md — generate-from-scratch versus restructure-existing — selected via pre-question or trigger inference.

**When to use:** When both paths share the same output format and nearly the same reference loading, but differ in what the user provides as input.

**Example (mode inference section):**

```markdown
## Modes

| Mode | Default | Behavior |
|------|---------|----------|
| `direct` | Yes | Single-pass abstract output using 5-sentence formula |
| `batch` | | Not supported — abstract requires full paper context |

**Path selection:**
- User provides raw content (methods, results, contribution) → generate path
- User provides an existing abstract → restructure path
- Ambiguous input → ask before proceeding
```

**Key constraint:** Both paths must produce the same locked 5-sentence formula output. The restructure path preserves the user's content; the generate path synthesizes from provided materials.

### Pattern 2: Phase-Separated Analysis Skill (Experiment Skill)

**What:** A two-phase workflow — analysis phase (identify patterns from data) then generation phase (write discussion paragraphs) — with a user decision point between phases.

**When to use:** When the first phase produces an intermediate artifact the user may want to review and correct before the second phase runs. The De-AI Skill (Phase 5) uses the same two-phase detect-then-rewrite pattern.

**Example (workflow sketch):**

```markdown
### Phase 1: Analyze

- Accept result data: table (file or pasted), list of statistics, or narrative description
- Identify patterns: significant comparisons, notable trends, outliers
- Present structured analysis: "Finding N: [observation] — [evidence]"
- Ask user to confirm, correct, or add before proceeding

### Phase 2: Generate Discussion

- Write discussion paragraph(s) for each confirmed finding
- Connect each finding to stated research question(s)
- Load results-and-discussion.md for expression patterns
- Load conclusions-and-claims.md for calibrated interpretation language
```

**Key constraint:** Phase 2 cannot run if Phase 1 is skipped. If user provides result data and research questions together, offer to run both phases sequentially with a single confirm.

### Pattern 3: 5-Sentence Formula as Locked Output Contract

**What:** The abstract formula (contribution, difficulty, method, evidence, key result) is not a soft guideline — it is the locked output shape for SECT-01. The Skill must label each sentence explicitly so users can verify alignment.

**When to use:** Any Skill where the output has a mandated structure (similar to how the Reviewer Simulation Skill locks its report format with explicit section headings).

**Example output contract block:**

```markdown
## Output Contract

| Sentence | Formula Position | Content |
|----------|-----------------|---------|
| 1 | Contribution | "We introduce/demonstrate/propose..." |
| 2 | Difficulty/Importance | Why this problem is hard or matters |
| 3 | Method | How the problem is solved (with key terminology) |
| 4 | Evidence | What was measured or compared |
| 5 | Key Result | Most important number or qualitative outcome |
```

### Anti-Patterns to Avoid

- **Formula-silent output:** Producing a good abstract without labeling which sentence maps to which formula position. Users cannot verify or iterate on unlabeled output.
- **Restructure path that loses content:** When restructuring an existing abstract, discarding sentences that do not fit neatly into one formula position. The requirement says "preserving content" — missing content must be flagged, not silently dropped.
- **Experiment Skill accepting vague input:** Proceeding with a discussion paragraph when the user only says "my results show improvement." The Skill must require a measurable finding (value, direction, comparison group) before generating text.
- **Connecting to literature without user input:** The success criteria says "connect findings to existing literature." The Skill cannot invent literature connections; it must ask the user to name the relevant references or prior findings to connect to.
- **Single combined Skill:** Putting both abstract and experiment logic into one SKILL.md will exceed the line budget and make trigger discovery confusing.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Abstract formula guidance | Inline the 5-sentence rules in SKILL.md | Reference CLAUDE.md's Farquhar formula; use conclusions-and-claims.md and introduction-and-gap.md for expression patterns | Formula already documented in project CLAUDE.md; expression modules cover the language |
| Result language for discussion | Write custom expression tables | Load `references/expression-patterns/results-and-discussion.md` | Already covers quantified comparison, pattern interpretation, heterogeneity, and limitation-aware language |
| Claim calibration | Write custom hedging rules | Load `references/expression-patterns/conclusions-and-claims.md` | Already covers `suggests`, `indicates`, scope boundaries, and avoid-list |
| Anti-AI vocabulary screening | Inline word blacklist | Load `references/anti-ai-patterns/vocabulary.md` leaf as needed | Pattern already established in Phases 3-5; consistency requires using the same reference |
| Journal style adaptation | Hardcode CEUS rules in SKILL.md | Load `references/journals/ceus.md` conditionally | Same pattern all other Skills use; refusal-on-missing convention applies here too |

**Key insight:** Phase 7 Skills are consumers of the reference system built in Phases 1-2. No new reference content needs to be created — only Skill logic that correctly loads and applies existing materials.

---

## Common Pitfalls

### Pitfall 1: Abstract Skill Diverges from Formula on Restructure Path

**What goes wrong:** When restructuring an existing abstract, the Skill reorganizes prose but does not ensure each of the 5 formula positions is filled. The output reads well but fails the formula contract.

**Why it happens:** LLMs naturally reorganize text to improve flow without auditing structural completeness.

**How to avoid:** After restructuring, run an explicit internal audit: "Sentence 1 maps to contribution? Sentence 2 maps to difficulty?" before presenting output. If a formula position is missing, flag it with `[MISSING: difficulty statement]` rather than silently omitting it.

**Warning signs:** Abstract output has fewer than 5 sentences, or two sentences cover the same formula position while another is empty.

### Pitfall 2: Experiment Skill Generates Discussion Without Grounded Evidence

**What goes wrong:** Discussion paragraphs contain interpretive claims ("this suggests the model is superior") without citing the specific number or comparison that supports the claim.

**Why it happens:** The generation phase conflates interpretation with evidence. The expression patterns module uses `suggests` correctly, but without grounding the claim first.

**How to avoid:** Enforce a generation rule: every interpretive sentence in a discussion paragraph must be preceded by or contain a grounding evidence clause. Template: "X outperforms Y by Z on metric M, suggesting that [interpretation]."

**Warning signs:** Discussion paragraphs that use `suggests`, `indicates`, or `demonstrates` without a preceding quantified statement.

### Pitfall 3: Literature Connection Hallucination

**What goes wrong:** SECT-03 requires discussion paragraphs that "connect findings to existing literature." If the Skill tries to make those connections autonomously, it may hallucinate citation relationships.

**Why it happens:** The generation instruction is open-ended; the Skill fills the slot with plausible-sounding references.

**How to avoid:** The Skill must ask the user to provide the relevant prior work to connect to: "Which papers or findings should this discussion connect to?" If the user cannot provide them, the Skill writes a placeholder: `[CONNECT TO: describe the prior finding here]`.

**Warning signs:** Discussion paragraphs contain author names or citation-like phrases that the user did not provide as input.

### Pitfall 4: Exceeding the Line Budget

**What goes wrong:** The Abstract Skill tries to include the full 5-sentence formula examples inline, and the Experiment Skill includes extended pattern tables, causing both to exceed 300 lines.

**Why it happens:** Both skills have structured output formats that look like tables, and the temptation is to make them self-contained.

**How to avoid:** Keep formula guidance minimal in the SKILL.md body. Load expression-patterns leaf files at runtime. Existing Skills (196-227 lines) show that the budget is achievable with proper reference discipline.

**Warning signs:** SKILL.md draft exceeds 250 lines during authoring.

### Pitfall 5: Abstract Skill Output Format Ambiguity

**What goes wrong:** The Skill produces a plain paragraph abstract without labeling formula positions. The user cannot verify formula compliance or iterate on a specific sentence.

**Why it happens:** Generating a clean paragraph is simpler than generating labeled output.

**How to avoid:** Always present the abstract with labeled formula markers, then offer a clean version for clipboard use. Pattern borrowed from De-AI Skill's detection report format:

```
[1: Contribution] We propose...
[2: Difficulty] Despite growing interest in X, existing approaches fail to...
[3: Method] We address this by...
[4: Evidence] Evaluated on N datasets...
[5: Key Result] Our approach achieves...

---
Clean version:
We propose... Despite growing interest...
```

---

## Code Examples

Verified patterns from existing Skills in this project:

### Frontmatter Shape for Abstract Skill

```yaml
---
name: abstract-skill
description: >-
  Generate or optimize academic paper abstracts using the 5-sentence formula.
  Supports generate-from-scratch and restructure-existing modes. 摘要生成与优化。
triggers:
  primary_intent: generate or optimize academic paper abstract
  examples:
    - "Write an abstract for my paper"
    - "帮我写摘要"
    - "Optimize my existing abstract"
    - "改写我的摘要，符合五句话公式"
    - "Generate a submission-ready abstract for CEUS"
tools:
  - Read
  - Write
  - Structured Interaction
references:
  required:
    - references/expression-patterns.md
  leaf_hints:
    - references/expression-patterns/introduction-and-gap.md
    - references/expression-patterns/results-and-discussion.md
    - references/expression-patterns/conclusions-and-claims.md
    - references/expression-patterns/methods-and-data.md
input_modes:
  - file
  - pasted_text
output_contract:
  - labeled_abstract
  - clean_abstract
---
```

### Frontmatter Shape for Experiment Skill

```yaml
---
name: experiment-skill
description: >-
  Analyze experiment results and generate discussion paragraphs for academic papers.
  Accepts tables, statistics, or result descriptions. 实验分析与讨论段落生成。
triggers:
  primary_intent: analyze experiment results and generate discussion paragraphs
  examples:
    - "Analyze my experiment results"
    - "帮我分析实验结果"
    - "Generate discussion for my results table"
    - "把我的实验数据写成讨论段"
    - "Write discussion paragraphs for my findings"
tools:
  - Read
  - Write
  - Structured Interaction
references:
  required:
    - references/expression-patterns.md
  leaf_hints:
    - references/expression-patterns/results-and-discussion.md
    - references/expression-patterns/conclusions-and-claims.md
    - references/expression-patterns/methods-and-data.md
input_modes:
  - file
  - pasted_text
  - structured_data
output_contract:
  - pattern_analysis
  - discussion_paragraphs
---
```

### Ask Strategy for Dual-Path Skill (Abstract)

```markdown
## Ask Strategy

**Before starting, ask about:**
1. Path: does the user have an existing abstract to restructure, or raw content to synthesize from?
2. Target journal (if not specified) — CEUS has specific abstract length conventions
3. Word limit (if the user knows it) — abstracts typically 150-300 words

**Path inference:**
- User pastes/provides a text that reads like an abstract → restructure path
- User provides sections, bullet points, or "my paper is about..." → generate path
- Ambiguous → ask "Do you have an existing abstract to restructure, or shall I generate one from your materials?"

**Rules:**
- Never ask more than 2 questions before starting
- Skip path question if inference is clear from input
```

### Two-Phase Workflow Pattern (Experiment Skill)

```markdown
### Phase 1: Analyze Results

1. Accept input: Read table file OR pasted results block OR narrative description
2. Extract measurable findings: comparisons, magnitudes, trends, outliers
3. Present structured finding list:
   ```
   Finding 1: [Method A] outperforms [Method B] by [X%] on [metric]
   Finding 2: Performance degrades in [condition] (N vs. M)
   Finding 3: [Subgroup] shows the largest effect ([value])
   ```
4. Ask: "Are these findings accurate? Add, correct, or remove before I write discussion."

### Phase 2: Generate Discussion

1. For each confirmed finding, write one discussion paragraph:
   - Open with evidence sentence using results-and-discussion.md patterns
   - Follow with interpretation sentence using calibrated language from conclusions-and-claims.md
   - Close with connection to research question (user must provide) or placeholder
2. If user provided prior work to connect to, add one sentence per finding connecting to literature
3. Anti-AI check: load vocabulary.md, avoid flagged patterns in output
```

---

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Abstract as freeform prose guidance | 5-sentence Farquhar formula with mandatory position labeling | Established in project CLAUDE.md | Skills must produce labeled output, not just good prose |
| Discussion generation as holistic writing | Phase-separated analysis-then-generation with user confirmation between phases | Established by De-AI Skill pattern (Phase 5) | Prevents hallucinated interpretation; user validates findings first |
| Literature connections by AI inference | Literature connections require user-provided prior work | Established by CLAUDE.md citation principles (~40% error rate) | Skill must request or placeholder, never invent, citation connections |
| Single-file Skills with all logic inline | Lean Skills with runtime reference loading | Phase 2 conventions | Both new Skills follow existing lean pattern; no new reference content needed |

**Patterns established across prior phases that this phase must continue:**

- LaTeX comment annotation format: if editing files, use distinct prefix (e.g., `% [Abstract]` or `% [Experiment]`)
- Journal-template-missing → refuse, not fallback-to-general (Translation, Polish, De-AI, Reviewer all do this)
- Anti-AI vocabulary awareness: load leaf hints from `references/anti-ai-patterns/` even if detection is not the primary task
- Bilingual Chinese/English triggers in frontmatter `examples` field
- Chinese inline translation optional but valuable: reviewer skill demonstrates this pattern for bilingual output

---

## Open Questions

1. **Should the Abstract Skill output to a file or only to conversation?**
   - What we know: abstracts are short (150-300 words), pasted-text workflows dominate in prior Skills
   - What's unclear: whether users want to save generated abstracts to a `.tex` or `.md` file automatically
   - Recommendation: default to conversation output; offer Write-to-file as an optional Step 4 if user confirms

2. **Should the Experiment Skill accept LaTeX table markup as input?**
   - What we know: Translation Skill handles LaTeX and preserves table environments; users work in LaTeX
   - What's unclear: whether the Experiment Skill needs to parse LaTeX table syntax or just read the data
   - Recommendation: treat LaTeX tables as readable text; the Skill reads data values and captions, not typesetting

3. **How many discussion paragraphs per finding?**
   - What we know: SECT-03 says "generate discussion paragraphs" (plural) without a fixed count
   - What's unclear: whether one paragraph per finding or a consolidated multi-paragraph discussion is expected
   - Recommendation: one paragraph per confirmed finding by default; offer consolidation as a follow-up option

4. **Does the Abstract Skill need to enforce journal-specific word count limits?**
   - What we know: CEUS does not have a hard word limit in the loaded template; abstract word counts vary by journal
   - What's unclear: whether to add word-count guidance to the CEUS template or handle it in the Skill
   - Recommendation: ask the user for their target word count during Ask Strategy; do not hardcode limits per journal

---

## Validation Architecture

`nyquist_validation` is `true` in `.planning/config.json` — this section is required.

### Test Framework

| Property | Value |
|----------|-------|
| Framework | None (markdown-only Skills; no runtime executable) |
| Config file | None |
| Quick run command | Manual structural inspection (line count + section audit) |
| Full suite command | Validate both SKILL.md files for conventions compliance |

### Phase Requirements → Test Map

| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| SECT-01 | Abstract Skill generates labeled 5-sentence abstract | manual-only | `grep -c "\[1:\|2:\|3:\|4:\|5:" output` (spot check) | ❌ Wave 0 |
| SECT-01 | Abstract Skill restructures existing abstract preserving content | manual-only | Read SKILL.md, verify restructure path exists in Workflow section | ❌ Wave 0 |
| SECT-03 | Experiment Skill accepts tables/statistics and identifies patterns | manual-only | Read SKILL.md, verify Phase 1 input handling | ❌ Wave 0 |
| SECT-03 | Experiment Skill generates discussion paragraphs connecting to research questions | manual-only | Read SKILL.md, verify Phase 2 connects to research questions or uses placeholder | ❌ Wave 0 |

**Manual-only justification:** Skills are markdown prompts, not executable code. The only runtime artifact is a SKILL.md read by a Claude instance. Automated structural tests (line count, section presence) are feasible; behavioral tests require human invocation.

### Sampling Rate

- **Per task commit:** `wc -l .claude/skills/abstract-skill/SKILL.md .claude/skills/experiment-skill/SKILL.md` (line budget check)
- **Per wave merge:** Structural audit — `grep -n "^## " .claude/skills/abstract-skill/SKILL.md .claude/skills/experiment-skill/SKILL.md` (required sections present)
- **Phase gate:** Both SKILL.md files pass structural audit and at least one manual invocation test before `/gsd:verify-work`

### Wave 0 Gaps

- [ ] `.claude/skills/abstract-skill/SKILL.md` — covers SECT-01 (generate and restructure paths)
- [ ] `.claude/skills/experiment-skill/SKILL.md` — covers SECT-03 (analyze and generate phases)
- [ ] Manual invocation test for each Skill (note in VALIDATION.md)

---

## Sources

### Primary (HIGH confidence)

- `.planning/REQUIREMENTS.md` — SECT-01 and SECT-03 requirement text
- `.planning/ROADMAP.md` — Phase 7 success criteria (all 4 criteria)
- `references/skill-conventions.md` — Authoring rules; read in full for this research
- `references/skill-skeleton.md` — Copyable starting point structure
- `.claude/skills/reviewer-simulation-skill/SKILL.md` — Structural reference (most recently authored Skill)
- `.claude/skills/de-ai-skill/SKILL.md` — Two-phase workflow pattern
- `.claude/skills/polish-skill/SKILL.md` — Dual-mode (direct + guided) pattern
- `.claude/skills/translation-skill/SKILL.md` — Direct mode, file output, ask strategy
- `references/expression-patterns/results-and-discussion.md` — Experiment Skill primary reference
- `references/expression-patterns/conclusions-and-claims.md` — Abstract and Experiment Skill secondary reference
- `references/expression-patterns/introduction-and-gap.md` — Abstract Skill primary reference
- `.planning/STATE.md` — Accumulated decisions from all prior phases
- `~/.claude/CLAUDE.md` — 5-sentence Farquhar abstract formula; anti-hallucination citation rule

### Secondary (MEDIUM confidence)

- `.planning/phases/02-skill-conventions/02-RESEARCH.md` — Background on conventions decisions; confirmed patterns still valid

### Tertiary (LOW confidence)

- None — this phase depends entirely on project-local contracts, not external dependencies.

---

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — all components are existing project files, verified by reading
- Architecture patterns: HIGH — drawn directly from four existing Skills and the conventions document
- Pitfalls: HIGH — derived from explicit success criteria gaps and patterns visible in prior Skill implementations
- Validation: HIGH — constraint (markdown-only, no test framework) confirmed by project structure

**Research date:** 2026-03-12
**Valid until:** 2026-04-12 (stable domain — no external dependencies)

---

*Phase: 07-abstract-and-experiment-skills*
*Research completed: 2026-03-12*
*Ready for planning: yes*
