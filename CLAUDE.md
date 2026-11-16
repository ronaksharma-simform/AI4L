# Claude Code Instructions

@AGENTS.md

See @AGENTS.md for the globals, hard rules, repo layout and MCP servers. Read `prompts/ER.md` before creating or auditing an ER. Each agent's behaviour is defined in `.claude/agents/` (`er-creator.md`, `er-auditor.md`, `er-fixer.md`, `er-combiner.md`).


## Commands

There is no build, test or lint step. The entry points are skills:

* `/er create|audit|fix|combine|iterate|full|compare` — create, audit, fix, combine, iterate and compare evidence reviews (`.claude/skills/er/SKILL.md`)
* `/check` — verify versions, item numbering and MCP config consistency (`.claude/skills/check/SKILL.md`)


## Gotchas

* `.claude/` is the source of truth for agents and skills; the copies in `.codex/`, `.agents/`, `.opencode/` and `.cursor/` are generated from it. Edit `.claude/`, not the copies.
* `creation/*` is gitignored, so git cannot restore a damaged or deleted ER or QA file.
* Take timestamps from `date` in the shell, never from the model's clock.
* The built-in `WebFetch` tool is denied in `.claude/settings.json`; use the MCP verification chain in AGENTS.md.
