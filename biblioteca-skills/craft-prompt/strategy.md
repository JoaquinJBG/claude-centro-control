# Execution strategy

Read **always**, before writing the prompt. It decides **how** the task is approached; the prompt
comes after and depends on this. What improves results most is not the model nor the number of
subagents, but: splitting well, Claude having a real check, a short approved plan, and then letting
it work without interruptions.

## 0. A command before (or instead of) a prompt?

Look at the history (current conversation; progress, plan or memory if it continues work from
another session). Recommend the command only if its signal really shows up; if none does, skip
this section.

| Signal | Recommend |
|---|---|
| The current session went down a bad path: discarded approach, several failed attempts, correction after correction | `/rewind` (or `Esc Esc`) to the point before the detour and relaunch from there with the corrected prompt: it takes the bad path out of the context and keeps the cache. It restores Claude's file edits, not shell or subagent changes. |
| The work lives in another session | `claude --resume` / `/resume` instead of re-explaining the context in the prompt |
| Try an alternative without losing the current line | `/fork` |
| Quick side question in the middle of a task | `/btw` — no prompt needed |
| Long or unattended task with a checkable "done" | `/goal <condition>` together with the prompt: files that must exist, checks that must pass, and "stop after N turns and report the blocker". More reliable than "keep working until done" inside the prompt. `/goal clear` removes it. |
| The session is already long (`/context` > ~300k) or switches task | State to `progress.md` + `/clear` or a new session; `/context` if it is unclear what weighs |
| Talks about spend or tokens | `/usage` (breakdown by skill, subagent and MCP), plus any token report the user's CLAUDE.md mentions |
| Asks to optimize or improve skills, CLAUDE.md or saved prompts | Before the prompt: `/doctor prompt-audit` (patterns written for older models) and `/skill-doctor` (context cost and usage per skill); ask the user to paste the output. The improvement prompt measures with `claude plugin eval` before and after the change (see `skill-evals.md`). |

## 1. Is a prompt worth it?

- **Trivial** (typo, rename, one line): say so and recommend doing it directly in the current
  session. If they still want it, the prompt is 1-3 lines.
- **Ill-defined** (nobody knows what has to change or what "done" looks like): the first chunk is
  an **exploration/plan** session (plan mode, no edits) whose deliverable is the plan in a file.
  Do not write the implementation prompt until that plan exists.

## 2. Shape

| Situation | Shape | Notes |
|---|---|---|
| Small or medium, 1 known area | **1 session** | No subagents. If the target is Opus 5.5, row 6 of `opus-best-practices.md` (delegation brake). |
| Several **independent** parts (different tickets, paired api/web repos with a fixed contract, modules with no shared state) | **N sessions in parallel**, one per worktree | One prompt per session. Worktree where the user's CLAUDE.md says; by default `<repo>/.worktrees/<branch>` with `git worktree add`. If api and web share an unfixed contract: first fix the contract (short session), then parallel. At most 2-3 live sessions: more cannot be supervised. |
| Big with **dependent** parts | **Phases** | Phase 0 in plan mode → plan in a file (`docs/plan-<topic>.md` or similar). Each later phase is a new session that gets `@plan` + "implement phase N", with its check and a dated progress entry in the plan when it ends. Clean context per phase. |
| Long, autonomous, clear goal and a check available | **Fable as the main model**, separate session | Describe the result, not the steps; short approved plan and then no interruptions. Better than Opus split across many turns. |
| Mechanical sweep over many files, exhaustive audit, multi-source research | **Ultracode** with a Budget block | See the ultracode rubric in SKILL.md. |
| Reviewing a PR/MR | Built-in **`/code-review`** (`--comment` posts inline on the PR/MR in the project's forge) | `/code-review ultra` only for big or critical MRs (paid). Follow the review rules in the user's CLAUDE.md (e.g. threads vs. a summary report) if it has any. |
| New UI / design (new app, view or component) | **1 session**, with visual directions first | If the user's CLAUDE.md asks for design references (e.g. Mobbin) and that MCP is connected, references before proposing (or ask if it is small). Check = a Playwright screenshot. Adding a field or column to an existing UI is **not** new design: follow the existing patterns, no references. |

When torn between 1 session and splitting: if it fits in one session with a clear check,
**1 session**. Splitting has a cost (coordination, merges, repeated context).

## 3. The three axes you always assess

- **Verification**: what real check can Claude run? Tests, type-check, build, the changed command,
  Playwright (UI), staging/prod logs or a DB query (data bugs). Name it in the prompt. **If there
  is none, the first chunk is creating it** (a test that reproduces the bug, a Playwright script, a
  query that shows the symptom). With no check, warn that the result will not be verified.
- **Human intervention**: mark where it has to stop and ask: production, Kubernetes writes with
  `kubectl` (staging included), production releases, pushing or opening a PR/MR when the user's
  CLAUDE.md reserves that for them, posting on other people's PRs/MRs, data migrations, deletions. Staging releases that the user's CLAUDE.md lets run on their own (e.g.
  through a release skill) are not a gate. Everywhere else, let it work without asking for permission.
- **Interaction**: interactive (the user will keep watching) or unattended (`claude -p`, in the
  background, a parallel session while doing something else). Unattended only if there is a check
  and no risky step without a gate. If unattended and the target is Opus 5.5, row 15 of
  `opus-best-practices.md`.

## 4. Model and effort per chunk

Each chunk gets its own model and effort per the rubrics in SKILL.md: the plan phase can go on Opus
`high` and the mechanical implementation ones on Sonnet `medium`. Do not recommend the same model
for everything out of inertia.
