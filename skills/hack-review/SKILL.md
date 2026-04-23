---
name: hack-review
description: Perform a scoped, coverage-led review of a working tree, staged diff, commit range, branch diff, PR, or suspicious implementation to find hack-like implementation risks. Use when Codex must audit brittle shortcuts such as impossible-state fallbacks, masked root causes, duplicate abstractions, hardcoded special cases, boundary bypasses, hidden temporal coupling, write-then-fix-up flows, or other ownership problems, and must write a Markdown report that enumerates all distinct hack-risk findings discovered within the reviewed scope plus coverage gaps and intentional exceptions.
---

# Hack Review

## Overview

Treat the change set as a scoped hack-risk audit with an engineering gate recommendation. The primary job is to enumerate every distinct structural liability that can be reasonably identified within the reviewed scope, not only the most severe or easiest findings.

Always produce a Markdown report file and a short terminal summary.

Keep the overall recommendation mechanically aligned with the highest-severity unresolved finding and the coverage state. Do not let prose tone drift the gate up or down.

## Skill Boundary

- Use `hack-review` to create or refresh the report.
- Use `receiving-hack-review` to consume the report and decide what to do next.
- Use `regression-review` instead when the main question is user-visible behavioral regression.
- Use `receiving-code-review` instead when the feedback is general review feedback rather than a hack-risk gate.

## Set Scope First

- Prefer an explicit scope from the user: working tree, staged changes, last commit, commit range, branch diff, PR, or a named implementation slice.
- Strictly honor the user-specified scope. Do not silently widen the review to unstaged changes, neighboring refactors, or the whole branch unless the user explicitly asks for that broader range.
- If the user does not specify scope, default to the staged diff.
- If nothing is staged and no scope was specified, stop and ask whether to review the working tree, last commit, commit range, branch diff, or a named implementation slice. Do not silently fall back to the working tree.
- If the requested scope is too large to review completely in one pass, do not silently sample it. Review the highest-risk boundaries first, mark the report `Incomplete`, list the exact files or boundaries not covered, and set the recommendation no lower than `Discuss` unless the uncovered area is demonstrably not implementation-relevant.
- If requirements, issue text, design docs, migration notes, or architecture docs exist, read them before judging whether a shortcut is accidental or deliberate.
- Separate bounded intentional exceptions from accidental hacks.

## Completeness Contract

- Output every distinct hack-risk finding discovered within the reviewed scope. Do not stop after the top 3.
- Use `Must-review now` only as a short priority preview. The full findings sections and `Complete Hack-Risk Index` must include all findings.
- Maintain an `Ownership Coverage Ledger` that maps every touched implementation boundary, invariant, abstraction, adapter, serializer, store, cache, lifecycle path, or extension point to one of:
  - `Finding F#`
  - `Intentional Exception I#`
  - `Reviewed - no hack-risk found`
  - `Not hack-relevant`
  - `Not covered`
- Do not drop lower-severity shortcuts. Put them in `Watch` or `Intentional Exceptions` when they are real but not blocking.
- If a candidate issue is investigated and dismissed, record the dismissal in the evidence appendix when it explains coverage or prevents duplicate review.
- If token budget, missing context, runtime setup, or diff size prevents complete coverage, say exactly where coverage stopped. A partial review must not be presented as complete.

## Output Rules

- Always write a Markdown file.
- If the repo already has an obvious location for reviews or reports, follow that convention.
- Otherwise write to `tmp/reviews/YYYY-MM-DD-hack-review-report.md`.
- End with a short terminal summary that includes the report path, the gate recommendation, completion status, counts by action, and the top risks.
- Structure the report in this order:
  - `Scope`
  - `Gate Snapshot`
  - `Complete Hack-Risk Index`
  - `Block`
  - `Discuss`
  - `Watch`
  - `Intentional Exceptions`
  - `Ownership Coverage Ledger`
  - `Evidence Appendix`
- Keep the primary view optimized for review reading speed, but do not omit findings for brevity.
- Do not use wide summary tables as the main presentation shape for finding details. Tables are appropriate for the findings index, coverage ledger, and evidence appendix.

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
- Else if coverage is incomplete for an implementation-relevant or unknown-impact boundary, the report recommendation MUST be `Discuss`.
- Else if any unresolved finding is `Watch`, the report recommendation MUST be `Pass with caveat`.
- Else use `Pass`.

Additional rules:

- Do not write `Pass with caveat` when a `Discuss` item is still open.
- Do not write `Discuss` when the body contains no `Discuss` items and no incomplete implementation-relevant coverage.
- If a suspected hack lacks enough proof to justify `Block`, downgrade the finding to `Discuss` instead of keeping `Block` with hand-wavy evidence.
- If the report contains both `Block` and `Discuss`, keep both sections, but the top-level recommendation remains `Block`.

## What Counts As One Finding

- Count one item per distinct structural liability, not per file, function, smell label, or repeated syntax pattern.
- Merge multiple code changes that express the same shortcut or ownership violation.
- Split items when different boundaries, invariants, owners, or failure modes are affected in materially different ways.
- Focus on findings that matter during review:
  - impossible-state fallbacks, retries, or extra validation that hide a broken invariant
  - temporary patches that make the symptom disappear without fixing the source
  - new helpers, hooks, services, serializers, validators, or adapters that recreate an existing abstraction
  - hardcoded special cases keyed on one ID, tenant, route, flag, environment, or literal string
  - bypassing shared boundaries such as caches, stores, permissions, serializers, or lifecycle ownership
  - write-then-fix-up or retry-until-it-works sequences that stand in for real coordination
  - shadow state or duplicated truth that lets later fixes land in the wrong layer

## Evidence Standard

- Prefer the strongest feasible evidence for the suspected shortcut, not one fixed verification mode.
- Runtime checks are valuable, but they are not mandatory when static ownership tracing, existing-abstraction search, or output inspection already provides strong evidence.
- Search the codebase for the current owner of the invariant, abstraction, normalization path, serializer, cache, lifecycle, or permission boundary before calling something a duplicate wheel or bypass.
- For impossible-state fallback findings, identify who is supposed to make the state impossible.
- For root-cause masking findings, separate symptom suppression from source repair.
- For duplicate-abstraction findings, show the incumbent abstraction and explain what ownership now overlaps.
- For boundary-bypass findings, show the shared boundary and the path that reaches around it.
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
   - Record the exact scope and baseline in the report.
2. Build a diff inventory.
   - List touched files and classify each as implementation boundary, implementation dependency, test-only, docs-only, generated, config, or unknown.
   - Use repository structure, exports, call sites, dependency injection, lifecycle hooks, feature flags, and ownership boundaries to avoid missing indirect implementation paths.
3. Build the ownership coverage ledger before writing findings.
   - Identify every helper, hook, service, adapter, middleware, cache, serializer, validator, store, permission boundary, lifecycle path, config default, feature flag, invariant, and extension point touched by the diff.
   - Add each boundary to the ledger even if it later has no finding.
4. Trace shortcut deltas for every ledger boundary.
   - Compare before and after ownership for each boundary.
   - Look for added fallback branches for supposedly unreachable states, patch layers that only absorb symptoms, duplicated local helpers, hardcoded special cases, timing hacks, retry loops, shadow state, and bypassed shared abstractions.
5. Gather proof and code pointers.
   - Collect the strongest runtime, output, search, fixture, log, test, or code-path evidence available.
   - Trace each finding back to concrete code lines.
6. De-duplicate and classify.
   - Merge candidates with the same structural liability.
   - Rank within each section by engineering risk first, confidence second.
7. Write the report from `references/report-template.md`.
   - Include all findings in `Complete Hack-Risk Index` and the matching action sections.
   - Put priority highlights in `Must-review now`, but never use that list as the full result.
8. Run the report self-check.
   - Every touched implementation-relevant or unknown boundary is present in `Ownership Coverage Ledger`.
   - Every finding in the action sections appears in `Complete Hack-Risk Index`.
   - Every `Finding F#` in `Ownership Coverage Ledger` has a matching card.
   - Every `Not covered` row has a reason and a concrete next step.
   - The recommendation matches the mapping rules.

## Card Format

Each risk should be written as a short review card, not a spreadsheet row. Repeat the card format for every finding in the section; do not cap sections at one item.

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

- Start with the gate decision and completion status.
- `Gate Snapshot` should fit on one screen when possible.
- `Gate Snapshot` should usually include:
  - `Recommendation`
  - `Completion`
  - `Why now`
  - `Must-review now`
  - `Findings count`
  - `Coverage confidence`
  - `Biggest blind spot`
- Limit `Must-review now` to the top 3 items, and explicitly point to `Complete Hack-Risk Index` for the full list.
- In each review card, keep the first sentence about engineering impact, not code mechanics.
- Use exactly 1 or 2 links under `Look here first`.
- Put the full causal chain in `Evidence Appendix`, not in the main card.
- Use absolute file paths when the environment supports clickable local links. Include line anchors when available, for example `[normalizer.ts](/abs/path/lib/normalizer.ts#L128)` or `[normalizer.ts](/abs/path/lib/normalizer.ts:128)`.
- `Intentional Exceptions` should be compact and easy to skim.
- `Ownership Coverage Ledger` must include every reviewed implementation-relevant or unknown boundary, not just boundaries with findings.
- `Evidence Appendix` may use small tables.
- If no hack-risk findings are found, still write the report:
  - Say the recommendation is `Pass` or `Pass with caveat`
  - Include an empty `Complete Hack-Risk Index`
  - Include the full `Ownership Coverage Ledger`
  - State the strongest blind spot
  - Document what was verified
- If the report is based mainly on static reasoning, say that plainly in both `Gate Snapshot` and `Ownership Coverage Ledger`.
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
  - Does the code branch on one ID, tenant, route, environment, flag, or literal to patch a local problem without fixing the general model?
- Hidden temporal coupling:
  - Do sleeps, retries, ordering assumptions, or "check one more time" logic stand in for explicit lifecycle or synchronization ownership?
- Boundary bypass:
  - Does a caller reach around shared APIs, caches, permission gates, serializers, stores, or lifecycle owners to make one path work?

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
- Do not hide lower-severity shortcuts because stronger shortcuts already exist.
- Do not claim complete coverage unless the ownership coverage ledger accounts for every touched implementation-relevant or unknown boundary.
- If a runtime check cannot be performed, say so and downgrade confidence rather than hiding the gap.
- Do not discard a strong duplicate-abstraction lead just because tests pass.
- Do not promote a finding to `Block` unless the hack risk itself, not just the code smell, is strongly supported.

## Reference

- Use [references/report-template.md](references/report-template.md) as the default output shape.
