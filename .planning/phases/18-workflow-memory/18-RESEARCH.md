# Phase 18: Workflow Memory - Research

**Researched:** 2026-03-18
**Domain:** Claude Code Skill workflow instrumentation, user behavior pattern detection, JSON file-based state management
**Confidence:** HIGH

## Summary

Phase 18 adds two capabilities uniformly to all 12 existing Skills: (1) recording each invocation to a shared history file `.planning/workflow-memory.json`, and (2) detecting frequent sequences (2-3 consecutive Skills appearing 5+ times in the last 50 entries) and offering recommendations via `AskUserQuestion` at the start of the next Skill invocation. The user decisions in CONTEXT.md are highly prescriptive, leaving minimal discretion areas.

The implementation is entirely within Markdown SKILL.md files and two JSON config files. There is no executable code, no library dependencies, and no build system. Every change is a text edit to an existing file or creation of a new JSON file. The primary risk is the line budget: adding ~15-20 lines of workflow memory steps to each of the 12 Skills, some of which are already near the 300-line budget.

**Primary recommendation:** Structure the implementation as three waves: (1) create the config and empty history file + update skill-conventions.md and skill-skeleton.md, (2) add the recording step and Step 0 pattern detection to all 12 Skills, (3) verify end-to-end by reading the final state of all modified files.

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions
- History lives in `.planning/workflow-memory.json` -- a standalone file separate from GSD config
- Configuration (thresholds, sequence length) lives in `.planning/config.json` under a `workflow_memory` key
- `workflow-memory.json` stores only the call log: `[{"skill": "<name>", "ts": "<ISO timestamp>"}, ...]`
- Maximum 50 entries retained; on write, if length would exceed 50, drop the oldest entry first
- All 12 Skills participate in recording (translation, polish, de-ai, reviewer-simulation, abstract, experiment, logic, caption, cover-letter, visualization, literature, repo-to-paper)
- Write one entry after the Skill's first-step validation completes -- i.e., when the Skill begins producing output, not on invocation
- This applies uniformly including repo-to-paper-skill (no special handling for multi-step Skills)
- A "pattern" is a consecutive chain of 2 or 3 Skills in the call log (A->B or A->B->C)
- No time constraint -- only order in the log matters; cross-session sequences count
- A pattern triggers a recommendation only after appearing 5 or more times in the last 50 entries
- Pattern matching reads the log at Skill startup, before any other action
- Recommendations are presented at the start of the triggered Skill (not at the end of the previous one)
- Use `AskUserQuestion` with two options: "Yes, proceed" / "No, continue normally"
- If user accepts: run the current Skill in `direct` mode -- skip all Ask Strategy questions
- If user declines: continue in normal mode as if no recommendation was shown
- Recommendation must not interrupt or alter the Skill's output contract
- The "write call record" step is added to `references/skill-conventions.md` as a mandatory Workflow step for all new Skills
- Existing 12 Skills are updated as part of this phase; future Skills inherit the rule from conventions
- The convention documents the exact write location (`.planning/workflow-memory.json`) and the 50-entry cap

### Claude's Discretion
None explicitly listed -- all decisions are locked.

### Deferred Ideas (OUT OF SCOPE)
- GitHub auto-update Skill: a dedicated Skill to pull the latest version from `https://github.com/Lylll9436/Paper-Polish-Workflow-skill` and update local `.claude/skills/`. Candidate for Phase 19.
- Sequence length > 3 (e.g., 4-item chains): out of scope; threshold is 2-3 items only
- Per-Skill opt-out from recording: not needed for now; all 12 Skills participate uniformly
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| UXFIX-03 | Skill automatically records user's frequent workflow sequences, saves as project-level config, and offers as recommendations on next invocation | Full research: recording mechanism (JSON append with 50-entry cap), pattern detection algorithm (consecutive chain matching), recommendation UX (AskUserQuestion two-option dialog), and integration points (all 12 SKILL.md files + skill-conventions.md + config.json) are fully documented below |
</phase_requirements>

## Standard Stack

### Core

This phase has no library dependencies. All changes are text edits to Markdown and JSON files.

| Component | Location | Purpose | Why Standard |
|-----------|----------|---------|--------------|
| `workflow-memory.json` | `.planning/workflow-memory.json` | Call history log | Project-level JSON per CONTEXT.md decision |
| `config.json` update | `.planning/config.json` | Threshold and limit configuration | Existing config file; new `workflow_memory` key |
| SKILL.md (x12) | `.claude/skills/*/SKILL.md` | Skill workflow definitions | Each gets Step 0 (pattern check) + record-write step |
| `skill-conventions.md` | `references/skill-conventions.md` | Mandatory authoring rules | Canonical convention document for all Skills |
| `skill-skeleton.md` | `references/skill-skeleton.md` | Copyable template | Must stay in sync with conventions |

### Supporting

None. No external dependencies.

### Alternatives Considered

None -- all implementation decisions are locked by CONTEXT.md.

## Architecture Patterns

### File Structure Changes

```
.planning/
  config.json                    # ADD: workflow_memory key
  workflow-memory.json           # NEW: call history log (created empty as [])

references/
  skill-conventions.md           # UPDATE: add Workflow Memory section
  skill-skeleton.md              # UPDATE: add Step 0 and record-write step template

.claude/skills/
  abstract-skill/SKILL.md        # UPDATE: add Step 0 + record-write
  caption-skill/SKILL.md         # UPDATE: add Step 0 + record-write
  cover-letter-skill/SKILL.md    # UPDATE: add Step 0 + record-write
  de-ai-skill/SKILL.md           # UPDATE: add Step 0 + record-write
  experiment-skill/SKILL.md      # UPDATE: add Step 0 + record-write
  literature-skill/SKILL.md      # UPDATE: add Step 0 + record-write
  logic-skill/SKILL.md           # UPDATE: add Step 0 + record-write
  polish-skill/SKILL.md          # UPDATE: add Step 0 + record-write
  repo-to-paper-skill/SKILL.md   # UPDATE: add Step 0 + record-write
  reviewer-simulation-skill/SKILL.md  # UPDATE: add Step 0 + record-write
  translation-skill/SKILL.md     # UPDATE: add Step 0 + record-write
  visualization-skill/SKILL.md   # UPDATE: add Step 0 + record-write
```

### Pattern 1: Call History Recording

**What:** After the Skill's first-step validation completes (i.e., when the Skill begins producing output), append one entry to `.planning/workflow-memory.json`.

**When to use:** Every Skill invocation, uniformly.

**Exact insertion point per Skill (derived from reading all 12 SKILL.md files):**

| Skill | Current First Step | Record-Write Insertion Point |
|-------|-------------------|------------------------------|
| translation | Step 1: Collect Context | After Step 1 completes (references loaded, input read) |
| polish | Quick-fix Step 1 / Guided Step 1 | After Step 1 completes |
| de-ai | Phase 1 Step 1: Prepare | After Phase 1 Step 1 completes |
| reviewer-simulation | Step 1: Collect Context | After Step 1 completes |
| abstract | Step 1: Collect Context | After Step 1 completes |
| experiment | Phase 1 Step 1: Prepare | After Phase 1 Step 1 completes |
| logic | Step 1: Input Guard | After Step 1 completes (full-paper guard passed) |
| caption | Step 1: Collect Context | After Step 1 completes |
| cover-letter | Step 1: Collect Context | After Step 1 completes |
| visualization | Step 1: Collect Context | After Step 1 completes |
| literature | Step 2: Collect Search Query | After Step 2 (Step 1 is MCP pre-flight; Step 2 confirms query) |
| repo-to-paper | Step 1: Scan Repository | After Step 1 completes (repo scanned, categories displayed) |

**Record entry format:**
```json
{"skill": "polish-skill", "ts": "2026-03-18T14:30:00.000Z"}
```

**Cap enforcement:** Before appending, read current array. If length >= 50, remove oldest entry (index 0). Then append.

### Pattern 2: Sequence Detection at Startup (Step 0)

**What:** At the very start of the Skill, before any existing Step 1 logic, read the call history and check for recognized patterns.

**Algorithm:**
1. Read `.planning/workflow-memory.json`. If file missing or empty, skip to normal workflow.
2. Look at the last 1-2 entries in the log to form candidate patterns:
   - If the last entry is Skill A and current Skill is B: check if A->B appears >= 5 times in the log
   - If the last two entries are Skill A then Skill B and current Skill is C: check if A->B->C appears >= 5 times in the log
3. "Appears >= 5 times" means: scan the entire log for consecutive sequences matching the pattern. Count each occurrence.
4. If a matching pattern is found, present recommendation via AskUserQuestion.

**Pattern matching specifics (derived from CONTEXT.md):**
- Only consecutive entries matter. If the log is [polish, de-ai, polish, translation, polish, de-ai, polish, de-ai, polish, de-ai, polish, de-ai], the pattern polish->de-ai appears 5 times (entries at indices 0-1, 6-7, 8-9, 10-11, with gaps from the translation entry breaking one chain).
- Correction: the pattern counts non-overlapping consecutive pairs. In the log [A, B, A, B, A, B, A, B, A, B, A, B], the pair A->B appears at positions (0,1), (2,3), (4,5), (6,7), (8,9), (10,11) = 6 times. Each time A is immediately followed by B counts as one occurrence.

**Recommendation prompt example (from CONTEXT.md specifics):**
```
AskUserQuestion({
  question: "检测到常用流程：polish -> de-ai（已出现 5 次）。是否直接以 direct 模式运行 de-ai？",
  options: [
    { label: "Yes, proceed", description: "Run in direct mode, skip Ask Strategy questions" },
    { label: "No, continue normally", description: "Proceed with normal workflow" }
  ]
})
```

**Behavior on accept/decline:**
- Accept: set mode to `direct` for this invocation. Skip all AskUserQuestion calls in the Ask Strategy. Proceed directly to the operational workflow.
- Decline: continue in normal mode. The recommendation had no effect.

### Pattern 3: Skill That Does Not Support Direct Mode

**What:** repo-to-paper-skill explicitly states `direct` mode is "Not supported -- outline generation inherently requires user validation at each level."

**How to handle:** If the user accepts a workflow recommendation for repo-to-paper, the CONTEXT.md says "run the current Skill in `direct` mode -- skip all Ask Strategy questions." For repo-to-paper, this means: skip the Ask Strategy questions (repo path, journal) but still present the guided workflow with confirmation checkpoints. The "direct mode" behavior means skipping pre-questions, not skipping the guided Steps 2-5 confirmations which are inherent to the Skill's output contract.

This interpretation is consistent because:
- The CONTEXT.md says "Recommendation must not interrupt or alter the Skill's output contract"
- repo-to-paper's output contract requires user confirmation at each heading level
- Skipping Ask Strategy (2 questions: repo path + journal) is the maximum safe optimization

### Anti-Patterns to Avoid

- **Recording at invocation time instead of after validation:** CONTEXT.md explicitly says "when the Skill begins producing output, not on invocation." If the Skill refuses (e.g., journal template missing, MCP unavailable), no record should be written.
- **Time-based pattern matching:** CONTEXT.md explicitly says "No time constraint -- only order in the log matters." Do not add timestamp comparisons.
- **Breaking the output contract:** The recommendation changes the mode, not the output. The same deliverables must still be produced.
- **Recommending the current Skill to itself:** A->A sequences should not trigger recommendations. But this is naturally handled since the pattern must be 2+ different Skills in sequence.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Structured user prompt | Custom dialogue text | `AskUserQuestion` with two options | Already enforced by skill-conventions.md; Phase 12 established this pattern |
| Configuration storage | New config file | `.planning/config.json` `workflow_memory` key | Existing config infrastructure |

**Key insight:** Everything in this phase is text editing of Markdown/JSON files. There is no code to "build" -- the implementation is entirely declarative Skill definitions.

## Common Pitfalls

### Pitfall 1: Line Budget Overflow
**What goes wrong:** Adding ~15-20 lines of workflow memory steps pushes Skills over the 300-line budget.
**Why it happens:** Several Skills are already near the limit (logic: 276, experiment: 242).
**How to avoid:** Keep the Step 0 and record-write additions extremely concise. Use a reference to `skill-conventions.md` rather than duplicating the full algorithm in every SKILL.md. Target ~10-12 lines per Skill for both additions combined.
**Warning signs:** Any Skill exceeding 300 lines after modification (except repo-to-paper which has existing justification).

### Pitfall 2: Inconsistent Record-Write Placement
**What goes wrong:** Some Skills record too early (on invocation, before validation) or too late (after final output).
**Why it happens:** Each Skill has a different workflow structure (some have phases, some have steps).
**How to avoid:** The research table above identifies the exact insertion point for each Skill. The rule is: "after the Skill's first-step validation completes" -- meaning after the Skill has confirmed it CAN proceed (references loaded, guards passed, input validated).
**Warning signs:** A Skill that was invoked but refused (e.g., journal template missing) still wrote a record.

### Pitfall 3: Pattern Count Off-by-One
**What goes wrong:** The pattern count is wrong, causing recommendations to appear too early or never.
**Why it happens:** Ambiguity in how "consecutive" is defined -- does A->B->A->B count as 2 occurrences of A->B or 1?
**How to avoid:** Define clearly: scan the entire log left-to-right. For each position i, if log[i].skill == A and log[i+1].skill == B, that is one occurrence of A->B. Occurrences can overlap (if the log is [A, B, A, B], positions (0,1) and (2,3) each count, total = 2).
**Warning signs:** Recommendation appearing after 4 occurrences or not appearing after 6.

### Pitfall 4: AskUserQuestion Fallback Missing
**What goes wrong:** In environments without AskUserQuestion, the recommendation step crashes or blocks.
**Why it happens:** The recommendation dialog uses AskUserQuestion but the existing Fallback Rules say to degrade gracefully.
**How to avoid:** The Step 0 pattern must include a fallback: if AskUserQuestion is unavailable, present the recommendation as a plain-text question ("检测到常用流程：... 是否继续？(yes/no)") per the existing fallback convention.
**Warning signs:** Skill blocks or crashes when AskUserQuestion is not available.

### Pitfall 5: repo-to-paper Direct Mode Confusion
**What goes wrong:** Accepting a recommendation for repo-to-paper skips all guided Steps (2-5 confirmations), producing an unchecked outline.
**Why it happens:** Literal interpretation of "run in direct mode" when repo-to-paper says direct mode is "Not supported."
**How to avoid:** For repo-to-paper, "direct mode from recommendation" means: skip the 2 Ask Strategy questions (repo path + journal, assuming they are known from context or previous invocation), but keep all guided Step checkpoints. Document this explicitly in the SKILL.md update.
**Warning signs:** repo-to-paper produces an outline without any user confirmation checkpoints.

### Pitfall 6: Empty or Missing History File
**What goes wrong:** Step 0 crashes when `.planning/workflow-memory.json` does not exist or contains invalid JSON.
**Why it happens:** First-ever invocation, or file was manually deleted/corrupted.
**How to avoid:** Step 0 must handle gracefully: if file missing or unparseable, skip pattern detection and proceed to normal workflow. Create the file as `[]` on first record-write if it does not exist.
**Warning signs:** Skill errors on first invocation in a new project.

## Code Examples

### Example 1: workflow-memory.json Structure

```json
[
  {"skill": "translation-skill", "ts": "2026-03-18T10:00:00.000Z"},
  {"skill": "polish-skill", "ts": "2026-03-18T10:15:00.000Z"},
  {"skill": "de-ai-skill", "ts": "2026-03-18T10:30:00.000Z"},
  {"skill": "translation-skill", "ts": "2026-03-18T11:00:00.000Z"},
  {"skill": "polish-skill", "ts": "2026-03-18T11:20:00.000Z"},
  {"skill": "de-ai-skill", "ts": "2026-03-18T11:35:00.000Z"}
]
```

### Example 2: config.json Update

```json
{
  "mode": "yolo",
  "granularity": "fine",
  "parallelization": true,
  "commit_docs": true,
  "model_profile": "quality",
  "workflow": {
    "research": true,
    "plan_check": true,
    "verifier": true,
    "nyquist_validation": true,
    "_auto_chain_active": false
  },
  "workflow_memory": {
    "threshold": 5,
    "max_history": 50,
    "max_chain": 3
  }
}
```

### Example 3: Step 0 Template (for SKILL.md)

```markdown
### Step 0: Workflow Memory Check

- Read `.planning/workflow-memory.json`. If file missing or empty, skip to Step 1.
- Check if the last 1-2 entries form a recognized pattern with the current Skill (e.g., polish -> de-ai) that has appeared >= `workflow_memory.threshold` times (default: 5) in the log.
- If a pattern is found, present recommendation via AskUserQuestion:
  - Question: "检测到常用流程：[pattern]（已出现 N 次）。是否直接以 direct 模式运行 [current skill]？"
  - Options: "Yes, proceed" / "No, continue normally"
- If user accepts: set mode to `direct` for this invocation, skip Ask Strategy questions.
- If user declines or AskUserQuestion unavailable: continue in normal mode.
```

### Example 4: Record-Write Step Template (for SKILL.md)

```markdown
**Record workflow:** After Step 1 validation completes, append `{"skill": "[skill-name]", "ts": "<ISO timestamp>"}` to `.planning/workflow-memory.json`. If file does not exist, create it as `[]` first. If log length >= 50, drop the oldest entry before appending.
```

### Example 5: skill-conventions.md Addition

```markdown
## Workflow Memory

All Skills must participate in the project-wide workflow memory system.

### Recording

- After the Skill's first-step validation completes (i.e., when the Skill begins producing output, not on invocation), append one entry to `.planning/workflow-memory.json`.
- Entry format: `{"skill": "<skill-name>", "ts": "<ISO timestamp>"}`
- Maximum 50 entries retained. If log length >= 50, drop the oldest entry before appending.
- If the file does not exist, create it as `[]` before appending.
- If the Skill refuses or exits before completing first-step validation, do NOT write a record.

### Pattern Detection (Step 0)

- At the very start of the Workflow, before any existing Step 1 logic, read `.planning/workflow-memory.json`.
- Check if the last 1-2 entries form a consecutive pattern with the current Skill that appears >= threshold times (configured in `.planning/config.json` under `workflow_memory.threshold`, default: 5).
- If a matching pattern is found, present a bilingual recommendation via AskUserQuestion with two options: "Yes, proceed" (activates direct mode) / "No, continue normally".
- If the user accepts: skip all Ask Strategy questions and run in `direct` mode.
- If the user declines or AskUserQuestion is unavailable: proceed in normal mode.
- Pattern detection must not block the workflow. If the history file is missing, empty, or unparseable, skip silently to Step 1.
```

## State of the Art

This phase does not involve external libraries or evolving ecosystems. All implementation is project-internal Markdown and JSON. No "state of the art" changes apply.

## Skill-by-Skill Impact Analysis

Critical input for the planner -- exact line counts and modification impact:

| Skill | Current Lines | Est. Lines Added | Est. Final | Budget Status |
|-------|--------------|------------------|------------|---------------|
| abstract | 228 | 12 | 240 | OK (under 300) |
| caption | 217 | 12 | 229 | OK |
| cover-letter | 219 | 12 | 231 | OK |
| de-ai | 228 | 12 | 240 | OK |
| experiment | 242 | 12 | 254 | OK |
| literature | 226 | 12 | 238 | OK |
| logic | 276 | 12 | 288 | OK (tight) |
| polish | 213 | 12 | 225 | OK |
| repo-to-paper | 448 | 15 | 463 | Over (existing justified exception) |
| reviewer-simulation | 230 | 12 | 242 | OK |
| translation | 212 | 12 | 224 | OK |
| visualization | 171 | 12 | 183 | OK |

**Strategy to minimize line increase:** Reference `skill-conventions.md > Workflow Memory` for the full algorithm. In each SKILL.md, add only:
- Step 0: ~6-7 lines (brief version referencing conventions)
- Record-write: ~2-3 lines (one-liner after Step 1 completion note)
- Total: ~10-12 lines per Skill

## Modification Order and Dependencies

The planner should sequence modifications as follows:

1. **First: Infrastructure files** (no dependencies)
   - `.planning/config.json` -- add `workflow_memory` key
   - `.planning/workflow-memory.json` -- create as `[]`
   - `references/skill-conventions.md` -- add Workflow Memory section
   - `references/skill-skeleton.md` -- add Step 0 and record-write to the template

2. **Second: All 12 SKILL.md files** (depend on conventions being updated first)
   - Each Skill gets the same two additions: Step 0 at the top of Workflow, record-write after first validation step
   - Order within this wave does not matter

3. **Third: Verification** (depends on all modifications being complete)
   - Read all 16 modified files and confirm consistency

## Open Questions

1. **Pattern counting with gaps**
   - What we know: CONTEXT.md says "consecutive chain of 2 or 3 Skills in the call log" and "only order in the log matters"
   - What's unclear: If the log is [A, B, C, A, B, D, A, B], does A->B count as 3 occurrences (positions 0-1, 3-4, 6-7)?
   - Recommendation: Yes -- count every position where A is immediately followed by B. This is the most natural reading of "consecutive" and aligns with the user's intent of detecting frequent sequences.

2. **3-item chain counting**
   - What we know: Patterns can be 2 or 3 items (A->B or A->B->C)
   - What's unclear: Should both 2-item and 3-item patterns be checked? If A->B->C has 5 occurrences AND A->B has 7 occurrences, which recommendation is shown?
   - Recommendation: Check 3-item patterns first (more specific). If a 3-item pattern matches, show that recommendation. If not, check 2-item patterns. This avoids showing a less-specific recommendation when a more-specific one is available.

3. **Multiple patterns matching simultaneously**
   - What we know: Only one recommendation should be shown at startup
   - What's unclear: If both polish->de-ai and translation->polish->de-ai both have 5+ occurrences, which takes priority?
   - Recommendation: Longer chain wins (3-item over 2-item). If tied in length, higher count wins. If still tied, most recent occurrence wins. Document this priority in skill-conventions.md.

## Validation Architecture

### Test Framework

| Property | Value |
|----------|-------|
| Framework | None -- this is a pure Markdown/JSON configuration project with no executable code |
| Config file | N/A |
| Quick run command | N/A |
| Full suite command | N/A |

### Phase Requirements -> Test Map

| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| UXFIX-03 (recording) | Each Skill writes to workflow-memory.json after first validation | manual-only | Read all 12 SKILL.md files, verify each has record-write step in correct position | N/A |
| UXFIX-03 (detection) | Step 0 detects patterns and offers recommendation | manual-only | Read all 12 SKILL.md files, verify each has Step 0 with pattern detection | N/A |
| UXFIX-03 (recommendation) | AskUserQuestion with two options, accept -> direct mode | manual-only | Read all 12 SKILL.md files, verify Step 0 includes AskUserQuestion dialog | N/A |
| UXFIX-03 (conventions) | skill-conventions.md updated with Workflow Memory rules | manual-only | Read skill-conventions.md, verify Workflow Memory section exists | N/A |
| UXFIX-03 (config) | config.json has workflow_memory key | manual-only | Read config.json, verify workflow_memory key with threshold/max_history/max_chain | N/A |

**Justification for manual-only:** This project contains no executable code -- it is a collection of Claude Code Skill definitions (Markdown files), reference documents, and JSON configuration. There is no test runner, no build system, and no code to unit-test. Validation consists of reading the modified files and confirming their textual content matches the specification.

### Sampling Rate
- **Per task commit:** Read modified files and verify textual content
- **Per wave merge:** Read all 16 modified files end-to-end
- **Phase gate:** Complete file review before `/gsd:verify-work`

### Wave 0 Gaps
None -- no test infrastructure needed for a pure configuration project.

## Sources

### Primary (HIGH confidence)
- **All 12 SKILL.md files** -- read in full to understand current Workflow structure, modes, line counts, and insertion points
- **references/skill-conventions.md** -- read in full; the canonical convention document that must be updated
- **references/skill-skeleton.md** -- read in full; the copyable template that must stay in sync
- **.planning/config.json** -- read; existing config structure verified
- **18-CONTEXT.md** -- read in full; all decisions locked

### Secondary (MEDIUM confidence)
- None needed. All information is available from project files.

### Tertiary (LOW confidence)
- None.

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH -- all files are project-internal, fully read and understood
- Architecture: HIGH -- modification pattern is uniform and well-defined by CONTEXT.md
- Pitfalls: HIGH -- derived from concrete analysis of all 12 Skills' current structure and line counts

**Research date:** 2026-03-18
**Valid until:** Indefinite (project-internal configuration, no external dependencies)
