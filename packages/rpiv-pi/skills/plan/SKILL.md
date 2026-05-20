---
name: plan
description: Convert a design artifact into a phased implementation plan with parallelized atomic phases and explicit success criteria, written to .rpiv/artifacts/plans/. Use after the design skill when the user wants a design turned into an actionable, phase-by-phase plan to hand to the implement skill. Prefer plan when a straightforward phased breakdown is sufficient, and prefer blueprint when iterative vertical-slice micro-checkpoints between phases are needed.
argument-hint: "[design artifact path]"
shell-timeout: 10
---

# Plan

You are tasked with creating phased implementation plans from design artifacts. The design artifact contains all architectural decisions, full implementation code, and ordering constraints. Your job is to decompose that design into parallelized atomic phases with success criteria that implement can execute.

## Input

`$ARGUMENTS` — path to a design artifact (`.rpiv/artifacts/designs/*.md`).

## Metadata

```!
node "${SKILL_DIR}/../_shared/now.mjs"
echo
node "${SKILL_DIR}/../_shared/git-context.mjs"
```

- `now.mjs` (line 1) — `<iso>\t<slug>` tab-separated.

Copy values verbatim — do not reformat the timezone offset.

## Flow

1. Input → 2. Decompose into phases → 3. Write plan → 4. Independent Coverage Review → 5. Triage & iterate → 6. Follow-ups

The final artifact is implement-ready.

## Steps

### Step 1: Read Design Artifact

When this command is invoked:

1. **Determine input mode**:

   **Design artifact provided** (path to a `.md` file in `.rpiv/artifacts/designs/`):
   - Read the design artifact FULLY using the Read tool WITHOUT limit/offset
   - Extract: Architecture (the code changes), File Map, Ordering Constraints, Verification Notes, Performance Considerations, Scope
   - These are the inputs for phasing
   - Design decisions are settled — do not re-evaluate them
   - If the design has unresolved questions, STOP — tell the developer to return to design

   **No arguments provided**:
   ```
   I'll create an implementation plan from a design artifact. Please provide the path:

   `/skill:plan .rpiv/artifacts/designs/2025-01-20_09-30-00_feature.md`

   Run `/skill:design` first to produce the design artifact. There is no standalone path.
   ```
   Then wait for input.

2. **Read any additional files mentioned** in the design's References — research documents, tickets. Read them FULLY for context.

### Step 2: Decompose into Phases

Read the Ordering Constraints and File Map from the design artifact. Apply phasing rules:

1. **Independently implementable**: Each phase must compile and pass tests on its own — no cross-phase runtime state
2. **Parallelizable**: Phases that don't depend on each other are explicitly marked (e.g., "Phases 2 and 3 can run in parallel")
3. **Worktree-sized**: Each phase should be appropriate for a single implement session in a worktree (~3-8 files changed, 1-3 components touched)
4. **Dependency-ordered**: Phase ordering follows the design artifact's Ordering Constraints
5. **Grouped coherently**: Related file changes go in the same phase (e.g., import change + hook setup + JSX modification for one component)

**If the design's Ordering Constraints say "all files independent"**, consider whether a single phase is appropriate. Don't split into phases just for the sake of it — if all changes can be done in one worktree session, one phase is correct.

Present phase outline and get developer feedback BEFORE writing details:

```
Here's my proposed plan structure based on the design at {path}:

## Implementation Phases:
1. {Phase name} - {what it accomplishes} ({N} files)
2. {Phase name} - {what it accomplishes} ({N} files)
3. {Phase name} - {what it accomplishes} ({N} files)

Phases {2} and {3} can run in parallel after Phase 1.
Total: {N} files across {M} phases.

Does this phasing make sense? Should I adjust the order or granularity?
```

Use the `ask_user_question` tool to confirm the phase structure. Question: "{N} phases, {M} total files. Does this structure work?". Header: "Phases". Options: "Proceed (Recommended)" (Write the detailed plan with code blocks and success criteria); "Adjust phases" (Split, merge, or reorder phases before writing); "Change scope" (Add or remove files from the plan).

Get feedback on structure before writing details.

### Step 3: Write Plan

After structure approval, write the plan **incrementally** — skeleton first, then fill each phase:

1. **Write the plan skeleton** to `.rpiv/artifacts/plans/<slug>_<description>.md` (use `<slug>` from the Metadata block's `now.mjs` line 1; copy `<iso>` verbatim into frontmatter `date:` and `last_updated:`).
   - Format: `<slug>_<description>.md` where:
     - `<slug>` comes from `now.mjs` (second tab-separated field on line 1)
     - description is a brief kebab-case description (may include ticket number)
   - Examples:
     - With ticket: `2025-01-08_14-30-00_ENG-1478-parent-child-tracking.md`
     - Without ticket: `2025-01-08_14-30-00_improve-error-handling.md`
   - The skeleton includes everything EXCEPT large code blocks: frontmatter, Overview, Desired End State, What We're NOT Doing, full phase structure (Overview, Changes Required with file paths and change summaries, Success Criteria, parallelism annotations), Testing Strategy, Performance Considerations, References. All phasing and structural decisions happen in this pass.

2. **Fill code blocks using Edit** — one phase at a time:
   - For each phase, Edit to insert the before/after code blocks from the design's Architecture section into the Changes Required subsections

3. **Use this template structure**:

```markdown
---
date: {Current date and time with timezone in ISO format}
author: {`author:` from Metadata block}
commit: {Current commit hash}
branch: {Current branch name}
repository: {Repository name}
topic: "{Feature/Task Name}"
tags: [plan, relevant-component-names]
status: in-progress
parent: "{path to design artifact}"
last_updated: {Same ISO timestamp as `date:` above}
last_updated_by: {`author:` from Metadata block}
---

# {Feature/Task Name} Implementation Plan

## Overview

{Brief description of what we're implementing and why. Reference design artifact.}

## Desired End State

{From design artifact's Desired End State / Summary — what "done" looks like and how to verify it}

## What We're NOT Doing

{From design artifact's Scope → Not Building}

## Phase 1: {Descriptive Name}

### Overview
{What this phase accomplishes}

### Changes Required:

#### 1. {Component/File Group}
**File**: `path/to/file.ext`
**Changes**: {Summary of changes}

```{language}
// Code from design artifact's Architecture section
```

### Success Criteria:

#### Automated Verification:
- [ ] Type checking passes: `pnpm typecheck`
- [ ] Linting passes: `pnpm lint`
- [ ] Tests pass: `pnpm test`

#### Manual Verification:
- [ ] {From design's Verification Notes — specific visual/behavioral check}
- [ ] {Component-specific verification}

---

## Phase 2: {Descriptive Name}

{Similar structure with both automated and manual success criteria...}

---

## Testing Strategy

### Automated:
- {Standard project checks from success criteria}

### Manual Testing Steps:
1. {From design's Verification Notes — converted to step-by-step}
2. {Another verification step}

## Performance Considerations

{From design artifact — copied directly}

## Migration Notes

{From design artifact — copied directly. If applicable: schema changes, data migration, rollback strategy, backwards compatibility. Empty if not applicable.}

## Developer Context

{Empty at skeleton write; Step 4.4 fallback notes and any post-write developer interactions land here.}

## References

- Design: `.rpiv/artifacts/designs/{file}.md`
- Research: `.rpiv/artifacts/research/{file}.md`
- Original ticket: `thoughts/me/tickets/{file}.md`
```

### Step 4: Independent Coverage Review

After Step 3 finalizes the artifact, dispatch an independent coverage-review subagent to walk every verification-intent entry against the plan's phases at HEAD.

The plan's emitted code originates from the design artifact, where it was the subject of code review at design's Step 8 (subject to design's Step 8.4 fallback). Coverage review is the only review dispatched here; code review belongs upstream. Every `## Verification Notes` entry must land in a phase's `### Success Criteria:` bullet or as a visible code mirror.

#### 4.0. Flip status to in-review

Before dispatching the reviewer, Edit frontmatter `status: in-progress` → `status: in-review` (Step 5 flips to `ready` after triage — keeps consumers off an artifact still being edited).

#### 4.1. Dispatch the artifact-coverage-reviewer subagent

Reuse the exact `file_path` string passed to `Write` at Step 3 — the runtime already resolved it for this platform; do not rebuild it from `pwd`. `ls` to verify it still exists; abort dispatch on miss.

```
Agent({
  subagent_type: "artifact-coverage-reviewer",
  description: "post-finalization plan coverage review",
  prompt: `Plan artifact: {Step-3 Write file_path, ls-verified}

Review the finalized plan's verification-intent coverage. Walk every ## Verification Notes entry (and any carried-forward precedent-lesson content); for each, verify it lands in either a phase's ### Success Criteria: bullet or as a visible code mirror. Emit one severity-tagged row per uncovered entry.`
})
```

#### 4.2. Persist the coverage table to the artifact

The agent returns a markdown table with columns `plan-loc | codebase-loc | severity | dimension | finding | recommendation`. Append it to the plan artifact as a new section, with a `resolution` column appended (initially blank, filled progressively at Step 5):

```markdown
## Plan Coverage Review (Step 4)

_Independent post-finalization verification-coverage review. Findings triaged at Step 5._

| plan-loc          | codebase-loc                | severity   | dimension             | finding   | recommendation   | resolution         |
| ----------------- | --------------------------- | ---------- | --------------------- | --------- | ---------------- | ------------------ |
| {plan-loc}        | <n/a>                       | {severity} | verification-coverage | {finding} | {recommendation} | (filled at Step 5) |
| ...               |                             |            |                       |           |                  |                    |
```

If the agent emits zero rows, still emit the section with a single line: `_No findings — coverage reviewer cleared the artifact._`. Persistence is mandatory regardless of finding count — the section is the durable audit trail.

#### 4.3. Tally findings for Step 5's prompt

Count rows by severity. Store the counts in main context for Step 5's developer prompt:

```
{B} blockers, {C} concerns, {S} suggestions
```

Do NOT auto-apply any finding. The orchestrator never makes the apply / defer / dismiss judgment alone — that lives with the developer at Step 5. The reviewer's role is to surface; the developer's role is to triage.

#### 4.4. Failure handling

If artifact-coverage-reviewer errors out (subprocess crash, malformed output, timeout):
- Skip Step 4's findings; do not block on the failure.
- Append `_Step 4 coverage review failed: {one-line cause}._` under the `## Plan Coverage Review (Step 4)` heading instead of the row table.
- Record the fallback in a `## Developer Context` section (create if absent): `Step 4 review unavailable; proceeded to developer review without reviewer findings.`
- Proceed to Step 5.

### Step 5: Review & Iterate

1. **Triage Step 4 reviewer findings first** (skip if Step 4 returned no findings):

   Present the Plan Coverage Review table from Step 4 to the developer with severity-grouped framing:

   ```
   Coverage-reviewer findings: {B} blockers, {C} concerns, {S} suggestions

   Triage each row before the freeform review below:
   - applied — change made; I'll Edit the affected `### Success Criteria:` block (or add a code-guard reference) and fill the row's resolution as `applied: {one-line summary}`
   - deferred — noted but not fixing now; resolution cites why (e.g., "out of scope for this plan", "follow-up commit")
   - dismissed — not a real issue; resolution explains why the reviewer was wrong (e.g., "X is intentional because Y")
   ```

   Use `ask_user_question` with options "applied / deferred / dismissed":
   - **applied**: Edit the affected `### Success Criteria:` block per the recommendation; fill `resolution`.
   - **deferred** / **dismissed**: fill `resolution` with the reason.

   **Order and batching**: blockers sequentially (resolution may invalidate later rows). Concerns and suggestions: batch up to 4 independent rows per `ask_user_question` call (Step 4's rule). Independent = different files / different intents AND neither recommendation references the other's location; otherwise sequential.

2. **Flip status to ready**: once every row has a `resolution` (or the table is empty per Step 4's no-findings / failure-fallback path), Edit frontmatter `status: in-review` → `status: ready`. Artifact is now implement-ready.

3. **Present the plan location** (after triage is complete):
   ```
   Implementation plan written to:
   `.rpiv/artifacts/plans/{filename}.md`

   {N} phases, {M} total file changes. {T} reviewer findings triaged at Step 5 ({A} applied, {D} deferred, {DD} dismissed). When Step 4 hit the failure-fallback path, render this line as `Step 4 review unavailable — proceeded without findings.`

   Please review:
   - Are the phases properly scoped for worktree execution?
   - Are the success criteria specific enough?
   - Any phase that should be split or merged?

   ---

   💬 Follow-up: describe the change in chat to append a timestamped Follow-up section to this artifact, or use `/skill:revise <plan-path>` for surgical phase edits. Re-run `/skill:plan` for a fresh artifact.

   **Next step:** `/skill:implement .rpiv/artifacts/plans/{filename}.md Phase 1` — start execution at Phase 1 (omit `Phase 1` to run all phases sequentially).

   > 🆕 Tip: start a fresh session with `/new` first — chained skills work best with a clean context window.
   ```

### Step 6: Handle Follow-ups

- **Edit in-place.** Use the Edit tool to update the plan artifact directly. Phase numbering stays stable when possible — renumber only when a phase is split or merged.
- **Bump frontmatter.** Update `last_updated` + `last_updated_by`; set `last_updated_note: "<one-line summary>"`.
- **Phase-level moves.** Split large phases, merge small phases, adjust success criteria, reorder phases — all in-place. Continue refining until the developer is satisfied.
- **When to re-invoke instead.** For surgical edits driven by review findings, prefer `/skill:revise <plan-path>`. Re-run `/skill:plan` only when the underlying design changed materially. The previous block's `Next step:` stays valid for the existing plan.

## Guidelines

1. **Trust the Design**:
   - Design decisions are fixed — do not re-evaluate architectural choices
   - If something in the design seems wrong, flag it to the developer
   - Don't silently change the approach or add scope
   - The design is the source of truth for what to build

2. **Be Interactive**:
   - Don't write the full plan in one shot
   - Get buy-in on phase structure first
   - Allow course corrections on granularity
   - Work collaboratively

3. **Be Practical**:
   - Focus on incremental, testable changes
   - Each phase should leave the codebase in a working state
   - Think about what can be verified independently
   - Include "what we're NOT doing" from the design's scope

4. **Phase for Worktrees**:
   - Each phase should be implementable in an isolated worktree
   - No phase should depend on another phase's uncommitted changes
   - If the design says "all independent," one phase may be correct
   - Don't split for the sake of splitting

5. **Track Progress**:
   - Use a todo list to track planning tasks
   - Mark planning tasks complete when done

6. **No Open Questions in Final Plan**:
   - If you encounter open questions during planning, STOP
   - If the design artifact has unresolved questions, send the developer back to design
   - Do NOT write the plan with unresolved questions
   - The implementation plan must be complete and actionable

## Success Criteria Guidelines

**Always separate success criteria into two categories:**

1. **Automated Verification** (can be run by execution agents):
   - Commands that can be run: `make test`, `npm run lint`, etc.
   - Specific files that should exist
   - Code compilation/type checking
   - Automated test suites

2. **Manual Verification** (requires human testing):
   - UI/UX functionality
   - Performance under real conditions
   - Edge cases that are hard to automate
   - User acceptance criteria

**Format example:**
```markdown
### Success Criteria:

#### Automated Verification:
- [ ] Database migration runs successfully: `make migrate`
- [ ] All unit tests pass: `go test ./...`
- [ ] No linting errors: `golangci-lint run`
- [ ] API endpoint returns 200: `curl localhost:8080/api/new-endpoint`

#### Manual Verification:
- [ ] New feature appears correctly in the UI
- [ ] Performance is acceptable with 1000+ items
- [ ] Error messages are user-friendly
- [ ] Feature works correctly on mobile devices
```

**Convert design's Verification Notes to success criteria:**
- Prose warnings → specific automated commands or manual steps
- "Test production builds" → `pnpm build && verify in built app`
- "Verify scrollbar appearance" → `[ ] Open {component}, scroll, observe slim scrollbar`
- "Do NOT use X" → `grep -r "X" src/ should return 0 matches`

## Subagent Usage

| Context | Agents Spawned |
|---|---|
| Step 4 post-finalization coverage review (mandatory) | artifact-coverage-reviewer |

## Important Notes

- NEVER edit source files — this skill produces a plan document, not implementation
- Always read the design artifact FULLY before decomposing into phases
- The plan template must be compatible with implement — preserve the phase/success criteria structure
- If the design artifact has unresolved questions, STOP — send the developer back to design
- Code in the plan comes from the design artifact's Architecture section — do not invent new code
- ALWAYS dispatch artifact-coverage-reviewer at Step 4 after Step 3 write, BEFORE the developer review at Step 5
- NEVER auto-apply a Step 4 reviewer finding; triage is the developer's call at Step 5
- ALWAYS hold `status: in-review` from Step 4.0 through Step 5; flip to `ready` only after every row has a `resolution` (or the table is empty)
- **Frontmatter consistency**: Always include frontmatter, use snake_case for multi-word fields, keep tags relevant
