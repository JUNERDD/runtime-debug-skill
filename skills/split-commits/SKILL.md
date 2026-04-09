---
name: split-commits
description: Split a large or mixed Git working tree into multiple focused local commits. Use when changes span unrelated concerns, combine refactors with behavior changes, mix generated files with source edits, or contain separable hunks in the same file, and Codex needs to stage one logical batch at a time, invoke `$git-commit` to draft the staged commit message, present the staged batch for user confirmation, run `git commit` only after the user explicitly approves that batch, and repeat without pushing.
---

# Split Commits

Turn an overgrown working tree into a short sequence of focused local commits. Assemble each commit in the index first, hand it to `$git-commit` for the message, then stop for user confirmation before running `git commit`. Do not push as part of this skill.

## Workflow

1. Inspect the current Git state.
   - `git status --short`
   - `git diff --stat`
   - `git diff --cached --stat`
2. Decide whether the work should be split. Split when one or more of these are true:
   - the changes contain multiple independent concerns
   - refactors and behavior changes are mixed
   - tests belong to only one part of the implementation
   - generated or lock files should travel with only one subset
   - a file contains unrelated hunks that would produce a misleading single commit
3. Write a short commit plan before staging.
   - Aim for 2-5 commits.
   - Give each commit one purpose and one likely Conventional Commit type.
   - Order commits so each one is understandable on its own.
4. Respect the current index.
   - If staged changes already match the next planned commit, keep them.
   - If staged changes mix unrelated concerns, stop and ask before repartitioning the index.
5. Stage only the next logical batch.
   - Use `git add <path>` for whole files.
   - Use `git add -p <path>` when only some hunks belong.
   - Avoid `git add .` unless the remaining work is intentionally one batch.
6. Verify the staged batch.
   - `git diff --cached --stat`
   - `git diff --cached`
   - Confirm no secrets, unrelated files, or accidental churn are included.
7. Invoke `$git-commit` and use it to draft the commit message from the staged diff only.
8. Review the proposed message against the staged diff.
   - Keep it if accurate.
   - Tighten it if it overstates the change or merges multiple concerns in the wording.
   - If `$git-commit` indicates the staged set is still mixed, do not commit it. Refine the staged batch and rerun `$git-commit`.
9. Present the staged diff summary and the finalized message to the user, then ask for explicit confirmation for that one commit.
   - Wait for confirmation before changing Git history.
   - If the user declines, refine the staged batch or the message and ask again.
10. Run `git commit` with the finalized message only after the user explicitly approves that staged batch.
11. Re-run `git status --short` and continue with the next planned batch until the intended local commits are created.

## Grouping Heuristics

Prefer splitting by intent, not by directory alone.

Good commit boundaries:

- a bug fix with the tests that prove it
- a pure rename or move isolated from behavior changes
- a mechanical formatting or codemod sweep isolated from semantic edits
- a dependency bump paired with the code or config required for that bump
- documentation that belongs to a single feature or refactor

Poor commit boundaries:

- "backend files" vs "frontend files" when both implement one feature
- tests separated from the code they validate unless the repo already expects that pattern
- arbitrary file-count balancing
- one commit per file when the behavior change spans several files

If two changes cannot be described honestly with one subject line, they likely deserve separate commits.

## Guardrails

- Do not push.
- Do not amend unless the user explicitly asks.
- Do not run `git commit` until the user explicitly approves the current staged batch and message.
- Do not use destructive commands such as `git reset --hard` or `git checkout --`.
- Do not silently unstage or restage a mixed index; ask first if the existing staged state must be reshaped.
- Do not ask `$git-commit` to summarize unstaged changes. It must see staged changes only.
- Stop and ask if the remaining edits are too entangled to split without guessing intent.

## Completion

Before finishing:

- ensure every requested local commit exists
- leave any intentionally uncommitted changes in the working tree
- report the created commit subjects or SHAs to the user
- do not push or open a PR
