# Review Gate: Hack-Risk Assessment

## Scope

- Review date: `YYYY-MM-DD`
- Scope reviewed: `[working tree | staged diff | commit range | branch diff | implementation slice]`
- Baseline: `[HEAD | commit SHA | branch | design doc | existing abstraction]`
- Assumptions: `[if scope was not specified, say that the review defaulted to the staged diff; record any ownership or runtime assumptions separately]`

## Gate Snapshot

- Recommendation: `[Block | Discuss | Pass with caveat | Pass]`
- Why now: `[one sentence that explains the review decision]`
- Must-review now:
  1. `F1` `[short title]`
  2. `F2` `[short title]`
  3. `F3` `[short title]`
- Coverage confidence: `[high | medium | low]`
- Biggest blind spot: `[short phrase]`

## Block

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

- `I1` `[short shortcut]` - `[why it appears deliberate and bounded]` - `[link](/abs/path/file.ts#L10)`

## Evidence Appendix

### Coverage

| Area | Status | Note |
| --- | --- | --- |
| Runtime verification | `[done | partial | not done]` | `[what was actually executed]` |
| Ownership trace | `[strong | partial | weak]` | `[which invariant owners or abstractions were traced]` |
| Existing abstraction search | `[done | partial | not done]` | `[what incumbent helpers or wrappers were checked]` |
| Test coverage | `[strong | partial | weak]` | `[tests run or missing]` |

### Verification Commands

- `[command and key outcome]`
- `[command and key outcome]`

### Supporting Code Links

| ID | Role | Link | Why it matters |
| --- | --- | --- | --- |
| F1 | incumbent | `[existing abstraction](/abs/path/file.ts#L10)` | `[where ownership used to live]` |
| F1 | shortcut | `[new fallback](/abs/path/file.ts#L42)` | `[where the hack enters]` |
| F1 | sibling path | `[other caller](/abs/path/file.tsx#L88)` | `[who still depends on the incumbent path]` |

### Blind Spots

| Area | Risk introduced by the blind spot | What would resolve it |
| --- | --- | --- |
| `[unverified path]` | `[how this limits the review decision]` | `[specific next verification step]` |
