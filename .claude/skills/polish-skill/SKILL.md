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
    - "帮我润色这段摘要"
    - "Polish all sections in my paper"
tools:
  - Read
  - Edit
  - Structured Interaction
references:
  required:
    - references/expression-patterns.md
    - references/bilingual-output.md
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
  - bilingual_conversation
---

## Purpose

This Skill polishes English academic text for journal submission through two modes: Quick-fix (default, intelligent single-pass) and Guided (fixed three-step: structure, logic, expression). For file input, it edits the original file in-place using the Edit tool and preserves originals as LaTeX comment annotations for traceability. For pasted text, polished output is presented directly in conversation. The Skill adapts to journal-specific style when a target journal is specified, detects translationese automatically, and avoids high-frequency AI vocabulary by loading anti-AI patterns proactively.

## Trigger

**Activates when the user asks to:**
- Polish, improve, or refine English academic text
- 润色、改善、优化英文学术文本

**Example invocations:**
- "Polish this paragraph for CEUS" / "润色这段英文"
- "Guided polish my methods section" / "引导模式润色方法部分"
- "Quick fix my introduction" / "帮我快速润色引言"
- "Polish all sections in my paper"

## Modes

| Mode | Default | Behavior |
|------|---------|----------|
| `direct` (Quick-fix) | Yes | Intelligent single-pass polish, minimal interaction |
| `guided` | | Three-step flow: structure, logic, expression with checklist at each step |
| `batch` | | Quick-fix across multiple sections/files with same settings |

**Default mode:** `direct` (Quick-fix). User says "polish this" and gets polished output.

**Mode inference:** "guided polish", "step by step", or "引导模式" switches to `guided`. "Polish all sections" or "batch" switches to `batch`.

## References

### Required (always loaded)

| File | Purpose |
|------|---------|
| `references/expression-patterns.md` | Academic expression patterns overview and module index |

### Leaf Hints (loaded when needed)

| File | When to Load |
|------|--------------|
| `references/expression-patterns/introduction-and-gap.md` | Polishing introduction or background content |
| `references/expression-patterns/methods-and-data.md` | Polishing methods, data, or study area content |
| `references/expression-patterns/results-and-discussion.md` | Polishing results or discussion content |
| `references/expression-patterns/conclusions-and-claims.md` | Polishing conclusion content |
| `references/expression-patterns/geography-domain.md` | Content involves spatial, urban, or planning topics |
| `references/anti-ai-patterns/vocabulary.md` | Always -- loaded proactively for vocabulary screening |
| `references/anti-ai-patterns/sentence-patterns.md` | Always -- loaded proactively for sentence pattern screening |
| `references/anti-ai-patterns/transitions-and-tone.md` | Always -- loaded proactively for transition screening |

### Loading Rules

- Load expression patterns overview at the start; select the appropriate leaf based on section type.
- Load ALL three anti-AI pattern leaves proactively (vocabulary, sentence-patterns, transitions-and-tone).
- When a target journal is specified, also load `references/journals/[journal].md`.
- Load `geography-domain.md` when spatial, urban, or planning content is detected.
- If a reference file is missing, warn the user and proceed with reduced capability (except journal templates; see Fallbacks).

## Ask Strategy

**Before starting, ask about:**
1. Target journal (if not specified in trigger) -- determines style loading
2. Which section this text belongs to (only if ambiguous from content or headers)
3. In guided mode only: confirm scope before starting

**Rules:**
- In `direct` (Quick-fix) mode, skip pre-questions when the user provides enough context.
- If the user specifies a journal with no template at `references/journals/[journal].md`, **refuse** and instruct the user to add a template.
- Never ask more than 3 questions before producing output.
- Use Structured Interaction when available; fall back to plain-text questions otherwise.

## Workflow

### Step 0: Workflow Memory Check

- Read `.planning/workflow-memory.json`. If file missing or empty, skip to Step 1.
- Check if the last 1-2 log entries form a recognized pattern with `polish-skill` that has appeared >= threshold times in the log. See `skill-conventions.md > Workflow Memory > Pattern Detection` for the full algorithm.
- If a pattern is found, present recommendation via AskUserQuestion:
  - Question: "检测到常用流程：[pattern]（已出现 N 次）。是否直接以 direct 模式运行 polish-skill？"
  - Options: "Yes, proceed" / "No, continue normally"
- If user accepts: set mode to `direct`, skip Ask Strategy questions.
- If user declines or AskUserQuestion unavailable: continue in normal mode.

### Quick-fix (Direct) Mode

**Step 1 -- Collect Context:**
- Determine input type: file path (use Edit tool) or pasted text (output in conversation).
- Load references: expression pattern leaf by section type, all anti-AI pattern leaves, journal template if specified.
- Detect input characteristics: translationese presence, section type, text length for smart adaptation.
- **Opt-out check:** Scan the user's trigger prompt for any of these phrases (case-insensitive, exact phrase match): `english only`, `no bilingual`, `only english`, `不要中文`. Store result as `bilingual_mode` (true/false). This flag governs Step 5 bilingual output below.
- **Record workflow:** Append `{"skill": "polish-skill", "ts": "<ISO timestamp>"}` to `.planning/workflow-memory.json`. Create file as `[]` if missing. Drop oldest entry if log length >= 50.

**Step 2 -- Polish:**
- Single intelligent pass covering expression, logic, and structure as needed based on text quality.
- Priority: expression issues first, then logic coherence, then structural adjustments.
- Apply journal style preferences from loaded template.
- Avoid anti-AI vocabulary; flag and rewrite AI-sounding phrases.
- Pay special attention to translationese if detected (literal structures, calques, excessive "of" constructions).
- For file input: use Edit tool; add `% [Polish] Original: <original text>` annotation before each modification.
- For pasted text: present polished output directly in conversation with key changes highlighted.

**Step 3 -- Smart Adaptation:**
- Short text (up to ~3 paragraphs): polish entirely in one pass.
- Long text (4+ paragraphs or multiple `\section{}` markers): split by section, process sequentially, maintain terminology consistency across sections.

**Step 4 -- Summary:**
- Generate concise report: change count, modification types (expression/logic/structure), notes.
- Include "Recommend running De-AI Skill for further detection check" when substantial rewrites were made.

**Step 5 -- Bilingual Display (file input only):**
- If `bilingual_mode` is true and input was a file: for each paragraph that was modified in Step 2, display a `> **[Chinese]** ...` blockquote in conversation showing the Chinese translation of the polished English text.
- Use a section header in conversation: "**双语对照 / Bilingual Comparison:**" before the first blockquote.
- Format per paragraph:

  > **[Chinese]** [Chinese translation of the polished paragraph]

- Do not insert Chinese into the .tex file. The file remains English-only and submission-ready.
- If `bilingual_mode` is false (opt-out detected): skip this step entirely.
- Pasted text input: if `bilingual_mode` is true, append the `> **[Chinese]** ...` blockquote immediately after each polished paragraph in the conversation output.

### Guided Mode

**Step 1 -- Collect Context:** Same as Quick-fix Step 1, plus confirm scope with user.

**Steps 2-4 follow the same pattern:** Analyze current text, present numbered checklist of problems found, user selects which to fix ("fix 1, 3" or "all"), apply selected fixes in batch using Edit tool with annotations, report changes. Re-read the file before each step to see the result of previous edits.

| Step | Focus | Key Checks |
|------|-------|------------|
| 2. Structure | Paragraph organization, topic sentences, section flow | Redundancy, ordering, paragraph purpose, section transitions |
| 3. Logic | Argument coherence, claim support, terminology consistency | Argument chains, evidence links; cross-section coherence when multi-section input |
| 4. Expression | Word choice, sentence clarity, tone, conciseness | Anti-AI patterns, translationese, academic register, hedging calibration |

**Step 5 -- Summary:** Same as Quick-fix Step 4.

**Step 6 -- Bilingual Display:** Same as Quick-fix Step 5. If `bilingual_mode` is true, display `> **[Chinese]** ...` blockquotes in conversation for each modified paragraph. If false, skip entirely.

## LaTeX Annotation Format

- Format: `% [Polish] Original: <original text>` on the line immediately before the replacement text.
- Multi-line originals: each line gets its own `% [Polish] Original:` prefix.
- Annotations are valid LaTeX comments -- the file still compiles with them present.
- Cleanup: after user confirms acceptance, remove all lines matching `^% \[Polish\] Original:` pattern.
- If existing `% [Polish] Original:` annotations are found, clean them up before adding new ones.

## Output Contract

| Output | Format | Condition |
|--------|--------|-----------|
| `polished_text` | In-place edits (file) or conversation output (pasted text) | Always produced |
| `change_annotations` | LaTeX comments (`% [Polish] Original:`) | File input only |
| `summary_report` | Markdown in session (not in file) | Always produced |
| `bilingual_conversation` | `> **[Chinese]** ...` blockquotes in session | File input: modified paragraphs only. Pasted text: after each output paragraph. Skipped when opt-out detected. |

## Edge Cases

| Situation | Handling |
|-----------|----------|
| Input too short (single sentence) | Polish but warn limited context for structural assessment |
| No section headers in input | Default to expression-focused polishing |
| Pasted text input | Output directly in conversation; no file operations |
| File input | Use Edit tool only; never Write (do not overwrite entire file) |
| Journal specified but no template | Refuse; instruct user to add template at `references/journals/[journal].md` |
| Very long input (10+ sections) | Process in batches; maintain cross-section terminology awareness |
| Existing `% [Polish] Original:` annotations | Clean up old annotations before adding new ones |
| Mixed LaTeX and text | Preserve all LaTeX commands; polish surrounding natural language only |
| Text with obvious translationese | Flag in summary; prioritize translationese fixes in expression pass |

## Fallbacks

| Scenario | Fallback |
|----------|----------|
| Structured Interaction unavailable | Ask 1-3 plain-text questions covering highest-impact gaps |
| Expression pattern leaf missing | Use overview entrypoint for general patterns |
| Journal template missing for specified journal | **Refuse** -- instruct user to add the template first |
| Anti-AI patterns file missing | Proceed without; rely on Claude's vocabulary awareness |
| File is read-only or Edit fails | Present changes as a diff in conversation; user applies manually |

---

*Skill: polish-skill*
*Conventions: references/skill-conventions.md*
