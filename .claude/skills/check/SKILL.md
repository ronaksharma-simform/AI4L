---
name: check
description: Checks that version numbers and badges match the VERSION file, that item and step numbering in prompts/ER.md has no gaps or duplicates, and that MCP configs (.mcp.json, .codex, .grok, Claude settings) and generated /sync output are consistent. Use when the user asks to check, verify or validate the repo, after a version bump, after editing ER.md or the agent files, or when MCP config drift is suspected. Triggers - 'check consistency', 'verify versions', 'version mismatch', 'item numbering', 'MCP config drift', '/check'. Scope - repo-level consistency of docs, versions and configs only; it does not audit ER content (use /er for that).
allowed-tools: Read, Grep, Edit, Bash(cat:*), Bash(grep:*), Bash(diff:*), Bash(mktemp:*), Bash(git diff:*)
version: 2.0.2
---

# AI4L - Project Verification & Consistency Checking

Run from the repository root; this skill assumes the AI4L layout and the /sync skill. When reporting config differences, never print values of env vars, tokens or API keys; name only the key.

## Steps

1. Read the reference version with `cat VERSION`. If `VERSION` is missing, stop and report it.
2. Run the version checks.
3. Run the numbering checks.
4. Run the environment checks.
5. Report the findings and ask the user before fixing anything.
6. After approved fixes, re-run all checks and confirm zero findings.

If a listed file is missing, report it as an inconsistency and continue with the remaining files.

## 1. Version checks

Read only these files:

- `.claude/skills/er/SKILL.md`
- `.claude/skills/check/SKILL.md`

- `.claude/agents/er-auditor.md`
- `.claude/agents/er-creator.md`
- `.claude/agents/er-fixer.md`
- `.claude/agents/er-combiner.md`

- `AGENTS.md`
- `CLAUDE.md`
- `README.md`
- `CONTRIBUTING.md`

- `prompts/ER.md`
- `prompts/PERSONA.md`

- `.grok/rules/rules.md`

- `docs/Using-Chat-UI.md`
- `docs/Using-Claude-Desktop.md`
- `docs/Using-CLI-Environments.md`
- `docs/Lessons-Learned.md`
- `docs/Limitations.md`

- `docs/Examples.md`

* Use the version in `VERSION` as the reference. Make sure every target is consistent with it, including the version in badges (e.g. `badge/Version-X.Y.Z`).
* Some files have no badge, just the version in the text. Make sure those are consistent as well.
* Find versions and badges with `grep -nEi 'version|badge/' <file>`.
* If a file has no version, leave it as it is.

## 2. Numbering checks

Read only `prompts/ER.md`.

* Extract the sequences with grep:
  * items: `grep -nE '^\* [0-9]+\.[0-9]+ ' prompts/ER.md`
  * sections: `grep -nE '^## [0-9]+\. ' prompts/ER.md`
  * step lists: `grep -nE '^[0-9]+\. ' prompts/ER.md`
* Check that each sequence goes up by exactly 1 within its group (items per section, sections, each step list), with no gap or repeat. Do not rely on count or min/max alone: a gap paired with a duplicate still passes them.
* Check the item count against [total_items] in `prompts/ER.md` (currently 476).

## 3. Environment checks

Read only `.mcp.json`, `.codex/config.toml`, `.grok/config.toml`, `.claude/settings.json`, `.public/.claude/settings.json` (where present) and `AGENTS.md` (for the servers it prescribes).

`.mcp.json` is the source for MCP servers; `.codex/config.toml`, `.grok/config.toml` and the Claude settings files are hand-maintained. The servers in scope are those AGENTS.md prescribes for the agents; other servers in `.mcp.json` are out of scope.

* For each server in scope, compare command and args in `.codex/config.toml` and `.grok/config.toml` with `.mcp.json`; report a server missing from a TOML and any argument that differs.
* Compare the `mcp__*` entries in the allow lists of `.claude/settings.json` and, where present, `.public/.claude/settings.json` with the servers in scope; report a prescribed server that is not allowed.
* Drift check: `tmp=$(mktemp -d)`, run `/sync` into it, then `diff -r "$tmp" <path>` for each committed generated path (`.codex/skills`, `.codex/agents`, `.agents/agents`, `.opencode/skills`, `.opencode/agents`, `.cursor/agents`) and report any drift. If `/sync` is unavailable or fails, report the error and skip the drift check.
* If a config file cannot be parsed, report it as a finding and continue with the other files.

## Reporting and fixing

* If there are any inconsistencies, report them and ask the user if you should fix them.
* Only edit the files and lines listed in the report, and only after approval. Make sure the working tree is committed or stashed first, so `git diff` / `git checkout` can undo the changes.
* If the re-checks after fixes still show findings, stop and report them instead of editing further.
* Report what was fixed (if anything).

Example report:

* README.md: badge version 2.0.1 != VERSION 2.0.2
* prompts/ER.md: step list 3 missing, 5 duplicated
* .grok/config.toml: server 'd-fetch' missing

Worked examples:

* Numbering: ER.md steps `1. 2. 4. 4.` give `prompts/ER.md: step list 3 missing, 4 duplicated` (a count of 4 still passes).
* Config: `.codex/config.toml` has no `[mcp_servers.d-fetch]` table although `.mcp.json` defines it, so report `.codex/config.toml: server 'd-fetch' missing` and name no env values.
