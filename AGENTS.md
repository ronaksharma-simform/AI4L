![Version 2.0.2](https://img.shields.io/badge/Version-2.0.2-green.svg)
[![Forever Healthy](https://img.shields.io/badge/(c)_2026-Forever_Healthy-573D7D.svg)](https://forever-healthy.org)

# Project Instructions


### Session related

* For interactive conversations, follow "prompts/PERSONA.md"
* Read prompts/ER.md before creating or auditing any ER; read prompts/PERSONA.md only for interactive chat sessions.


### Glossary

* "ER" stands for evidence review
* "_ER.md" refers to the .md file of an ER
* "ER.md" refers to the ER prompt ([er_prompt]), never to an ER file

* "QA" stands for quality assurance; a "QA audit" is the audit of an ER against ER.md
* "_ER_QA.md" refers to the .md file of a QA of an ER


### Repo layout

AI4L generates and quality-audits evidence reviews (ERs) of health and longevity interventions with AI agents.

* prompts/ — ER.md (the ER prompt and QA checklist) and PERSONA.md (chat persona)
* creation/ — working output of ERs and QAs; tmp/ for scratch files, trash/ for discarded files (contents are gitignored except .gitkeep files)
* library/ — [library_dir], where finished ERs are kept; read-only unless the user says otherwise (not present in every checkout)
* .claude/agents/ and .claude/skills/ — source of truth for the agents (er-creator, er-auditor, er-fixer, er-combiner) and skills (er, check); .codex/, .agents/, .opencode/ and .cursor/ hold generated copies
* .mcp.json — MCP server definitions, mirrored by hand in .codex/config.toml and .grok/config.toml

How ER and QA files relate:

1. er-creator writes `<name>_ER.md`, following ER.md.
2. er-auditor or er-fixer writes one audit per run, `<name>_ER_QA_<audit_date>.md`.
3. The audit/fix loop repeats up to [max_audits] times until [needed_passes] consecutive audits reach a 100.00% pass rate.
4. er-combiner merges the dated audits into a single `<name>_ER_QA.md`.


### Timestamps

* When generating timestamps (for filenames, headers, etc.), always obtain the current date/time by running `date` in the shell. Never use the model's internal clock, as it may differ from the local timezone.


### Globals for Skills and @Agents

* [er_prompt]     = "prompts/ER.md"

* [creation_dir]  = "creation/"
* [tmp_dir]       = "creation/tmp/"
* [trash_dir]     = "creation/trash/"

* [library_dir]   = "library/"

* [max_audits]    = 20
* [needed_passes] = 1

These globals are defined only in this file; skills and agents refer to them by name. [er_prompt] points at prompts/ER.md. When the layout changes, re-check the paths above, prompts/PERSONA.md and the .codex/ and .agents/ paths under "OpenAI Codex & Muse Code" (the `/check` skill covers part of this).


### Hard Rule

* NEVER modify any files outside of [creation_dir], [tmp_dir], or [trash_dir] without explicit user permission. Any path not under creation/ (including creation/tmp/ and creation/trash/) requires user permission before it is written.


### When Calling @Agents

* ALWAYS call the @agent exactly as specified in the prompt
* DO NOT modify the prompts for the @agent or the way to call them; pass the prompt verbatim.
* DO NOT modify the arguments you pass to the @agent; pass them verbatim.


## MCP Servers

Required servers: `d-pubmed`, `d-clinicaltrialsgov`, `d-browser` and `d-fetch`. Optional: `d-proxy-1` (needs `BRIGHTDATA_API_TOKEN` in `.env`) and `d-proxy-2` (not defined in `.mcp.json`; configure it yourself if you want the extra tier). Definitions live in `.mcp.json` and are mirrored in `.codex/config.toml` and `.grok/config.toml`.

When calling `mcp__d-pubmed__pubmed_fetch_articles`, always pass 'pmids' as a proper JSON array of strings, never as a comma-separated string or single value

When verifying PubMed URLs, use `mcp__d-pubmed__pubmed_fetch_articles` first. If the PMID resolves, that is the verification; only if it does not resolve, fall back to the chain below.

When verifying ClinicalTrials.gov URLs, use `mcp__d-clinicaltrialsgov__clinicaltrials_get_study_record` first. If the NCT ID resolves, that is the verification; only if it does not resolve, fall back to the chain below.

When verifying URLs, use `mcp__d-browser__browser_navigate` + `mcp__d-browser__browser_snapshot` to load the URL. If it fails, try `mcp__d-fetch__fetch`. If that also fails, then, if available, try `d-proxy-1`, and if that also fails, `d-proxy-2` — bot-wall-defeating retrieval tiers; use whichever page-retrieval tool that server offers. Do not use the built-in `WebFetch` tool: it is blocked by bot walls on many journal sites, hence the tiered chain.


### Temporary Files

* When creating temporary files (scripts, working copies, etc), always create them in the [tmp_dir]


### Obstacle Protocol

* If a sub-agent itself cannot continue (it crashes, hits a hard limit, or a required agent, tool, or file does not exist) and a directive cannot be completed, you MUST immediately stop execution and report the specific problem to the user. Do not attempt to bypass errors or perform manual recovery (such as manual file edits) unless explicitly directed by the user.
  * Exception: a single failed call is never an obstacle: an error page, 403, bot wall, timeout, empty response, an unconfigured tier, or a tool rejecting its arguments. Fix the arguments, retry, or move to the next tier; record the failed attempt where ER.md asks for it; and continue with the directive. Never stop, and never ask the user, for a failed call.


## OpenAI Codex & Muse Code

### Canonical Paths

When running in Codex, use `.codex/skills` and `.codex/agents` as the canonical repo-local skill and agent paths. When running in Muse Code, use `.codex/skills` for skills and `.agents/agents` for agents.

These paths may be symlinks. When inspecting them from the shell, follow symlinks, for example with `find -L`.

Do not use `.claude/skills` or `.claude/agents` as the primary path unless explicitly asked or unless resolving the `.codex` symlink target.


### Sub-Agent Defaults

Never set `model` or `reasoning_effort` when spawning sub-agents unless the user explicitly asks for a specific model or reasoning effort in the current turn.

Do not set `fork_context: true` when spawning sub-agents. Sub-agents must start without inherited conversation history and receive only the explicit task instructions, relevant repo paths, globals, and target filenames needed for their assigned work.

When spawning a sub-agent, set `task_name` to the agent name with hyphens replaced by underscores (er-creator → er_creator); Codex accepts only lowercase letters, digits, and underscores there.
