# Review Gate: User-Visible Regression Assessment

## Scope

- Review date: `YYYY-MM-DD`
- Scope reviewed: `[working tree | staged diff | commit range | branch diff]`
- Baseline: `[HEAD | commit SHA | branch]`
- Assumptions: `[state defaults if scope or runtime setup was inferred]`

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

User impact: `[what users will notice or fail to complete]`
Review reason: `[why this is worth blocking in review]`
Surface: `[route, feature, command, output, email, etc.]`
Confidence: `[high | medium | low]`

Look here first:
- `[primary](/abs/path/file.ts#L10)`
- `[secondary](/abs/path/file.tsx#L42)`

Behavior delta:
- Before: `[behavior before this change]`
- After: `[behavior after this change]`

Evidence:
- `[runtime repro, targeted test, log, screenshot, fixture, or code-path evidence]`

Reviewer action:
`[block until fixed | block until disproven | request targeted test | request runtime verification]`

## Discuss

### F2 Discuss - [Short title]

User impact: `[what users may notice]`
Review reason: `[why this deserves review discussion]`
Surface: `[route, feature, command, output, email, etc.]`
Confidence: `[high | medium | low]`

Look here first:
- `[primary](/abs/path/file.ts#L10)`
- `[secondary](/abs/path/file.tsx#L42)`

Behavior delta:
- Before: `[behavior before this change]`
- After: `[behavior after this change]`

Evidence:
- `[what supports the concern and what is still missing]`

Reviewer action:
`[raise in review | confirm intent | ask for proof | request follow-up test]`

## Watch

### F3 Watch - [Short title]

User impact: `[minor or narrower visible effect]`
Review reason: `[why it is worth noting but not blocking]`
Surface: `[route, feature, command, output, email, etc.]`
Confidence: `[high | medium | low]`

Look here first:
- `[primary](/abs/path/file.ts#L10)`

Behavior delta:
- Before: `[behavior before this change]`
- After: `[behavior after this change]`

Evidence:
- `[what was checked]`

Reviewer action:
`[approve with caveat | monitor | add follow-up test | note for later]`

## Intentional Changes

- `I1` `[short visible change]` - `[why it appears intentional]` - `[link](/abs/path/file.ts#L10)`

## Evidence Appendix

### Coverage

| Area | Status | Note |
| --- | --- | --- |
| Runtime verification | `[done | partial | not done]` | `[what was actually executed]` |
| Platform coverage | `[broad | partial | narrow]` | `[desktop, mobile, locale, auth state, browser, CLI, etc.]` |
| Test coverage | `[strong | partial | weak]` | `[tests run or missing]` |

### Verification Commands

- `[command and key outcome]`
- `[command and key outcome]`

### Supporting Code Links

| ID | Role | Link | Why it matters |
| --- | --- | --- | --- |
| F1 | entry | `[submit handler](/abs/path/file.ts#L10)` | `[where the journey starts]` |
| F1 | behavior | `[guard removed](/abs/path/file.ts#L42)` | `[where the change happens]` |
| F1 | output | `[render path](/abs/path/file.tsx#L88)` | `[where the user-visible result appears]` |

### Blind Spots

| Area | Risk introduced by the blind spot | What would resolve it |
| --- | --- | --- |
| `[unverified path]` | `[how this limits the review decision]` | `[specific next verification step]` |
