---
name: plan-mode
description: Create or update an editable Markdown implementation plan with code references, decisions, and todos. Use when the user requests a plan, planning-only work, a saved plan file, or architecture and tradeoff analysis as the deliverable. Task complexity or multiple files alone should not activate a planning-only approval gate during authorized implementation.
---

# Plan Mode

## Overview

Create a disk-backed editable Markdown plan, research the codebase into it, resolve material questions, and keep file references and todos current. The plan file records the work; the user's instructions determine whether to stop at planning or continue into implementation.

Read `references/architecture.md` only for complex planning cases involving tool policy, read-only subagent exploration, diagrams, multi-system data flow, or non-trivial handoff from an approved plan to execution.

## Core Contract

Resolve the requested phase from the conversation before applying the planning boundary:

- For a plan-only request, prepare the plan and stop before implementation. A request to research or plan does not authorize code changes.
- For a request to plan and implement, or to execute an already approved plan, record the existing authorization and continue through the authorized work. Do not require a second approval merely because a plan file was created or this skill was loaded.
- Do not select this skill solely because an implementation spans many files or needs research. Use an ordinary working plan within the existing task when that is sufficient.

During a planning-only phase:

- Do not edit implementation files, delete files, change settings, stage commits, install packages, start write-oriented scripts, or run commands with side effects while in plan mode.
- Use read-only inspection: read files, search code, inspect diagnostics, check existing terminal state, review documentation, and ask focused questions.
- Create and update planning artifacts only: the required Markdown plan file and `$grill-me`'s Q&A log and planning-ready outcome files when `$grill-me` is invoked.
- Treat the plan as an editable file. If the user edits it directly, reread it before changing the plan or building from it.
- Treat user changes and dirty working trees as user-owned. Plan around them; do not revert or normalize them.
- If no plan deliverable is requested and the task is already authorized, proceed with the task without introducing a planning-only phase.

## Planning Workflow

1. Create or reuse the Markdown plan file immediately, then cite its path.
2. Research the codebase, docs, diagnostics, and relevant conventions with read-only tools.
3. Update the plan with concrete file paths, code references, discovered constraints, and unresolved questions.
4. Ask focused clarifying questions when requirements would change the plan. After each answer, update the plan file before moving on.
5. Maintain `Plan Todos` as editable checklist items. Include dependencies, selected todos, and enough detail for another agent to build from the plan.
6. Pressure-test meaningful assumptions and failure modes against available evidence. Invoke `$grill-me` when the user requests an interview or unresolved user decisions benefit from one; complexity alone does not require a new question loop. Store pointers to its finalized transcript and outcome when used.
7. Mark the plan ready to build only when blocking questions are resolved, the todos are concrete, and validation is named. Otherwise keep it draft and identify the exact blocker.
8. Run the artifact check and fix any missing section or placeholder before handoff or execution.
9. For planning-only work, deliver the concrete plan and request implementation approval only if that is the requested handoff. If execution is already authorized, record its scope and proceed without another approval question. Chat or platform-plan output summarizes and links the file.
10. Build from the plan within the authorized scope. If the user approves only selected todos, execute only those items.

## Grill-Me Pressure Test

Use `$grill-me` the way `$split-commits` uses `$git-commit`: delegate the specialized step instead of recreating it.

- Use `$grill-me` for an explicitly requested interview or material unresolved decisions that require the user's judgment. First resolve what the repository and existing conversation already establish.
- Skip the interview when evidence, settled decisions, or reasonable in-scope assumptions resolve the plan. If `$grill-me` is unavailable and was not explicitly requested, perform the needed analysis directly.
- Let `$grill-me` ask one logged question at a time and follow its logging/finalization rules. Do not ask ad hoc pressure-test questions and then log them later.
- Treat `$grill-me`'s transcript and planning-ready outcome as planning artifacts allowed before implementation approval.
- After `$grill-me` finalizes, write links to the transcript and outcome paths in the plan. Include only a terse status or one-line summary in the plan; do not paste the full outcome content because it may be large.

## Plan Artifact Rules

The Markdown plan file is the required editable planning artifact.

- Do not use the platform plan or chat message as the only approval artifact. Those may summarize the plan, but they do not replace the file.
- Skip the plan file only when the user explicitly says not to write one or the environment cannot write files. In that case, say why no plan file was written.
- If the user provides a path, use it. Otherwise reuse the repository's existing plan location or template. If none exists, use `docs/plans/<YYYY-MM-DD>-<short-topic>.md`.
- Keep planning artifacts as the only allowed writes in plan mode. Creating parent directories for a requested plan file or `$grill-me` log/output is allowed only when needed for those artifacts.
- Include status, summary, clarifying questions, file/code references, plan todos, build instructions, validation, risks, and approval state.
- Use Markdown checkboxes for todos. Keep them editable and stable enough that selected todos can be handed to a new execution pass.
- Record draft/completion status separately from authorization. Use `Draft - awaiting approval` only when implementation approval is actually pending; otherwise record the existing authorization and selected todos in the approval fields. The helper's initial draft text is a placeholder for that state, not a new permission requirement.
- If the plan changes after feedback, update the artifact. Seek renewed approval only for work outside the existing authorization or an unresolved decision that materially changes the outcome.
- In the chat response, summarize the plan briefly and provide the plan file path.

### Plan Artifact Helper

Use `scripts/plan_artifact.py` to lower the chance of forgetting the file.

Resolve the helper:

```bash
workspace_root="$(git rev-parse --show-toplevel 2>/dev/null || pwd)"
helper=""
for candidate in \
  "$workspace_root/skills/plan-mode/scripts/plan_artifact.py" \
  "$HOME/.agents/skills/plan-mode/scripts/plan_artifact.py" \
  "$HOME/.codex/skills/plan-mode/scripts/plan_artifact.py"
do
  if [ -f "$candidate" ]; then
    helper="$candidate"
    break
  fi
done
[ -n "$helper" ] || { echo "plan_artifact.py not found" >&2; exit 1; }
```

Create a draft plan file before final approval:

```bash
plan_file="$(python3 "$helper" init --workspace "$workspace_root" --title "<plan title>")"
```

If the user gave a path:

```bash
plan_file="$(python3 "$helper" init --workspace "$workspace_root" --title "<plan title>" --path "<path>" --reuse-existing)"
```

After filling the plan file, verify it before asking for approval:

```bash
python3 "$helper" check "$plan_file"
```

If the check fails because required sections or placeholders remain, update the plan file and rerun the check. Do not ask for implementation approval until the check passes or until you explicitly report why it cannot pass.

## Clarification Rules

Ask questions early when the answer changes the plan.

- Ask at most one or two critical questions at a time.
- Use structured choices when the environment provides a question or clarification tool.
- Apply a sensible, low-risk default within scope when one exists and record material assumptions. Keep independent authorized work moving while a blocking decision is pending.
- Do not ask about trivia that can be answered by reading the codebase or existing docs.

## Research Rules

Keep research proportional to risk.

- For small tasks, read the directly relevant files and stop.
- For large codebases, use read-only subagents or semantic search to map ownership boundaries, routes, data contracts, and verification surfaces.
- If using subagents, give each one a bounded read-only objective and ask for paths, evidence, blockers, and residual risks.
- Avoid redoing a delegated investigation in the foreground unless the result is blocking and unavailable.

## Plan Structure

Make the plan easy to accept or reject.

- Name the files or modules likely to change, with code references when useful.
- Keep a live todo checklist that can be edited, selected, and built from.
- Include build notes that say how to execute the plan after approval.
- Include validation commands and any expected non-goals.
- Call out tradeoffs only when they affect the chosen approach.
- Include concise code snippets only when they clarify a non-obvious target.
- Use Mermaid diagrams inside the plan when they reduce ambiguity.
- Resolve blocking questions before marking the plan ready to build. If they cannot be resolved, deliver the completed research and name the exact blocked decision without implying approval or readiness.

## Handoff To Execution

After approval, build from the plan file.

- Re-read the latest user message and the plan file before acting, especially after a long pause or context transition.
- Execute only approved todos. If the user selected a subset, leave the other plan todos untouched.
- Keep edits scoped to the authorized outcome. Update the plan for evidence-backed implementation adjustments within that scope; pause only the affected work when a new product decision, scope expansion, or missing authority requires user input.
- Run the validation promised in the plan and required repository checks, or explain why a check could not run. Choose checks for affected behavior and contracts; repeat or broaden them only for a new change, failure, or unresolved risk.
