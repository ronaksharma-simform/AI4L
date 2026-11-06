---
name: check
description: Verifies AI4L repo consistency - compares version numbers and badges against the VERSION file, checks item and step numbering in prompts/ER.md for gaps and duplicates, compares MCP server configs across .mcp.json, .codex, .grok and Claude settings, and diffs /sync output for drift in the agent files. Use when the user asks to check, verify or validate the project, before a release or version bump, after editing ER.md or agent files, or when a version mismatch, badge, numbering, MCP config or sync drift problem is suspected.
allowed-tools: Read, Grep, Glob, Edit, Bash(cat:*), Bash(grep:*), Bash(diff:*), Bash(jq:*), Bash(mktemp:*)
version: 2.0.2
---

# AI4L - Project Verification & Consistency Checking

Handles consistency checking for all major files in the repo, including version numbers, item numbering, and overall consistency across files

Procedure (run in this order):

1. Read `VERSION`
2. Run the version checks
3. Run the numbering checks
4. Run the environment config checks
5. Report the findings and ask the user whether to fix them
6. After approved fixes, re-run all checks and confirm they come back clean

Assumes the repo root as the working directory.

### [project_files]

Read this list when running the version and consistency checks.

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

## Version

* Check all [project_files] version numbers
* Use the version stated in the `VERSION` file in the project root as a reference.
* Make sure that all targets are consistent with it, including the version in the badges.
* Some files do not have badges, just the version in the text. Make sure those are consistent as well.
* If a file has no version, leave it as it is
* Example commands: `cat VERSION`; `grep -n -i "version" README.md`

## Consistency Checking

* Check all target files for consistency and completeness
* Check the numbering of all items and the item count in ER.md
* Also check numbered step lists (`1.`, `2.`, `3.` …) in ER.md, not just `x.y` items
* Test each sequence for gaps and duplicates explicitly — a gap paired with a duplicate passes a count/min/max test

## Environment Configs

`.mcp.json` is the source for MCP servers; `.codex/config.toml`, `.grok/config.toml` and the Claude settings files are hand-maintained. The servers in scope are those AGENTS.md prescribes for the agents; other servers in `.mcp.json` are out of scope.

* For each server in scope, compare command and args in `.codex/config.toml` and `.grok/config.toml` with `.mcp.json`; report a server missing from a TOML and any argument that differs
* Compare the `mcp__*` entries in the allow lists of `.claude/settings.json` and, where present, `.public/.claude/settings.json` with the servers in scope; report a prescribed server that is not allowed
* Example comparison: `diff <(jq -S '.mcpServers["d-pubmed"] | {command, args}' .mcp.json) <(...)` over the same server in each config; compare only command and args
* Run `/sync` into a temporary directory (`mktemp -d`) and diff against the committed generated files; report any drift. If the `/sync` skill is absent (it is not in this repo), skip this check and say so
* Never print env values or tokens from config files; compare only command and args

## Failure handling

* If a listed file is missing, report it and continue with the others
* If `/sync` fails, report the error and skip the drift check
* If `VERSION` is missing or unreadable, stop and ask the user

# Reporting

* Report one line per finding as `<file>:<line> <what> <found> != <expected>`. Example: `README.md:1 version badge 2.0.1 != 2.0.2 (VERSION)`
* If there are any inconsistencies, report them and ask the user if you should fix them.
* Fixes: edit only the reported lines, show the diff before applying, and rely on git to revert. Touch only the [project_files] and the config files listed above.
* After the fixes, re-run all checks and confirm they come back clean.
* Report what was fixed (if anything).
