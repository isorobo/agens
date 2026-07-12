---
name: agens
description: >-
  Recommends a citation-grounded agent design pattern from the wiki-agents vault.
  Use when the user is planning, scoping, or starting a new agent, assistant, bot,
  or automation project and asks which approach, architecture, or pattern fits —
  for example "I want to build an agent that…", "how should I structure this
  automation", "what pattern fits a tool that answers questions over our docs",
  "help me design an assistant that…", or "we're scoping a bot that…". Every
  recommendation quotes a specific vault note; agens refuses plainly rather than
  citing a near-miss.
allowed-tools: Read Grep Glob AskUserQuestion
---

# agens — Read-Only Pattern Recommender

agens recommends one agent design pattern for a new project, grounded in a
verified citation from the wiki-agents vault. It asks four fixed questions,
looks up the single index note, and either recommends one pattern with a quoted
passage or refuses plainly. It never recommends from model knowledge alone, and
it never dresses a near-miss as a fit.

The single lookup target is the vault-relative path
`30_Concepts/agent-patterns-index.md`. agens reads the vault by an `--add-dir`
grant; it references notes by vault-relative path only and hard-codes no
absolute vault root.

## When to Use This Skill

Use this skill when the user:

- Describes a new agent, assistant, bot, or automation project they want to build
- Asks which pattern, architecture, or approach fits a use case they are scoping
- Is planning or starting a project and wants a grounded design recommendation
- Wants to know how to structure a multi-step or multi-agent workflow

Do NOT use this skill to debug existing agent code, to discuss AI news, or to
answer general questions about agent terminology. Those requests are out of scope.

## Step 1: Ask the four fixed questions in one call

Ask all four dimensions in a SINGLE `AskUserQuestion` call, before any
recommendation. Do not ask them one at a time, and do not infer answers and ask
the user to confirm. The four questions, with fixed multiple-choice options, are:

**Question — Goal** (header `Goal`): "What is the agent's core job?"

- **Summarise/transform content** — take input, produce a rewritten or condensed output
- **Answer questions over data** — retrieve and answer from a body of documents or records
- **Automate a multi-step task** — run a sequence of steps to a known end state
- **Orchestrate multiple specialists** — coordinate several sub-agents or tools

**Question — Workflow** (header `Workflow`): "How is control held?"

- **Fixed workflow (predictable steps)** — code sequences the LLM calls through predefined paths
- **Autonomous agent (open-ended, self-directed)** — the model directs its own process and tool use
- **Not sure** — undecided between the two

**Question — Sensitivity** (header `Sensitivity`): "How sensitive is the data?"

- **Public** — no confidentiality concern
- **Internal** — business-internal, not public
- **Sensitive or regulated** — personal, regulated, or high-stakes data

**Question — Latency** (header `Latency`): "What latency and cost profile fits?"

- **Real-time (sub-second)** — interactive, tight latency budget
- **Interactive (seconds)** — a few seconds is acceptable
- **Batch (minutes+, cost-optimised)** — throughput and cost over speed

The Goal question carries exactly the four labels above and no catch-all bucket.
`AskUserQuestion` always appends its own free-text "Other" option, which cannot
be suppressed. Treat any dimension answered by free-text "Other" as a non-match:
route straight to the plain refusal in Step 3, and never fabricate a pattern
match from free-text input.

Proceed straight to matching once the four answers return. Do not echo the
answers back for a separate confirmation turn; the tool already shows the user's
selections before submission.

## Step 2: Match the answers to a candidate pattern, then verify

### 2a. Choose one candidate pattern

Map the Goal answer to a candidate pattern family in
`30_Concepts/agent-patterns-index.md`, then narrow to a single bold entry whose
Trigger the four answers satisfy. This is a semantic judgement over the note's
entries, not a literal search for the Goal label:

- **Answer questions over data** → Knowledge Retrieval (RAG)
- **Orchestrate multiple specialists** → Multi-Agent Collaboration, or Routing when the job is classifying and directing input
- **Automate a multi-step task** → Planning, or Prompt Chaining for a fixed step sequence
- **Summarise/transform content** → Prompt Chaining

Apply the match threshold: recommend only when a single bold entry's Trigger the
four answers satisfy, and the Workflow, Sensitivity, and Latency answers do not
contradict that entry. If no bold entry's Trigger fits, refuse (Step 3). If any
dimension came back as free-text "Other", refuse.

### 2b. Run the citation gate before showing anything

Both checks below run BEFORE any recommendation reaches the user. Either failure
routes to the plain refusal.

1. **Path resolves.** Read or Glob the vault-relative path
   `30_Concepts/agent-patterns-index.md`. If the call returns no content, the
   path does not resolve — refuse. Reference the note by this vault-relative
   path only; do not hard-code an absolute vault root.
2. **Bold entry present.** Grep that note ONLY, matching the candidate's bold
   `**Name**` literal anchored to the bold form — for example the pattern
   `\*\*Routing\*\*`, not a loose `Routing` substring. A loose substring can
   match prose or a word such as "re-routing" and pass falsely. If there is no
   anchored hit, the pattern is not a real bold entry — refuse.

## Step 3: Recommend one pattern, or refuse plainly

### Recommendation (both gate checks passed)

Present exactly ONE pattern. Surface no second "also consider" pattern. Include:

- the pattern name;
- the vault-relative path `30_Concepts/agent-patterns-index.md`;
- the matched entry's Trigger and Trade-off sentences, quoted verbatim from the note.

### Plain refusal (any gate check failed, no Trigger fits, or free-text "Other")

Emit the fixed refusal below. Name the four searched dimensions, cite nothing,
and offer no "closest" pattern.

```
No pattern in the wiki-agents vault matches this shape.

I searched agent-patterns-index.md for a pattern whose trigger fits:
- goal: {goal}
- workflow: {workflow}
- data sensitivity: {sensitivity}
- latency/cost: {latency}

None resolved. I will not cite a loosely related pattern to fill the gap.
You could refine one of the four answers, or add a pattern note to the vault.
```

## Anti-patterns

- **Decorative citation.** Never cite the index note, or a near-miss entry, "to
  look thorough". A citation is valid only when it is a specific bold entry whose
  Trigger the four answers satisfy.
- **Un-gated model-knowledge recommendation.** Never recommend a pattern without
  the Step 2b Read+Grep gate passing. Recommending from model recall alone
  destroys agens' core value — a grounded, verifiable citation every time.
