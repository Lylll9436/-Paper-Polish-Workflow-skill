# Skill Skeleton

A copyable starting point for new Claude Code Skills in this project.

Copy this file, rename it to `SKILL.md` inside your new Skill directory, and fill in each section. See [`references/skill-conventions.md`](skill-conventions.md) for the full authoring rules.

---

```yaml
---
name: example-skill
description: Brief description with trigger keywords. Include both English and Chinese phrases when applicable
triggers:
  primary_intent: what the skill does in one phrase
  examples:
    - "English trigger phrase"
    - "Chinese trigger phrase"
tools:
  - Read
  - Write
  - Structured Interaction
references:
  required:
    - references/expression-patterns.md
    - references/journals/ceus.md
  leaf_hints:
    - references/expression-patterns/results-and-discussion.md
input_modes:
  - file
  - pasted_text
output_contract:
  - polished_english
  - latex
---
```

## Purpose

State what the Skill does in one paragraph. Include who the Skill serves and what the output is used for.

> Example: This Skill translates Chinese academic drafts into polished English text ready for journal submission. It produces LaTeX-formatted output that preserves technical terminology and follows the target journal's style preferences.

## Trigger

Describe when the Skill activates and list example invocations.

**Activates when the user asks to:**
- [English trigger description]
- [Chinese trigger description]

**Example invocations:**
- "Help me [action] my [target]"
- "[Chinese equivalent]"

## Modes

| Mode | Default | Behavior |
|------|---------|----------|
| `interactive` | | Full step-by-step with confirmation at each decision |
| `guided` | Yes | Multi-pass with confirmation at key checkpoints |
| `direct` | | Single-pass output, minimal interaction |
| `batch` | | Same operation across multiple inputs |

**Default mode:** `guided`

**Mode inference:** If the user says "quickly" or "just fix", switch to `direct`. If the user says "step by step" or "help me revise", use `interactive`.

## References

### Required (always loaded)

| File | Purpose |
|------|---------|
| `references/expression-patterns.md` | Academic expression patterns overview |
| `references/journals/ceus.md` | CEUS journal contract (when targeting CEUS) |

### Leaf Hints (loaded when needed)

| File | When to Load |
|------|--------------|
| `references/expression-patterns/results-and-discussion.md` | When working on results or discussion sections |

### Loading Rules

- Load required references at the start of the workflow.
- Load leaf files only when the current task matches their scope.
- If a reference file is missing, warn the user and proceed with reduced capability.
- If the loaded reference does not match the current task, ask the user for clarification.

## Ask Strategy

**Before starting, ask about:**
1. Target journal (if not already known)
2. Which section or scope to work on
3. Preferred interaction mode (if ambiguous from trigger)

**Rules:**
- Never ask more than 3 questions before producing initial output.
- In `direct` mode, skip pre-questions if the user provided enough context.
- Use structured interaction when available; fall back to plain-text questions otherwise.
- See `skill-conventions.md > AskUserQuestion Enforcement` for when to use `AskUserQuestion` vs plain text.

## Workflow

### Step 1: Collect Context

- Load required references.
- Identify target journal and apply journal-specific constraints.
- Read the user's input (file or pasted text).

### Step 2: [Primary Action]

- [Describe the main operation the Skill performs]
- [Present intermediate output for user review in `interactive` and `guided` modes]

### Step 3: [Refinement]

- [Describe any refinement, checking, or polishing step]
- [In `direct` mode, this step runs automatically without user confirmation]

### Step 4: Output

- Present the final result.
- Write to file after user confirmation (or automatically in `batch` mode).
- Report word count and any journal constraint violations.

## Output Contract

| Output | Format | Condition |
|--------|--------|-----------|
| Polished text | Markdown or LaTeX | Always produced |
| Word count | Integer | Always reported |
| Journal compliance notes | Bullet list | When a target journal is specified |

> **Bilingual eligibility:** If this Skill's `output_contract` produces academic text, it must support bilingual paragraph-by-paragraph comparison (ON by default). See `skill-conventions.md > Bilingual Output Eligibility` for the full classification.

## Edge Cases

| Situation | Handling |
|-----------|----------|
| Input is too short to analyze | Warn the user and offer to proceed with limited context |
| Input language does not match expected language | Ask the user to confirm before proceeding |
| Journal not in `references/journals/` | Proceed with general academic style; inform the user |
| Multiple sections provided but scope unclear | Ask which section to process first |

## Fallbacks

| Scenario | Fallback |
|----------|----------|
| Structured interaction unavailable | Ask 1-3 plain-text questions covering the highest-impact gaps |
| Reference file missing | Log the missing file, proceed with reduced capability, warn the user |
| Target journal not specified | Ask once; if declined, use general academic style |
| PDF analysis tool unavailable | Ask the user to paste relevant text instead of providing a PDF path |

---

*Skeleton: references/skill-skeleton.md*
*Conventions: [references/skill-conventions.md](skill-conventions.md)*
