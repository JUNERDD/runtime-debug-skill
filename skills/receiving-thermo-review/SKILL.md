---
name: receiving-thermo-review
description: Resolve thermo-review reports or structural quality feedback. Verify responsibility boundaries, decomposition gaps, 350-line findings, and behavior parity before applying scoped fixes or evidence-backed waivers. Use to process existing structural findings, challenge stale or misleading claims, and account for unresolved coverage without changing unrelated code or Git state.
---

# Receiving Thermo Review

Treat structural review findings as claims to verify against current ownership, cohesion, dependencies, and behavior. A lower line count alone does not resolve structural risk.

## Boundary

- Use `thermo-review` to create or refresh the structural report.
- Use `receiving-thermo-review` to consume the report and decide what to fix, disprove, narrow, waive, or carry forward.
- Use `receiving-code-review` for general correctness, security, contract, or test feedback.
- Use `receiving-hack-review` for hack-risk ownership gates.
- Use a scoped `regression-review` when the user requests that gate or direct behavior-parity checks leave material uncertainty.
- Use `exhaustive-code-slimmer` when the user wants a fresh deletion-first slimming pass instead of a response to an existing thermo report.

## Scope and Authorization

- Resolve the source, selected items, and requested outcome from the whole current conversation. A feedback assessment or review-only request ends with dispositions; implement only when the user has authorized fixes.
- When review and repair are already requested together, continue after the source report is complete. A phase change or a pre-edit plan does not require the user to repeat that authorization.
- Keep the source artifact unchanged. Record current evidence and dispositions in a concise final ledger or a separate resolution report; a report's recommendation is not permission to edit.
- Preserve unrelated work and pre-existing staged contents. Leave fixes unstaged unless the user has already authorized the specific staging, commit, or publication action for this task; repair permission alone does not include those actions.
- Do not stage to make a follow-up review easier. Verify working-tree fixes against the recorded baseline and disclose that the staged snapshot may still contain the original issue. Avoid tools with unrequested staging side effects; if the index is accidentally changed, restore only the known workflow delta without disturbing user work and disclose it.

## Workflow

1. Read the complete available report, governing requirements, and current scope. For PR comments or unstructured feedback, normalize the material claims into stable IDs and record the actual source; missing template sections alone do not require a new review.
2. Build a disposition ledger before code edits. Include these source items:
   - every `F#` in the findings index or severity cards and every standalone `D#` decomposition gap
   - recursive coverage, candidate-sweep, and line-count rows linked to findings, unknown impact, missing coverage, threshold crossings, or waivers
   - mismatches between the source index, cards, scope, line counts, and ledgers
3. Reconcile scope and evidence per item. Mark already-fixed, disproved, stale, out-of-scope, or unmappable claims explicitly. Reconstruct affected evidence locally where possible; regenerate only the affected scope when the source cannot support reliable item matching. An unclear target or contract blocks dependent edits, while independent confirmed work may continue. Keep unresolved coverage and integrity gaps visible; do not claim the whole gate is clear.
4. Verify each claim against its current behavior path and governing product or architecture intent. Reuse source traces only after checking that relevant code, inputs, contracts, and scope still match and that the evidence supports the claim; reconstruct changed, missing, or disputed portions. Current code and tests are behavioral evidence, not product authority by themselves. Lint, typecheck, and unrelated green tests do not prove a claim false.
5. Present a concise plan naming accepted items, affected files or boundaries, expected behavior, risks, and verification. Continue within existing authorization. Resolve technical risk through evidence, focused checks, or a narrower fix; ask only when an unresolved user decision or additional authority is needed, and pause only dependent work.
6. Apply the smallest cohesive changes for proven in-scope issues, preserving approved behavior and bounded exceptions. Prioritize blockers; an unresolved independent item does not prevent other confirmed fixes, but it still affects the final gate.
7. Verify affected behavior and repository-required checks, then update each disposition. A check may cover several related items. Reuse recorded results only when they apply to the final code and inputs, and identify their source; never describe reuse as a new run. Repeat or broaden checks only for new edits, failures, unresolved concerns, or required gates.

For threshold items, map responsibilities, canonical owners, dependency direction, and the proposed seam before choosing a remedy. Preserve package/layer and public-contract boundaries unless their change is already authorized.

## Regression Guard

Before editing, build a small behavior-parity ledger for every file or module the fix may touch.

Classify each touched surface as:

- `User-visible`: route, component, command, API response, persisted write, email/export/output, job, config default, flag path, or permission/session behavior.
- `User-visible dependency`: helper, adapter, type, parser, formatter, query, cache key, retry path, or state transform feeding a visible surface.
- `Not user-visible`: tests, docs, generated-only, fixture-only, or dead code with evidence.
- `Unknown impact`: any path whose visible effect cannot be traced locally.

For each `User-visible`, `User-visible dependency`, or `Unknown impact` surface, compare the planned before/after path:

- Source parity: does visible behavior still read from the same input, state slice, request field, serialized form, fixture, env var, or feature flag?
- Guard parity: do auth, permission, validation, debounce, duplicate-submit, confirmation, retry, ordering, empty-state, and error guards still run before the effect?
- Output parity: does the final renderer, API response, CLI text, exporter, email body, request builder, or persisted record still receive the expected shape and format?
- Extension-point parity: if a local override, callback, prop, branch, or special case is removed, what preserves the same behavior?
- Intent split: are structural cleanup and intentional product-visible behavior changes separated and named?

Record each surface as `Preserved`, `Intentional change`, `Potential regression`, or `Not covered`.

Do not hide behavior risk behind passing typecheck, lint, or unrelated tests. If behavior parity is uncertain, either run targeted verification, narrow the structural fix, invoke `regression-review`, or carry the uncertainty forward explicitly.

## Item Handling

### `Blocker`

Treat as do-not-merge until fixed, disproven, narrowed below blocker severity, or explicitly justified.

- Verify the structural failure and why it blocks maintainable change.
- Prefer behavior-preserving simplification, decomposition, ownership correction, or type-boundary clarification.
- Disprove or downgrade only with stronger evidence than the report has.

### `Major`

Treat as fix or answer before approval.

- Verify maintainability, abstraction, ownership, type-contract, duplication, or decomposition risk.
- Apply focused fixes that delete complexity or clarify ownership.
- Promote to `Blocker` if verification shows serious ongoing delivery risk.

### `Minor`

Treat as decide-and-record.

- Fix when mitigation is cheap and reduces reasoning cost without broad churn.
- Carry forward when impact is low, the fix is noisy, or approved broader work should own it.
- Keep the item in the final ledger either way.

### `Question`

Treat as approval-affecting missing context.

- Answer with local evidence, specs, architecture notes, ownership conventions, or user clarification when possible.
- Convert to a fix, disproof, waiver, or carried-forward open question.

### `Decomposition Gaps`

Treat standalone `D#` rows as structural review items.

- Verify whether extraction, module split, helper reuse, state-model cleanup, or type-boundary clarification reduces net reasoning cost.
- Close a gap only when behavior remains preserved and verification is clear.
- Carry forward broad boundary work with owner, scope, and trigger instead of starting an unapproved architecture refactor.

### Recursive Ledgers And Sweep Logs

Treat recursive coverage and candidate sweep rows as gate evidence.

- Ensure every `Finding F#` and `merged into F#` row maps to the disposition ledger.
- Challenge `Not review-relevant` only when the path affects maintained source structure, ownership, type contracts, duplication, tests, config, generated output, or visible behavior.
- For `Not covered` or ambiguous rows, run the missing trace, refresh the gate, or carry a concrete next step.

### Line Count Ledger

Treat the 350-line threshold as a structural signal, not a mechanical ban.

- Recompute current line counts before fixing or waiving a threshold item.
- Before choosing a remedy, map the file's independent reasons to change, shared state, side effects, callers, callees, dependency clusters, and concepts with an existing canonical owner.
- When responsibilities are mixed, prefer returning a complete concern to its canonical owner or extracting one domain, policy, state, adapter, presentation, or orchestration responsibility behind a narrow contract. Delete code only when current evidence independently proves it redundant.
- Accept a split only when the original file owns less, the destination has a clear domain purpose, dependencies stay one-way without cycles, and the change does not create another mixed-responsibility hotspot.
- Reject dense formatting, compressed control flow, shortened names, removal of useful documentation or types, line-range extractions, thin forwarding modules, and moves into catch-all `utils`, `helpers`, or `common` files. None of these closes a threshold item.
- For `crossed 350`, implement the smallest cohesive, behavior-preserving separation available within scope or document a defensible cohesive-file waiver. Crossing back below the number is supporting evidence, not the acceptance criterion.
- For `already over 350`, avoid adding another responsibility; place new behavior with its rightful owner or narrow an existing responsibility when the approved scope permits it. If a cohesive file still grows, refresh its waiver against the added behavior.
- Treat a contract-preserving extraction inside the current package and architectural layer as scoped cleanup. For a move across either boundary, follow any already-approved scope; otherwise carry it forward with the proposed scope, risk, and verification plan until authorized.
- Exclude generated files, lockfiles, vendored artifacts, snapshots, fixtures, and intentionally monolithic external formats unless manually maintained source.

## Push Back When

- The report was generated for a different scope, stale branch, stale baseline, or stale line count.
- Completion is incomplete but presented as resolved.
- Indexes, severity sections, decomposition gaps, ledgers, sweep logs, and self-checks disagree.
- The suggested simpler path broadens behavior, moves architecture boundaries, weakens contracts, or merely relocates or compresses code without reducing structural load.
- The alleged oversized file is generated, vendored, fixture-like, or intentionally external-format code.
- The alleged duplicated or misplaced logic is actually the canonical owner.
- Current code, call sites, tests, docs, project conventions, or behavior-parity tracing contradict the report.
- A `Not covered` row requires credentials, data, platform access, or runtime setup that is unavailable.

Push back with evidence: code path, line count, owner, canonical helper, behavior trace, command output, test, fixture, or doc.

## Completion and Follow-Up

Finish with the source identity, per-item dispositions, focused changes, verification, remaining risks, and actual Git state. Do not claim resolution while a source item lacks a disposition or an implemented item lacks appropriate evidence. Separate a resolved subset from an unresolved overall gate.

Use at most one post-fix review in this resolution when the user requested it or independent review would materially improve confidence. Limit it to the implementation delta and affected boundaries, carrying forward settled intent and disproved claims unless relevant code, contracts, or evidence changed. Return its remaining findings; do not automatically start another receiving cycle. A changed line count, new artifact, or phase handoff alone does not require another full review.

Before final verification, correct failures caused by the current patch when evidence supports an in-scope repair. Report materially distinct discoveries outside the accepted item set without silently expanding the task.

## Final Ledger

Use this shape when multiple items were consumed:

```md
| ID / area | Original status | Disposition | Structural evidence | Behavior / regression evidence | Next action |
| --- | --- | --- | --- | --- | --- |
| F1 | Blocker | Fixed | Moved discount policy into a cohesive domain owner; checkout now owns orchestration only, dependencies remain one-way, and the file dropped from 382 to 319 lines. | Submit source, duplicate-submit guard, and persisted payment payload are unchanged; checkout tests pass. | None |
| F2 | Major | Narrowed | Shared adapter is the canonical owner; only the local wrapper was unnecessary. | API response shape is unchanged by static trace. | Remove wrapper |
| D1 | Decomposition gap | Carried forward | Requires approved boundary change across three packages. | Not edited, so no behavior delta. | Propose scoped refactor |
| app/page.tsx | Crossed 350 | Waived | Generated route table; not manually maintained source. | Not user-visible source logic. | Keep excluded |
| Payment state candidates | Not covered | Open | Requires staging event logs not available locally. | Behavior parity not covered; risk remains unknown. | Regenerate with logs |
```

Mention changed files are left unstaged unless the user asked otherwise.
