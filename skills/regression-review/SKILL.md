---
name: regression-review
description: Evaluate how the current code changes may create user-visible behavioral regressions and turn the result into a reviewer-friendly Markdown gate report. Use when reviewing a working tree, staged diff, commit range, or PR and the goal is to decide whether the change should be blocked, discussed, watched, or treated as an intentional visible change during code review.
---

# Regression Review

## Overview

Treat the change set as a code review gate, not as an audit archive. The primary question is whether a reviewer or TL should block the change because users will notice a broken or degraded behavior.

Always produce a Markdown report file and a short terminal summary.

Keep the overall recommendation mechanically aligned with the highest-severity unresolved finding. Do not let prose tone drift the gate up or down.

## Set Scope First

- Prefer an explicit scope from the user: working tree, staged changes, last commit, commit range, or branch diff.
- If the user does not specify scope, compare the current working tree against `HEAD` and state that assumption in the report.
- If requirements, issue text, or a design doc exist, read them before judging regressions.
- Separate intended product changes from accidental regressions.

## Output Rules

- Always write a Markdown file.
- If the repo already has an obvious location for reviews or reports, follow that convention.
- Otherwise write to `tmp/reviews/YYYY-MM-DD-user-visible-regression-report.md`.
- End with a short terminal summary that includes the report path, the gate recommendation, and the top risks.
- Structure the report in this order:
  - `Gate Snapshot`
  - `Block`
  - `Discuss`
  - `Watch`
  - `Intentional Changes`
  - `Evidence Appendix`
- Keep the primary view optimized for code review reading speed.
- Do not use wide summary tables as the main presentation shape.

## Review Actions

Use these action labels as the primary classification:

- `Block`
  - Strong evidence or strong risk that a user-visible behavior is broken, degraded, or dangerously changed.
  - A reviewer should stop the change from shipping until the issue is fixed or disproven.
- `Discuss`
  - The user-visible impact may be meaningful, but severity, intent, or actual runtime effect is still unclear.
  - A reviewer should raise it in review and resolve the uncertainty before approval.
- `Watch`
  - The behavior change is worth noticing, but does not justify blocking by itself.
  - A reviewer may approve with caveat, added test, follow-up, or monitoring.
- `Intentional`
  - The user-visible change appears deliberate and should not be framed as a regression.

Do not use `confirmed`, `probable`, or `possible` as top-level groupings. Keep them as evidence language inside the card when useful.

## Recommendation Mapping

Use the top-level `Recommendation` field in `Gate Snapshot` with this exact mapping:

- If any unresolved finding is `Block`, the report recommendation MUST be `Block`.
- Else if any unresolved finding is `Discuss`, the report recommendation MUST be `Discuss`.
- Else if any unresolved finding is `Watch`, the report recommendation MUST be `Pass with caveat`.
- Else use `Pass`.

Additional rules:

- Do not write `Pass with caveat` when a `Discuss` item is still open.
- Do not write `Discuss` when the body contains no `Discuss` items.
- If a suspected issue lacks enough proof to justify `Block`, downgrade the finding to `Discuss` instead of keeping `Block` with hand-wavy evidence.
- If the report contains both `Block` and `Discuss`, keep both sections, but the top-level recommendation remains `Block`.

## What Counts As One Finding

- Count one item per distinct user-facing outcome, not per file, function, or root cause.
- Merge multiple code changes that lead to the same user-visible symptom.
- Split items only when different user journeys are affected in materially different ways.
- Focus on findings that matter during review:
  - broken or degraded task completion
  - changed defaults or fallback behavior
  - loading, empty, or error state changes
  - permission or session behavior changes
  - stale data, cache, retry, or ordering changes
  - destructive or duplicate actions
  - output changes in emails, generated files, CLI output, or exported data

## Evidence Standard

- Prefer the strongest feasible evidence for the affected surface, not one fixed verification mode.
- Runtime checks are valuable, but they are not mandatory when static path tracing or output inspection already provides strong evidence.
- For frontend behavior, use the evidence that best matches the risk: code-path tracing, generated output inspection, logs, focused tests, or runtime repro when needed.
- For non-UI changes, inspect outputs, fixtures, generated files, CLI responses, logs, and focused tests.
- If evidence is missing, lower confidence and say so plainly.
- Separate verified facts from inferred consequences.
- Do not present speculation as fact.
- Do not treat `eslint`, `typecheck`, or passing unrelated tests as proof that a user-visible regression is disproven. They are hygiene evidence only.
- When a spec, issue, or design doc exists, use it to judge intent, but do not treat it as proof that the runtime/output behavior is still correct.

## Workflow

1. Define the comparison baseline.
   - Use `git diff`, `git diff --staged`, `git show`, or an explicit range.
   - Record the exact scope in the report.
2. Map affected user surfaces.
   - Identify routes, pages, API consumers, commands, scheduled jobs, config defaults, and feature flags touched by the diff.
   - Build a short list of the user journeys most relevant to review risk.
3. Trace the behavior delta.
   - Compare before and after behavior for each journey.
   - Look for removed guards, changed branching, altered ordering, serialization changes, loading changes, and error-path changes.
   - For refactors, wrappers, adapters, shared utilities, or "just telemetry" changes, explicitly trace what now supplies the user-visible input, what still enforces guards, and what produces the final output.
4. Gather proof and code pointers.
   - Collect the strongest runtime or output evidence available.
   - Trace each finding back to concrete code lines.
5. Classify for review action.
   - Decide whether the reviewer should `Block`, `Discuss`, `Watch`, or mark the change `Intentional`.
   - Rank within each section by user impact first, confidence second.
6. Write the report.
   - Use `references/report-template.md`.
   - Lead with the gate recommendation and the top items a reviewer should open first.

## Card Format

Each risk should be written as a short review card, not a spreadsheet row.

Use this shape:

```md
### F1 Block - Checkout can submit twice

User impact: Users can trigger duplicate payment attempts during a slow checkout.
Review reason: Direct money-path risk with obvious user-facing failure.
Surface: Checkout submit flow
Confidence: High

Look here first:
- [submit handler](/abs/path/app/checkout.tsx#L128)
- [removed guard](/abs/path/lib/payment.ts#L42)

Behavior delta:
- Before: First submit disabled repeat submission while the request was pending.
- After: Repeat clicks can issue another submission before the first request finishes.

Evidence:
- Reproduced locally in browser with throttled network.
- No remaining test covering duplicate submit protection.

Reviewer action:
Block until the guard is restored or equivalent idempotency is proven elsewhere.
```

## Writing Rules

- Start with the gate decision, not with counts.
- `Gate Snapshot` should fit on one screen when possible.
- `Gate Snapshot` should usually include:
  - `Recommendation`
  - `Why now`
  - `Must-review now`
  - `Coverage confidence`
  - `Biggest blind spot`
- Limit `Must-review now` to the top 3 items.
- In each review card, keep the first sentence about user impact, not code mechanics.
- Use exactly 1 or 2 links under `Look here first`.
- Put the full causal chain in `Evidence Appendix`, not in the main card.
- Use absolute file paths when the environment supports clickable local links. Include line anchors when available, for example `[checkout.tsx](/abs/path/app/checkout.tsx#L128)` or `[checkout.tsx](/abs/path/app/checkout.tsx:128)`.
- `Intentional Changes` should be compact and easy to skim.
- `Evidence Appendix` may use small tables.
- If no user-visible regressions are found, still write the report:
  - Say the recommendation is `Pass` or `Pass with caveat`
  - State the strongest blind spot
  - Document what was verified
- If the report is based mainly on static reasoning, say that plainly in both `Gate Snapshot` and `Coverage`.
- When evidence is mixed, write the verified code-path facts first, then the inferred user impact second.

## Behavior-Preserving Refactor Checklist

When the diff centralizes behavior behind a shared helper, adapter, wrapper, middleware, base class, hook, or common handler, check these explicitly before deciding it is "just a refactor":

- Source parity: did the user-visible behavior start reading from a different input, field, state slice, request payload, or serialized form?
- Guard parity: did auth, permission, debounce, duplicate-submit, confirmation, retry, ordering, or empty-state guards move, disappear, or become bypassable?
- Output parity: does the final renderer, exporter, request builder, CLI output, email body, or persisted record still consume the expected format and shape?
- Extension-point parity: if a local override, callback, prop, or branch was removed, is there a new path that preserves the same behavior, not just the same UI or API surface?
- Intent split: are telemetry-only, naming-only, or cleanup-only changes actually mixed with behavior changes in the same refactor?

For any user-visible flow, explicitly compare:

- user-visible input source
- transformed/intermediate payload
- final output or side effect
- gates that must still run before the effect happens

If those layers are no longer aligned, treat that as a regression lead even before runtime verification.

## Guardrails

- Do not call a deliberate product change a regression just because users will notice it.
- Do not mistake "tests passed" for "user impact disproven".
- Do not overcount the same symptom across multiple files.
- If the diff is large, cover the highest-risk journeys first and say where coverage stops.
- If a runtime check cannot be performed, say so and downgrade confidence rather than hiding the gap.
- Do not discard a strong static regression lead just because the change was introduced during a refactor or telemetry cleanup.
- Do not promote a finding to `Block` unless the user-visible breakage itself, not just the code smell, is strongly supported.

## Reference

- Use [references/report-template.md](references/report-template.md) as the default output shape.
