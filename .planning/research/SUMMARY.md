# Project Research Summary

**Project:** paper-polish-workflow v2.0
**Domain:** Claude Code Skill suite for academic paper writing — v2.0 extension
**Researched:** 2026-03-17
**Confidence:** HIGH

## Executive Summary

This project extends an established 11-Skill Claude Code suite for academic paper writing (Chinese-English researchers, targeting venues like NeurIPS, ICML, CEUS) with three new v2.0 capabilities: a repo-to-paper Skill that generates structured paper drafts from experiment code repositories, bilingual paragraph-by-paragraph comparison output standardized across relevant Skills, and a prompt engineering fix for broken AskUserQuestion usage in the legacy paper-polish-workflow. The v1.0 architecture — flat multi-Skill pattern, shared reference library, YAML-frontmatter SKILL.md format, Semantic Scholar MCP, ~300-line budget — is validated and unchanged. All v2.0 work is additive with zero breaking changes to existing Skills.

The recommended approach is a five-phase build ordered by dependency: (Phase 0) tech debt and convention fixes before any new authoring, (Phase 1) AskUserQuestion fix as a zero-risk P0 bug fix, (Phase 2) bilingual output standardization using the translation-skill's proven LaTeX comment pattern as the canonical model, (Phase 3) repo-to-paper Skill core structure and repo scanning, (Phase 4) literature integration at the H2 stage, and (Phase 5) body generation with expression patterns and bilingual output. The repo-to-paper Skill is the only new Skill; all other changes are modifications to existing files. Two new reference files are needed: `references/repo-patterns.md` (file pattern registry) and `references/bilingual-format.md` (bilingual output specification).

The primary risk is context window exhaustion during multi-file repo scanning combined with top-down generation, which can cause Claude to silently forget early files when writing body paragraphs. This is mitigated by a two-pass scanning strategy (directory structure first, selective deep reads just-in-time per section), a strict file budget of 10-15 files per session, and writing intermediate outputs (H1/H2 outlines, ref files) to disk between phases. A secondary risk is hallucinated experiment results — avoided by enforcing evidence-first generation for quantitative sections, mandatory `[SOURCE: filename:line]` annotations on all factual claims, and explicit placeholders (`[RESULTS NEEDED]`) when no result files are found in the repo.

---

## Key Findings

### Recommended Stack

The v1.0 stack is unchanged and carries forward entirely. No new tools, packages, or external services are added. The v2.0 repo-to-paper feature uses only existing Claude Code built-in tools (Glob, Grep, Read, Write) in a defined scanning sequence — Glob for directory discovery, Grep for pattern extraction (model names, metric values, API surface), and Read for selective deep reads of key files. The Semantic Scholar MCP integration is extended from the existing literature-skill pattern, with automated multi-section batch queries added for the H2 literature collection phase.

**Core technologies (all v1.0, unchanged):**
- SKILL.md with YAML frontmatter: authoring format, skill identification, tool declarations
- Shared `references/` library: expression patterns, anti-AI patterns, journal templates — loadable knowledge base
- Semantic Scholar MCP (`mcp__semantic-scholar__papers-search-basic`, `get-paper-abstract`): citation retrieval, anti-hallucination guardrail
- AskUserQuestion tool: structured multi-option interaction at user checkpoints
- Claude Code built-in tools (Read, Write, Edit, Glob, Grep): all file operations, no npm packages needed

**New reference files (v2.0 additions):**
- `references/repo-patterns.md`: file pattern registry mapping repo file types to paper sections (40-80 lines)
- `references/bilingual-format.md`: bilingual output specification defining Variant A (LaTeX comment) and Variant B (Markdown blockquote)

### Expected Features

**Must have — table stakes for v2.0 launch:**
- AskUserQuestion fix in paper-polish-workflow (P0 bug: `mcp_question` does not exist; actual tool is `AskUserQuestion`) — fixes broken interactive UX across the entire workflow
- Bilingual output reference file defining the two standardized patterns — foundation for all bilingual implementation
- Bilingual paragraph-by-paragraph mode — user-requested, patterns already exist in v1.0
- Repo scan with progressive file discovery — foundation for entire repo-to-paper feature
- Top-down H1/H2/H3 generation with user checkpoints at each level — core UX promise
- Literature integration at H2 stage with ref files (title, abstract summary, BibTeX, relevance note)
- Body generation using expression patterns, evidence-first rule, and anti-hallucination placeholders

**Should have — competitive differentiators:**
- Code-aware paper structure (outline derived from actual codebase, not from a topic prompt)
- Ref files with full abstracts for anti-hallucination in Related Work body generation
- Evidence-before-interpretation enforced throughout Results/Discussion sections
- Anti-hallucination placeholders: `[CITATION NEEDED]`, `[EXACT VALUE: metric]`, `[VERIFY: claim]`, `[SOURCE: filename:line]`

**Defer to v2.x:**
- Retrofit bilingual mode to existing v1.0 Skills (polish-skill, abstract-skill, experiment-skill)
- Figure inventory sidebar: list repo image files, user selects which to include
- Iterative revision mode: re-enter generation at H1/H2/body without full regeneration

**Defer to v3+:**
- Multi-repo paper support
- Non-Chinese bilingual pairs (Japanese-English, Korean-English)
- Paper-to-repo reverse direction

**Anti-features (explicitly excluded):**
- One-click full paper generation with no checkpoints — produces hallucinated content
- Auto-formatting for conference LaTeX templates — changes yearly, out of scope
- Real-time bilingual co-editing — creates version conflicts; Chinese is a review aid, not co-authored source
- Bidirectional paper-to-code sync — different lifecycles, enormous complexity

### Architecture Approach

The architecture is a single new Skill (`repo-to-paper-skill`) added to the flat multi-Skill pattern, with targeted modifications to two existing files (paper-polish-workflow/SKILL.md tool name fix, polish-skill/SKILL.md bilingual output option). The reference library is unchanged. No structural changes to v1.0 components. Generated literature metadata goes to a `.paper-refs/` directory in the user's working context (not the shared `references/` library). The repo-to-paper Skill is a standalone orchestrator, not a Skill-chaining coordinator, because Claude Code does not reliably support Skill-calling-Skill.

**Major components:**
1. `repo-to-paper-skill/SKILL.md` — new Skill: six-step workflow (MCP preflight, repo scan, H1 checkpoint, H2+literature checkpoint, H3 checkpoint, body generation + output)
2. `paper-polish-workflow/SKILL.md` — modified: replace `mcp_question`/`mcp_read`/`mcp_write` with correct Claude Code tool names
3. `polish-skill/SKILL.md` — modified: add optional bilingual before-vs-after comparison output mode
4. `translation-skill/SKILL.md` — verify: confirm existing bilingual pattern matches v2.0 standard (likely no changes needed)
5. `.paper-refs/` directory convention — new output location for per-paper literature metadata files; add to `.gitignore`

### Critical Pitfalls

1. **Context window exhaustion during repo scanning** — mitigate with two-pass scan (Glob-only first pass for structure, selective Read second pass just-in-time per section), strict 10-15 file budget, and writing H1/H2 outlines plus repo summaries to disk as intermediate files so body generation starts in a fresh context with only relevant files loaded.

2. **Hallucinated experiment results and unsupported claims** — mitigate with evidence-first generation (extract and confirm evidence table before writing Results), mandatory `[SOURCE: filename:line]` annotations on all quantitative claims, and hard separation between "what code does" (from code analysis) and "what code produced" (requires actual result files or user-pasted metrics). If no result files are found in repo scan, insert `[RESULTS NEEDED]` placeholders rather than generating plausible numbers.

3. **Bilingual output doubling context consumption** — mitigate by (a) writing bilingual output to a separate file (`_bilingual.tex` or `_bilingual.md`), not into conversation, (b) generating paragraph-by-paragraph progressively, and (c) making bilingual an opt-in mode flag, not the default.

4. **AskUserQuestion over-enforcement breaking direct mode** — the fix must be a per-Skill audit identifying which specific questions benefit from structured interaction, not a global "MUST" mandate. Preserve the direct-mode exemption: if the user provides sufficient context in the trigger, the Skill proceeds without pre-questions. Multi-option questions (journal, mode, scope) are good AskUserQuestion candidates; open-ended questions (research question, data description) should stay freeform.

5. **Repo-to-paper SKILL.md exceeding the ~300-line budget** — estimated at ~430 lines including all workflow phases. Mitigate by extracting scanning strategy heuristics to `references/repo-patterns.md` (~80 lines saved) and bilingual output rules to the shared bilingual reference file (~30 lines saved), targeting ~320 lines. Do not sacrifice anti-hallucination rules to meet the budget; a 320-line Skill with robust guards is correct; a 300-line Skill with vague hallucination rules is not.

---

## Implications for Roadmap

Based on combined research, the architecture research's suggested five-phase build order is well-justified and should be adopted directly. Each phase has verified dependencies and clear acceptance criteria.

### Phase 0: Tech Debt and Convention Foundation
**Rationale:** Convention changes are high-leverage and high-risk — they propagate to all 11 existing Skills. Resolving v1.0 tech debt and adding AskUserQuestion guidance to conventions before any new authoring prevents regression and cascading confusion in later phases.
**Delivers:** Updated `skill-conventions.md` with AskUserQuestion usage rules (per-Skill audit approach, not blanket mandate), `required: []` escape hatch documentation prominently marked, and bilingual scope categorization (7 bilingual-appropriate Skills vs. 4 English-only Skills); fixed `literature-skill` and `cover-letter-skill` frontmatter tech debt items.
**Addresses:** Pitfall 4 (AskUserQuestion over-enforcement), Pitfall 6 (convention regression in existing Skills), Pitfall 10 (v1.0 tech debt interactions), Pitfall 11 (bilingual added to inappropriate Skills)
**Avoids:** Blanket "MUST use AskUserQuestion" rule that would break direct mode for power users

### Phase 1: AskUserQuestion Fix
**Rationale:** P0 bug fix — the current paper-polish-workflow references `mcp_question` (a non-existent tool), breaking the entire interactive UX. Zero dependencies, zero risk, ~30 minutes of work.
**Delivers:** paper-polish-workflow/SKILL.md with correct tool names (`AskUserQuestion`, `Read`, `Write`, `Edit`) and explicit AskUserQuestion invocation examples matching the actual tool schema (questions[1-4], options[2-4], header max 12 chars)
**Addresses:** TS-5 (AskUserQuestion correct usage)
**Avoids:** Full rewrite of the legacy SKILL.md — preserve the existing sentence-by-sentence confirmation workflow philosophy; fix only tool names

### Phase 2: Bilingual Pattern Standardization
**Rationale:** Establishes the canonical bilingual format before the repo-to-paper Skill needs it. Adding bilingual to polish-skill (simpler) validates the approach at low risk before implementing it in the complex new Skill. Must be done in a single coordinated batch to prevent inconsistent formats across the suite.
**Delivers:** `references/bilingual-format.md` defining Variant A (LaTeX comment, for body text output) and Variant B (Markdown blockquote, for analysis output); polish-skill extended with opt-in before-vs-after bilingual comparison; translation-skill bilingual pattern verified as canonical
**Addresses:** TS-4 (bilingual paragraph-by-paragraph output), D-4 (bilingual as cross-Skill capability)
**Avoids:** Pitfall 3 (context doubling) via file-based output and opt-in mode flag; Pitfall 8 (format inconsistency) via single-batch coordinated implementation

### Phase 3: Repo-to-Paper Core Structure
**Rationale:** Core Skill structure (scanning, H1/H2/H3 outline generation with user checkpoints) can be built and tested independently before literature integration adds external API complexity.
**Delivers:** `.claude/skills/repo-to-paper-skill/SKILL.md` with frontmatter, repo scan strategy (progressive discovery, 10-15 file budget, structured summaries), H1/H2/H3 checkpoint workflow using AskUserQuestion, and `references/repo-patterns.md` file pattern registry
**Addresses:** TS-1 (repo scan), TS-2 (top-down generation), D-1 (code-aware paper structure)
**Avoids:** Pitfall 1 (context exhaustion) via two-pass scan and intermediate file writes; Pitfall 5 (budget overage) via reference file extraction; Pitfall 9 (code vs. results gap) via early probe for result files and methodology-vs-results distinction

### Phase 4: Literature Integration
**Rationale:** Literature integration depends on stable H2 section structure from Phase 3. Separating it into its own phase allows testing structure generation without external API failure modes.
**Delivers:** Semantic Scholar MCP batch search at H2 stage with user-confirmed search queries per section; `.paper-refs/` directory convention and ref file format; graceful degradation to `[CITATION NEEDED]` placeholders when MCP is unavailable
**Addresses:** TS-3 (literature integration at H2), D-2 (ref files with metadata)
**Avoids:** Pitfall 7 (literature bottleneck) via decoupled literature collection pass, capped 3-5 refs per section, and on-demand abstract loading per paragraph during body generation

### Phase 5: Body Generation and Bilingual Output
**Rationale:** Body generation consumes outputs from all previous phases — structure (Phase 3), literature refs (Phase 4), bilingual pattern (Phase 2), expression patterns and anti-AI patterns (v1.0 shared library). It is the final assembly step.
**Delivers:** Full body text generation per confirmed H2 subsection using section-appropriate expression-pattern leaves; evidence-before-interpretation enforcement for Results/Discussion; anti-AI vocabulary screening; bilingual output (Pattern A, LaTeX comments) for draft; all output files written to disk (`{topic}_draft.tex`, `{topic}_draft_bilingual.tex`, `.paper-refs/*.md`)
**Addresses:** D-3 (evidence-before-interpretation), D-5 (anti-hallucination placeholders throughout)
**Avoids:** Pitfall 2 (hallucinated results) via evidence table confirmation before quantitative section generation and `[SOURCE: filename:line]` annotation requirement on all factual claims

### Phase Ordering Rationale

- Phase 0 is a hard prerequisite: convention changes propagate to all 11 existing Skills, and making them after new Skill authoring begins creates regression risk.
- Phase 1 (AskUserQuestion fix) is independent of all other phases and can complete in a single focused session; completing it early removes the highest-urgency user-reported problem.
- Phase 2 (bilingual) must precede Phase 3 (repo-to-paper core) because repo-to-paper needs the bilingual format for its output. Validating the pattern on the existing polish-skill first is lower-risk than implementing it first in a new Skill.
- Phase 3 (structure) must precede Phase 4 (literature) because H2 section titles must exist before literature search queries can be derived from them.
- Phase 5 (body generation) is last because it consumes all prior outputs and has the highest failure risk (hallucination in quantitative claims) — a clean tested foundation is essential.

### Research Flags

Phases with standard, well-documented patterns (skip `/gsd:research-phase` during planning):
- **Phase 0:** Convention file edits; all patterns are established in v1.0 skill-conventions.md and do not require new research
- **Phase 1:** Tool name replacement; correct AskUserQuestion schema is fully documented and verified from official sources
- **Phase 2:** Translation-skill's bilingual pattern is the canonical model; only standardization work is needed, not discovery

Phases likely needing deeper research or empirical validation during implementation:
- **Phase 3:** Repo scan heuristics for non-Python repos are LOW confidence. The file pattern registry is calibrated for Python ML projects. Validate against 2-3 real user repos before finalizing.
- **Phase 4:** Semantic Scholar rate limit behavior under batch queries (20-30 calls per paper) needs empirical testing in a real session before finalizing the decoupled collection pass design.
- **Phase 5:** Body generation quality across diverse repo types (NLP vs. CV vs. urban computing vs. non-ML) is untested. Evidence-first discipline and placeholder enforcement may need iterative tuning based on first real-world outputs.

---

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | All v2.0 tools are v1.0 tools already validated in production. Only 2 new reference files with well-defined content. No new external dependencies. |
| Features | MEDIUM-HIGH | Table stakes and differentiators are well-grounded in existing codebase patterns. LOW confidence items: repo scanning heuristics for non-ML repos, actual SKILL.md line count after drafting. |
| Architecture | HIGH | All integration points verified against actual codebase by direct file reading. Single new Skill, targeted modifications to 2 existing files, zero structural changes to v1.0 architecture. |
| Pitfalls | HIGH | Grounded in analysis of all 11 existing Skills plus v1.0 retrospective lessons. Context management and hallucination patterns supported by external research on Claude Code behavior. |

**Overall confidence:** HIGH

### Gaps to Address

- **Repo scan heuristics for non-Python repos:** the file pattern registry is calibrated for Python ML projects. If early users have R, Julia, or non-ML repos (e.g., urban computing simulation in Fortran), scanning priorities may need adjustment. Handle by: including an explicit user-correction checkpoint at H1 ("does this structure match your repo's actual organization?") and making the pattern registry an editable reference file rather than hardcoded Skill logic.

- **SKILL.md line count for repo-to-paper:** estimated at ~430 lines before extraction, ~320 after extracting to reference files. Actual count depends on how much prose is needed for anti-hallucination guards. Handle during implementation: write the Skill fully first, then extract to reference files. Do not pre-compromise anti-hallucination rules to reach budget.

- **AskUserQuestion enforcement effectiveness:** the fix approach (explicit invocation examples in each Skill's Ask Strategy + updated convention rule) is theoretically sound but untested. Claude's actual behavior change requires empirical verification after Phase 1 is deployed. Flag as a validation step in Phase 1 acceptance testing.

- **Bilingual output quality for generated Chinese in repo-to-paper:** in repo-to-paper, the Chinese lines are parallel-generated comprehension summaries, not translations of a Chinese source. Quality expectations differ from translation-skill's bilingual output. Clearly label in output file header and validate with a test paper in Phase 5.

---

## Sources

### Primary (HIGH confidence)
- Existing codebase — all 11 SKILL.md files, skill-conventions.md, skill-skeleton.md, full references/ library, paper-polish-workflow/SKILL.md (direct reading, all integration points verified)
- v1.0 STACK.md, FEATURES.md, ARCHITECTURE.md, PITFALLS.md, MILESTONE-AUDIT.md — validated stack decisions and retrospective lessons
- Claude Code AskUserQuestion tool description (github.com/Piebald-AI/claude-code-system-prompts) — exact tool schema, confirmed interface parameters
- Claude Code internal tools implementation (gist.github.com/bgauryy) — AskUserQuestion: questions[1-4], options[2-4], header max 12 chars

### Secondary (MEDIUM confidence)
- Anthropic Building Effective Agents — prompt chaining pattern: outline-then-body as standard LLM workflow
- PaperCoder / Paper2Code (arXiv:2504.17192) — multi-agent three-stage planning->analysis->generation; v2.0 does the code->paper inverse
- Competitor analysis (Gatsbi, Paperguide, SciSpace) — all topic-based, no code-aware structure, no bilingual output; confirms v2.0 differentiation
- AskUserQuestion community usage guides (atcyrus.com, torqsoftware.com, egghead.io) — real-world behavior patterns and timeout behavior

### Tertiary (LOW confidence, needs validation)
- Repo file categorization heuristics — based on Python ML project conventions; non-Python or non-ML repos may need different patterns
- 430-line SKILL.md estimate — section-by-section estimation; actual count will vary during implementation
- Semantic Scholar batch query performance — documented 1 RPS limit; real behavior at 20-30 call scale untested

---
*Research completed: 2026-03-17*
*Ready for roadmap: yes*
