---
phase: 02-delegation-wiring
reviewed: 2026-07-12T00:00:00Z
depth: standard
files_reviewed: 1
files_reviewed_list:
  - .claude/skills/agens/SKILL.md
findings:
  critical: 2
  warning: 4
  info: 2
  total: 8
status: issues_found
---

# Phase 2: Code Review Report

**Reviewed:** 2026-07-12
**Depth:** standard
**Files Reviewed:** 1
**Status:** issues_found

## Summary

Phase 2 adds a `Skill` grant, a Step 0 framework-fit detector, and a "Delegate a
framework-fit question" section (combined pre-flight gate, invocation, present-as-is
instruction, fixed failure message) to `.claude/skills/agens/SKILL.md`. The fail-closed
shape of the two gate checks (Check A presence, Check B phase parse) is sound and
mirrors Phase 1's citation gate correctly — both checks refuse on any non-confirmation
rather than proceeding on uncertainty, and neither leaks a stack trace or an absolute
filesystem path in the fixed failure message.

Two BLOCKER-level defects were found. First, the invocation instructions require
passing "the four questionnaire answers" as background context, but Step 0 explicitly
instructs skipping Step 1 — the only step that collects those answers — when a
framework-fit match fires. As written, the delegation branch has no source for the
values it is told to interpolate. Second, Step 0's trigger vocabulary matches bare
framework/SDK keyword mentions (not just "which framework fits"-style questions),
which risks misrouting away from agens' own core job on plausible openers that this
project's own description text would otherwise handle (e.g. "I want to build an agent
using the Claude Agent SDK that summarises documents").

Four WARNING-level issues also surface: the `Skill` tool grant is technically
unscoped (Claude Code has no allowed-tools syntax to restrict *which* skill name may
be invoked), so the "only ever target `gsd-ai-integration-phase`" rule is enforced by
prompt text alone; Check B validates only that a phase integer exists, not that the
phase is actually active/in-progress; the fixed failure message's `{if A}`/`{if B}`
conditional-line instructions are ambiguous about whether the literal marker tokens
should be stripped before the text reaches the user; and the skill's auto-trigger
`description` was not extended with framework/SDK vocabulary, so Step 0's new logic
may never run for users who rely on auto-invocation rather than the `/agens` slash
command.

## Critical Issues

### CR-01: Delegation invocation requires four answers that Step 0 instructs skipping

**File:** `.claude/skills/agens/SKILL.md:62-65, 216-229`
**Issue:** Step 0 states: "On a match: skip Steps 1-3 entirely and branch to the `## Delegate a framework-fit question` section. Do not run the pattern questionnaire" (lines 62-65). Step 1 is the *only* place in the file that runs `AskUserQuestion` to collect the four answers (goal, workflow, sensitivity, latency) — see line 76-80. Yet the Delegate section's invocation instructions require passing those same four answers as labelled background prose:

```
Background from agens (do not re-ask the user these):
- goal: <goal answer>
- workflow: <workflow answer>
- data sensitivity: <sensitivity answer>
- latency/cost: <latency answer>
```

(lines 224-229). When Step 0 fires, no `<goal answer>` etc. exist to substitute — Step 1, where they are collected, was explicitly skipped. The instructions as written force one of three bad outcomes: the model fabricates plausible-sounding answers to fill the template (directly contradicting this skill's own "never fabricate" ethos used elsewhere for citations), it emits the literal placeholder tokens verbatim to `gsd-ai-integration-phase`, or it silently re-runs Step 1's `AskUserQuestion` anyway — which Step 0 explicitly forbids ("Do not run the pattern questionnaire"). None of these is a specified, correct behaviour.
**Fix:** Either (a) have the Delegate section itself collect the four answers via its own `AskUserQuestion` call before invocation (and stop calling this "not running the pattern questionnaire" if Step 1's exact call is reused), or (b) drop the four-answer background prose from the invocation entirely when routed via Step 0, and only include it when the same session already has answers from a prior Step 1 run. Document which of these applies explicitly; do not leave the contradiction for the executing model to resolve ad hoc.

### CR-02: Step 0 keyword match can misroute agens' own core pattern-recommendation job

**File:** `.claude/skills/agens/SKILL.md:57-60`
**Issue:** "Match a framework-fit question by keyword or semantic signal on framework and SDK terminology. Name at least: LangChain, CrewAI, LangGraph, "Agent SDK", "which framework", "which SDK", and "which library should I use"." The list mixes two different signal types without distinguishing them: genuine "which X fits" question phrasings (which correctly indicate a framework-fit question) and bare product-name keywords ("Agent SDK", "LangChain", "CrewAI", "LangGraph") that also appear in ordinary pattern-scoping openers. This project's own frontmatter description gives as an example trigger phrase "I want to build an agent that…" — extend that naturally to "I want to build an agent using the Claude Agent SDK that summarises documents" (a highly plausible opener, since this very project's `Ecosystem` constraint is "Anthropic Claude Agent SDK / Claude Code Skills only") and Step 0, as literally written, matches on the bare "Agent SDK" keyword and routes the request to the delegation branch — skipping the four-question pattern intake that is agens' entire reason to exist — even though the user asked a pattern-fit question, not a framework-fit one.
**Fix:** Require the "which X fits/should I use" question shape (or an unambiguous semantic equivalent) for every match, not bare keyword presence. Rewrite the list to give examples of qualifying *phrasings* ("which framework should I use", "LangChain or CrewAI?", "is LangGraph a good fit") rather than bare product names that can appear inside a pattern-scoping sentence that never asks for framework advice.

## Warnings

### WR-01: `Skill` tool grant is unscoped; the single-target restriction is prompt-only

**File:** `.claude/skills/agens/SKILL.md:12, 232-234`
**Issue:** `allowed-tools` now includes bare `Skill` (line 12). Claude Code's `allowed-tools` frontmatter has no syntax to scope the `Skill` tool to one named target (unlike, e.g., `Bash(git:*)`), and this file's own instructions confirm the general rule ("Claude invokes any skill lacking `disable-model-invocation: true`" — see `CLAUDE.md`). The restriction to `gsd-ai-integration-phase` — "Never target `gsd-framework-selector`... Never reach for the `Agent` tool... The only delegation route is `gsd-ai-integration-phase`" (lines 232-234) — is therefore enforced purely by natural-language instruction, not by a technical control. CLAUDE.md's own Constraints section names exactly this class of risk ("a direct mitigation against the self-mutation/rug-pull risk pattern documented in wiki-agents' MCP Security note") and requires a human-approval gate for any write/build-capable delegation. That gate exists only for the not-yet-built `agens-build` skill; nothing here technically prevents a future prompt-injected vault note, a malformed detection match, or a model reasoning slip from invoking a different, write-capable skill (e.g. `build-mcp-server`) via the same `Skill` grant.
**Fix:** No code fix removes this — it is a genuine platform limitation shared with the `gsd-ns-workflow` analog. At minimum, document the residual risk explicitly in this file's threat model / anti-patterns (it is currently only in `02-CONTEXT.md`'s STRIDE table, not in the shipped SKILL.md), and keep this constraint visible for the `agens-build` phase so its human-approval gate is not skipped on the assumption that the `Skill` grant is otherwise safe.

### WR-02: Check B verifies a phase number exists, not that the phase is active

**File:** `.claude/skills/agens/SKILL.md:205-209`
**Issue:** Check B is titled "active GSD phase" and its stated intent (per `02-CONTEXT.md` D-02) is to gate on "a current/in-progress phase." The actual check only parses "the leading integer" of the `Phase:` line under `## Current Position` and fails on an absent/unparseable value — it never inspects a status field (e.g. the "— EXECUTING" / "— COMPLETE" suffix visible in this project's own live `STATE.md`). A `STATE.md` whose current-position phase has already completed (but still carries a trailing integer) passes Check B and delegates as if a phase were actively in progress.
**Fix:** Either parse and require an in-progress status token alongside the integer, or rename the check/documentation to accurately describe it as "a resolvable phase number exists," not "an active phase," so downstream expectations match actual behaviour.

### WR-03: Ambiguous instruction for stripping `{if A}`/`{if B}` template markers

**File:** `.claude/skills/agens/SKILL.md:244-258`
**Issue:** "Emit this text exactly, keeping only the `{if A}` and `{if B}` lines for whichever condition failed" is internally inconsistent: "exactly" suggests verbatim reproduction of the block shown (which contains the literal tokens `{if A}` and `{if B}` as line prefixes), while "keeping only ... lines for whichever condition failed" implies a transformation (dropping non-applicable lines). Neither this sentence nor the template itself tells the executing model to strip the `{if A}`/`{if B}` prefix tokens from the lines it does keep. A literal reading of "emit this text exactly" risks the raw marker text (e.g. "- {if A} The gsd-ai-integration-phase skill is not installed...") reaching the user verbatim, which looks like a broken template rather than a clean failure message.
**Fix:** State explicitly: "drop the `{if A}` / `{if B}` prefix tokens themselves; keep only the sentence for whichever condition failed." Optionally show a worked example for the single-failure and both-failure cases.

### WR-04: Auto-trigger `description` was not extended for framework/SDK phrasing

**File:** `.claude/skills/agens/SKILL.md:3-11, 51-65`
**Issue:** Claude Code's auto-invocation of a Skill (as opposed to explicit `/agens` invocation) is driven by the frontmatter `description` field, which Phase 2 left unchanged — it still names only pattern-recommendation trigger phrasings ("I want to build an agent that…", "what pattern fits a tool that answers questions over our docs", etc.) and never mentions framework, SDK, LangChain, CrewAI, or "which framework" language. Step 0's new detection logic (lines 51-65) only ever runs once agens has already loaded into the conversation. For a user whose very first message in a fresh session is a pure framework-fit question with no pattern-fit language in it, agens may never auto-trigger at all, so Step 0 never executes and Claude is free to answer the framework question from its own knowledge — the exact outcome DELEGATE-01 exists to prevent — unless the user happens to invoke `/agens` explicitly first.
**Fix:** Extend the `description` field with representative framework/SDK trigger phrasing (mirroring Step 0's vocabulary) so auto-invocation reaches the router for framework-fit openers too, not only pattern-fit ones.

## Info

### IN-01: "Initial message" undefined for a skill that can auto-trigger mid-session

**File:** `.claude/skills/agens/SKILL.md:51-70`
**Issue:** Step 0 scopes detection to "the user's INITIAL message only" and states mid-conversation framework language is out of scope. For an auto-triggered skill, "initial" is ambiguous: it could mean the first message of the overall conversation, or the first message after agens loads (which may be turns into an existing conversation). The file does not define which.
**Fix:** Clarify explicitly: "the message that caused this skill to load" vs. "the first message of the session," since these can differ for an auto-triggered skill.

### IN-02: Minor duplication of the "no directory enumeration" rule

**File:** `.claude/skills/agens/SKILL.md:207-209, 277-278`
**Issue:** "Never enumerate the `phases/` directories to guess a phase number" appears near-verbatim in both Check B's description and the "Inventing a phase number" anti-pattern entry. Harmless, but a future edit to one copy risks drifting from the other.
**Fix:** Consider stating the rule once in Check B and having the anti-pattern entry cross-reference it, rather than restating it.

---

_Reviewed: 2026-07-12_
_Reviewer: Claude (gsd-code-reviewer)_
_Depth: standard_
