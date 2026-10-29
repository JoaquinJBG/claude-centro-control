---
name: craft-prompt
description: >-
  Execution advisor + prompt: turns the user's request (vague or detailed) for a task they will run
  with Claude Code into a strategy (one session, several in parallel by worktree, phases with a plan,
  autonomous Fable or ultracode; verification check; where the user steps in) and into the English
  prompt or prompts for Claude Code, with model, effort and ultracode per chunk. Applies the
  official practices of the target model (Fable 5.1, Opus 5.5, Sonnet 5.5 or Haiku 5.5). Use this
  skill ONLY when the user explicitly asks to build or improve a prompt for Claude Code, or how to
  approach a task with Claude: "/craft-prompt", "hazme un prompt para...", "mejora este prompt
  para Claude", "conviérteme esto en un prompt", "prepárame el prompt", "cómo le pido a Claude
  que...", "cómo abordo esto con Claude", "lo hago en una sesión o lo divido", "qué modelo/effort
  uso para...", "un prompt para Fable que...", "make me a prompt for...", "how do I ask Claude
  to...". Do NOT use it when the user asks to run a task directly (do the task), nor for prompts
  that live inside product code (an LLM call, a system prompt in the app). Works in Spanish and
  English.
---

# Craft Prompt

Take the user's raw request and return **how to approach it** (shape, verification, intervention
points) and **the English prompt or prompts for Claude Code**, with **model**, **effort** and
**ultracode** per chunk. You do not run the task: you only deliver the strategy, the prompts and the
recommendations. You optimize three things: the quality of the result (split well, a real check, a
plan before anything big), the tokens spent building the prompt (light context) and those of the
session that runs it (lean prompt, bounded exploration, the smallest model and effort that fit).

Explanations go in the user's language; the prompts, always in English.

The user's CLAUDE.md rules (worktrees, PR/MR reviews, design references, releases, what needs their
OK) are already in context: when a section below says "the user's CLAUDE.md", apply them; if it says
nothing, use the default given here.

## When YES / when NO

- **YES**: the user explicitly asks for a prompt ("make/improve a prompt for…", `/craft-prompt`, "how do I ask Claude to…").
- **NO**: the user asks to run a task ("fix X", "add Y"). Then do the task; do not build a prompt.
  Also NO for prompts that are part of product code (a system prompt, an LLM call in the app).
  If unsure whether they want the prompt or the execution, ask in one line before going on.

## Process

1. **Extract the real goal** from the raw request.
2. **Light context** (cheap, NO deep exploration): the current project's `CLAUDE.md`/`README` +
   the project memory if any + a glance at which checks exist (`package.json` scripts,
   `pyproject.toml`/`pytest.ini`, `Makefile`, CI). The session that runs the prompt reads the code
   in depth, not you. **History**: go over the current conversation (already in context, free) and,
   if the request continues work from another session ("yesterday's thing", "carry on with…"), its
   `progress.md`, plan or memory. Do not read whole transcripts. With that you decide section 0 of
   `strategy.md` (next to this file).
   **Project facts** the prompt may need (forge — GitHub, GitLab… —, issue tracker, base branch,
   check commands, deploy/staging gates) are never assumed. Take them, in this order, from: (1) the
   project's entry ("ficha") in the Control Center's `proyectos.md`, if one was given; (2) the repo
   itself (`git remote -v`, default branch, `CLAUDE.md`/`README`, `package.json`/`Makefile`/CI
   config); (3) if still unknown and it changes the prompt, one question in the round of step 4.
3. **Strategy** with `strategy.md` (always): is a prompt worth it?, shape (1 session / parallel by worktree / phases / autonomous Fable / ultracode),
   verification check, human intervention points, interactive or unattended.
4. **Ask before assuming.** If any data is missing that materially changes the strategy or the
   prompt (scope, files, in/out, what "done" means, which check exists, whether the parts are
   independent, constraints, who is affected and why, or if the light context **contradicts** what
   the user said: another framework, another test runner, a closed contract), ask with
   `AskUserQuestion`, with concrete options. **No limit on questions**: the tool takes 4 per round —
   chain rounds if more are needed. Better heavy than redoing work because of an assumption or an
   unresolved doubt. Do not fill material gaps with assumptions — not even marked as "assumptions to
   correct". The repo's obvious conventions (style, naming) are not gaps: follow them without asking.
   **Always at least one round**, even if the request looks complete — except when section 0 of
   `strategy.md` solves it without a prompt ("No prompt needed:"): then there are no questions and no
   alternative prompts; answer in 2-5 lines with the exact command and stop. Besides the gaps,
   include 1-2 questions about **what the user may not be taking into account**: who will read or
   use it, what happens if it goes wrong (rollback, data, prod), whether it must last or be kept up
   to date, dependencies on other people or repos, cost/time, what counts as "done". Each option
   with its consequence in the description, so the question teaches something.
   **Do not state what you do not know for sure**: if a premise of the strategy or the prompt depends
   on something you have not checked (which stack the repo uses, what exists, how a tool behaves),
   ask or check it cheaply; never take it for granted. In the output, whatever stays unconfirmed is
   marked as such.
5. **Model** with the rubric, per chunk. If the user set the model explicitly, respect it (warn in
   one line if the rubric suggests another).
6. **Target-model add-ons** (requested or recommended), in this skill's folder (next to this file):
   Fable → `fable-best-practices.md`; Opus 5.5 → `opus-best-practices.md`; Sonnet 5.5 →
   `sonnet-best-practices.md`; Haiku 5.5 → `haiku-best-practices.md`.
   **Only the file of each target model** — they do not share practices (verification, subagents and
   verbosity diverge). Graft the blocks whose condition applies to this task — never all of them.
7. **Write the prompt or prompts in English** (one per session/phase) with the anatomy + token
   economy below.
8. **Effort** and **ultracode** with their rubrics.
9. **Return ONLY** the output format. Do not run anything.

## Prompt anatomy (in English)

Include only the sections that add something; a small task can be 3-5 lines without headings.

- **Context / why** — 1-3 sentences: what the app is, the problem, why the change.
- **Task** — clear goal in the imperative with an action verb ("implement/fix/change", never "can you suggest").
- **Scope & files** — which files to touch (referenced with `@path`) and what is **out of scope**.
- **Patterns to follow** — existing examples in the repo to imitate ("follow the pattern in @…").
- **Approach** — if not trivial: "explore first, then plan, then implement". If small: direct.
- **Verification** — the **concrete check** Claude can run (test/build/screenshot) and what counts
  as "fixed"; "address the root cause, don't suppress the error"; evidence (the command's real
  output), not claims. Name the check, do not ask to "verify" in the abstract: generic orders
  ("include a final verification step", "double-check", "re-verify", "use a subagent to verify")
  are superfluous on Opus 5.5 and make it over-verify. With Sonnet 5.5 and Haiku 5.5 the explicit
  step does help.
- **Output** — what you want back and how short ("report the outcome and evidence briefly").
- **Delivery flow** — if the task ends in a pull/merge request, add the line "Follow the
  ticket-flow skill." (a generic `ticket-flow` skill sits next to this one: ticket → branch → PR/MR
  in review) and give in the prompt only what is specific to this task. If the change is code only,
  with no ticket and no PR/MR, say so explicitly: "No ticket, no PR/MR."

Writing rules: specific context, not vague; point to the source that answers a doubt (git
history, a concrete file); symptom + likely location + what "fixed" looks like; always a check it
can run.

## Prompt token economy

The shortest prompt that is still unambiguous wins — cut content, not clarity.

- **Bound the exploration**: name the relevant files/dirs; without that, the executing session
  scans the whole repo.
- **No over-prompting**: current models obey the first time. No "CRITICAL/YOU MUST", repeated
  orders or "be extremely thorough" — they inflate exploration and thinking. Say what to do,
  positively, once.
- **Ask for lean output** (Output section): it trims the narration of the answer.
- **Long data** (logs, docs) **at the top and the task at the end**; XML tags only if the prompt
  mixes instructions with data/examples — in a short prompt they are noise.
- **Examples**: only if the output format matters; one good one is enough.
- For analysis/diagnosis, the cheap anti-hallucination: "If unsure, say so. Read a file before
  making claims about it."
- **Heavy stuff, out of the main context**: visual QA with screenshots, design-reference searches
  and long logs go in a subagent that returns only the verdict or the selection; every screenshot
  or large result that enters the main session is re-read on every following turn.
- **Short sessions**: if the work spans more than one task or phase, the prompt leaves the state in
  a file (`progress.md` or the plan) and the next part goes in a new session, instead of a single
  session of hundreds of turns.

## Model rubric

Model = capability ceiling; effort = depth per step. Independent levers: do not move up a model
when raising effort is enough, nor the other way round.

| Model | When | Examples |
|--------|--------|----------|
| **Haiku 5.5** | Mechanical or high volume, low risk, speed/cost rule; also a cheap subagent in ultracode | mass renames, formatting, summaries, classifying, extracting data, DB/log queries |
| **Sonnet 5.5** | Default for well-bounded coding/agentic work, **including end-to-end implementation of something well specified** (ticket with clear criteria, plan already written, QA or review fixes) | normal feature/bugfix, tests, CRUD, scripts, ultracode implementers and fixers |
| **Opus 5.5** | Hard agentic coding and intelligence-sensitive work: multi-file without a prior plan, big refactors, high-risk code review/bug-finding, design, unfamiliar code, knowledge work with figures/sources, reading charts/diagrams | cross-module refactor, bug without a clear repro, code audit, architecture, financial model |
| **Fable 5.1** | The hardest: long-horizon, ambiguous, multi-thread, hours/days of autonomy. As **the main model in a separate session**: you describe the result and it plans and checks on its own | end-to-end migration, complex prod incident, deep audit, project from an idea |

- Torn between two and the cost of being wrong is high → the bigger one; repetitive/disposable task → the smaller.
- Fable and Opus 5.5 run classifiers (biology, high-risk dual-use offensive cybersecurity; Opus 5.5 also reasoning extraction). Looking for vulnerabilities in source code is allowed.
- Opus 5.5 does best with **the full specification up front and no interruptions**: if the task is
  big, put everything in the prompt instead of splitting it across turns of the same session
  (phases in new sessions with a written plan are something else: see `strategy.md`).
- In Claude Code the model is changed with `/model` (and `/fast` on Opus = same intelligence,
  faster, for interactive sessions).
- Whether the user has access to Fable or to ultracode depends on their plan: if unsure, recommend
  it with the Opus/one-session alternative next to it.

## Effort rubric

| Effort | When | Examples |
|--------|--------|----------|
| **low** | Mechanical, obvious scope, no design | typo, rename, add a log, formatting, copy change |
| **medium** | Well-bounded feature/bugfix, 1 known area | standard CRUD endpoint, fix with a clear repro, a single-case test |
| **high** | Multi-file, unfamiliar code, needs design judgment or something ambiguous | cross-module refactor, integrate OAuth, design a new API |
| **xhigh** | Subtle correctness, hard architecture, security, deep reasoning | concurrency/races, delicate data migration, financial logic |
| **max** | The hardest: debugging without a repro, novel algorithms, high-risk prod decisions | intermittent bug, critical optimization, prod incident on real data |

Recommend the minimum that fits (too much effort = burned tokens). Torn between two levels: high
cost of being wrong → the higher; low → the lower.

**If the target is Fable**: use this same rubric without inflating it — Fable already does well at
low/medium. Do not go up to xhigh just "because it's Fable"; only if the xhigh/max row fits on its own.

**If the target is Opus 5.5**: the default is `medium`, and it already matches or beats Opus 5 at
`high` in coding and knowledge work; in coding `low` gets close at much lower cost. At the same
level it thinks more per turn than Opus 5, especially at `xhigh`/`max`. Hence: recommend `medium`
as the starting point for what the rubric marks medium/high, `low` for the mechanical, and `high`
for what it marks xhigh/max; `xhigh`/`max` only if the task is truly frontier (max row). When in
doubt, go lower.

**If the target is Sonnet 5.5**: recalibrated effort (do not carry over Sonnet 5's). `medium` for
well-specified agentic coding, `high` for the long or hard; `xhigh`/`max` almost never (it launches
review rounds on its own).

**If the target is Haiku 5.5**: `medium` by default, `low` for high-volume mechanical work, `high`
if it needs to follow instructions to the letter. `xhigh`/`max` no: at that cost, Sonnet 5.5.

## Ultracode rubric

`ultracode` = deploying **multiple agents in parallel** (Workflow). It costs many tokens; recommend it
only when **parallelism/breadth** or **independent verification** add real value. It is
**orthogonal to effort** (effort = depth per step; ultracode = breadth/parallelism).

| Situation | Ultracode? |
|-----------|-------------|
| Change in 1-few files, a one-off question, a feature in a known area | **No** — a single Claude |
| Mechanical migration/sweep over many files | **Yes** — one agent per file/batch |
| Exhaustive audit/review with independent verification | **Yes** — finders + adversarial verifiers |
| Deep multi-source research | **Yes** — search fan-out + synthesis |
| Work too big for a single context | **Yes** — break it into phases |
| You need confidence via independent perspectives (design trade-offs) | **Yes** — panel of judges |
| Normal, iterative development with shared state | **No** — the overhead does not pay off |

If you recommend ultracode, add in one line **how to break it down** (e.g. "one agent per file",
"3 finders + verification", "fan-out research + synthesis").

**Ultracode budget (mandatory rule):** when ultracode is "yes", the prompt must ALWAYS include
a `Budget` block in English with:
- **Maximum total agents** (by default ≤ 6; go higher only if the breakdown truly demands it, and
  never more than 10), and maximum agents alive at once.
- **Model and effort per phase**: Haiku 5.5 for the mechanical and high-volume (renames, sweeps,
  formatting, extracting data, summarizing, DB/log queries); Sonnet 5.5 to explore, review and
  verify findings, and also to **implement and fix** when there is a clear ticket or plan; Opus only
  to design, integrate and decide. **At most 1–2 Opus agents per ultracode**, each with a line
  justifying why Sonnet is not enough (Opus row of the rubric). Set `model:` **and** `effort:` on
  every agent (otherwise the model falls back to `CLAUDE_CODE_SUBAGENT_MODEL` or the session's, and
  effort to the global one).
- **Do not relaunch agents that already finished well**: if something fails, retry only the failed
  agent and reuse the good results.
- **Stop and ask** if the plan needs more agents than the maximum, instead of growing it on its own.
The user does not need to ask for it: if this block is missing, the prompt is incomplete. If the
user's CLAUDE.md sets other limits, those win.

If the target is **Opus 5.5 and ultracode is "no"**, consider pasting the subagent brake block
(row 6 of `opus-best-practices.md`): Opus 5.5 delegates on its own more than it should on small tasks.

## Output format (this is the only thing you deliver)

1. One sentence: the goal you understood.
2. **How to approach it** (3-6 bullets). If section 0 of `strategy.md` applies, the first bullet is
   **"Before the prompt:"** with the exact command and why (or "No prompt needed:" if the command
   solves it alone). Then: shape (1 session / N sessions in parallel with their worktree and branch /
   phases / autonomous Fable / ultracode) and why; verification check (or "first chunk: create the
   check"); where the user has to step in; interactive or unattended; risks. If the task is not
   worth a prompt, say so here and stop.
3. The **English prompts**, one per session/phase, each in its own copyable code block with a short
   title ("Session 1 — api", "Phase 0 — plan"). Under each, a **"How to launch it"** line:
   `Model · Effort · Ultracode: yes/no · Worktree/branch · Plan mode: yes/no` + a brief reason.
   If the model is not the active one, remind `/model`. If ultracode is yes, prefix `ultracode` and
   the suggested breakdown. If there is an order between sessions, say it ("launch 2 once 1 has
   fixed the contract").
4. (Only if you made minor non-material decisions) one line "Minor decisions: …".

Headings and bullets in the user's language (Spanish: «Cómo abordarlo», «Cómo lanzarlo»,
«Antes del prompt:», «No hace falta prompt:»). You do not run the task. You do not offer to run it.
You only return the text.

## Common mistakes

- Intercepting a request that was to run something, not to build a prompt. → When in doubt, ask.
- Exploring the code in depth. → Only light context (CLAUDE.md/README/memory); the rest is done by the executing session.
- Filling a material gap with an "assumption" instead of asking. → If it changes the prompt, ask first. No limit on questions: better heavy than assuming.
- Skipping the question round because "it's already clear". → Always one round: gaps + 1-2 blind spots.
- Filler questions (obvious or already answered). → Each question must change the prompt or uncover something the user had not considered.
- Stating something unchecked (stack, a tool's behavior, the state of something). → Ask, check it cheaply or mark it "unconfirmed".
- Writing the prompt in Spanish. → The prompt ALWAYS in English (the explanation, in the user's language).
- Forgetting the check in the prompt. → Every prompt must give Claude something concrete to run.
- Adding generic verify/re-check orders with Opus 5.5 ("double-check", "verify at the end", "use a subagent to verify"). → It already does it; adding them burns tokens with no gain. Name the check, not the order.
- In a review prompt, writing "report only the serious stuff" or "be conservative". → Opus 5.5 obeys literally and under-reports: ask for full coverage with severity/confidence and filter in a separate step.
- With Opus 5.5, asking "think carefully/step by step" or "write your reasoning in the answer". → Depth comes from effort; the second can trigger a `reasoning_extraction` refusal. Ask for the justification of the decision, not the thinking.
- Bloated prompt: empty headings, emphatic CAPITALS, repeated orders, unnecessary XML. → The shortest that is still unambiguous.
- Recommending too much model/effort "just in case". → The rubrics rule; extra capability = cost with no return.
- Reading the add-ons of a model that is not the target, or mixing several models'. → One file, the target model's, and only the blocks that apply to that task.
- Spitting out the prompt without a strategy. → Always "How to approach it" first: shape, check, human intervention.
- Splitting for the sake of it (parallel sessions or phases for something that fits in one session). → Splitting costs coordination; only if the parts are independent or the task does not fit in one context.
- Implementation prompt with no check available. → The first chunk is creating the check (test, Playwright, query), or warning that it will not be verified.

Origen: adaptada de una versión interna de craft-prompt.
