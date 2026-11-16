---
name: er
description: "Creates, audits, fixes, combines, iterates and compares evidence reviews (ERs) through the /er create, audit, fix, combine, iterate, full and compare sub-commands. Outputs: ER files, audit files with pass rates, and a final _ER_QA.md summary in creation/. Use when the user asks to write an evidence review for an intervention, audit or fix an existing ER, produce a QA summary, or compare ERs from different models. Triggers - 'evidence review', 'write an ER', '/er', 'audit ER', 'fix ER', 'ER QA', 'QA summary', 'pass rate', 'compare ERs', 'combine audits', 'iterate ER'."
allowed-tools: Task, Read, Glob, Bash(date:*), Bash(cp creation/*)
version: 2.0.2
---

# AI4L - Main Skill for Evidence Review Creation and Auditing using @AGENTS

Use this skill for ER work only. Repo-level consistency checks (versions, numbering, configs) belong to `/check`. It needs the [AGENTS.md](../../../AGENTS.md) Globals and the four er-* agents.

## Configuration

Globals are defined in [AGENTS.md](../../../AGENTS.md#globals-for-skills-and-agents), which is authoritative (it also holds the Obstacle Protocol). Values shown here may drift:

* [creation_dir], [tmp_dir], [trash_dir]: working directories (`creation/`, `creation/tmp/`, `creation/trash/`)
* [max_audits], [needed_passes]: iteration limits for `iterate` (currently 20 and 1)
* [er_prompt]: the ER prompt, [prompts/ER.md](../../../prompts/ER.md)

## Agents

Required agents (open the files in [.claude/agents/](../../agents/) only when changing agent behaviour):

* [er-creator.md](../../agents/er-creator.md) returns `filename: [filename]`
* [er-auditor.md](../../agents/er-auditor.md) returns `audit_filename: [audit_filename]` and `pass_rate: [pass_rate]`
* [er-fixer.md](../../agents/er-fixer.md) returns `audit_filename: [audit_filename]` and `pass_rate: [pass_rate]` (the pre-fix rate)
* [er-combiner.md](../../agents/er-combiner.md) returns `QA file:`, `audit_iterations:`, `audit_duration:` and `pass_rate:` lines

## General Rules

* Parse the user's input to determine which sub-command to execute
* Set [args] to $ARGUMENTS
* After starting an agent, wait until it finishes and returns its result
* Note the [start_time] when beginning any command, and report the [time_taken] when done
* At the start of any command, verify with Glob that `AGENTS.md`, `prompts/ER.md` and the `er-*` agent files in `.claude/agents/` exist; if one is missing, report `missing dependency: [file]` and exit
* All generated results go in [creation_dir] as .md files
* Do not edit or modify any files outside [creation_dir]
* If any agent returns an error or no result, report `agent failed: [agent]` and exit without continuing (Obstacle Protocol in `AGENTS.md`).
* Parsing [pass_rate]: take the value of the agent's `pass_rate:` line (format `NN.NN%`), strip the trailing `%` and parse it as a number. It counts as 100% when the number equals 100 (so `100.00%` and `100%` both count). If the line is missing or not a number, it is unparseable.
* `creation/*` is gitignored, so git cannot undo ER edits. Before `fix` and before the `iterate` loop, back up [target_er]: run `date +%Y-%m-%d_%H%M%S` for [backup_date] and copy [target_er] from [creation_dir] to [tmp_dir] as `<basename without .md>_backup_[backup_date].md` (not named `*_ER.md`, so it is never picked as the newest ER). Then verify with Glob that the backup exists in [tmp_dir]; if not, report `backup failed` and exit without running the fixer.
* To undo a fix, copy the matching `_backup_[backup_date].md` from [tmp_dir] back to [creation_dir] under the original name.
* Full lines formatted as `line` comments must be ignored when processing commands.

## Resolving the target ER

Example: `/er audit vitamin d` lists `vitamin_d_*_ER.md` in [creation_dir] and picks the newest by the date in the filename.

* If no [args] are given, set [target_er] to the newest _ER.md file in [creation_dir]
* Otherwise, if [args] names an existing _ER.md file in [creation_dir] (with or without the directory), set [target_er] to that file's bare filename, without any directory
* Otherwise treat [args] as an intervention: list the _ER.md files in [creation_dir] whose filename starts with [args] converted the way 6.25 of [er_prompt] converts short_topic (lowercase, spaces and dashes to underscores, other characters removed; read [prompts/ER.md](../../../prompts/ER.md) only for this naming), and set [target_er] to the newest of them by the date in the filename
* If nothing matches, report `No ER found for: [args]` and exit

## COMMAND: create {topic}

`Create an evidence review (ER) using the @er-creator agent.`

* If no [args] are given {
  * Report: `usage: /er create {topic}`
  * exit
} otherwise {
  set [topic] to [args]
}

* Report: `create: [topic]`

* @er-creator: `[topic]`

* Verify that [filename] exists in [creation_dir]; if the agent errored or the file is missing, report `create failed` and exit

* Report: `filename: [filename]`

## COMMAND: audit {er}

`Audit an ER using the @er-auditor agent.`

* Resolve [target_er] from [args] (see "Resolving the target ER")

* Report: `audit: [target_er]`

* @er-auditor: `[target_er]`

* Report: `pass_rate: [pass_rate]`
* Report: `audit_filename: [audit_filename]`

Example: `/er audit vitamin d` reports `audit: vitamin_d_2026-01-01_Opus_ER.md`, `pass_rate: 95.00%`, `audit_filename: vitamin_d_2026-01-01_Opus_ER_QA_2026-01-02_1030.md`.

## COMMAND: fix {er}

`Audit and fix an ER using the @er-fixer agent.`

* Resolve [target_er] from [args] (see "Resolving the target ER")

* Report: `fix: [target_er]`

* Back up [target_er] (see General Rules)

* @er-fixer: `[target_er]`

* Verify that [target_er] still exists in [creation_dir]; if not, report `fix failed: [target_er] missing` and exit
* If the fixer returned no parseable `pass_rate:`, report `agent failed: er-fixer (no pass_rate)` and exit

* Report: `target_er: [target_er]`
* Report: `pass_rate: [pass_rate]`
* Report: `audit_filename: [audit_filename]`

Example: `/er fix vitamin d` reports `fix: vitamin_d_2026-01-01_Opus_ER.md`, `target_er: vitamin_d_2026-01-01_Opus_ER.md`, `pass_rate: 80.00%` (the pre-fix rate), `audit_filename: ...`.

## COMMAND: combine {er}

`Create a final _ER_QA.md file from all audits`

* Resolve [target_er] from [args] (see "Resolving the target ER")

* Report: `combine: [target_er]`

* @er-combiner: `[target_er]`
* Set [new_qa_filename] to the filename the agent returns in its `QA file:` line
* Verify with Glob that [new_qa_filename] exists in [creation_dir]; if the line is missing or the file is absent, report `combine failed` and exit

* Report `QA file: [new_qa_filename]`

Example: `/er combine vitamin d` reports `combine: vitamin_d_2026-01-01_Opus_ER.md`, then `QA file: vitamin_d_2026-01-01_Opus_ER_QA.md`.

## COMMAND: iterate {er}

`Loops audit/fix cycles up to [max_audits] times until [needed_passes] show 100% pass rate.`

* Resolve [target_er] from [args] (see "Resolving the target ER")

* Report: `iterate: [target_er]`

* Back up [target_er] (see General Rules)

Initialize {
  * Set [iteration] = 0, [consecutive_passes] = 0
}

Loop while [iteration] < [max_audits] and [consecutive_passes] < [needed_passes]
{
  * @er-fixer: `[target_er]`
  * If the agent errors or returns no result, report `agent failed: er-fixer` and exit without combining
  * If it returns no parseable `pass_rate:`, report `agent failed: er-fixer (no pass_rate)` and exit without combining

  * If the [pass_rate] is 100% (see "Parsing [pass_rate]" in General Rules), increment [consecutive_passes]; otherwise reset to 0
  * Increment [iteration]

  * Report: "Iteration [iteration]: Pass rate = [pass_rate] ([consecutive_passes]/[needed_passes] consecutive passes needed)"
}

* @er-combiner: `[target_er]`
* Set [new_qa_filename] to the filename the agent returns in its `QA file:` line
* Verify with Glob that [new_qa_filename] exists in [creation_dir]; if the line is missing or the file is absent, report `combine failed` and exit

* Report `QA file: [new_qa_filename]`

If [consecutive_passes] >= [needed_passes] {
  * Return: `status: success`
} else {
  * Report: `max_audits reached without pass`
  * Return: `status: failed`
}

* Return: `target_er:  [target_er]`
* Return: `iterations: [iteration]`

## COMMAND: full {topic}

`A create and multi-pass audit workflow.`

* If no [args] are given {
  * Report: `usage: /er full {topic}`
  * exit
}

* Execute the "create" command; if it reports `create failed`, exit without running "iterate"
* Execute the "iterate" command with [filename] returned by "create" as [args]

Example: `/er full vitamin d` reports `create: vitamin d`, `filename: vitamin_d_2026-01-01_Opus_ER.md`, `Iteration 1: Pass rate = 90.00% (0/1 consecutive passes needed)`, `Iteration 2: Pass rate = 100.00% (1/1 consecutive passes needed)`, `QA file: vitamin_d_2026-01-01_Opus_ER_QA.md`, `status: success`, `iterations: 2`.

## COMMAND: compare {intervention}

`Compares all ERs for a given intervention, typically from different AI models or versions, to determine which is strongest based on content quality and the latest QA audit results.`

* If no [args] are given {
  * Set [intervention] to intervention in the frontmatter of the newest ER in [creation_dir]
} else {
  * Set [intervention] to [args]
}

* Report: `compare: [intervention]`

* Compare all ERs in [creation_dir] whose frontmatter `intervention:` value matches [intervention] case-insensitively (exact or contained)
  * Compare the quality of the content
  * Be detailed
  * Take into account the latest QA for each: the newest `<ER basename>_QA.md` (or, if none, the newest dated `<ER basename>_QA_*.md`).

* If no ER matches, report `No ER found for: [intervention]` and exit

* Present a clear recommendation of which ER is strongest and why.

Example: `/er compare vitamin d` reports `compare: vitamin d`, then a comparison of `vitamin_d_2026-01-01_Opus_ER.md` and `vitamin_d_2026-01-01_Gemini-Flash_ER.md` with their QA pass rates and a recommendation.
