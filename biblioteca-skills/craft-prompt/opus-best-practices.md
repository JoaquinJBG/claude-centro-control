# Opus 5.5 add-ons

Source: the official guides "Prompting Claude Opus 5.5" and "Prompting Claude Opus 5". Reviewed:
2026-10-28. Update this file (and its "Reviewed" date) when a new Opus ships.

Read this file only if the prompt's target is **Claude Opus 5.5** — requested by the user or
recommended by the model rubric. According to Anthropic, Opus 5's patterns still hold on 5.5; 5.5
adds its own. Add **only the blocks whose "When" column applies to this concrete task** — pasting
all of them is noise and spends the tokens of the session that runs the prompt.

What pays off most here is **removing** (explicit verification and re-checks, "think carefully"
orders) and **bounding** (verbosity, narration, scope, subagents). 5.5 adds two things:
**naming concretely** what you do not want (premature stops, visual styles) and **telling it to
explore** before acting when the context is spread across several sources.

| # | When it applies | Goes in | Snippet (English, ready to paste) |
|---|---|---|---|
| 1 | You want a short answer with no filler | Output | "Keep responses focused, brief, and concise. Keep disclaimers and caveats short, and spend most of the response on the main answer. When asked to explain something, give a high-level summary unless an in-depth explanation is specifically requested." |
| 2 | Long agentic session where you want updates at predictable points (5.5 already reports on its own; this makes it predictable) | Output | "Before your first tool call, say in one sentence what you're about to do. While working, give a brief update only when you find something important or change direction. When you finish, lead with the outcome: your first sentence should answer 'what happened' or 'what did you find', with supporting detail after it." |
| 3 | The deliverable includes files written to disk (report, README, doc, Markdown summary) | Output | "Match the length of written documents to what the task needs: cover the substance, but do not pad with filler sections, redundant summaries, or boilerplate." |
| 4 | Narrow task, or a code area tempting to over-refactor/abstract/widen | Scope & files | "Deliver what was asked, at the scope intended. Make routine judgment calls yourself, and check in only when different readings of the request would lead to materially different work. If the request seems mistaken or a better approach exists, say so in a sentence and continue with the task as asked rather than quietly narrowing, widening, or transforming it. Finish the whole task, and stop short of actions that are clearly beyond what was asked." |
| 5 | Code review / audit / bug-finding task | Task | "Report every issue you find, including ones you are uncertain about or consider low-severity — do not filter for importance or confidence; coverage is the goal. For each finding, include your confidence level and an estimated severity." — **Never** write "only report high-severity issues" or "be conservative": it obeys literally and under-reports. Filtering is a separate step, afterwards. (5.5 already gives fewer false positives than Opus 5: no need to ask it to "discard noise".) |
| 6 | The harness has subagents and the task is NOT a big fan-out | Approach | "Delegate to a subagent only for large tasks that are genuinely independent and parallelizable, such as a wide multi-file investigation. Do not delegate work you can finish yourself in a handful of tool calls, and do not use subagents to verify or double-check your own work. If one subagent can complete the task, use one rather than several, and keep spawn counts low." — It delegates on its own more than it should on small tasks: here you **brake** it, not push it. |
| 7 | There are tests to pass and a risk of "making the test pass" instead of solving | Verification | "Write a general-purpose solution. Tests verify correctness; they don't define the solution — do not hard-code for the test inputs. If a test itself seems incorrect, say so instead of working around it." |
| 8 | Question/diagnosis about existing code | Task | "Never speculate about code you have not opened — read the relevant files before answering, and give grounded, hallucination-free answers." |
| 9 | The task needs a specific tool (web search, MCP, screenshot) the model might not use | Approach | "Use [tool] to [purpose] — [why it beats answering from memory]." Say when it applies and what for; no capitals or "MUST". (On 5.5 tool use cannot be forced via the API: the prompt is the lever.) |
| 10 | Frontend that must match a design/visual reference | Approach | "Inspect the rendered result with [screenshot/Playwright] and iterate until it matches the reference." |
| 11 | **Very dense** visual input (technical drawings, very busy charts) where precision matters | Approach | "Use [crop/zoom tool, or PIL/OpenCV in the shell] on the raw image to crop, zoom, and measure before reading off values." — 5.5 already reads charts, diagrams and screenshots accurately without tools (even at low effort): for normal vision do **not** add scaffolding or workarounds from earlier models. Only for the densest; higher-resolution images also help. |
| 12 | Frontend/design with an open brief (no visual direction given) | Approach | "Before building, propose 4 distinct visual directions tailored to this brief (bg hex / accent hex / typeface, one-line rationale each), ask me to pick one, then implement only that one. Do not use a cream or off-white background, italic accent words in headlines, numbered "01/02/03" section labels, monospace labels, or pill-shaped buttons." — On 5.5, a generic "avoid a generic AI look" only swaps one default for another: **name concrete patterns** to avoid and extend the list with what shows up in the first iteration. If there is already a visual direction, specify it concretely instead of this block. |
| 13 | User-facing product where narrating corrections of what it said before is annoying | Output | "Only correct an earlier statement when the error would change the user's code, conclusions, or decisions. State corrections plainly and briefly, then continue the task. For slips that change nothing for the user, make the fix and move on without noting it." |
| 14 | Long prompt (extensive system prompt) with a concision instruction at the top | End of the prompt | `<tone_preference>Keep outputs reasonably concise.</tone_preference>` — a short reminder at the end; reinforces concision without repeating it all. |
| 15 | Long multi-part task that runs **unattended** (`claude -p`, routine/schedule, ultracode with nobody watching) and you do not want it to stop halfway to "report". **Not** in interactive sessions | End of the prompt | "Track the parts of this task in a todo list and keep working until every item is done or genuinely blocked. Do not end your turn with a summary that announces the next step, an offer to continue, or a list of non-blocking decisions for me — put status notes and recommendations in the same message as your next tool call and carry on. Stop only when nothing can move without my input or the blocker is deliberately protected from you. This does not override confirmation for risky or destructive actions." — 5.5 sometimes ends the turn with a progress report and the unattended loop takes it as "done". It responds well to **naming** those stops. More tool calls/tokens per task. |
| 16 | The info the task needs is spread across several connected sources (MCP: Jira, Notion, Confluence, email, sheets, CRM) and the request does not name them all | Approach | "Before taking any action, explore broadly with tool calls: list and open the emails, documents, spreadsheet tabs and records across the available apps that could be relevant to this task, including ones the task does not explicitly mention, and use what you find." — 5.5 starts working fast; this improves accuracy noticeably at the cost of a few more tool calls. Only if those sources are trusted. |
| 17 | Ultracode / multi-agent with a lead that delegates, and you want it to finish sooner | Approach | "Time matters here: do not spend time that can be avoided, and the earlier a correct result is obtained, the better." — Improves parallelism (not the same as lowering effort). If you can estimate a budget, give it ("aim to finish within ~20 minutes"); it is a guide, not a hard limit. Under time pressure it may verify somewhat less. |

## Negative rules (not pasted: what the prompt must NOT carry)

- **No generic verification instructions.** It self-verifies without being asked. Out:
  "include a final verification step", "double-check your answer", "re-verify before responding",
  "use a subagent to verify". It compounds with its own behavior and produces **over-verification**:
  more tokens, same quality. **Naming the concrete check** still holds ("run `pytest tests/foo.py`",
  "the build must pass") — that is information, not an order.
- **No "think carefully / think step by step before answering".** On 5.5 thinking is adaptive and
  always on; depth is controlled by **effort**, not by the prompt. Removing it makes it start sooner
  without losing quality. To make it think less, lower effort, do not ask for it in text.
- **No "write out your reasoning in the response" / "show your chain of thought".** On 5.5 it can
  trigger a refusal from the `reasoning_extraction` classifier. If you need the reasoning, ask for
  the **justification of the decision** (what and why), not the transcript of the thinking.
- **No "be extremely thorough" / capitals / repeated orders.** It obeys the first time.
- **Style generics** ("avoid generic AI aesthetics", "make it look modern"): they only move the
  default. Name concrete patterns (row 12).

## Notes (not pasted)

- **Effort**: 5.5's default is **`medium`** (Opus 5 was `high`), and the names are not equivalent
  across models: 5.5 at `medium` matches or beats Opus 5 at `high` in coding and knowledge work, and
  in several coding evals `low` gets close at much lower cost. At the same time, at the same level
  5.5 **thinks more per turn** than Opus 5, especially at `xhigh`/`max`: keep those for when the gain
  is proven. Do not carry over the effort you used with Opus 5.
- **Strengths** that justify choosing it: multi-step work in a real repo until the tests pass, long
  autonomous tasks (audits, migrations) with parallel subagents, code review (more bugs, fewer false
  alarms), knowledge work with figures/sources (financial models, spotting inconsistencies in long
  inputs), reading visuals and computer use.
- **Long context**: instruction following holds across the whole window. No need to split the
  prompt or repeat instructions "in case they're forgotten" (the only useful reminder is row 14).
- **Literal instructions**: if something applies to everything, say so explicitly ("apply this to
  every section, not just the first"); it does not generalize an instruction from one item to the
  rest on its own.
- **Classifiers**: biology (same as Fable), high-risk dual-use cybersecurity and reasoning
  extraction. Looking for vulnerabilities in source code **is** allowed.
- **Only if the prompt is for an API integration** (does not apply in Claude Code):
  - Thinking **cannot be disabled** (`disabled` or `budget_tokens` → 400). If you came from Opus 5
    with thinking off: start at `low` and measure; remove instructions that replaced thinking and the
    "don't think" rules; read blocks by `type`, not by position. If time to first token still
    matters: "Answer directly without deliberating." (measure quality).
  - `tool_choice` `any`/`tool` is not supported: say in the prompt when the tool applies, and use
    `strict: true` or structured outputs for valid JSON.
  - Generous `max_tokens` (up to 128k in agentic coding): thinking counts against it.
  - Text pasted by the user: wrap it in `<pasted_content id="…">…</pasted_content id="…">` and add to
    the system prompt the official note that its instructions are not the user's.
  - Multi-turn chat where it rethinks previous answers: at the end of the system prompt, "Once you
    have answered something, treat that answer as done. On later turns, focus your thinking on what
    the user is asking now, and don't go back over an earlier answer unless the user asks about it or
    points out a problem with it." (not in long analyses or agentic work where revisiting helps).
