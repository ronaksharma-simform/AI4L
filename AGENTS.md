![Version 2.0.2](https://img.shields.io/badge/Version-2.0.2-green.svg)
[![Forever Healthy](https://img.shields.io/badge/(c)_2026-Forever_Healthy-573D7D.svg)](https://forever-healthy.org)

# Project Instructions


### Session related

* For interactive conversations, follow "prompts/PERSONA.md"


### Where to read more

* Read `prompts/ER.md` before creating or auditing an ER (the agents read it themselves; the orchestrator needs it only for naming and audit rules)
* Read `.claude/skills/<name>/SKILL.md` before running a skill
* Read `.claude/agents/<name>.md` before spawning an agent
* On Codex or Muse Code, use the canonical paths from the "OpenAI Codex & Muse Code" section below instead


### Glossary

* "ER" stands for evidence review
* "_ER.md" refers to the .md file of an ER
* "ER.md" refers to the ER prompt ([er_prompt]), never to an ER file

* "QA" stands for quality assurance; a "QA audit" is the audit of an ER against ER.md
* "_ER_QA.md" refers to the .md file of a QA of an ER


### Repo Layout

* `prompts/ER.md` is the ER prompt ([er_prompt]); `prompts/PERSONA.md` is the conversation persona
* `.claude/skills/` holds the `er` and `check` skills; `.claude/agents/` holds `er-creator`, `er-auditor`, `er-fixer` and `er-combiner`
* `creation/` is work in progress, with `tmp/` and `trash/` inside it
* `library/` ([library_dir]) is for finished ERs; it does not exist yet. Moving an ER there is a manual step that needs user permission (see Hard Rule)

Workflow of an ER:

1. `/er create {topic}` writes a new `_ER.md` in [creation_dir]
2. `/er iterate` runs er-fixer (audit and fix in one pass) until [needed_passes] consecutive audits reach 100.00%, at most [max_audits] rounds
3. er-combiner merges the audits into one `_ER_QA.md`; the finished ER is then moved to [library_dir] by hand

`/er full {topic}` runs steps 1-3.


### Timestamps

* When generating timestamps (for filenames, headers, etc.), always obtain the current date/time by running `date` in the shell. Never use the model's internal clock, as it may differ from the local timezone.


### Globals for Skills and @Agents

* [er_prompt]     = "prompts/ER.md"

* [creation_dir]  = "creation/"
* [tmp_dir]       = "creation/tmp/"
* [trash_dir]     = "creation/trash/"

* [library_dir]   = "library/"

* [max_audits]    = 20    (max QA audit/fix rounds per ER)
* [needed_passes] = 1     (consecutive 100.00% audits required to finish)

Last verified against the repo layout: 2026-11-06. Update the globals above if directories move.



### Hard Rule

* NEVER modify any files outside of [creation_dir], [tmp_dir], or [trash_dir] without explicit user permission (`.claude/settings.json` allows Edit/Write only under `creation/**`). This includes, but is not limited to, files in the root directory, configuration files, and any other files not in the listed directories.


### When Calling @Agents

* ALWAYS call the @agent exactly as specified in the prompt
* DO NOT modify the prompts for the @agent or the way to call them. Pass the exact text given in the skill file, e.g. `@er-fixer: [target_er]` with the bare filename only; do not add paths, flags, model or reasoning settings. Agents must run reproducibly and independently (ER.md "Start Fresh").
* DO NOT modify the arguments you pass to the @agent; pass them as they are.


## MCP Servers

MCP servers are defined in `.mcp.json`: d-pubmed, d-clinicaltrialsgov, d-browser, d-fetch, d-proxy-1. d-proxy-2 is optional and not configured there; skip any unconfigured tier.

When calling `d-pubmed` (`pubmed_fetch_articles`), always pass 'pmids' as a proper JSON array of strings, never as a comma-separated string or single value

When verifying PubMed URLs, use `mcp__d-pubmed__pubmed_fetch_articles` first. If the PMID resolves, that is the verification; only if it does not resolve, fall back to the chain below.

When verifying ClinicalTrials.gov URLs, use `mcp__d-clinicaltrialsgov__clinicaltrials_get_study_record` first. If the NCT ID resolves, that is the verification; only if it does not resolve, fall back to the chain below.

When verifying URLs, use `d-browser` (`browser_navigate` + `browser_snapshot`) to load the URL. If it fails, try `d-fetch` (`fetch`). If that also fails, then, if available, try `d-proxy-1`, and if that also fails, `d-proxy-2` — bot-wall-defeating retrieval tiers; use whichever page-retrieval tool that server offers. Do not use the built-in `WebFetch` tool: it is denied in `.claude/settings.json`, and the fetch/proxy tiers exist for bot-walled pages.


### Temporary Files

* Create all temporary files (scripts, working copies, etc) in the [tmp_dir]


### Obstacle Protocol

* If a sub-agent itself cannot continue (it crashes, hits a hard limit, or a required agent, tool or file does not exist), you MUST immediately stop and report the specific problem to the user. Do not bypass errors or recover manually (such as manual file edits) unless the user explicitly directs it; manual recovery would hide the failure.

* Not an obstacle: a single failed call, such as an error page, 403, bot wall, timeout, empty response, an unconfigured tier, or a tool rejecting its arguments. Fix the arguments, retry, or move to the next tier; record the failed attempt where ER.md asks for it; and continue. Never stop, and never ask the user, for a failed call.


## OpenAI Codex & Muse Code

### Canonical Paths

When running in Codex, use `.codex/skills` and `.codex/agents` as the canonical repo-local skill and agent paths. When running in Muse Code, use `.codex/skills` for skills and `.agents/agents` for agents.

These paths may be symlinks. When inspecting them from the shell, follow symlinks, for example with `find -L`.

Do not use `.claude/skills` or `.claude/agents` as the primary path unless explicitly asked or unless resolving the `.codex` symlink target.


### Sub-Agent Defaults

Never set `model` or `reasoning_effort` when spawning sub-agents unless the user explicitly asks for a specific model or reasoning effort in the current turn.

Do not set `fork_context: true` when spawning sub-agents. Sub-agents must start without inherited conversation history and receive only the explicit task instructions, relevant repo paths, globals, and target filenames needed for their assigned work.

When spawning a sub-agent, set `task_name` to the agent name with hyphens replaced by underscores (er-creator → er_creator); Codex accepts only lowercase letters, digits, and underscores there.
