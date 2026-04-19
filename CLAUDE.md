# Project Instructions

## Two-Layer AI Workflow

This project uses a strict separation between a reasoning layer (Claude Pro) and an execution layer (OpenCode Go). Neither layer may operate outside its designated role.

### Layer 1 — Reasoning (Claude Pro, $20/month)

Claude Pro is responsible exclusively for:

- Ingesting project context and existing code before making any plan
- Writing feature specs and saving them to `planning/<feature>.md`
- Making architectural decisions (component structure, data flow, API design, state management patterns)
- Reviewing completed work for correctness against spec acceptance criteria

Claude Pro must never write implementation code directly into source files.
All output from this layer is a spec file. Code generation belongs to Layer 2.

### Layer 2 — Execution (OpenCode Go, ~$60/month credit budget)

OpenCode Go consumes spec files from `planning/` and implements the diff. It must:

- Read the relevant `planning/<feature>.md` in full before starting any task
- Implement exactly what the spec describes — no additions, no refactors beyond scope
- Never make architectural decisions; if a spec is ambiguous on architecture, stop and flag it rather than deciding independently
- Never touch `.env` or any secrets file

A spec file must exist in `planning/` before any OpenCode Go task runs. No exceptions.

---

## Model Assignment (OpenCode Go)

Model selection is credit-sensitive. Assign models by task complexity as follows.

**Complex tasks** — multi-file changes, new features, API integration, data transformation logic
Use **Kimi K2.5** or **GLM-5.1** as primary models. These tasks require strong reasoning over context spread across multiple files.

**Routine / high-volume tasks** — styling fixes, copy changes, minor component edits, type corrections, test boilerplate
Use **MiniMax M2.5** or **Qwen3.5 Plus**. Reserve stronger models for tasks that genuinely need them.

| Task type | Preferred model |
|---|---|
| New feature implementation (multi-file) | Kimi K2.5 |
| Complex refactor or architectural migration | GLM-5.1 |
| Single-component edits, style fixes | MiniMax M2.5 |
| High-volume repetitive tasks (tests, stubs) | Qwen3.5 Plus |

When in doubt, default down to the cheaper model and escalate only if output quality is insufficient.

---

## Spec File Format (`planning/<feature>.md`)

Every spec file must contain exactly the following sections in this order. Prose should be moderate — one to two sentences per step with key decisions noted inline.

```markdown
# Feature: <name>

## Goal
One sentence stating what this feature accomplishes and why it exists.

## Context
Relevant background: which existing files are affected, what external APIs or data shapes are involved, and any constraints inherited from prior decisions.

## Steps
Numbered implementation steps. Each step names the file(s) to modify and describes the change in enough detail that the executing model requires no architectural judgment.

1. …
2. …

## File Targets
Explicit list of files to be created or modified. No other files should be touched.

- `src/components/X.jsx` — create
- `src/lib/strava.js` — modify

## Acceptance Criteria
Verifiable conditions that define "done." Should be checkable without running a test suite if possible.

- [ ] …
- [ ] …
```

Specs must be committed to `planning/` before execution begins. Do not delete or overwrite a spec after execution starts; append a `## Notes` section for any amendments.

---

## UI Work & Design

All UI-related work must reference [`DESIGN.md`](./DESIGN.md) before making any visual changes.

- `DESIGN.md` is the single source of truth for visual decisions: color palette, typography, spacing scale, component patterns, and layout conventions.
- Before implementing any UI feature or style change, read `DESIGN.md` in full.
- After completing UI work, update `DESIGN.md` if new patterns or decisions were introduced.
- OpenCode Go must not make visual/design decisions independently. If `DESIGN.md` does not cover a case, stop and flag it for the reasoning layer.

---

## Deployment

**Every merged change must be deployed to Vercel immediately after merge.**

- All feature branches are merged to `main` via pull request.
- After a PR merges to `main`, trigger a Vercel deployment (automatic if Vercel is connected to the repo via GitHub integration; otherwise run `vercel --prod` manually).
- Do not consider a task complete until the deployment succeeds and the change is live on the production Vercel URL.
- If a deployment fails, treat it as a blocker — open a new spec or flag the issue before starting the next task.

---

## Cost Constraints

Monthly budget: $60 credit equivalent across all OpenCode Go usage.
Per-task limit: No hard ceiling per task. Monthly total is the governing constraint — monitor it yourself.
Model discipline is the primary cost lever. Routine tasks running on MiniMax M2.5 instead of Kimi K2.5 represent the most significant savings opportunity. Apply the model assignment table above consistently.

---

## Non-Negotiable Rules

1. **Layer separation is absolute.** OpenCode Go must never make architectural decisions. Claude Pro must never write implementation code into source files. If either constraint would be violated to complete a task, stop and restructure the workflow.
2. **Spec-first execution.** No OpenCode Go task may begin without a corresponding spec in `planning/`. If a spec does not exist, create it in the reasoning layer first.
3. **No secrets, ever.** Neither AI layer may read from, write to, or reference `.env` or any file containing credentials, tokens, or API keys. Tokens are managed manually outside the AI workflow.
4. **No direct commits to main.** All changes must be made on a feature branch and merged via pull request.
5. **No file deletions without explicit spec instruction.** A spec must name a file for deletion explicitly. OpenCode Go must not delete files as a side effect of refactoring.
6. **No scope creep.** OpenCode Go implements the spec as written. If the implementer identifies a better approach, it must surface that as a comment or flag — not implement it unilaterally.
7. **Deploy every change.** Every merged PR must produce a successful Vercel deployment before the task is considered done.

---

## Behavioral Guidelines (Both Layers)

### 1. Think Before Coding
Do not assume. Do not hide confusion. Surface tradeoffs before starting.

Before implementing anything: state assumptions explicitly, and ask if uncertain. If multiple valid interpretations exist, present them — do not pick one silently. If a simpler approach exists, say so and push back when warranted. If something is genuinely unclear, stop, name what is confusing, and ask.

### 2. Simplicity First
Write the minimum code that solves the problem. Nothing speculative.

Do not add features beyond what was asked. Do not introduce abstractions for single-use code, or add "flexibility" and "configurability" that was not requested. Do not write error handling for impossible scenarios. If an implementation reaches 200 lines and could be 50, rewrite it.

The test: would a senior engineer say this is overcomplicated? If yes, simplify before submitting.

### 3. Surgical Changes
Touch only what you must. Clean up only your own mess.

When editing existing code, do not improve adjacent code, comments, or formatting. Do not refactor things that are not broken. Match existing style even if you would do it differently. If unrelated dead code is noticed, mention it — do not delete it.

When your changes create orphans (unused imports, variables, or functions), remove them. Do not remove pre-existing dead code unless the spec explicitly asks for it.

The test: every changed line should trace directly to the spec or the user's request.

### 4. Goal-Driven Execution
Define success criteria. Loop until verified.

Transform tasks into verifiable goals before starting. For multi-step tasks, state a brief plan with a verification check at each step:

1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]

Strong success criteria allow independent looping to completion. Weak criteria ("make it work") require constant clarification and should trigger a stop-and-ask before implementation begins.

---

## Escalation Path

If OpenCode Go encounters any of the following, it must stop and surface the issue rather than proceeding:

- The spec is ambiguous about which file to modify
- Completing the task requires touching a file not listed under File Targets
- An architectural decision is implied but not stated
- The task would require reading or modifying `.env`
- A UI decision is required but not covered in `DESIGN.md`
- A Vercel deployment failed after merge

Escalation means: output a clear description of the blocker, then halt. Do not guess.

---

## Commands

```bash
# Build
npm run build

# Test
npm test
npm test -- path/to/file

# Lint & Format
npm run lint
npm run lint:fix
npm run typecheck

# Dev
npm run dev

# Deploy
vercel --prod
```

## Workflow

- Run typecheck after making a series of code changes
- Prefer fixing the root cause over adding workarounds
- When unsure about approach, use plan mode (`Shift+Tab`) before coding
- After every PR merge: deploy to Vercel and confirm the production URL is live

## Don'ts

- Don't modify generated files (`*.gen.ts`, `*.generated.*`)
- Don't make UI decisions without reading `DESIGN.md` first
- Don't consider a task done until Vercel deployment succeeds
