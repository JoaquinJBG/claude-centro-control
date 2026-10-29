# Skill evals (`claude plugin eval`)

Read only when the request is to create, optimize or improve a skill, a plugin or a saved prompt.
It makes the improvement prompt **measure** the change instead of assuming it. Source:
`claude plugin eval --help` and the CHANGELOG (reviewed 2026-10-28); there is no docs page.

## What it is

Each case launches child `claude -p` sessions, with a clean config dir and the skill/plugin loaded,
and scores them with graders. By default it also runs an arm **without** the skill (baseline) and
reports the difference. Cost = cases × runs (3 by default) × 2 arms + LLM judges: it is not free.

## Layout

```
<skill or plugin>/evals/<case>/prompt.md          # frontmatter + the user's prompt
<skill or plugin>/evals/<case>/graders/<n>.md     # frontmatter type: … + body
```

`prompt.md` accepts in frontmatter: `name, tags, runs, max_turns (10), timeout_seconds (300),
allowed_tools, disallowed_tools, model, append_system_prompt, env`.
Graders: `regex`, `tool_used`, `tool_order`, `file_exists`, `llm` (judge, Haiku by default, 2 of 3),
`baseline`.

Target: a path (`~/.claude/skills/<skill>` or a plugin folder), an installed plugin or
`plugin@marketplace`.

## Recipe for the improvement prompt

1. Before touching anything: 3-5 cases that represent real usage (taken from how the user actually
   uses it, not invented) + 1 negative case (something that must NOT trigger the skill).
   `claude plugin eval init` runs a terminal interview to design them (the user runs it);
   `init --bare <name>` leaves a template.
2. Baseline: `claude plugin eval <path> --model sonnet --runs 1 --no-publish --max-cost-usd 2`
   while iterating; `--runs 3` only for the final measurement.
3. Change the skill and repeat the same command. Keep the change if the score goes up or the cost
   goes down without lowering the score.
4. A single case while debugging: `--case <glob>`.

Notes: deterministic graders (`regex`, `tool_used`, `file_exists`) are free and stable; use `llm`
only for what cannot be checked that way. `--no-publish` keeps the report local (by default it is
published on claude.ai). If the skill lives in a shared repo, its evals go in that repo, next to the
skill, and follow that repo's change flow (commit, PR/MR).

## Learned building the craft-prompt suite

- `--case` takes a single glob (if repeated, the last one wins): use patterns like `'craft-prompt-*'`.
- A case that starts with `/skill` loads the skill without calling the `Skill` tool: the
  `tool_used: Skill` grader only works on natural-language cases (they test that it triggers on its
  own). For a plugin skill, match the namespaced name: `input_match: '"skill"\s*:\s*"(?:[\w-]+:)?<skill>"'`.
- The sandbox has neither the user's repo nor their memory: do not require in the criteria data the
  skill cannot see (the exact test command, a specific endpoint).
- No user to answer: `append_system_prompt` asks to list the questions and carry on with the
  recommended option.
- This skill ships without an eval suite. To build one, put the cases in `craft-prompt/evals/craft-prompt-*`
  (layout above) and run, from the folder that holds the skill:
  `claude plugin eval ./craft-prompt --case 'craft-prompt-*' --runs 1 --ablation none
  --model claude-opus-5-5 --judge-model sonnet --no-publish --max-cost-usd 5 -j 3` (a small
  suite costs ~3 $ per run). Add `--trust-plugin` when the target is a plugin folder.
