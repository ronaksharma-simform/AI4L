![Version 2.0.2](https://img.shields.io/badge/Version-2.0.2-green.svg)
[![Forever Healthy](https://img.shields.io/badge/(c)_2026-Forever_Healthy-573D7D.svg)](https://forever-healthy.org)

@AGENTS.md  <!-- repo layout, globals, workflow, MCP and hard-write rules; read before making changes -->

## Commands

- There is no build, test or lint tooling: this is a prompts and docs repo.
- Verify the repo with `/check` (versions and badges, `prompts/ER.md` numbering, MCP config drift).
- Run the ER workflow with `/er {create|audit|fix|combine|iterate|full|compare}`.

## Layout

- `prompts/`: ER prompt (`ER.md`) and conversation persona (`PERSONA.md`)
- `.claude/skills/`, `.claude/agents/`: the `er` and `check` skills and the four ER agents
- `creation/`: work in progress (with `tmp/` and `trash/`); `docs/`: user documentation
- AGENTS.md has the full map and the globals.

## Gotchas

- Writes are limited to `creation/` (plus tmp and trash) without user permission; `.claude/settings.json` allows Edit/Write only under `creation/**`.
- `WebFetch` is denied. Use the d-browser, d-fetch and d-proxy MCP tiers; they handle bot-walled pages.
- `.codex/`, `.opencode/` and `.cursor/` hold copies of the `.claude` agents and skills. Edit the `.claude` sources first and keep the copies in sync, or `/check` reports drift.
- `/check` calls a `/sync` skill that is not in this repo. Skip that check and say so when it is absent.
