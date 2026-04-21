# Installing JUNERDD Skills For Agent Runtimes

Expose this repository to local agent runtimes through a single symlink:

```text
~/.agents/junerdd-skill -> <repo>/skills
```

The goal is to link the repository's `skills/` directory into `~/.agents` without copying files.

## Installation

1. Determine the absolute path of this repository.
   - If the agent is already working inside a local checkout of `JUNERDD/skills`, use that checkout.
   - Otherwise clone `https://github.com/JUNERDD/skills.git` to a stable local path first, then use that checkout.

2. Ensure the target parent directory exists:

   ```bash
   mkdir -p ~/.agents
   ```

3. Inspect `~/.agents/junerdd-skill` before changing it.
   - If it does not exist, continue.
   - If it is already a symlink to `<repo>/skills`, leave it in place.
   - If it is a symlink to some other target, refresh it.
   - If it is a real file or directory, stop and ask before replacing it.

4. Create or refresh the symlink so it points to this repository's `skills/` directory:

   ```bash
   ln -sfn "<repo>/skills" ~/.agents/junerdd-skill
   ```

5. Verify the result:
   - Show the symlink target with `ls -la ~/.agents/junerdd-skill`.
   - Confirm it resolves to `<repo>/skills`.
   - Confirm at least one skill entry is reachable through the link, for example `~/.agents/junerdd-skill/git-commit/SKILL.md`.

## Updating

If `~/.agents/junerdd-skill` points at an existing local checkout, updates come from updating that checkout. The symlink does not need to be recreated unless the checkout path changes.

## Uninstalling

Remove the symlink:

```bash
rm ~/.agents/junerdd-skill
```
