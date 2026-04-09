---
name: comment-strategist
description: Add or rewrite code comments so they explain intent, constraints, data meaning, and control-flow decisions instead of translating syntax. Use when Codex is asked to document existing code, improve low-value comments, add structured comments to functions, interfaces, classes, types, or exported constants, or insert guided step comments inside complex logic while preserving the repository's existing comment language and style.
---

# Comment Strategist

Add durable, high-value comments that help a reader understand what the code is responsible for, why a branch exists, what a field means, and where misuse or misunderstanding is likely.

Prefer TypeScript and JavaScript conventions when the target code is TS/JS. Apply the same semantic rules to other languages while matching the local documentation style.

## Workflow

1. Read the target file and identify the top-level definitions first.
2. Detect the prevailing comment language, style, and density from the current file and nearby files.
3. Rank comment candidates by value:
   - top-level functions, classes, interfaces, types, exported constants, and core configuration objects
   - complex public helpers and reusable internal utilities
   - non-obvious control-flow blocks inside complex functions
4. Replace or remove low-value comments before adding new ones so the file keeps one clear documentation voice.
5. Add comments only where they improve comprehension of intent, boundaries, semantics, or sequencing.
6. Re-read the edited file and verify the comments still hold if small implementation details change.
7. Stop after finishing the requested comment edits and any explicitly requested verification. Do not stage, commit, amend, push, or otherwise advance Git state unless the user explicitly asked for that Git action in the current request.

## Priorities

### Top-level definitions first

Prioritize comments for top-level definitions before touching smaller internal details. In most files this means:

- exported functions
- public or shared classes
- interfaces and type aliases
- exported constants with business meaning
- configuration objects or maps whose keys are not self-evident

If the file is only partially documented, spend the budget on these definitions first.

### Preserve signal

Do not add comments just because a definition exists. Skip comments when the name, signature, and surrounding code already make the intent obvious and there are no hidden constraints.

## Comment Standards

### Explain semantics, not syntax

Write comments that explain one or more of the following:

- the responsibility of a definition
- the business or domain meaning of a value
- required preconditions or assumptions
- important side effects
- failure behavior or edge-case handling
- why a branch, transformation, or ordering requirement exists

Do not restate the code in prose. Avoid comments such as "Increment count" or "Check whether user exists" unless there is a deeper semantic reason worth explaining.

### Follow project language and style

Follow the repository's existing comment language. Infer it from the current file first, then nearby files of the same area. If the project mixes languages, follow the dominant style of the current file.

Preserve the local comment format when one already exists:

- JSDoc or TSDoc blocks for documented definitions
- line comments for short contextual notes
- block comments only when the surrounding codebase already uses them for that purpose

Default to JSDoc-compatible block comments for documented TS/JS top-level definitions when no stronger local pattern exists.

### Prefer durable wording

Write comments that survive small refactors. Prefer explaining intent and contract over implementation detail. Avoid documenting temporary mechanics unless that mechanic is the whole reason the code is hard to understand.

## Top-level Rules by Definition

### Functions

Document top-level functions and any non-trivial reusable helper whose misuse would be costly.

For documented functions, include:

- a one- or two-sentence summary of the function's responsibility
- `@param` entries for each meaningful parameter
- `@returns` only when the function has a meaningful return value

Use `@returns` to explain the meaning of the returned value, not just its type. Omit `@returns` for procedures that only produce side effects.

Capture important contracts when relevant:

- required parameter relationships
- mutation or side effects
- ordering guarantees
- fallback behavior
- failure or nullability semantics

### Interfaces and types

Add an interface-level summary for every documented interface. Then document each top-level field when its meaning is not fully obvious from the name alone.

Field comments should explain:

- what the field represents
- whether it is required for a specific behavior
- special units, formats, or constraints
- coupling to external systems or business rules

For simple aliases or literal unions, add a top-level comment only when the type has domain meaning that is not evident from its shape.

### Classes

Document the role of the class, especially its lifecycle, ownership boundary, and collaboration with other components. Document methods when their behavior is non-obvious or part of the external contract.

### Exported constants and config objects

Comment constants and object properties when they encode policy, thresholds, feature gates, lookup semantics, or business categories that a reader would not safely infer from names alone.

## Internal Guide Comments

Add internal guide comments only when they materially help a reader navigate complex logic. Good candidates include:

- multi-stage normalization or validation flows
- branching with business-specific rules
- state transitions
- retry, fallback, or recovery logic
- ordering-sensitive transformations

When internal guide comments are needed, use numbered headings in ascending order:

- `1.`
- `1.1`
- `2.`
- `2.1`
- `2.2`

Use these comments to describe the intent of each stage, not every statement inside the stage.

Example pattern:

```ts
// 1. Normalize external input into a shape that downstream validation can reason about.
// 1.1 Preserve the caller's explicit overrides before applying defaults.
// 2. Reject combinations that would produce conflicting scheduling behavior.
```

Do not use numbered comments in simple functions, single-branch helpers, or straightforward CRUD wrappers.

## Rewrite and Removal Rules

Replace existing comments when they are:

- literal translations of syntax
- outdated after refactors
- redundant with strong naming
- inconsistent with the current file's terminology

Remove comments instead of rewriting them when the code is already fully clear and the comment adds no durable meaning.

Never stack a better comment under a bad one. Clean the old one first.

## TS and JS Defaults

For TypeScript and JavaScript, prefer:

- JSDoc-style comments for top-level functions, classes, interfaces, and exported constants
- property comments for interface fields
- short line comments for internal numbered guide steps

When documenting params and returns, keep type information in the type system and use the comment to explain semantic meaning, constraints, or caller expectations.

## Guardrails

- Do not comment every line.
- Do not invent business meaning that the code and nearby context do not support.
- Do not add `@returns` to void procedures.
- Do not document internal implementation trivia unless it prevents likely misunderstanding.
- Do not add field comments to nested inline object literals unless the request explicitly calls for that depth.
- Do not let comments drift into a second source of truth for obvious type information.
- Do not run `git add`, `git commit`, `git commit --amend`, `git push`, create tags, or open PRs unless the user explicitly requested that Git action in the current request.
- Do not treat "comment the code", "document this file", or similar requests as permission to prepare a commit.
- If the user only asked for comments, leave version-control state untouched apart from the edited working-tree files themselves.

## Completion Check

Before finishing, verify all of the following:

- Top-level definitions were considered first.
- Function comments include `@param` and only include `@returns` when warranted.
- Interfaces have an overview and comments for each top-level field that needs semantic clarification.
- Internal numbered comments appear only where staged guidance materially helps.
- New comments explain intent, constraints, and meaning rather than rephrasing code.
- Comment language and style match the surrounding project.
- No Git state-changing command was run unless the user explicitly requested it.
