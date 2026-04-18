---
name: receiving-hack-review
description: Use when Codex is given a `hack-review` report, PR feedback derived from one, or a request to address `Block`, `Discuss`, `Watch`, or `Intentional Exceptions` items and must verify whether the reported shortcut or ownership problem still applies before changing code.
---

# Receiving Hack Review

## Overview

Use this skill after `hack-review` has produced a gate report. Treat the report as an evidence-backed design review artifact, not as an instruction list to execute blindly.

## Skill Boundary

- Use `hack-review` to create the report.
- Use `receiving-hack-review` to consume the report and decide what to do next.
- Use `regression-review` instead when the concern is user-visible behavior rather than implementation shortcuts.
- Use `receiving-code-review` instead when the feedback is general code review rather than a hack-risk gate.

## Core Principle

Verify the current ownership model before implementing a fix.

Keep gate integrity aligned with evidence:

- Do not dismiss a `Block` item without stronger counter-evidence.
- Do not "fix" an `Intentional Exceptions` item into a sweeping refactor unless product or architecture intent changed.
- Do not treat lint, typecheck, or unrelated passing tests as proof that a shortcut is gone.
- Do not remove necessary edge guards merely because the report called out a nearby fallback. Confirm which boundary actually owns the invariant first.

## Response Pattern

WHEN receiving a hack review report:

1. Read the full report, including `Scope`, `Gate Snapshot`, and `Evidence Appendix`.
2. Confirm the report still applies to the exact scoped change set the user asked to review.
3. Restate each finding as a concrete design liability, not as a code edit.
4. Verify the finding against current code, outputs, tests, runtime behavior, and ownership boundaries.
5. Decide the disposition: fix, disprove, narrow, clarify, or confirm as intentional.
6. Address findings in gate order and verify each change before moving on.
7. Refresh the gate by rerunning `hack-review` or updating the reviewer with concrete evidence.

## Intake Checklist

Before changing code, confirm:

- Which scope was reviewed: working tree, staged diff, commit range, branch diff, or a named implementation slice.
- Whether that scope matches the user's requested review range exactly.
- If the user did not specify scope, whether the report was generated from the staged diff.
- Which baseline the report compares against.
- Whether the current checkout still matches that scope.
- Which items are unresolved in `Block`, `Discuss`, and `Watch`.
- Which incumbent abstraction, invariant owner, or shared boundary the report points to.
- Which blind spots limit confidence.

If scope, baseline, or owning abstraction is stale or unclear, stop and regenerate or clarify the report before implementing.
If the report was generated from a broader or different range than the user asked for, do not consume it as-is. Regenerate the gate for the correct scope first.

## Handle Each Section

### `Block`

Treat `Block` as "stop the change from shipping until fixed or disproven."

For each `Block` item:

- Reproduce the shortcut, or trace the current code path strongly enough to show the report is still correct.
- Fix the ownership problem or disprove the finding with stronger evidence than the report currently has.
- Do not skip ahead to lower-severity cleanup while a real `Block` item remains unresolved.

Valid outcomes:

- Fix the owning layer and remove the shortcut.
- Prove the reported shortcut no longer exists or never owned the cited concern.
- Downgrade to `Discuss`, but only with concrete evidence.

### `Discuss`

Treat `Discuss` as "resolve uncertainty before approval."

- Clarify intent when the change may be a deliberate transition, migration shim, or compatibility layer.
- Gather the missing proof the report asked for.
- Prefer focused verification over speculative rewrites.

### `Watch`

Treat `Watch` as "note it, then decide whether cheap mitigation is worth it."

- Add a targeted test, owner note, TODO with exit trigger, or follow-up task when the debt matters.
- Avoid unnecessary churn when the risk is minor and already understood.

### `Intentional Exceptions`

Treat `Intentional Exceptions` as protected shortcuts unless evidence says otherwise.

- Confirm scope, owner, and exit condition against the spec, issue, PR description, or user instruction.
- Leave the code alone if the exception is deliberate and bounded.
- Move it back into `Discuss` only when intent is unclear, the exception has grown beyond its stated scope, or the exit condition is gone.

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
- The shortcut is a deliberate migration or compatibility layer with a clear owner and exit condition.
- The alleged impossible-state fallback is actually guarding a real external, legacy, or backward-compatibility boundary.
- The alleged duplicate abstraction owns a materially different seam.
- Current runtime or output evidence contradicts the report.

Push back with evidence, not tone:

- Cite the current code path, runtime result, or owning abstraction.
- State what the report got right and what no longer applies.
- Say what additional verification would settle the disagreement if proof is still incomplete.

## Implementation Order

For multi-item reports:

1. Clarify stale or unclear scope first.
2. Regenerate the report if it does not match the user's exact review range, or if no scope was specified and the report was not based on the staged diff.
3. Fix or disprove every unresolved `Block` item.
4. Resolve `Discuss` items with proof or intent clarification.
5. Decide whether `Watch` items need mitigation now.
6. Leave `Intentional Exceptions` alone unless intent changed.
7. Re-run targeted verification for every touched boundary.
8. Refresh the hack gate if your changes materially altered the implementation strategy.

## Response Style

Do not use performative agreement. Use short technical acknowledgments.

Good:

- `F1 still applies on the current diff. The new fallback is hiding a broken invariant in [file]. Restoring ownership upstream.`
- `F2 is narrower than reported. The extra guard protects legacy input at the boundary, so downgrading from Block to Watch.`
- `F3 is real duplicated logic. Reusing the existing serializer instead of keeping parallel mapping.`

Bad:

- `You're absolutely right.`
- `Great catch, I'll refactor all of this now.`
- `Thanks for the detailed report.`

## Common Mistakes

- Treat the report title as the bug instead of the underlying ownership problem.
- Delete the fallback without fixing who should have prevented the bad state.
- Keep the symptom patch and add another patch upstream.
- Replace one duplicated wheel with a different duplicated wheel.
- Downgrade a `Block` item without stronger evidence.
- Stop after code changes without rerunning the affected path, ownership trace, or targeted checks.

## Bottom Line

A hack review report is a gate artifact. Consume it the same way a strong reviewer would: verify the current ownership model, preserve the severity semantics, then fix, challenge, narrow, or confirm each finding with evidence.
