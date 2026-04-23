# Hack-Risk Audit

## Scope

- Review date: `YYYY-MM-DD`
- Scope reviewed: `[working tree | staged diff | commit range | branch diff | PR | implementation slice]`
- Baseline: `[HEAD | commit SHA | branch | design doc | existing abstraction]`
- Completion: `[Complete within reviewed scope | Incomplete - reason]`
- Assumptions: `[if scope, ownership, runtime setup, credentials, or architecture context was inferred]`

## Gate Snapshot

- Recommendation: `[Block | Discuss | Pass with caveat | Pass]`
- Completion: `[Complete within reviewed scope | Incomplete - exact uncovered boundary]`
- Why now: `[one sentence that explains the review decision]`
- Must-review now: `[top 1-3 items only; full list is in Complete Hack-Risk Index]`
  1. `F#` `[short title]`
  2. `F#` `[short title]`
  3. `F#` `[short title]`
- Findings count: `Block [n] | Discuss [n] | Watch [n] | Intentional [n]`
- Coverage confidence: `[high | medium | low]`
- Biggest blind spot: `[short phrase, or None identified]`

## Complete Hack-Risk Index

If no findings exist, write `No hack-risk findings identified in the reviewed scope.`
Otherwise add one row for every `F#` finding in the report.

| ID | Action | Boundary / surface | Structural liability | Confidence |
| --- | --- | --- | --- | --- |
| `F1` | `[Block | Discuss | Watch]` | `[helper, adapter, store, lifecycle path, command, etc.]` | `[one-line liability]` | `[high | medium | low]` |

## Block

If none exist, write `None.`
Otherwise repeat this card for every `Block` finding. Continue numbering across all finding sections as needed.

### F1 Block - [Short title]

Engineering impact: `[how this shortcut increases fragility, divergence, or hidden coupling]`
Review reason: `[why this is worth blocking in review]`
Surface: `[module, flow, abstraction, command, adapter, serializer, store, etc.]`
Confidence: `[high | medium | low]`

Look here first:
- `[primary](/abs/path/file.ts#L10)`
- `[secondary](/abs/path/file.tsx#L42)`

Implementation delta:
- Before: `[how ownership or abstraction worked before this change]`
- After: `[how the new shortcut changes that ownership or abstraction]`

Evidence:
- `[runtime repro, targeted test, search result, log, output inspection, or code-path evidence]`

Reviewer action:
`[block until shared abstraction is reused | block until root cause is fixed | block until ownership is explicit | block until disproven]`

## Discuss

If none exist, write `None.`
Otherwise repeat this card for every `Discuss` finding. Continue numbering across all finding sections as needed.

### F2 Discuss - [Short title]

Engineering impact: `[what can drift, split, or become misleading]`
Review reason: `[why this deserves discussion]`
Surface: `[module, flow, abstraction, command, adapter, serializer, store, etc.]`
Confidence: `[high | medium | low]`

Look here first:
- `[primary](/abs/path/file.ts#L10)`
- `[secondary](/abs/path/file.tsx#L42)`

Implementation delta:
- Before: `[how ownership or abstraction worked before this change]`
- After: `[how the current implementation may be narrowing, bypassing, or duplicating it]`

Evidence:
- `[what supports the concern and what is still missing]`

Reviewer action:
`[raise in review | confirm intent | ask for proof | request migration plan]`

## Watch

If none exist, write `None.`
Otherwise repeat this card for every `Watch` finding. Continue numbering across all finding sections as needed.

### F3 Watch - [Short title]

Engineering impact: `[narrower debt or bounded shortcut]`
Review reason: `[why it is worth noting but not blocking]`
Surface: `[module, flow, abstraction, command, adapter, serializer, store, etc.]`
Confidence: `[high | medium | low]`

Look here first:
- `[primary](/abs/path/file.ts#L10)`

Implementation delta:
- Before: `[how the previous implementation stayed aligned]`
- After: `[how the new path introduces a small or bounded shortcut]`

Evidence:
- `[what was checked]`

Reviewer action:
`[approve with caveat | monitor | add follow-up test | note for later]`

## Intentional Exceptions

If none exist, write `None.`

- `I1` `[short shortcut]` - `[why it appears deliberate and bounded]` - `[owner or exit condition]` - `[link](/abs/path/file.ts#L10)`

## Ownership Coverage Ledger

Every touched implementation-relevant or unknown boundary must appear here, including boundaries with no findings.

| Boundary / path | Touched files or entry points | Status | Result | Evidence |
| --- | --- | --- | --- | --- |
| `[boundary]` | `[file or entry point links]` | `[Finding F# | Intentional Exception I# | Reviewed - no hack-risk found | Not hack-relevant | Not covered]` | `[short result]` | `[ownership trace, search, runtime check, fixture, or reason not covered]` |

## Evidence Appendix

### Diff Inventory

| File or area | Classification | Boundary considered |
| --- | --- | --- |
| `[path]` | `[boundary | dependency | config | test-only | docs-only | generated | unknown]` | `[helper, service, adapter, store, cache, lifecycle, none, or unknown]` |

### Candidate Sweep Log

Use this section for meaningful candidates that were investigated and dismissed, or for duplicate candidates merged into another finding. Omit trivial unchanged paths.

| Candidate | Decision | Reason |
| --- | --- | --- |
| `[candidate shortcut]` | `[dismissed | merged into F# | intentional I#]` | `[evidence or reasoning]` |

### Verification Commands

- `[command and key outcome]`
- `[command and key outcome]`

### Supporting Code Links

| ID | Role | Link | Why it matters |
| --- | --- | --- | --- |
| `F1` | `incumbent` | `[existing abstraction](/abs/path/file.ts#L10)` | `[where ownership used to live]` |
| `F1` | `shortcut` | `[new fallback](/abs/path/file.ts#L42)` | `[where the hack enters]` |
| `F1` | `sibling path` | `[other caller](/abs/path/file.tsx#L88)` | `[who still depends on the incumbent path]` |

### Blind Spots

| Area | Risk introduced by the blind spot | What would resolve it |
| --- | --- | --- |
| `[unverified boundary]` | `[how this limits the review decision]` | `[specific next verification step]` |

### Report Self-Check

- `[yes | no]` Every touched implementation-relevant or unknown boundary appears in `Ownership Coverage Ledger`.
- `[yes | no]` Every finding in an action section appears in `Complete Hack-Risk Index`.
- `[yes | no]` Every `Finding F#` ledger row has a matching card.
- `[yes | no]` Every `Not covered` row has a reason and next verification step.
- `[yes | no]` Recommendation follows the mapping rules from the skill.
