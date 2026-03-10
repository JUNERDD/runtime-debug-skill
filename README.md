# JUNERDD Skills

Reusable AI agent skills published from a single repository. The repository is structured to grow over time; right now it ships the `debug` skill for evidence-first runtime debugging.

This repository packages installable skills under [`skills/`](./skills/). Each subfolder is a self-contained skill that can be installed independently. The current skill lives at [`skills/debug/`](./skills/debug/). [`skills/debug/agents/openai.yaml`](./skills/debug/agents/openai.yaml) is only an optional metadata adapter for runtimes that support OpenAI-style skill discovery.

## Current Skills

- [`debug`](./skills/debug/) for hypothesis-driven runtime debugging with local log capture

## Included Today

- A reusable `debug` skill that forces hypothesis-driven debugging instead of speculative fixes
- A detailed operator reference in [`skills/debug/references/runtime-debugging.md`](./skills/debug/references/runtime-debugging.md)
- A bundled local NDJSON collector and dashboard in [`skills/debug/scripts/local_log_collector/`](./skills/debug/scripts/local_log_collector/)
- Optional UI metadata for compatible runtimes in [`skills/debug/agents/openai.yaml`](./skills/debug/agents/openai.yaml)

## Architecture

```mermaid
flowchart LR
  User["Developer / Operator"] --> Agent["Agent Runtime"]
  Agent --> Skill["SKILL.md<br/>workflow + guardrails"]
  Agent --> Ref["runtime-debugging.md<br/>bootstrap + log format"]
  Agent --> App["Target app under debug"]
  Agent --> Logs["Temporary instrumentation"]
  Logs --> Collector["Local NDJSON collector<br/>same-origin dashboard + APIs"]
  App --> Logs
  Collector --> File["Session log file"]
  Collector --> UI["Live dashboard"]
  File --> Agent
  UI --> Agent
  Agent --> Fix["Proven fix + post-fix verification"]
```

## Why This Exists

Most debugging prompts collapse into code reading, guesses, and defensive patches. This skill pushes the agent into a stricter loop:

1. Generate precise hypotheses.
2. Attach to or start an authoritative logging session.
3. Add minimal temporary instrumentation.
4. Ask for a reproduction run.
5. Read the log file and mark each hypothesis `CONFIRMED`, `REJECTED`, or `INCONCLUSIVE`.
6. Apply a fix only after the root cause is proven.
7. Verify with fresh post-fix logs before removing instrumentation.

That makes the debugging process easier to audit, easier to repeat, and much less likely to ship guesswork.

## Dashboard Preview

![Runtime Debug dashboard preview](./docs/images/dashboard-overview.png)

## What The Skill Enforces

- Evidence-first debugging instead of inspection-only reasoning
- Minimal instrumentation with explicit cleanup after verification
- Per-hypothesis logging and before/after comparison
- Local collector bootstrap when the host does not already provide logging
- Guardrails against stale logs, speculative fallback code, and leaked debug scaffolding
- Browser-first log transport for frontend debugging, with explicit prohibition on app-local proxy routes unless direct delivery is proven blocked

## Runtime Compatibility

The skill is intentionally portable. You can use it with:

- OpenAI Codex and similar local-skill runtimes
- Agent shells that read `~/.agents/skills/<name>/SKILL.md`
- Custom agent frameworks that mount a skill folder and inject `SKILL.md` into context
- Internal toolchains that want the collector, references, or workflow as reusable assets

If your runtime ignores [`skills/debug/agents/openai.yaml`](./skills/debug/agents/openai.yaml), that is fine. The core logic is still fully available through [`skills/debug/SKILL.md`](./skills/debug/SKILL.md).

## Install

Preferred install for the current `debug` skill via the `skills` CLI:

```bash
npx skills add JUNERDD/skills --skill debug
```

For a global Codex install:

```bash
npx skills add JUNERDD/skills --skill debug -g -a codex -y
```

Manual install still works if your runtime does not use the CLI. Copy [`skills/debug/`](./skills/debug/) as a local skill named `debug`.

| Runtime | Install path | Notes |
| --- | --- | --- |
| Codex-style runtimes | `~/.codex/skills/debug` | Copy `skills/debug` there |
| Agents-style runtimes | `~/.agents/skills/debug` | Copy `skills/debug` there |
| Custom runtimes | Any mounted `debug/` folder | Copy `skills/debug`, then load `SKILL.md` and optionally parse `agents/openai.yaml` |

Example:

```bash
mkdir -p ~/.agents/skills
cp -R ./skills/debug ~/.agents/skills/
```

If your runtime supports metadata discovery, keep [`skills/debug/agents/openai.yaml`](./skills/debug/agents/openai.yaml) inside the copied skill folder.

## How To Invoke It

The exact invocation depends on the host:

- In runtimes with skill commands or chips, call `debug` or `$debug`
- In plain-text agent runtimes, tell the agent to load the `debug` skill before investigating the bug
- In custom frameworks, inject [`skills/debug/SKILL.md`](./skills/debug/SKILL.md) as the active debugging workflow

Example prompts:

```text
Use the debug skill to investigate why checkout stays disabled after the cart updates.
```

```text
Load the debug workflow from SKILL.md and debug this flaky save action using runtime evidence.
```

## Local Collector

The bundled collector is a zero-dependency Python app built on the standard library. It accepts JSON log events, appends them to an NDJSON file, and serves a same-origin dashboard for live inspection.

For frontend and browser debugging, the intended transport is direct client-to-collector HTTP posting. The collector already handles CORS and preflight, so the skill should not create temporary Next.js API routes or other app-local proxy layers unless direct browser delivery has been proven blocked in the current host.

Endpoints:

- `POST /ingest`
- `GET /health`
- `GET /api/state`
- `GET /api/logs`
- `GET /api/logs/detail`
- `POST /api/clear`
- `POST /api/shutdown`

Minimal smoke test:

```bash
mkdir -p .debug-logs
python3 skills/debug/scripts/local_log_collector/main.py \
  --log-file "$PWD/.debug-logs/demo.ndjson" \
  --ready-file "$PWD/.debug-logs/demo.json" \
  --session-id "demo-session"
```

The ready file contains the active endpoint, dashboard URL, log file path, and session metadata.

## Repository Layout

```text
. 
├── LICENSE
├── README.md
├── docs/
│   └── images/
│       └── dashboard-overview.png
└── skills/
    └── debug/
        ├── SKILL.md
        ├── agents/
        │   └── openai.yaml
        ├── references/
        │   └── runtime-debugging.md
        └── scripts/
            └── local_log_collector/
                ├── main.py
                ├── collector_server.py
                ├── collector_state.py
                ├── collector_browser.py
                └── static/
```

## Customize It

- Edit [`skills/debug/SKILL.md`](./skills/debug/SKILL.md) to change workflow rules, guardrails, or response shape
- Edit [`skills/debug/references/runtime-debugging.md`](./skills/debug/references/runtime-debugging.md) to refine bootstrap commands and logging templates
- Edit [`skills/debug/agents/openai.yaml`](./skills/debug/agents/openai.yaml) only when you need runtime-specific metadata tweaks

## License

Released under the [MIT License](./LICENSE).
