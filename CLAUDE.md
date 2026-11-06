# Project Instructions

Read @AGENTS.md for globals (ER/QA glossary, `[creation_dir]` etc.), the Hard Rule, @agent calling rules, MCP server usage and Codex/Muse path conventions. See `CONTRIBUTING.md` when changing `prompts/ER.md`, agents or skills, and `docs/` for user-facing guides.

## Key rules

- NEVER modify files outside `creation/`, `creation/tmp/` or `creation/trash/` without explicit user permission. `.claude/settings.json` only allows Edit/Write on `creation/**`.
- Get timestamps from the `date` shell command, never from the model's clock.
- Call the @agents (`er-creator`, `er-auditor`, `er-fixer`, `er-combiner`) exactly as the prompt specifies; do not alter their prompts or arguments.
- Put temporary files in `creation/tmp/`.
- Verify PubMed URLs with `mcp__d-pubmed__pubmed_fetch_articles` (pass `pmids` as a JSON array of strings) and ClinicalTrials.gov URLs with `mcp__d-clinicaltrialsgov__clinicaltrials_get_study_record`. Never use the built-in `WebFetch` tool (denied in `.claude/settings.json`).
- `.claude/` is the source for agents and skills. The copies in `.codex/`, `.opencode/` and `.agents/` are generated from it, so edit only `.claude/` (see `CONTRIBUTING.md`).
- Obstacle Protocol: if a sub-agent itself cannot continue, stop and report to the user. A single failed tool call is not an obstacle.

## Commands

- There is no build, test or lint tooling; this is a prompt repo.
- `/er create|audit|fix|combine|iterate|full|compare <args>`: run the evidence-review workflow (`.claude/skills/er/SKILL.md`).
- `/check`: verify versions against `VERSION` and check consistency across prompts, agents and skills. Run it after editing any prompt, agent or skill.

## Layout

- `prompts/`: `ER.md` (the evidence-review prompt) and `PERSONA.md` (interactive persona).
- `.claude/`: source agents (`agents/`), skills (`skills/er`, `skills/check`) and `settings.json`.
- `.codex/`, `.opencode/`, `.agents/`, `.grok/`: per-environment generated copies and config. `.mcp.json` defines the MCP servers.
- `creation/`: git-ignored output (`tmp/`, `trash/`).
- `docs/`: user guides and limitations.
- `VERSION`, `CHANGELOG.md`, `CONTRIBUTING.md`, `README.md`.
