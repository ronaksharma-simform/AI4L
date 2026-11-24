# Claude Code Instructions

@AGENTS.md

Read `prompts/ER.md` before creating or auditing an ER. Each agent's behaviour is defined in `.claude/agents/` (`er-creator.md`, `er-auditor.md`, `er-fixer.md`, `er-combiner.md`).


## Layout

* `creation/`: generated ERs and QA files (gitignored, except `tmp/` and `trash/` placeholders)
* `prompts/ER.md`: ER format and rules
* `.claude/agents/` and `.claude/skills/`: source of truth for agents and skills; `.codex/`, `.agents/`, `.opencode/` and `.cursor/` hold generated copies

Hand-off: er-creator writes the ER, er-auditor or er-fixer audits it, er-combiner merges the dated audits into one `_ER_QA.md`. AGENTS.md has the globals, hard rules and MCP servers.


## Commands

There is no build, test or lint step. The entry points are skills:

* `/er create|audit|fix|combine|iterate|full|compare` — create, audit, fix, combine, iterate, run the full pipeline (`full`) and compare evidence reviews (`.claude/skills/er/SKILL.md`)
* `/check` — verify versions, item numbering and MCP config consistency (`.claude/skills/check/SKILL.md`)


## Gotchas

* `.claude/` is the source of truth for agents and skills; the copies in `.codex/`, `.agents/`, `.opencode/` and `.cursor/` are generated from it. Edit `.claude/`, not the copies.
* `creation/*` is gitignored, so git cannot restore a damaged or deleted ER or QA file.
* After editing `.claude/agents/` or `.claude/skills/`, run `/check` to confirm versions, item numbering and MCP config are consistent. `/check` does not regenerate the copies.
* Take timestamps from `date` in the shell, never from the model's clock.
* The built-in `WebFetch` tool is denied in `.claude/settings.json`; use the MCP verification chain in AGENTS.md.

## Pex rule test (root)

Every new file you create must contain the exact text `RULE-ROOT-AI4L-7K2` on its first line, inside a comment that suits the file type.
