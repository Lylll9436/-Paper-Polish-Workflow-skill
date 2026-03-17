# Pitfalls Research: v2.0 Features (Repo-to-Paper, Bilingual Output, AskUserQuestion Fix)

**Domain:** Adding repo-to-paper generation, bilingual paragraph-by-paragraph comparison, and AskUserQuestion enforcement to an existing 11-Skill Claude Code suite for academic paper writing
**Researched:** 2026-03-17
**Confidence:** HIGH (grounded in codebase analysis of all 11 existing Skills, v1.0 retrospective lessons, and verified web research on context management, hallucination patterns, and Claude Code Skills 2.0 testing)

---

## Critical Pitfalls

Mistakes that cause rewrites, break existing Skills, or undermine the v2.0 value proposition.

---

### Pitfall 1: Context Window Exhaustion During Repo Scanning

**What goes wrong:**
The repo-to-paper Skill reads an entire experiment repository -- source files, configs, READMEs, data descriptions, notebooks -- and the accumulated file content exceeds usable context before generation even begins. The Skill either crashes with a truncation error, or Claude silently forgets earlier files during the top-down H1-to-body generation cascade. Later sections (H3 body paragraphs) are generated without awareness of files read at the H1 planning stage.

**Why it happens:**
- A typical ML experiment repo contains 20-50 files. Each file read costs 2,000-8,000 tokens. Reading 30 files at ~4,000 tokens each = 120K tokens consumed before any generation begins.
- The v2.0 design specifies a top-down workflow (H1 -> H2 -> H3 -> body) with user checkpoints at each level. Each checkpoint adds conversation turns that accumulate in context. By the time body paragraphs are being generated, the context is already 60-80% full.
- Claude Code's compaction mechanism drops the oldest tokens first when the buffer fills. This means files read during the initial repo scan get compacted away precisely when they are needed most -- during the final body generation step.
- Even with the 1M context window (GA for Opus 4.6 and Sonnet 4.6 as of 2026), 80% of context is consumed by file reads and tool results, leaving only 20% for the agent's reasoning and output. A large repo can still exhaust this.
- v1.0 Skills never faced this because they process a single paper section (1-5 pages), not an entire repo. The repo-to-paper Skill is the first Skill to require multi-file scanning.

**How to avoid:**
1. Implement a **two-pass scanning strategy**: Pass 1 reads only filenames, directory structure, and README/config files (low token cost) to build a repo map. Pass 2 reads only the files identified as relevant to each H2 section, loaded just-in-time before that section is generated.
2. Enforce a **file budget**: cap at 10-15 files read per session. If the repo has more, the Skill must ask the user which files contain the core experiment logic, results, and configuration.
3. Design the workflow to be **resumable across sessions**: write intermediate outputs (H1 outline, H2 section plans, collected findings) to disk files so the Skill can resume in a fresh context. Each generation session starts clean by reading only the relevant intermediate file and the specific source files needed for that section.
4. Use **structured summaries instead of raw file content**: after reading each source file, extract a 5-10 line structured summary (purpose, key functions/classes, metrics, results) and store it in an intermediate file. Subsequent generation steps read summaries, not raw code.
5. Never read binary files, data files (.csv, .parquet), or large notebooks in full. For notebooks, extract only markdown cells and output cells containing results.

**Warning signs:**
- Skill instructions say "Read all files in the repository" without a file budget
- No intermediate output files written between the scanning phase and the generation phase
- Body paragraphs in later sections contain generic claims that do not reference specific files or results
- User reports that Claude "forgot" details from files read earlier in the session
- Context compaction events visible in Claude Code logs during a single repo-to-paper run

**Phase to address:**
Phase 1 (Repo-to-Paper Skill design). The scanning strategy and file budget must be specified in the Skill architecture before any generation logic is written. Retrofitting context management is extremely difficult because it requires restructuring the entire workflow.

---

### Pitfall 2: Hallucinated Experiment Results and Unsupported Claims in Generated Paper

**What goes wrong:**
The repo-to-paper Skill generates paper text that makes claims not actually supported by the repository code or results. Common failure modes:
- Fabricating accuracy numbers, F1 scores, or p-values that do not appear in any output file
- Describing ablation studies or baseline comparisons that were not actually implemented
- Claiming the method handles edge cases (e.g., "robust to missing data") when the code contains no such handling
- Generating a Related Work section with hallucinated citations (the most dangerous variant, already documented in v1.0 Pitfall 1)
- Inferring experimental methodology from function names rather than actual implementation, leading to incorrect method descriptions

**Why it happens:**
- LLMs predict plausible-sounding text. A repo containing a model training script will trigger Claude to generate plausible-sounding results paragraphs even when the actual results files are not in the repo or were not read.
- The top-down workflow (H1 -> H2 -> H3 -> body) creates a commitment cascade: the H1 outline may promise sections ("Ablation Study", "Comparison with Baselines") that have no backing data in the repo. By the time body text is generated, the outline creates pressure to fill these sections with content.
- GPTZero's ICLR 2026 scan found 50+ papers with hallucinated citations that passed peer review by 3-5 expert reviewers. If expert humans miss these errors, authors using this tool are even less likely to catch them.
- Context compaction (Pitfall 1) makes this worse: if the actual results files were read early but compacted away, Claude generates from its training distribution rather than from the repo evidence.

**How to avoid:**
1. **Evidence-first, not outline-first**: reverse the generation order for quantitative sections. Before writing Results, the Skill must extract and present a structured evidence table (metric, value, source file, line number) for user confirmation. Only confirmed evidence appears in generated text. This mirrors the v1.0 experiment-skill's two-phase pattern (Phase 1 findings -> confirm -> Phase 2 discussion).
2. **Mandatory `[SOURCE: filename:line]` annotations**: every factual claim in generated text must include an inline source annotation pointing to the exact file and approximate line. Claims without source annotations are flagged as `[UNVERIFIED]`.
3. **No Related Work generation from memory**: the Skill must either (a) use the existing literature-skill via Semantic Scholar MCP to find references, saving them as ref files with metadata and abstracts per the v2.0 design, or (b) insert `[CITATION NEEDED]` placeholders. The existing v1.0 anti-hallucination rule (MCP data only for BibTeX) must be extended to the repo-to-paper Skill without exception.
4. **Outline validation checkpoint**: after generating the H1/H2 outline, explicitly ask the user: "Does your repo contain data/results for each of these sections?" Remove sections that lack backing data rather than generating empty content.
5. **Guard against commitment cascade**: if the outline includes a section (e.g., "Ablation Study") but no corresponding results are found in the repo, the Skill must flag this and remove or demote the section rather than fabricating content.

**Warning signs:**
- Generated text contains specific numbers (accuracy percentages, p-values) without `[SOURCE:]` annotations
- The outline includes sections like "Comparison with State-of-the-Art" when the repo contains no baseline implementations
- BibTeX entries appear in generated output without Semantic Scholar MCP verification
- The generated method description does not match the actual code logic

**Phase to address:**
Phase 1 (Repo-to-Paper Skill core design). The evidence-first pattern and source annotation requirement must be baked into the Skill workflow before any generation logic exists. This is the v2.0 equivalent of v1.0's anti-hallucination citation rule: design it in, do not retrofit it.

---

### Pitfall 3: Bilingual Output Doubling Context Consumption and Output Length

**What goes wrong:**
Adding paragraph-by-paragraph bilingual comparison (English + Chinese) to Skills that currently produce English-only output doubles the output token count. For a 3,000-word Results section, bilingual output requires ~6,000 words (12,000+ tokens for output alone). Combined with input (paper section + reference files + conversation), this pushes against the output length limit and context ceiling simultaneously. The Skill either truncates the bilingual output mid-section, or the Chinese translation quality degrades noticeably in later paragraphs as context fills.

**Why it happens:**
- Academic text averages ~1.4 tokens per word. Chinese text is more token-dense (each character is typically 1-2 tokens). A bilingual paragraph pair costs roughly 2.5x the tokens of the English version alone, not simply 2x.
- Research on long-output LLMs (2025) shows that text generation quality degrades beyond ~4,000 output tokens (~2,600 words). Bilingual output for a full paper section easily exceeds this threshold.
- The v1.0 bilingual pattern (reviewer-skill, logic-skill) uses inline Chinese blockquotes for individual concerns -- short, bounded translations of 2-4 sentences each. This is fundamentally different from paragraph-by-paragraph comparison of an entire section, which is continuous long-form translation.
- Skills that already consume significant context (polish-skill loads 6+ reference leaves, de-ai-skill loads 3 anti-AI leaves + expression leaves) have less remaining context budget for doubled output.

**How to avoid:**
1. **Chunk bilingual output by paragraph, not by section**: generate one bilingual paragraph pair at a time, present it, then generate the next. This keeps each output within the 2,600-word quality threshold. The user sees results progressively rather than waiting for the entire section.
2. **Separate bilingual output into a dedicated output file**: write the bilingual comparison to `[name]_bilingual.md` (or `.tex`) rather than including it in the conversation output. This moves the doubled content out of the conversation context and into a file, preserving context for subsequent paragraphs.
3. **Make bilingual output opt-in, not default**: add a bilingual mode flag rather than always producing both languages. Many users will want English-only output for most Skills and bilingual only for review/verification purposes.
4. **Differentiate the bilingual pattern by Skill type**: for analysis Skills (reviewer, logic, experiment) keep the existing inline blockquote pattern -- it works well for structured concerns. For generation Skills (translation, polish, abstract, repo-to-paper) use the paragraph-by-paragraph file output pattern. Do not force one pattern onto all Skills.
5. **Use the translation-skill's proven bilingual pattern**: the translation-skill already writes bilingual comparison to `[name]_bilingual.tex` with Chinese as LaTeX comments and English as body text. Extend this pattern rather than inventing a new one. The pattern works because it keeps Chinese out of the conversation context.

**Warning signs:**
- Bilingual output truncated mid-paragraph in conversation
- Chinese translation quality visibly drops after the third or fourth paragraph in a section
- Skills that were within the ~300-line budget now exceed it due to added bilingual output instructions
- User reports "output was cut off" during bilingual generation

**Phase to address:**
Phase 2 (Bilingual Enhancement). But the architectural decision -- file-based bilingual output vs. conversation output -- must be made during Phase 1 Skill design, because it affects how all Skills structure their Output Contract section.

---

### Pitfall 4: AskUserQuestion Over-Enforcement Breaking Direct Mode and Low-Interaction Skills

**What goes wrong:**
Fixing the "Claude uses plain dialogue instead of structured AskUserQuestion" problem swings to the opposite extreme: the convention update makes structured questions mandatory for ALL pre-task interactions, including cases where freeform dialogue is more natural. Skills that currently work well in direct mode with zero questions (visualization-skill with both inputs provided, logic-skill with file path in trigger) start pausing for unnecessary structured questions, slowing down power users.

Worse, Skills with legitimately empty required references (`required: []`) -- logic-skill, visualization-skill, literature-skill -- get flagged as convention violations if the AskUserQuestion enforcement rule does not preserve the existing `required: []` escape hatch.

**Why it happens:**
- The v1.0 audit identified that "Claude uses plain dialogue instead of structured questions." The natural fix is to add stronger language: "MUST use AskUserQuestion for all pre-task interactions." But this overrides the existing convention that direct mode should "skip pre-questions if the user provided enough context."
- The skill-conventions.md Ask Strategy section already says "never ask more than 3 questions before producing initial output" and "in direct mode with sufficient context, proceed without pre-questions." But if the fix adds a blanket enforcement rule, these nuanced conditions get drowned out.
- The `required: []` escape hatch for self-contained Skills (documented in skill-conventions.md at line 73) is not prominently flagged. A sweeping AskUserQuestion enforcement rule may conflict with this unless explicitly carved out.
- Claude Code's AskUserQuestion tool presents multiple-choice options. Not all questions are well-suited to multiple choice. "What is your research question?" is inherently freeform. Forcing it into structured interaction makes the UX worse, not better.

**How to avoid:**
1. **Fix the root cause, not the symptom**: the problem is not "Claude does not use AskUserQuestion" but "Claude uses plain dialogue WHEN structured questions would be better." The fix should be a per-Skill audit identifying which specific questions in each Skill's Ask Strategy should use structured interaction, not a global mandate.
2. **Preserve the direct-mode exemption**: the convention should continue to state that in direct mode with sufficient context provided in the trigger, the Skill proceeds without pre-questions. The fix should not override this.
3. **Distinguish structured-question-appropriate vs. freeform-appropriate interactions**: multi-option questions (target journal, figure type, interaction mode) are good candidates for AskUserQuestion. Open-ended questions (research question, data description, correspondence author details) should remain freeform plain text. Document this distinction in the updated conventions.
4. **Protect the `required: []` escape hatch**: the convention update should explicitly state that Skills with `required: []` are not affected by AskUserQuestion enforcement. These Skills have no required references to configure via questions.
5. **Add a "question necessity" test**: before adding a structured question to a Skill, ask: "Would a power user providing a well-formed trigger be annoyed by this question?" If yes, make it conditional.

**Warning signs:**
- After the fix, direct-mode invocations that previously worked in one turn now require two turns
- Users report "the tool keeps asking me questions I already answered in my request"
- Skills with `required: []` (logic, visualization, literature) start asking pre-task questions they did not ask before
- The convention document contains "MUST use AskUserQuestion" without conditional qualifiers

**Phase to address:**
Phase 0 (Convention updates / tech debt fix). This must be resolved before any new Skills are authored in v2.0, because the convention change affects all Skills including the new repo-to-paper Skill. But it must be done carefully -- audit each Skill's Ask Strategy individually rather than applying a blanket rule.

---

### Pitfall 5: Repo-to-Paper Skill Exceeds the ~300-Line Budget, Breaking the Lean Skill Pattern

**What goes wrong:**
The repo-to-paper Skill is the most complex Skill in the suite. Its workflow includes: repo scanning, file budgeting, structured summarization, H1 outline generation with user checkpoint, H2 section planning with user checkpoint, integrated literature search via Semantic Scholar MCP, H3 subsection planning, body paragraph generation with source annotations, and bilingual output. Fitting this into ~300 lines forces either (a) a bloated Skill that violates the budget, or (b) an under-specified Skill that omits critical anti-hallucination and context management rules.

**Why it happens:**
- The v1.0 Skills are each focused on a single, well-bounded task (translate one section, polish one paragraph, generate one abstract). The repo-to-paper Skill is a multi-stage pipeline orchestrating several sub-tasks.
- The v1.0 retrospective identified that "Skill budget forces good design" -- but this was for single-task Skills. A multi-stage orchestrator Skill may need a different architectural pattern.
- The integrated literature search at H2 stage adds another workflow branch to the Skill, further straining the budget.
- Anti-hallucination rules (source annotations, evidence-first generation, outline validation) each require explicit documentation in the Skill body. These cannot be compressed without losing effectiveness.

**How to avoid:**
1. **Decompose into a coordinator Skill + sub-Skills**: the repo-to-paper Skill acts as an orchestrator that defines the workflow stages and calls existing Skills (literature-skill for references, experiment-skill patterns for findings extraction, abstract-skill for the abstract). The coordinator Skill stays within budget because it delegates detail to sub-Skills.
2. **If decomposition is rejected, raise the budget with explicit justification**: the convention already allows exceeding 300 lines "if the author can justify why the extra length is necessary." Document the justification: "Repo-to-paper requires multi-stage workflow with anti-hallucination guards at each stage."
3. **Extract workflow-stage reference files**: move the scanning strategy, evidence table template, and source annotation format to a `references/repo-to-paper/` directory. The Skill references these files rather than embedding them inline.
4. **Do not sacrifice anti-hallucination rules to meet the budget**: if the budget forces a tradeoff between completeness of anti-hallucination documentation and Skill length, extend the budget. A 400-line Skill with robust anti-hallucination is better than a 300-line Skill that generates fabricated claims.

**Warning signs:**
- The repo-to-paper SKILL.md draft exceeds 400 lines
- Anti-hallucination rules (source annotations, evidence-first pattern) are absent or vaguely stated to save space
- The Skill contains inline examples of every workflow stage instead of referencing external templates
- The Skill tries to handle literature search inline rather than delegating to the existing literature-skill

**Phase to address:**
Phase 1 (Repo-to-Paper Skill design). The architectural decision -- single orchestrator vs. decomposed sub-Skills -- must be made at design time. This decision cascades to whether new sub-Skill directories are needed and how the reference file tree expands.

---

## Moderate Pitfalls

Mistakes that cause significant rework or reduced effectiveness but are recoverable.

---

### Pitfall 6: Regression in Existing 11 Skills When Updating Conventions

**What goes wrong:**
The v2.0 convention updates (AskUserQuestion enforcement, bilingual output pattern, new tool declarations) inadvertently change behavior of existing Skills that were working correctly in v1.0. Examples:
- Adding `Structured Interaction` to the tools list of Skills that do not need it triggers unnecessary AskUserQuestion calls
- Changing the bilingual output pattern in conventions causes the reviewer-skill and logic-skill (which already have working bilingual inline blockquotes) to switch to paragraph-by-paragraph format, breaking their output structure
- Updating the `required: []` convention to require justification forces edits to logic-skill, visualization-skill, and literature-skill that add no value

**Why it happens:**
- The v1.0 convention-first approach means ALL Skills follow conventions. Changing conventions propagates to all 11 existing Skills.
- Claude Code's Skills 2.0 (March 2026) introduced evals and regression testing, but this project has no evals set up for the existing 11 Skills. There is no automated way to verify that a convention change did not break an existing Skill.
- The v1.0 retrospective noted "convention-first approach paid off" -- but this cuts both ways. Convention changes are high-leverage and high-risk.

**How to avoid:**
1. **Convention changes must be backward-compatible**: any new convention rule must be additive (new Skills follow the new rule) rather than retroactive (existing Skills must be updated). If a retroactive change is necessary, explicitly list which existing Skills are affected and what changes they need.
2. **Scope bilingual conventions by Skill type**: define two bilingual output patterns in conventions -- inline blockquote (for analysis/review Skills) and paragraph-by-paragraph file output (for generation/translation Skills). Existing Skills keep their current pattern; new Skills choose the appropriate one.
3. **Create a regression checklist**: before merging any convention change, manually verify the five most interaction-heavy Skills (polish-skill, de-ai-skill, reviewer-skill, experiment-skill, translation-skill) still produce expected output with the updated conventions.
4. **Version conventions**: add a version marker to skill-conventions.md (e.g., `Convention version: 2.0`). Existing Skills that reference conventions continue to follow the v1.0 rules until explicitly updated. New Skills follow v2.0 conventions.
5. **Do not edit existing Skills unless required**: the v1.0 tech debt items (literature-skill tool name, cover-letter-skill required field, required: [] escape hatch) are safe to fix because they are scoped changes. But do not also "while we are at it" refactor working Skill workflows.

**Warning signs:**
- Convention change diff touches skill-conventions.md sections that affect all Skills (Mode Taxonomy, Ask Strategy, Reference Loading Rules)
- After convention update, an existing Skill that previously had zero pre-questions now asks questions
- The reviewer-skill or logic-skill bilingual output format changes without explicit intent
- More than 3 existing SKILL.md files need editing after a convention change

**Phase to address:**
Phase 0 (Convention updates). Convention changes should be the first phase, completed and verified before any new Skill authoring begins. Include a "backward compatibility" section in each convention change PR.

---

### Pitfall 7: Literature Integration in Repo-to-Paper Creates a Bottleneck

**What goes wrong:**
The v2.0 design specifies "integrated literature search: Semantic Scholar references collected by H2 stage, saved as ref files with metadata + abstracts." This creates two problems:
1. **Serialization bottleneck**: the H2 outline cannot be finalized until literature search completes for each section, and Semantic Scholar has a 1 RPS rate limit. Searching for references across 5-6 H2 sections requires 20-30 API calls, taking 20-30 seconds minimum.
2. **Context bloat from ref files**: saving metadata + abstracts for 20-30 references adds significant content. If these ref files are loaded into context during body generation, they compound the repo content already in context (Pitfall 1).

**Why it happens:**
- The v1.0 literature-skill is designed for single-topic search (one query, 5-10 results, user selects one). Scaling it to automated multi-section reference collection is a different usage pattern.
- Semantic Scholar MCP rate limits are per-key: 1 RPS for new keys, shared pool for unauthenticated. Batch reference collection hits these limits quickly.
- The v2.0 design says references are "collected by H2 stage" but does not specify whether they are collected automatically or with user confirmation at each search. Automatic collection risks hallucinated search queries (Claude generates a search query that does not match the actual repo content, returning irrelevant papers).

**How to avoid:**
1. **Decouple literature collection from the generation pipeline**: literature collection runs as a separate pass after the H2 outline is confirmed but before H3/body generation begins. This prevents the rate limit from blocking the outline workflow.
2. **User-confirmed search queries**: for each H2 section, the Skill proposes 2-3 search queries and the user confirms or modifies them before execution. This prevents hallucinated search queries.
3. **Ref files are summaries, not full metadata**: each ref file contains only: title, authors, year, one-sentence abstract summary, and Semantic Scholar ID. Full abstracts are loaded on-demand only when writing the specific paragraph that cites the paper.
4. **Limit references per section**: cap at 3-5 references per H2 section. The Skill is producing a draft, not a literature review. Users add more references manually.
5. **Reuse the existing literature-skill rather than reimplementing**: the repo-to-paper Skill should invoke the literature-skill's search pattern (with MCP-only BibTeX) rather than building its own search logic. This preserves the anti-hallucination guarantees.

**Warning signs:**
- Repo-to-paper workflow takes 10+ minutes due to sequential API calls
- Generated paper cites papers that are irrelevant to the actual repo content
- Ref files are loaded into context during body generation, consuming 20K+ tokens
- Literature search is embedded directly in the repo-to-paper Skill rather than delegating to literature-skill

**Phase to address:**
Phase 1 (Repo-to-Paper Skill design), with the literature integration designed as a decoupled sub-phase.

---

### Pitfall 8: Bilingual Output Inconsistency Across Skills

**What goes wrong:**
The v2.0 bilingual paragraph-by-paragraph comparison feature is added to some Skills but not others, or is implemented with different formats across Skills. The user experiences:
- Polish-skill produces bilingual `.tex` with Chinese in LaTeX comments
- Reviewer-skill produces bilingual inline blockquotes
- Repo-to-paper produces bilingual `.md` with side-by-side format
- Abstract-skill has no bilingual output at all

This inconsistency confuses users and makes the suite feel uncoordinated.

**Why it happens:**
- The v1.0 codebase already has two bilingual patterns: (a) translation-skill's paragraph-by-paragraph `.tex` file with Chinese as comments, and (b) reviewer-skill/logic-skill's inline blockquote pattern. These patterns serve different purposes and adding a third format creates fragmentation.
- Different Skills have different output types (LaTeX, Markdown, conversation-only) which constrain how bilingual output can be formatted.
- If the bilingual feature is implemented Skill-by-Skill across multiple phases, each Skill author may make independent formatting decisions.

**How to avoid:**
1. **Define exactly two bilingual patterns in conventions and document when to use each**:
   - **Pattern A: Inline blockquote** -- for structured analysis output (concerns, issues, findings). Used by reviewer-skill, logic-skill, experiment-skill. Chinese blockquote follows each English block.
   - **Pattern B: Paragraph-by-paragraph file** -- for continuous generated text (translation, polished text, paper draft). Used by translation-skill, polish-skill, abstract-skill, repo-to-paper-skill. Chinese as comments in `.tex` or as alternating blocks in `.md`.
2. **Decide which Skills get bilingual output in v2.0**: not all 11 Skills need bilingual output. Visualization-skill (recommendation cards), caption-skill (LaTeX captions), cover-letter-skill (formal letter) do not benefit from bilingual comparison. Explicitly mark these as "English-only output" in the design.
3. **Implement bilingual output in one batch phase, not per-Skill**: adding bilingual output across multiple Skills should happen in a single phase with a consistent template, not spread across separate Skill updates.

**Warning signs:**
- Three or more distinct bilingual output formats exist across the Skill suite
- A Skill has bilingual output but the format does not match either Pattern A or Pattern B
- Users must learn different bilingual formats for different Skills
- Convention document does not specify which pattern to use for which Skill type

**Phase to address:**
Phase 2 (Bilingual Enhancement), as a single coordinated update. The convention defining the two patterns should be written in Phase 0 (Convention updates).

---

### Pitfall 9: Repo-to-Paper Generates Claims from Code Structure, Not from Actual Results

**What goes wrong:**
The Skill reads source code and infers what the experiment does from function names, variable names, and code structure -- but does not read actual output files (results CSVs, log files, saved metrics). The generated paper describes the methodology correctly but fabricates quantitative results because no actual result data was available or read.

This is distinct from Pitfall 2 (general hallucination). This pitfall is specifically about the gap between "what the code does" and "what the code produced."

**Why it happens:**
- Source code is easy to read with Claude Code's Read tool. Result files (.csv, .json, .pkl, .log) may be binary, large, or in formats Claude cannot parse.
- Researchers often do not commit result files to their repos. The repo may contain the training script but the actual metrics are in a Weights & Biases dashboard, a separate experiment tracker, or local files not in version control.
- Function names like `evaluate_model()` and variable names like `best_accuracy` strongly suggest what results should look like, tempting Claude to generate plausible numbers even without actual data.

**How to avoid:**
1. **Separate "what the code does" from "what the code produced"**: the Skill must explicitly distinguish between methodology description (derived from code) and results reporting (derived from output files). Methodology sections can be generated from code analysis. Results sections CANNOT -- they require actual output data.
2. **Probe for result files early**: in the initial repo scan, specifically look for common result file patterns (`results/`, `outputs/`, `logs/`, `*.csv`, `*.json`, `wandb/`, `mlruns/`). If none are found, inform the user: "No result files found in repository. Results and Discussion sections will contain `[RESULTS NEEDED: provide data file or paste metrics]` placeholders."
3. **Never infer quantitative results from code**: if a function is named `compute_accuracy()`, the Skill must NOT generate "The model achieved X% accuracy." It must find the actual output value in a result file or user-provided input.
4. **User input as results source**: allow the user to paste experimental results (tables, metrics, key numbers) as supplementary input alongside the repo. The Skill combines code analysis (for methodology) with user-provided results (for quantitative claims).

**Warning signs:**
- Generated Results section contains specific numbers without corresponding `[SOURCE: filename]` annotations
- The paper describes experiments that produce results, but no result files exist in the repo
- Generated accuracy/F1/loss values are suspiciously round numbers (e.g., 85.0%, 92.3%) suggesting fabrication
- The Skill does not ask whether result data is available before generating quantitative sections

**Phase to address:**
Phase 1 (Repo-to-Paper Skill design). The methodology-vs-results distinction must be architecturally enforced from day one.

---

## Minor Pitfalls

Mistakes that cause inconvenience or suboptimal results but are easily corrected.

---

### Pitfall 10: v1.0 Tech Debt Items Interact Unexpectedly with v2.0 Features

**What goes wrong:**
The three known v1.0 tech debt items create friction when implementing v2.0 features:
1. **`skill-conventions.md` lacks escape hatch documentation for `required: []`**: when the AskUserQuestion enforcement rule is added, Skills with `required: []` may be incorrectly flagged as non-compliant.
2. **`literature-skill` uses vendor-specific "Semantic Scholar MCP" in description instead of "External MCP"**: when repo-to-paper Skill integrates literature search, the inconsistency between tool naming conventions creates confusion about whether the tool reference should use the capability category or the vendor name.
3. **`cover-letter-skill` has `required: []` but CEUS is functionally mandatory**: this sets a bad precedent. If the repo-to-paper Skill has similar "technically optional but functionally required" references, the same ambiguity will recur.

**Why it happens:**
- These items were identified in the v1.0 audit but deferred to v2.0. They interact with v2.0 features in ways that were not anticipated at the time of the audit.

**How to avoid:**
1. **Fix all three tech debt items in Phase 0 before any new Skill work**: these are small, scoped changes that take 5-10 minutes each. Fixing them first prevents cascading confusion.
2. **Fix 1**: add the escape hatch documentation to `skill-conventions.md` line 73 area -- the text is already there but should be more prominent (a subsection heading, not an inline note).
3. **Fix 2**: update `literature-skill` frontmatter `tools` to use `External MCP` per conventions (it currently says `External MCP` already -- verified in code). If the description text still mentions "Semantic Scholar MCP" as a vendor name, update it to reference the capability category in the tools list while keeping the specific MCP name in the workflow steps where it is needed.
4. **Fix 3**: move `references/journals/ceus.md` from `leaf_hints` to `required` in cover-letter-skill frontmatter, since the Skill refuses to operate without it.

**Warning signs:**
- Tech debt items are still in the "Known issues" list after Phase 0 completes
- New Skills copy patterns from the unfixed v1.0 Skills, propagating the debt
- AskUserQuestion enforcement rule conflicts with `required: []` Skills

**Phase to address:**
Phase 0 (Tech debt resolution). Must complete before any new Skill authoring.

---

### Pitfall 11: Bilingual Output Added to Skills Where It Adds No Value

**What goes wrong:**
The bilingual feature is applied uniformly to all Skills, including those where Chinese translation adds no value or makes output worse:
- **Caption-skill**: LaTeX captions are English-only by journal requirement. Adding Chinese translation doubles the output for no purpose.
- **Cover-letter-skill**: cover letters are formal English documents. A bilingual version is not submitted with the paper.
- **Visualization-skill**: chart type recommendations are language-agnostic. Translating "Grouped Bar Chart" to Chinese adds nothing.
- **Literature-skill**: BibTeX entries are language-independent. Translating author names or journal titles into Chinese would be incorrect.

**Why it happens:**
- The v2.0 feature description says "bilingual paragraph-by-paragraph comparison across Skills" which implies all Skills. But "across" may mean "available in those Skills where it is useful" rather than "in every Skill."
- Scope creep: once the bilingual pattern is defined, it seems easy to add it everywhere. But each addition increases Skill complexity and output length.

**How to avoid:**
1. **Explicitly categorize Skills into bilingual-appropriate and English-only**:
   - Bilingual-appropriate: translation-skill (already has it), polish-skill, abstract-skill, experiment-skill, reviewer-skill (already has it), logic-skill (already has it), repo-to-paper-skill (new)
   - English-only: caption-skill, cover-letter-skill, visualization-skill, literature-skill, de-ai-skill
2. **Document the categorization in the design phase**: include a "Bilingual scope" table in the roadmap that explicitly lists which Skills get bilingual output and which do not.
3. **The user should be able to request bilingual output for any Skill**, but it should not be the default for Skills categorized as English-only.

**Warning signs:**
- Cover-letter-skill produces a bilingual cover letter
- BibTeX entries include Chinese translations of titles
- Caption-skill outputs Chinese captions alongside English LaTeX captions
- All 11 Skills have bilingual mode regardless of appropriateness

**Phase to address:**
Phase 2 (Bilingual Enhancement). The scope table should be defined in Phase 0 (Convention updates) so Phase 2 has a clear target list.

---

## Technical Debt Patterns

Shortcuts that seem reasonable but create long-term problems.

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| Embedding repo-scanning logic directly in SKILL.md instead of extracting to reference file | Keeps everything in one file during prototyping | Skill exceeds 300 lines; scanning logic cannot be reused by other Skills | Never -- extract scanning strategy to `references/repo-to-paper/scanning-strategy.md` from the start |
| Generating bilingual output in conversation instead of writing to file | Quicker to implement; no file I/O needed | Context consumed by doubled output; later paragraphs lose quality | Acceptable only for short outputs (< 500 words total bilingual) |
| Hardcoding Semantic Scholar search queries in repo-to-paper instead of delegating to literature-skill | Avoids cross-Skill coordination complexity | Duplicates MCP interaction logic; anti-hallucination rules may diverge | Never -- always delegate to literature-skill's existing verified pattern |
| Adding bilingual output to all 11 Skills in one sweep | Feels complete and consistent | 4+ Skills gain unnecessary complexity; maintenance burden doubles | Never -- categorize and scope first |
| Fixing AskUserQuestion with a global "MUST" rule | Quick fix, one line in conventions | Breaks direct-mode UX for power users; forces unnecessary interactions | Never -- audit per-Skill instead |

## Integration Gotchas

Common mistakes when connecting v2.0 features to the existing v1.0 Skill ecosystem.

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|------------------|
| Repo-to-paper -> literature-skill | Building a separate MCP integration instead of reusing literature-skill's search workflow | Define a "search delegation" pattern: repo-to-paper Skill specifies search queries, literature-skill executes them with its existing anti-hallucination guards |
| Bilingual output -> translation-skill | Inventing a new bilingual format when translation-skill already has a proven pattern (Chinese as LaTeX comments) | Adopt translation-skill's `_bilingual.tex` pattern as the standard for paragraph-by-paragraph comparison. Adapt for `.md` output where needed |
| AskUserQuestion fix -> all Skills | Applying the fix globally via conventions without per-Skill audit | Audit each Skill's Ask Strategy section individually; update only the specific questions that should use structured interaction |
| Repo-to-paper -> experiment-skill | Duplicating the findings extraction logic instead of reusing experiment-skill's two-phase pattern | Adopt experiment-skill's Phase 1 (extract findings) -> confirm -> Phase 2 (generate discussion) pattern for each quantitative section |
| Bilingual output -> reviewer-skill | Changing the existing working inline blockquote pattern to paragraph-by-paragraph | Keep the inline blockquote pattern for reviewer-skill and logic-skill; they are analysis Skills, not generation Skills |

## Performance Traps

Patterns that work at small scale but fail as repo/paper size grows.

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|----------------|
| Reading all repo files into a single context | Works for a 5-file demo repo | Two-pass scanning with file budget | Repos with 15+ files (~60K+ tokens of file content) |
| Generating full bilingual output in conversation | Works for a single paragraph | File-based output with progressive generation | Sections longer than ~1,500 words English (~3,000+ bilingual tokens) |
| Sequential Semantic Scholar searches during outline generation | Works for 1-2 searches | Decoupled literature collection pass | 5+ sections requiring references (~20+ API calls at 1 RPS) |
| Loading all ref files into context during body generation | Works when only 2-3 references cited | On-demand ref loading per paragraph | 10+ references collected across all sections |

## UX Pitfalls

Common user experience mistakes when adding these features.

| Pitfall | User Impact | Better Approach |
|---------|-------------|-----------------|
| Repo-to-paper asks user to confirm at every H3 subsection heading | "I invoked one Skill and had to answer 20 questions" | Checkpoint at H1 and H2 only. H3 and body are generated in one pass per H2 section |
| AskUserQuestion presents 5+ options for every question | Decision fatigue; user picks randomly | Maximum 4 options per question. Use "Other (type your answer)" for open-ended needs |
| Bilingual output is always on, cannot be disabled | "I just want English output, stop giving me Chinese too" | Opt-in bilingual mode. Default matches existing Skill behavior (English-only for most, bilingual for reviewer/logic) |
| Repo-to-paper generates a 30-page paper draft from a small repo | Output far exceeds what the user expected or wants | Estimate output length from repo size and ask user to confirm scope before generation begins |
| AskUserQuestion triggers on Skills that previously had zero questions | Power users lose speed; "this used to be faster" | Only add structured questions where they prevent meaningful errors. Preserve zero-question direct mode for well-specified triggers |

## "Looks Done But Isn't" Checklist

Things that appear complete but are missing critical pieces.

- [ ] **Repo-to-paper Skill:** Often missing context management strategy -- verify the Skill has explicit file budget, two-pass scanning, and intermediate file writing
- [ ] **Repo-to-paper Skill:** Often missing methodology-vs-results separation -- verify the Skill distinguishes claims from code analysis vs. claims from result data, with different evidence requirements for each
- [ ] **Bilingual output:** Often missing per-Skill categorization -- verify a scope table exists listing which Skills get bilingual output and which do not
- [ ] **Bilingual output:** Often missing output length management -- verify file-based output is used for sections exceeding ~1,500 words
- [ ] **AskUserQuestion fix:** Often missing per-Skill audit -- verify each Skill's Ask Strategy was individually reviewed, not just a global convention change
- [ ] **AskUserQuestion fix:** Often missing direct-mode preservation -- verify that Skills with sufficient trigger context still proceed without questions
- [ ] **Literature integration:** Often missing anti-hallucination delegation -- verify repo-to-paper delegates to literature-skill's MCP-only BibTeX pattern rather than building its own
- [ ] **Convention updates:** Often missing backward compatibility check -- verify the 5 most interactive existing Skills (polish, de-ai, reviewer, experiment, translation) still work as expected
- [ ] **v1.0 tech debt:** Often deferred again -- verify all three items (escape hatch docs, literature-skill tool name, cover-letter required field) are actually fixed in Phase 0

## Recovery Strategies

When pitfalls occur despite prevention, how to recover.

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|----------------|
| Context exhaustion during repo scan (P1) | MEDIUM | Redesign as two-pass scan with intermediate files. Existing generated content is salvageable; only the workflow needs restructuring |
| Hallucinated experiment results (P2) | HIGH | Must audit ALL generated quantitative claims against actual repo data. Add `[SOURCE:]` annotations retroactively. Consider regenerating Results/Discussion sections from scratch |
| Bilingual output truncation (P3) | LOW | Switch from conversation output to file-based output. Re-generate truncated sections |
| AskUserQuestion over-enforcement (P4) | LOW | Roll back convention change. Audit per-Skill. Re-apply targeted fixes |
| Skill exceeds 300-line budget (P5) | MEDIUM | Extract sub-components to reference files or decompose into coordinator + sub-Skills. Requires architectural refactoring |
| Regression in existing Skills (P6) | MEDIUM-HIGH | Identify which convention change caused the regression. Revert the specific change. Apply backward-compatible alternative. Higher cost if multiple Skills are affected simultaneously |
| Literature integration bottleneck (P7) | LOW | Decouple literature collection into a separate pass. Does not require regenerating content |
| Bilingual format inconsistency (P8) | MEDIUM | Define the two-pattern standard. Reformat affected Skills. Lower cost if caught before v2.0 ships |
| Claims from code not results (P9) | HIGH | Same as P2 -- must audit all quantitative claims. Add explicit methodology-vs-results guards to Skill |
| Tech debt interaction (P10) | LOW | Fix the three items. 15 minutes total work |
| Bilingual applied to wrong Skills (P11) | LOW | Remove bilingual output from inappropriate Skills. Minimal effort per Skill |

## Pitfall-to-Phase Mapping

How roadmap phases should address these pitfalls.

| Pitfall | Prevention Phase | Verification |
|---------|------------------|--------------|
| P1: Context exhaustion during repo scan | Phase 1 (Repo-to-Paper design) | Skill specifies file budget and two-pass strategy; intermediate files are written; test with a 30-file repo |
| P2: Hallucinated experiment results | Phase 1 (Repo-to-Paper design) | Every quantitative claim has `[SOURCE:]` annotation; outline validation checkpoint exists; `[UNVERIFIED]` flags present where evidence is missing |
| P3: Bilingual output doubling | Phase 0 (Convention) + Phase 2 (Bilingual) | Bilingual output written to file, not conversation, for sections > 1,500 words; chunked generation verified |
| P4: AskUserQuestion over-enforcement | Phase 0 (Convention + per-Skill audit) | Direct-mode invocation with full context in trigger proceeds without questions; `required: []` Skills unchanged |
| P5: Skill exceeds budget | Phase 1 (Repo-to-Paper design) | Architectural decision (coordinator vs. single Skill) documented; Skill stays within budget or has justified exception |
| P6: Regression in existing Skills | Phase 0 (Convention updates) | Regression checklist run against 5 most interactive Skills after convention change; no behavior change detected |
| P7: Literature integration bottleneck | Phase 1 (Repo-to-Paper design) | Literature collection decoupled from outline generation; search queries user-confirmed; rate limit not hit during normal workflow |
| P8: Bilingual format inconsistency | Phase 0 (Convention) + Phase 2 (Bilingual) | Convention defines exactly two patterns; each Skill uses one pattern; no third format exists |
| P9: Claims from code not results | Phase 1 (Repo-to-Paper design) | Skill distinguishes methodology sections (from code) vs. results sections (from data); `[RESULTS NEEDED]` placeholders used when data missing |
| P10: v1.0 tech debt interaction | Phase 0 (Tech debt) | All three items resolved; no "Known issues" remain in PROJECT.md after Phase 0 |
| P11: Bilingual applied to wrong Skills | Phase 0 (Convention) + Phase 2 (Bilingual) | Scope table exists; only categorized Skills receive bilingual output |

## Sources

### Context Window Management
- [Context windows - Claude API Docs](https://platform.claude.com/docs/en/build-with-claude/context-windows)
- [Claude Code 1M Context Window: What It Means (2026)](https://claudefa.st/blog/guide/mechanics/1m-context-ga)
- [How Claude Code Got Better by Protecting More Context](https://hyperdev.matsuoka.com/p/how-claude-code-got-better-by-protecting)
- [Claude Code's 200K Context Window Is Not Enough for Large Projects](https://aiproductivity.ai/news/claude-code-200k-context-window-limits/)

### Hallucination in Academic Paper Generation
- [GPTZero uncovers 50+ Hallucinations in ICLR 2026](https://gptzero.me/news/iclr-2026/)
- [Duke University: It's 2026. Why Are LLMs Still Hallucinating?](https://blogs.library.duke.edu/blog/2026/01/05/its-2026-why-are-llms-still-hallucinating/)
- [Lakera: LLM Hallucinations Guide (2026)](https://www.lakera.ai/blog/guide-to-hallucinations-in-large-language-models)
- [A Survey on Hallucination in Large Language Models (ACM)](https://dl.acm.org/doi/10.1145/3703155)

### AskUserQuestion and Structured Interaction
- [Claude Code AskUserQuestion Tool Guide (SmartScope)](https://smartscope.blog/en/generative-ai/claude/claude-code-askuserquestion-tool-guide/)
- [Claude Code's Ask User Question Tool: First Impressions (Torq Software)](https://torqsoftware.com/blog/2026/2026-01-14-claude-ask-user-question/)
- [Handle approvals and user input - Claude API Docs](https://platform.claude.com/docs/en/agent-sdk/user-input)

### Bilingual Output and Long-Context Generation
- [Shifting Long-Context LLMs Research from Input to Output](https://arxiv.org/pdf/2503.04723)
- [Science Across Languages: Assessing LLM Multilingual Translation of Scientific Papers](https://arxiv.org/html/2502.17882v1)
- [LLM-based Translation Inference with Iterative Bilingual Understanding](https://arxiv.org/html/2410.12543v2)

### Skills Regression and Testing
- [Improving skill-creator: Test, measure, and refine Agent Skills (Anthropic)](https://claude.com/blog/improving-skill-creator-test-measure-and-refine-agent-skills)
- [Claude Code Skills 2.0: Evals, Benchmarks and A/B Testing](https://www.pasqualepillitteri.it/en/news/341/claude-code-skills-2-0-evals-benchmarks-guide)

### Repository-Level Code Analysis
- [Paper2Code: Automating Code Generation from Scientific Papers](https://arxiv.org/html/2504.17192)
- [On the Impacts of Contexts on Repository-Level Code Generation](https://aclanthology.org/2025.findings-naacl.82.pdf)

### Project-Specific Sources
- v1.0 Retrospective (`.planning/RETROSPECTIVE.md`)
- v1.0 Pitfalls Research (`.planning/research/PITFALLS.md`, archived)
- All 11 SKILL.md files analyzed for current patterns
- `references/skill-conventions.md` for existing Ask Strategy and reference loading rules

---
*Pitfalls research for: Paper Polish Workflow v2.0 (Repo-to-Paper, Bilingual Output, AskUserQuestion Fix)*
*Researched: 2026-03-17*
