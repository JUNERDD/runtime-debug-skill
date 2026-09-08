---
name: receiving-hack-review
description: Resolve hack-review reports or feedback about brittle shortcuts, duplicate ownership, masked causes, and boundary bypasses. Verify the current invariant owner and bounded intentional exceptions before applying scoped fixes. Use to challenge stale claims and account for every finding and ownership gap without blindly removing necessary guards.
---

# Receiving Hack Review

Treat hack-risk findings as claims about ownership and implementation liabilities. Verify the current invariant owner before changing a workaround, fallback, or competing abstraction.

## Skill Boundary

- Use `hack-review` to create or refresh the report.
- Use `receiving-hack-review` to consume the report and decide what to do next.
- Use `regression-review` instead when the concern is user-visible behavior rather than implementation shortcuts.
- Use `receiving-code-review` instead when the feedback is general code review rather than a hack-risk gate.

## Scope and Authorization

- Resolve the source, selected items, and requested outcome from the whole current conversation. A feedback assessment or review-only request ends with dispositions; implement only when the user has authorized fixes.
- When review and repair are already requested together, continue after the source report is complete. A phase change or a pre-edit plan does not require the user to repeat that authorization.
- Keep the source artifact unchanged. Record current evidence and dispositions in a concise final ledger or a separate resolution report; a report's recommendation is not permission to edit.
- Preserve unrelated work and pre-existing staged contents. Leave fixes unstaged unless the user has already authorized the specific staging, commit, or publication action for this task; repair permission alone does not include those actions.
- Do not stage to make a follow-up review easier. Verify working-tree fixes against the recorded baseline and disclose that the staged snapshot may still contain the original issue. Avoid tools with unrequested staging side effects; if the index is accidentally changed, restore only the known workflow delta without disturbing user work and disclose it.

## Workflow

1. Read the complete available report, governing requirements, and current scope. For PR comments or unstructured feedback, normalize the material claims into stable IDs and record the actual source; missing template sections alone do not require a new review.
2. Build a disposition ledger before code edits. Include these source items:
   - every `F#` from `Complete Hack-Risk Index` and the `Block`, `Discuss`, and `Watch` cards
   - every `I#` intentional exception and relevant `Ownership Coverage Ledger` row, including findings, unknown boundaries, and `Not covered` areas
   - mismatches between the source scope, index, action sections, and coverage ledger
3. Reconcile scope and evidence per item. Mark already-fixed, disproved, stale, out-of-scope, or unmappable claims explicitly. Reconstruct affected evidence locally where possible; regenerate only the affected scope when the source cannot support reliable item matching. An unclear target or contract blocks dependent edits, while independent confirmed work may continue. Keep unresolved coverage and integrity gaps visible; do not claim the whole gate is clear.
4. Verify each claim against its current behavior path and governing product or architecture intent. Reuse source traces only after checking that relevant code, inputs, contracts, and scope still match and that the evidence supports the claim; reconstruct changed, missing, or disputed portions. Current code and tests are behavioral evidence, not product authority by themselves. Lint, typecheck, and unrelated green tests do not prove a claim false.
5. Present a concise plan naming accepted items, affected files or boundaries, expected behavior, risks, and verification. Continue within existing authorization. Resolve technical risk through evidence, focused checks, or a narrower fix; ask only when an unresolved user decision or additional authority is needed, and pause only dependent work.
6. Apply the smallest cohesive changes for proven in-scope issues, preserving approved behavior and bounded exceptions. Prioritize blockers; an unresolved independent item does not prevent other confirmed fixes, but it still affects the final gate.
7. Verify affected behavior and repository-required checks, then update each disposition. A check may cover several related items. Reuse recorded results only when they apply to the final code and inputs, and identify their source; never describe reuse as a new run. Repeat or broaden checks only for new edits, failures, unresolved concerns, or required gates.

## Handle Each Item Type

### `Block`

Treat `Block` as "stop the change from shipping until fixed or disproven."

For each `Block` item:

- Reproduce the shortcut, or trace the current code path strongly enough to show the report is still correct.
- Fix the ownership problem or disprove the finding with stronger evidence than the report currently has.
- Prioritize proven blockers and preserve their gate effect while completing independent confirmed work.

Valid outcomes:

- Fix the owning layer and remove the shortcut.
- Prove the reported shortcut no longer exists or never owned the cited concern.
- Downgrade to `Discuss`, but only with concrete evidence.

### `Discuss`

Treat `Discuss` as "resolve uncertainty before approval."

- Clarify intent when the change may be a deliberate transition, migration shim, or compatibility layer.
- Gather the missing proof the report asked for.
- Prefer focused verification over speculative rewrites.
- Promote to `Block` if verification proves a serious ownership risk.
- Downgrade to `Watch` or `Intentional Exception` only with concrete evidence.

### `Watch`

Treat `Watch` as "note it, then decide whether cheap mitigation is worth it."

- Add a targeted test, owner note, TODO with exit trigger, or follow-up task when the debt matters.
- Avoid unnecessary churn when the risk is minor and already understood.
- Keep the item in the final disposition even when no code change is made.

### `Intentional Exceptions`

Treat `Intentional Exceptions` as protected shortcuts unless evidence says otherwise.

- Confirm scope, owner, and exit condition against the spec, issue, PR description, migration note, or user instruction.
- Leave the code alone if the exception is deliberate and bounded.
- Move it back into `Discuss` only when intent is unclear, the exception has grown beyond its stated scope, or the exit condition is gone.
- Do not refactor the exception away merely to silence the report.

### `Ownership Coverage Ledger`

Treat coverage rows as gate evidence, not background notes.

- For `Finding F#`, verify that the referenced finding is in the disposition ledger.
- For `Intentional Exception I#`, verify that the intentional exception has been confirmed or challenged.
- For `Reviewed - no hack-risk found`, leave the row alone unless current code or new evidence contradicts it.
- For `Not hack-relevant`, challenge the classification if the touched path owns an invariant, abstraction, lifecycle, cache, serializer, permission boundary, adapter, or extension point.
- For `Not covered`, either perform the missing ownership trace, regenerate the gate for that boundary, or leave it as an open coverage gap with a concrete next step.

## Verify Common Hack Leads Carefully

### Impossible-state fallback findings

- Identify who claims the state is impossible.
- Verify whether current code can still receive that state from a real external or legacy boundary.
- Fix the owner or the contract first. Deleting the fallback alone is not a fix if the invalid state is still produced upstream.

### Root-cause masking findings

- Trace where the bad state, invalid payload, or failure actually begins.
- Remove symptom patches only after the source is repaired or explicitly re-owned.
- Do not answer one local patch by adding a second patch earlier in the flow unless that earlier layer truly owns the invariant.

### Parallel wheel findings

- Compare the new implementation and the incumbent abstraction side by side.
- Prefer reusing or extending the incumbent boundary when it still owns the concern.
- If the new abstraction is actually better, migrate callers and retire the old one. Do not keep two accidental sources of truth.

## When to Push Back

Push back when:

- The report was generated for a different scope or stale branch.
- The report silently widened beyond the user-requested range.
- The report is incomplete but presents the gate as resolved.
- The `Complete Hack-Risk Index`, action sections, and `Ownership Coverage Ledger` disagree.
- The shortcut is a deliberate migration or compatibility layer with a clear owner and exit condition.
- The alleged impossible-state fallback is actually guarding a real external, legacy, or backward-compatibility boundary.
- The alleged duplicate abstraction owns a materially different boundary.
- Current runtime, output, search, or ownership evidence contradicts the report.
- A `Not covered` row requires context, credentials, data, platform access, or runtime setup that is not available in the current environment.

Push back with evidence, not tone:

- Cite the current code path, runtime result, owning abstraction, or search result.
- State what the report got right and what no longer applies.
- Say what additional verification would settle the disagreement if proof is still incomplete.

## Completion and Follow-Up

Finish with the source identity, per-item dispositions, focused changes, verification, remaining risks, and actual Git state. Do not claim resolution while a source item lacks a disposition or an implemented item lacks appropriate evidence. Separate a resolved subset from an unresolved overall gate.

Use at most one post-fix review in this resolution when the user requested it or independent review would materially improve confidence. Limit it to the implementation delta and affected boundaries, carrying forward settled intent and disproved claims unless relevant code, contracts, or evidence changed. Return its remaining findings; do not automatically start another receiving cycle. A changed line count, new artifact, or phase handoff alone does not require another full review.

Before final verification, correct failures caused by the current patch when evidence supports an in-scope repair. Report materially distinct discoveries outside the accepted item set without silently expanding the task.

## Disposition Ledger Format

Use this shape in the final response or a separate resolution report when multiple items were consumed:

```md
| ID / boundary | Original status | Disposition | Evidence | Next action |
| --- | --- | --- | --- | --- |
| F1 | Block | Fixed | Invariant now enforced in the normalizer; fallback removed from caller. | None |
| F2 | Discuss | Narrowed | Extra guard protects a legacy input boundary only. | Note in review |
| I1 | Intentional Exception | Confirmed | Migration note lists owner and removal trigger. | Leave unchanged |
| Cache invalidation path | Not covered | Open | Requires integration data not available locally. | Regenerate with staging logs |
```

Keep it concise, but account for every report item.
