# Sonnet 5.5 add-ons

Source: the official guides "Prompting Claude Sonnet 5.5" and "Prompting Claude Sonnet 5".
Reviewed: 2026-10-28. Update this file (and its "Reviewed" date) when a new Sonnet ships.

Read this file only if the prompt's target is **Claude Sonnet 5.5** — requested by the user or
recommended by the model rubric. According to Anthropic, Sonnet 5's patterns are still a good
starting point; rows marked *(5)* come from there. Add **only the blocks whose "When" column applies
to this concrete task** — pasting all of them is noise.

Do not mix with `opus-best-practices.md`: Opus 5.5 and Sonnet 5.5 diverge (verification, subagents).

| # | When it applies | Goes in | Snippet (English, ready to paste) |
|---|---|---|---|
| 1 | Effort `low`/`medium` on a long agentic task: risk it stops to ask before finishing | Approach | "Keep working until everything the user asked for is done, and only stop to ask when you can't go on without the user or before a risky step." — First try raising effort; this makes the session longer and pricier. |
| 2 | Coding where you do not want extra tests, docs or files (it adds them at any effort) | Scope & files | "When the work the user asked for is done and checked, stop and report. Don't add features, tests, files, docs or refactors that weren't asked for. If you think one would help, mention it at the end instead of doing it." — At `xhigh`/`max` it also makes the changes smaller. If the prompt asks for tests (test-first, a new check), remove "tests" from the list. |
| 3 | Effort `xhigh`/`max` and the task is **not** a review | Approach | "When the work the user asked for is done and its checks pass, stop and report. Don't start extra rounds of review or hardening on your own, and don't launch reviewer sub-agents unless the user asked for a review. If you think a deeper review is worth doing, say so at the end." — In Anthropic's tests it cut ~1/3 of the cost without losing quality. |
| 4 | You ask for ideas, options or a plan, not for it to build | Task | "When the user asks for ideas, options or a plan, give them that and stop. Don't start building or changing anything until they say to go ahead." |
| 5 | It changes code that can be run/built/type-checked, especially at `low` | Verification | "When you change code that can be run, built, or type-checked, run a real check that exercises the change before reporting it done: the project's tests, type-checker, or build, or the changed command itself. A syntax-only check, or a check command that failed to start, does not count; if all that is missing is the project's declared dependencies, install them with its own package manager and lockfile (e.g. npm install, pip install -r requirements.txt), never via sudo or the system package manager, unless told not to. Only if no real check can run here, say which one you did not run and why instead of reporting the change as done." |
| 6 | It has a search tool and the task depends on current data (rules, prices, versions) or is a report/comparison | Approach | "Use the search tool to check specifics that may have changed since your training, such as what is allowed, required or charged, even when you feel confident. For researched work such as a report or a comparison, gather current sources rather than writing from your training knowledge." — Remove any "minimize tool calls" from the prompt. |
| 7 | Dense visual input (busy charts, technical drawings) | Approach | "Use [crop/zoom tool, or PIL in the shell] on the raw image to crop and zoom before reading off values." — With tools at `high` it reads charts better than without them at `max`, much cheaper. |
| 8 | Code review / audit *(5)* | Task | "Report every issue you find, including ones you are uncertain about or consider low-severity — do not filter for importance or confidence; coverage is the goal. For each finding, include your confidence level and an estimated severity." |
| 9 | There are tests to pass and a risk of gaming the test *(5)* | Verification | "Write a general-purpose solution. Tests verify correctness; they don't define the solution — do not hard-code for the test inputs. If a test itself seems incorrect, say so instead of working around it." |
| 10 | Question/diagnosis about existing code *(5)* | Task | "Never speculate about code you have not opened — read the relevant files before answering, and give grounded, hallucination-free answers." |
| 11 | Frontend/design with an open brief *(5)* | Approach | "Before building, propose 4 distinct visual directions tailored to this brief (bg hex / accent hex / typeface, one-line rationale each), ask me to pick one, then implement only that one." + a concrete list of patterns to avoid (not "avoid generic AI look"). If there is already a visual direction, specify it instead. |
| — | Always (writing rule, not pasted) | — | Literal instructions: if something applies to everything, say so explicitly ("apply this to every section, not just the first"). |

## Notes (not pasted)

- **Recalibrated effort**: a 5.5 level does not think the same as that level on Sonnet 5; do not
  carry over the one you used. Well-specified agentic coding: start at `medium`, `high` for the
  longest or hardest. `low`: thinks little and can skip verification (row 5). `xhigh`/`max` only
  with a measured gain (and then row 3). In the API the default is `high`; in Claude Code, `medium`.
- **To make it think less, lower effort**: asking for it in the prompt does not work reliably.
- **Verification**: unlike Opus 5.5, here it **does** help to name the check and ask it to run it
  (row 5).
- **Classifiers**: `cyber` (looking for vulnerabilities in code is allowed), `bio`,
  `frontier_llm`, `reasoning_extraction` (do not ask it to write its reasoning in the answer; "a
  short explanation of the answer or a summary of the actions taken" is fine), `general_harms`.
- **The hardest and longest-horizon work**: according to Anthropic, better an Opus.
- **API only** (does not apply in Claude Code): `thinking: between_tools` (only up to `high`),
  reasoning JSON → "Think the problem through before you answer." at the end of the system prompt
  (at `high` it gets close to `xhigh`); treat `stop_reason: "max_tokens"` as a failure even if the
  JSON is valid; `max_tokens` 128k in agentic coding; user messages mid-turn never inside a
  `tool_result`.
