# Installing JUNERDD Skills For Agent Runtimes

Expose this repository to local agent runtimes through a single symlink:

```text
~/.agents/skills/junerdd-skill -> <repo>/skills
```

The goal is to link the repository's `skills/` directory into `~/.agents/skills` without copying files.

## Path Rules

There are two different paths in this setup. Do not mix them up.

- Repository checkout path: where the `JUNERDD/skills` git repository lives locally.
- Skill discovery path: the symlink path that agent runtimes read from.

The skill discovery path is fixed:

```text
~/.agents/skills/junerdd-skill
```

Do not install the symlink at `~/.agents/junerdd-skill`.
When the user talks about choosing an install location, they are choosing the repository checkout path, not the skill discovery path.

## Install Location Policy

Before cloning or linking anything, decide where the local checkout should live.

- If the user already gave an install path or asked to use the current local checkout, use that path.
- If the user did not specify an install location, stop and ask them where they want the repository installed.
- If the user explicitly says the agent can choose the install location, use `~/.junerdd/JUNERDD-skills` as the default checkout path.

## Installation

1. Determine the absolute path of this repository.
   - If the agent is already working inside a local checkout of `JUNERDD/skills`, use that checkout.
   - Otherwise, follow the install location policy above before cloning anything.
   - If the user lets the agent choose the install location, clone `https://github.com/JUNERDD/skills.git` to `~/.junerdd/JUNERDD-skills` and use that checkout.

2. Treat `~/.agents/skills/junerdd-skill` as the only valid skill discovery path.
   - Do not create `~/.agents/junerdd-skill`.
   - If `~/.agents/junerdd-skill` already exists from an older install, remove or replace it only after confirming it is safe to do so.

3. Ensure the target parent directory exists:

   ```bash
   mkdir -p ~/.agents/skills
   ```

4. Inspect `~/.agents/skills/junerdd-skill` before changing it.
   - If it does not exist, continue.
   - If it is already a symlink to `<repo>/skills`, leave it in place.
   - If it is a symlink to some other target, refresh it.
   - If it is a real file or directory, stop and ask before replacing it.

5. Create or refresh the symlink so it points to this repository's `skills/` directory:

   ```bash
   ln -sfn "<repo>/skills" ~/.agents/skills/junerdd-skill
   ```

6. Verify the result:
   - Show the symlink target with `ls -la ~/.agents/skills/junerdd-skill`.
   - Confirm it resolves to `<repo>/skills`.
   - Confirm at least one skill entry is reachable through the link, for example `~/.agents/skills/junerdd-skill/git-commit/SKILL.md`.
   - Confirm that `~/.agents/junerdd-skill` does not exist unless the user explicitly asked for that legacy layout.

## Updating

If `~/.agents/skills/junerdd-skill` points at an existing local checkout, updates come from updating that checkout. The symlink does not need to be recreated unless the checkout path changes.

## Uninstalling

Remove the symlink:

```bash
rm ~/.agents/skills/junerdd-skill
```
