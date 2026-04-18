---
name: receiving-regression-review
description: "Consume a `regression-review` gate report and turn it into the next technical action: verify whether each user-visible regression finding still applies, decide whether to fix, challenge, or leave it as intentional, and only then change code. Use when Codex is given a regression-review Markdown report, PR feedback derived from one, or a request to address `Block`, `Discuss`, `Watch`, or `Intentional Changes` items."
---

# Receiving Regression Review

## Overview

Use this skill after `regression-review` has produced a gate report. Treat the report as an evidence-backed review artifact, not as an instruction list to execute blindly.

## Skill Boundary

- Use `regression-review` to create the report.
- Use `receiving-regression-review` to consume the report and decide what to do next.
- Use `receiving-code-review` instead when the feedback is general code review rather than a user-visible regression gate.

## Core Principle

Verify the current user-visible behavior before implementing a fix.

Keep gate integrity aligned with evidence:

- Do not dismiss a `Block` item without stronger counter-evidence.
- Do not "fix" an `Intentional Changes` item back to the old behavior unless product intent changed.
- Do not treat lint, typecheck, or unrelated passing tests as proof that a finding is false.

## Response Pattern

WHEN receiving a regression review report:

1. Read the full report, including `Scope`, `Gate Snapshot`, and `Evidence Appendix`.
2. Confirm the report still applies to the current diff, branch, and baseline.
3. Restate each finding as a user-visible outcome, not as a code edit.
4. Verify the finding against current code, outputs, tests, or runtime behavior.
5. Decide the disposition: fix, disprove, clarify, or confirm as intentional.
6. Address findings in gate order and verify each change before moving on.
7. Refresh the gate by rerunning `regression-review` or updating the reviewer with concrete evidence.

## Intake Checklist

Before changing code, confirm:

- Which scope was reviewed: working tree, staged diff, commit range, or branch diff.
- Which baseline the report compares against.
- Whether the current checkout still matches that scope.
- Which items are unresolved in `Block`, `Discuss`, and `Watch`.
- Which blind spots limit confidence.

If scope or baseline is stale, stop and regenerate or clarify the report before implementing.

## Handle Each Section

### `Block`

Treat `Block` as "stop the change from shipping until fixed or disproven."

For each `Block` item:

- Reproduce the user-visible breakage, or trace the current code path strongly enough to show the report is still correct.
- Fix the behavior or disprove the finding with stronger evidence than the report currently has.
- Do not skip ahead to lower-severity cleanup while a real `Block` item remains unresolved.

Valid outcomes:

- Fix the regression.
- Prove an equivalent guard or output path still exists.
- Downgrade to `Discuss`, but only with concrete evidence.

### `Discuss`

Treat `Discuss` as "resolve uncertainty before approval."

- Clarify product intent when the change may be deliberate.
- Gather the missing proof the report asked for.
- Prefer focused verification over speculative fixes.

### `Watch`

Treat `Watch` as "note it, then decide whether cheap mitigation is worth it."

- Add a targeted test, monitor, or follow-up when the caveat matters.
- Avoid unnecessary churn when the risk is minor and already understood.

### `Intentional Changes`

Treat `Intentional Changes` as protected product deltas unless evidence says otherwise.

- Confirm intent against the spec, issue, PR description, or user instruction.
- Leave the change alone if it is deliberate.
- Move it back into `Discuss` only when intent is unclear or contradictory.

## When to Push Back

Push back when:

- The report was generated for a different scope or stale branch.
- The finding describes an intended product change, not a regression.
- Current runtime or output evidence contradicts the report.
- The user-visible path is no longer reachable.
- The report misses a guard, fallback, or idempotency control that now lives elsewhere.

Push back with evidence, not tone:

- Cite the current code path or runtime result.
- State what the report got right and what no longer applies.
- Say what additional verification would settle the disagreement if proof is still incomplete.

## Implementation Order

For multi-item reports:

1. Clarify stale or unclear scope first.
2. Fix or disprove every unresolved `Block` item.
3. Resolve `Discuss` items with proof or intent clarification.
4. Decide whether `Watch` items need mitigation now.
5. Leave `Intentional Changes` alone unless intent changed.
6. Re-run targeted verification for every touched user surface.
7. Refresh the regression gate if your changes materially altered user-visible behavior.

## Response Style

Do not use performative agreement. Use short technical acknowledgments.

Good:

- `F1 reproduces on the current diff. Restoring the pending-state guard.`
- `F2 does not reproduce on this branch; the empty-state fallback moved to [file].`
- `F3 appears intentional per the spec. Leaving behavior unchanged.`

Bad:

- `You're absolutely right.`
- `Great catch, I'll fix all of this now.`
- `Thanks for the detailed report.`

## Common Mistakes

- Treat the report title as the bug instead of the user-visible outcome.
- Fix `Intentional Changes` back to the old behavior.
- Downgrade a `Block` item without stronger evidence.
- Claim a finding is false because lint, typecheck, or unrelated tests passed.
- Keep implementing while scope or baseline is unclear.
- Stop after code changes without rerunning the affected flow or output check.

## Bottom Line

A regression review report is a gate artifact. Consume it the same way a strong reviewer would: verify the current behavior, preserve the severity semantics, then fix, challenge, or confirm each finding with evidence.
