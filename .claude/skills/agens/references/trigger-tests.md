# agens — Trigger and Walkthrough Acceptance Harness

This file is the behavioural oracle for the agens skill. It is authored before
`SKILL.md` and independent of the skill's `description` wording. A human uses it
at the Plan 03 checkpoint to confirm that agens fires on the right requests,
stays silent on the wrong ones, and produces a citation-grounded recommendation
or a plain refusal.

The should-trigger phrasings below are written from the user's mouth, not from
the skill's own vocabulary. None reuses a noun phrase from the `SKILL.md`
description. That independence is deliberate: a test set that echoes the
description proves only that the description matches itself.

---

## should-trigger

A fresh session with each phrasing below should auto-load agens (no `/agens`
typed). Each is a natural project-starting request to scope a new agent,
assistant, bot, or automation.

1. "I'm kicking off a new project and I want something that reads support tickets and drafts replies. Where do I start?"
2. "We keep copying numbers between two systems by hand. I'd like to hand that off to a bot. What shape should it take?"
3. "Thinking about a little helper that watches our inbox and files things into the right folder. How would you set it up?"
4. "My team wants a tool that answers staff questions from our internal handbook. What's the right approach?"
5. "I need to spin up an assistant that turns messy meeting notes into clean summaries. Which design makes sense?"
6. "We're scoping a system that plans a trip end to end — flights, hotels, the lot — on its own. What fits?"
7. "Could you help me figure out how to structure a thing that coordinates a few specialised workers to ship a report?"
8. "Starting fresh on an automation that pulls data from three APIs and decides what to do next. What pattern suits that?"
9. "I want to prototype an assistant that grades essays against a rubric. How should I think about the architecture?"
10. "We're planning a bot that routes each customer message to the right department. Advice on the setup?"

## should-not-trigger

Each phrasing below mentions agents, bots, or automation but is NOT a request to
scope a new project. agens must stay silent. These guard against over-firing.

1. "My existing agent is throwing a null-pointer error on line 42 — can you help me debug it?"
2. "Did you see the news about the new autonomous-agent research paper this week?"
3. "What does the word 'agentic' actually mean in plain terms?"
4. "Please refactor this bot's message handler to use async/await."
5. "Write me a unit test for the automation script I already have."
6. "Rename the variable `agent` to `worker` across this file."
7. "Summarise this article about AI automation trends for me."
8. "Our CI automation failed on the last push — what does this log line mean?"

---

## Manual walkthrough checklist

A human runs these steps at the Plan 03 checkpoint and ticks each one.

- [ ] Typing `/agens` loads the skill (explicit slash-command path works).
- [ ] Describing a new agent project (any should-trigger phrasing) auto-loads agens without a command.
- [ ] agens asks all four questions — Goal, Workflow, Sensitivity, Latency — in a single AskUserQuestion call, before any recommendation.
- [ ] Each question offers fixed multiple-choice options, matching the D-03 through D-06 labels.
- [ ] A clearly-matching project (for example "answer staff questions from our internal handbook") yields ONE pattern recommendation.
- [ ] The recommendation cites the resolvable vault path `30_Concepts/agent-patterns-index.md` and quotes the matched entry's Trigger and Trade-off sentences verbatim.
- [ ] The recommendation appears only after agens has Read/Glob-resolved the path and Grep-confirmed the bold `**Name**` entry (the citation gate ran).
- [ ] A clearly-unsupported project, or a free-text "Other" answer on any dimension, yields the plain refusal.
- [ ] The refusal names the four searched dimensions, cites nothing, and offers no "closest" pattern.
- [ ] No should-not-trigger phrasing fires agens.
