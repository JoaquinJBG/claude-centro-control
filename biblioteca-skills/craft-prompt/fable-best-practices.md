# Fable 5.1 add-ons

Source: the official guide "Prompting Claude Fable 5". Reviewed: 2026-10-28. Update this file
(and its "Reviewed" date) when a new Fable ships.

Read this file only if the prompt's target is **Claude Fable 5.1** — because the user asked for it
("para Fable", "for Fable") or because the model rubric recommended it for this task. Add **only the
blocks whose "When" column applies to this concrete task** — pasting all six every time is noise and
spends the tokens of whoever runs the prompt.

| # | When it applies | Goes in | Snippet (English, ready to paste) |
|---|---|---|---|
| 1 | Task with some ambiguity, or the model could over-explore/plan before acting | Next to Task | "When you have enough information to act, act. Do not re-derive facts already established, re-litigate a decision already made, or narrate options you will not pursue. If weighing a choice, give a recommendation, not an exhaustive survey." |
| 2 | Effort high/xhigh, or there is a code area tempting to over-refactor/abstract | Scope & files | "Don't add features, refactor, or introduce abstractions beyond what the task requires. Don't design for hypothetical future requirements — do the simplest thing that works well. Don't add error handling or validation for scenarios that can't happen; trust internal code and framework guarantees." |
| 3 | Long/multi-step task where the model could ask permission too often or stop halfway | Approach | "Pause for the user only when the work genuinely requires them: a destructive/irreversible action, a real scope change, or input only they can provide. Otherwise proceed and report at the end, not mid-task." |
| 4 | Domain where a silent failure is plausible (money, data, migrations) or there are steps that need human sign-off | Verification | "Before reporting progress, audit each claim against a tool result from this session. Only report work you can point to evidence for. If a step needs human sign-off, say so explicitly instead of marking it done." |
| 5 | Predictably long session (build + external fetch + extensive tests) | Approach (optional) | "You have ample context remaining. Do not stop, summarize, or suggest a new session on account of context limits." |
| 6 | Closing a long/async task the user did not watch in progress | Verification/closing (optional) | "Your final summary is for a reader who didn't see the work happen: open with the outcome in one sentence, then supporting detail in plain language — no shorthand or labels coined mid-task." |
| — | Whenever the target is Fable | Note, not pasted in the prompt | Effect on the effort rubric: Fable does well even at low/medium — keep using this skill's general rubric without inflating it, but when the result is "high", it can stay there instead of going up to xhigh unless the task fits the xhigh/max row. |
| — | Whenever the target is Fable | Negative rule, add nothing | Do not ask Fable to transcribe or explain its internal reasoning as answer text — it can trigger a `reasoning_extraction` refusal. If the reasoning needs to be seen, it is an API matter (`thinking` blocks), not a prompt one. |
| — | Offensive cybersecurity or biology/life-sciences task | Note | Fable runs safety classifiers in these two domains; it can refuse legitimate tasks. Does not apply to normal product/backend work — leave this note out unless the prompt touches those domains. |

Subagent delegation (Fable dispatching sub-Fables) and a persistent memory system are practices in
the guide that only help in harnesses that support them natively — Claude Code already has its own
mechanism (Task/Agent, session memory). Do not repeat them in the prompt unless the task is
explicitly multi-agent.
