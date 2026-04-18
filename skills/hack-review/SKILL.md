---
name: hack-review
description: Use when reviewing a working tree, staged diff, commit range, branch diff, or suspicious implementation and you need to decide whether the code relies on brittle shortcuts such as impossible-state fallbacks, masked root causes, duplicate abstractions, hardcoded special cases, or other hack-like implementation tactics that should be blocked, discussed, watched, or accepted as an intentional exception.
---

# Hack Review

## Overview

Treat the change set as an engineering review gate, not as a style audit. The primary question is whether a reviewer or TL should block because the implementation reaches correctness through brittle shortcuts, hidden ownership changes, or patch layers that are likely to rot.

Always produce a Markdown report file and a short terminal summary.

Keep the overall recommendation mechanically aligned with the highest-severity unresolved finding. Do not let prose tone drift the gate up or down.

## Skill Boundary

- Use `hack-review` to create the report.
- Use `receiving-hack-review` to consume the report and decide what to do next.
- Use `regression-review` instead when the main question is user-visible behavioral regression.
- Use `receiving-code-review` instead when the feedback is general review feedback rather than a hack-risk gate.

## Set Scope First

- Prefer an explicit scope from the user: working tree, staged changes, last commit, commit range, branch diff, or a named implementation slice.
- Strictly honor the user-specified scope. Do not silently widen the review to unstaged changes, neighboring refactors, or the whole branch unless the user explicitly asks for that broader range.
- If the user does not specify scope, default to the staged diff.
- If nothing is staged and no scope was specified, stop and ask whether to review the working tree, last commit, commit range, or branch diff. Do not silently fall back to the working tree.
- If requirements, issue text, design docs, or migration notes exist, read them before judging whether a shortcut is accidental or deliberate.
- Separate bounded exceptions from accidental hacks.

## Output Rules

- Always write a Markdown file.
- If the repo already has an obvious location for reviews or reports, follow that convention.
- Otherwise write to `tmp/reviews/YYYY-MM-DD-hack-review-report.md`.
- End with a short terminal summary that includes the report path, the gate recommendation, and the top risks.
- Structure the report in this order:
  - `Gate Snapshot`
  - `Block`
  - `Discuss`
  - `Watch`
  - `Intentional Exceptions`
  - `Evidence Appendix`
- Keep the primary view optimized for code review reading speed.
- Do not use wide summary tables as the main presentation shape.

## Review Actions

Use these action labels as the primary classification:

- `Block`
  - Strong evidence that the implementation reaches correctness through a brittle or misleading shortcut that should be stopped before merge.
  - Typical examples: impossible-state fallback creep, root-cause masking, duplicated ownership of an existing abstraction, hardcoded special-case logic, hidden timing coupling, or bypassing the real boundary that should own the fix.
- `Discuss`
  - The code smells like a hack, but intent, ownership, blast radius, or the better abstraction is still unclear.
  - A reviewer should resolve the uncertainty before approval.
- `Watch`
  - The shortcut or debt is real, but it is bounded enough that approval may proceed with a caveat, a follow-up task, a test, a comment, or a timebox.
- `Intentional Exception`
  - The shortcut appears deliberate, bounded, and documented enough that it should not be framed as an accidental hack.

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
- If a suspected hack lacks enough proof to justify `Block`, downgrade the finding to `Discuss` instead of keeping `Block` with hand-wavy evidence.
- If the report contains both `Block` and `Discuss`, keep both sections, but the top-level recommendation remains `Block`.

## What Counts As One Finding

- Count one item per distinct structural liability, not per file, function, or smell label.
- Merge multiple code changes that express the same shortcut or ownership violation.
- Split items only when different boundaries, invariants, or failure modes are affected in materially different ways.
- Focus on findings that matter during review:
  - impossible-state fallbacks, retries, or extra validation that hide a broken invariant
  - temporary patches that make the symptom disappear without fixing the source
  - new helpers, hooks, services, serializers, validators, or adapters that recreate an existing abstraction
  - hardcoded special cases keyed on one ID, tenant, route, flag, or literal string
  - bypassing shared boundaries such as caches, stores, permissions, or lifecycle ownership
  - write-then-fix-up or retry-until-it-works sequences that stand in for real coordination

## Evidence Standard

- Prefer the strongest feasible evidence for the suspected shortcut, not one fixed verification mode.
- Runtime checks are valuable, but they are not mandatory when static ownership tracing or output inspection already provides strong evidence.
- Search the codebase for the current owner of the invariant, abstraction, or normalization path before calling something a duplicate wheel.
- For impossible-state fallback findings, identify who is supposed to make the state impossible.
- For root-cause masking findings, separate symptom suppression from source repair.
- For duplicate-abstraction findings, show the incumbent abstraction and explain what ownership now overlaps.
- If evidence is missing, lower confidence and say so plainly.
- Separate verified facts from inferred maintenance or runtime risk.
- Do not present taste or aesthetic preference as fact.
- Do not treat `eslint`, `typecheck`, or passing unrelated tests as proof that a hack is disproven. They are hygiene evidence only.
- When a spec, issue, migration plan, or compatibility note exists, use it to judge whether an exception is intentional, but do not treat it as proof that the implementation is sound.

## Workflow

1. Define the comparison baseline.
   - Use the user-specified scope exactly as given.
   - If no scope was specified, use `git diff --staged`.
   - Only use `git diff`, `git show`, or an explicit range when the user requested that scope or clarified it after no staged diff was available.
   - Record the exact scope in the report.
2. Map the touched abstractions and ownership boundaries.
   - Identify helpers, hooks, services, adapters, middleware, caches, serializers, validators, stores, and feature flags touched by the diff.
   - Build a short list of the invariants and extension points this code is supposed to respect.
3. Trace the shortcut delta.
   - Compare before and after ownership for each relevant flow.
   - Look for added fallback branches for supposedly unreachable states, patch layers that only absorb symptoms, duplicated local helpers, hardcoded special cases, timing hacks, and bypassed shared abstractions.
4. Gather proof and code pointers.
   - Collect the strongest runtime, output, search, or code-path evidence available.
   - Trace each finding back to concrete code lines.
5. Classify for review action.
   - Decide whether the reviewer should `Block`, `Discuss`, `Watch`, or mark the change as an `Intentional Exception`.
   - Rank within each section by engineering risk first, confidence second.
6. Write the report.
   - Use `references/report-template.md`.
   - Lead with the gate recommendation and the top items a reviewer should open first.

## Card Format

Each risk should be written as a short review card, not a spreadsheet row.

Use this shape:

```md
### F1 Block - New fallback masks a broken invariant

Engineering impact: The code now maintains two competing truths for the same state, so later fixes can land in the wrong layer.
Review reason: The change handles a state that the upstream normalizer is supposed to make unreachable.
Surface: Order summary loader
Confidence: High

Look here first:
- [new fallback](/abs/path/app/order-summary.ts#L128)
- [existing normalizer](/abs/path/lib/normalize-order.ts#L42)

Implementation delta:
- Before: The loader consumed normalized records and failed at the owning layer when normalization broke.
- After: The loader fabricates a fallback record when normalization returns an impossible shape.

Evidence:
- The existing normalizer still claims ownership of this invariant.
- No upstream contract change or migration note accompanies the new fallback.

Reviewer action:
Block until the invariant is restored at the owning layer or the contract change is made explicit.
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
- In each review card, keep the first sentence about engineering impact, not code mechanics.
- Use exactly 1 or 2 links under `Look here first`.
- Put the full causal chain in `Evidence Appendix`, not in the main card.
- Use absolute file paths when the environment supports clickable local links. Include line anchors when available, for example `[checkout.tsx](/abs/path/app/checkout.tsx#L128)` or `[checkout.tsx](/abs/path/app/checkout.tsx:128)`.
- `Intentional Exceptions` should be compact and easy to skim.
- `Evidence Appendix` may use small tables.
- If no hack-risk findings are found, still write the report:
  - Say the recommendation is `Pass` or `Pass with caveat`
  - State the strongest blind spot
  - Document what was verified
- If the report is based mainly on static reasoning, say that plainly in both `Gate Snapshot` and `Coverage`.
- When evidence is mixed, write the verified code-path facts first, then the inferred engineering risk second.

## Common Hack Leads

When the diff claims to be a cleanup, guard, refactor, compatibility fix, or quick patch, check these explicitly before deciding it is harmless:

- Impossible-state fallback creep:
  - Did the code add fallback, retry, recovery, or extra validation for a state the owning abstraction says cannot happen?
  - If yes, ask whether the owner is broken or the contract changed.
- Root-cause masking:
  - Does the change suppress a symptom with default values, catch-and-ignore, shadow state, forced refresh, null guards, or extra retries without preventing the invalid state from being produced?
- Parallel wheel:
  - Does a new helper, wrapper, hook, serializer, validator, adapter, or store recreate a concern that an existing abstraction already owns?
- Hardcoded special case:
  - Does the code branch on one ID, tenant, route, environment, or literal to patch a local problem without fixing the general model?
- Hidden temporal coupling:
  - Do sleeps, retries, ordering assumptions, or "check one more time" logic stand in for explicit lifecycle or synchronization ownership?
- Boundary bypass:
  - Does a caller reach around shared APIs, caches, permission gates, serializers, or stores to make one path work?

For any suspected hack, explicitly compare:

- where the invariant is supposed to be owned
- where the invalid state is actually produced
- where the patch now absorbs it
- which sibling paths still depend on the old abstraction

If those layers are no longer aligned, treat that as a hack lead even before runtime breakage.

## Guardrails

- Do not call a deliberate migration shim, compatibility layer, or temporary exception a hack if it has a clear scope, an owner, and an exit trigger.
- Do not confuse "defensive against external input" with "papering over an impossible internal state."
- Do not block merely because code is ugly; block when the shortcut undermines ownership, truthfulness, or future correctness.
- If the diff is large, cover the highest-risk boundaries first and say where coverage stops.
- If a runtime check cannot be performed, say so and downgrade confidence rather than hiding the gap.
- Do not discard a strong duplicate-abstraction lead just because tests pass.
- Do not promote a finding to `Block` unless the hack risk itself, not just the code smell, is strongly supported.

## Reference

- Use [references/report-template.md](references/report-template.md) as the default output shape.
