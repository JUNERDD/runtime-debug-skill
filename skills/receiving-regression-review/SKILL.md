---
name: receiving-regression-review
description: Resolve regression-review reports or feedback about user-visible behavior changes. Verify current inputs, guards, outputs, intended product changes, and coverage before fixing proven regressions. Use to challenge stale or incorrect claims and produce a disposition for every finding, intentional change, and open surface.
---

# Receiving Regression Review

Treat regression findings as claims about user-visible outcomes. Verify the current path and product intent before restoring older behavior; a behavior-graph delta is supporting evidence, not an edit instruction.

## Skill Boundary

- Use `regression-review` to create or refresh the report.
- Use `receiving-regression-review` to consume the report and decide what to do next.
- Use `receiving-code-review` instead when the feedback is general code review rather than a user-visible regression gate.

## Scope and Authorization

- Resolve the source, selected items, and requested outcome from the whole current conversation. A feedback assessment or review-only request ends with dispositions; implement only when the user has authorized fixes.
- When review and repair are already requested together, continue after the source report is complete. A phase change or a pre-edit plan does not require the user to repeat that authorization.
- Keep the source artifact unchanged. Record current evidence and dispositions in a concise final ledger or a separate resolution report; a report's recommendation is not permission to edit.
- Preserve unrelated work and pre-existing staged contents. Leave fixes unstaged unless the user has already authorized the specific staging, commit, or publication action for this task; repair permission alone does not include those actions.
- Do not stage to make a follow-up review easier. Verify working-tree fixes against the recorded baseline and disclose that the staged snapshot may still contain the original issue. Avoid tools with unrequested staging side effects; if the index is accidentally changed, restore only the known workflow delta without disturbing user work and disclose it.

## Workflow

1. Read the complete available report, governing requirements, and current scope. For PR comments or unstructured feedback, normalize the material claims into stable IDs and record the actual source; missing template sections alone do not require a new review.
2. Build a disposition ledger before code edits. Include these source items:
   - every `F#` from `Complete Findings Index` and the `Block`, `Discuss`, and `Watch` cards
   - every `I#` intentional change and relevant `Coverage Ledger` row, including unknown or `Not covered` surfaces
   - behavior-graph deltas affecting inputs, guards, transforms, outputs, or effects, and mismatches with the source scope, findings, or coverage
3. Reconcile scope and evidence per item. Mark already-fixed, disproved, stale, out-of-scope, or unmappable claims explicitly. Reconstruct affected evidence locally where possible; regenerate only the affected scope when the source cannot support reliable item matching. An unclear target or contract blocks dependent edits, while independent confirmed work may continue. Keep unresolved coverage and integrity gaps visible; do not claim the whole gate is clear.
4. Verify each claim against its current behavior path and governing product or architecture intent. Reuse source traces only after checking that relevant code, inputs, contracts, and scope still match and that the evidence supports the claim; reconstruct changed, missing, or disputed portions. Current code and tests are behavioral evidence, not product authority by themselves. Lint, typecheck, and unrelated green tests do not prove a claim false.
5. Present a concise plan naming accepted items, affected files or boundaries, expected behavior, risks, and verification. Continue within existing authorization. Resolve technical risk through evidence, focused checks, or a narrower fix; ask only when an unresolved user decision or additional authority is needed, and pause only dependent work.
6. Apply the smallest cohesive changes for proven in-scope issues, preserving approved behavior and bounded exceptions. Prioritize blockers; an unresolved independent item does not prevent other confirmed fixes, but it still affects the final gate.
7. Verify affected behavior and repository-required checks, then update each disposition. A check may cover several related items. Reuse recorded results only when they apply to the final code and inputs, and identify their source; never describe reuse as a new run. Repeat or broaden checks only for new edits, failures, unresolved concerns, or required gates.

## Handle Each Item Type

### `Block`

Treat `Block` as "stop the change from shipping until fixed or disproven."

For each `Block` item:

- Reproduce the user-visible breakage, or trace the current code path strongly enough to show the report is still correct.
- Fix the behavior or disprove the finding with stronger evidence than the report currently has.
- Prioritize proven blockers and preserve their gate effect while completing independent confirmed work.

Valid outcomes:

- Fix the regression.
- Prove an equivalent guard or output path still exists.
- Downgrade to `Discuss`, but only with concrete evidence.

### `Discuss`

Treat `Discuss` as "resolve uncertainty before approval."

- Clarify product intent when the change may be deliberate.
- Gather the missing proof the report asked for.
- Prefer focused verification over speculative fixes.
- Promote to `Block` if verification proves a serious user-visible break.
- Downgrade to `Watch` or `Intentional` only with concrete evidence.

### `Watch`

Treat `Watch` as "note it, then decide whether cheap mitigation is worth it."

- Add a targeted test, monitor, or follow-up when the caveat matters.
- Avoid unnecessary churn when the risk is minor and already understood.
- Keep the item in the final disposition even when no code change is made.

### `Intentional Changes`

Treat `Intentional Changes` as protected product deltas unless evidence says otherwise.

- Confirm intent against the spec, issue, PR description, or user instruction.
- Leave the change alone if it is deliberate.
- Move it back into `Discuss` only when intent is unclear or contradictory.
- Do not make user-visible behavior match the old baseline merely to silence the report.

### `Behavior Graph Deltas`

Treat behavior graph rows as route evidence.

- Verify the current entry, input, guard, transform, and output/effect path before fixing or dismissing the linked item.
- If a graph delta shows a changed guard, input, transform, or output/effect that is not represented by a finding, intentional change, or coverage row, record an intake inconsistency and reconcile the affected items before dependent edits.
- If the report uses a direct trace instead of a graph, verify that path evidence. Record a coverage gap only when the affected behavior remains untraced or uncertain.
- Do not preserve the old behavior merely because the graph changed; first decide whether the delta is a regression, an intentional product change, or a harmless implementation detail.

### `Coverage Ledger`

Treat coverage rows as gate evidence, not background notes.

- For `Finding F#`, verify that the referenced finding is in the disposition ledger.
- For `Intentional I#`, verify that the intentional change has been confirmed or challenged.
- For `Reviewed - no user-visible regression found`, leave the row alone unless current code or new evidence contradicts it.
- For `Not user-visible`, challenge the classification if the touched path can affect routes, commands, outputs, persistence, scheduled jobs, or externally consumed data.
- For `Not covered`, either perform the missing verification, regenerate the gate for that surface, or leave it as an open coverage gap with a concrete next step.

## When to Push Back

Push back when:

- The report was generated for a different scope or stale branch.
- The report is incomplete but presents the gate as resolved.
- The `Complete Findings Index`, action sections, and `Coverage Ledger` disagree.
- `Behavior Graph Deltas` contradict or bypass the report's findings and coverage ledger.
- The finding describes an intended product change, not a regression.
- Current runtime or output evidence contradicts the report.
- The user-visible path is no longer reachable.
- The report misses a guard, fallback, or idempotency control that now lives elsewhere.
- A `Not covered` row requires credentials, data, platform access, or runtime setup that is not available in the current environment.

Push back with evidence, not tone:

- Cite the current code path, output, test, log, screenshot, or runtime result.
- State what the report got right and what no longer applies.
- Say what additional verification would settle the disagreement if proof is still incomplete.

## Completion and Follow-Up

Finish with the source identity, per-item dispositions, focused changes, verification, remaining risks, and actual Git state. Do not claim resolution while a source item lacks a disposition or an implemented item lacks appropriate evidence. Separate a resolved subset from an unresolved overall gate.

Use at most one post-fix review in this resolution when the user requested it or independent review would materially improve confidence. Limit it to the implementation delta and affected boundaries, carrying forward settled intent and disproved claims unless relevant code, contracts, or evidence changed. Return its remaining findings; do not automatically start another receiving cycle. A changed line count, new artifact, or phase handoff alone does not require another full review.

Before final verification, correct failures caused by the current patch when evidence supports an in-scope repair. Report materially distinct discoveries outside the accepted item set without silently expanding the task.

## Disposition Ledger Format

Use this shape in the final response or a separate resolution report when multiple items were consumed:

```md
| ID / surface | Original status | Disposition | Evidence | Next action |
| --- | --- | --- | --- | --- |
| F1 | Block | Fixed | Targeted test now passes; checkout guard restored. | None |
| F2 | Discuss | Disproved | Current serializer still emits legacy field. | Note in review |
| I1 | Intentional | Confirmed | Matches issue acceptance criteria. | Leave unchanged |
| Settings export | Not covered | Closed | Verified CLI output fixture. | None |
```

Keep it concise, but account for every report item.
