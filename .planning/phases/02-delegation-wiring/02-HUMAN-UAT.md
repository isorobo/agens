---
status: complete
phase: 02-delegation-wiring
source: [02-VERIFICATION.md]
started: 2026-07-12T12:15:03Z
updated: 2026-07-13T09:13:15Z
---

## Current Test

[testing complete]

## Tests

### 1. Live routing (DELEGATE-01)
expected: Open a fresh interactive session with agens loaded from the user-level install (`~/.claude/skills/agens/`), inside this GSD project (active Phase 2), with the opener: "Which framework should I use for this — LangChain or the Claude Agent SDK?" agens should skip the four-question pattern intake, run the combined gate (Check A hits the user-level literal path since `~/.claude/skills/gsd-ai-integration-phase/SKILL.md` exists; Check B reads STATE.md's Phase line and passes only if the status token still marks an in-progress phase), and invoke the Skill tool with skill name `gsd-ai-integration-phase` and the phase integer as `$ARGUMENTS`. No inline framework opinion, no questionnaire.
result: passed (on retest with explicit invocation). Attempt 1, bare opener with no invocation: agens did not auto-fire — base Claude answered the framework question inline from the agens project CLAUDE.md, made no Skill call, and referred to agens in the third person; recorded as an auto-trigger gap (see Gaps), not a delegation-wiring failure. Attempt 2, explicit invocation with the same opener: no four-question intake, combined gate ran, Skill call fired with `gsd-ai-integration-phase` and phase `2`, and the delegated skill ran end-to-end — GSD "AI-SPEC COMPLETE — PHASE 2" banner, `02-AI-SPEC.md` written and committed as 45a0cfe, framework answer delivered by the delegated framework selector (Claude Agent SDK; LangChain eliminated on the hard constraint), no inline agens opinion. Preconditions verified before attempt 1: both user-level SKILL.md files present; STATE.md line 28 read "Phase: 02 (delegation-wiring) — EXECUTING".

### 2. Live failure (DELEGATE-02)
expected: From a directory/session with no `.planning/STATE.md` (or a STATE.md whose Phase line status token is COMPLETE, not an in-progress token), open with the same framework-fit message. The fixed name+reason+next-step failure message should fire, naming the failed condition(s); no Skill call; no inline framework opinion.
result: passed. Run from `C:\Users\Simon\claude-projects` (no `.planning/`) with explicit `/agens` invocation and the same opener. agens recognised the framework-fit question, skipped the questionnaire, ran the pre-flight checks, and reported Check B failed with the reason (no `.planning/STATE.md`, no active GSD phase). The fixed name+reason+next-step message fired verbatim; no Skill call; no inline framework opinion. Cosmetic observation: agens appended its own elaboration after the fixed message, citing an example path that does not exist (`C:\Users\Simon\claude-projects-trader\` — the real path is `C:\Users\Simon\claude-projects\trader`); the fixed-message contract does not bar trailing text, but the invented path is worth tightening.

### 3. Unchanged pattern path
expected: Open with a non-framework opener such as "I want to build an assistant that answers questions over our internal docs." The Phase 1 four-question pattern intake should run unchanged; no delegation branch fires.
result: passed. Run in the agens project with explicit `/agens` invocation and the exact opener. Step 0 triage classified the request as pattern-fit; the four fixed questions ran; no gate checks, no Skill call, no delegation failure message. The session lacked the wiki-agents vault grant, so the citation gate failed on path resolution and agens refused plainly with the correct `--add-dir` remedy — correct Phase 1 behaviour. Two refusal-discipline observations logged in Gaps (template misstatement; model-knowledge hint); neither concerns the delegation wiring under test. Addendum: after `/add-dir` granted the vault in the same session, a re-run completed the happy path — intake answers reused, citation gate passed both checks, and agens recommended Knowledge Retrieval (RAG) with the Trigger and Trade-off lines quoted verbatim from `30_Concepts/agent-patterns-index.md`. The only variable between refusal and recommendation was vault access, confirming the gate keys on citation availability, not model knowledge.

### 4. Present-as-is hand-off (D-09)
expected: Confirm that after a successful Skill call, agens narrates nothing and `gsd-ai-integration-phase`'s own prompts/AI-SPEC.md output appear verbatim, not paraphrased.
result: issue — mid-run clean, tail breached. Verified against the Test 1 retest transcript. Mid-run: every message after agens' Skill call (transcript line 33) up to the completion banner belonged to the delegated workflow — its own progress banners (line 86) and validation narration (line 106); agens restated nothing; no AskUserQuestion dialogs fired (the selector, researchers, and eval-planner ran as four Agent sub-tasks). Tail: the final assistant message (line 117) contained the full GSD banner ending at "Next step: /gsd:plan-phase 2" — the defined completion boundary per workflows/ai-integration-phase.md:266-280 — then appended two agens-voice paragraphs in the same response: a restated framework answer and a section-by-section summary of 02-AI-SPEC.md. That paraphrase after the boundary is the narration D-09 bars. Fix: tighten the "Present the target's output as-is" section of agens SKILL.md — after the delegated skill's completion banner, agens emits nothing; the turn ends at the target's own "Next step" line.

## Summary

total: 4
passed: 3
issues: 1
pending: 0
skipped: 0
blocked: 0

## Gaps

- Auto-trigger gap: with agens installed at user level and the exact framework-fit opener from DELEGATE-01, the skill was not selected on a bare opener; base Claude answered inline from project CLAUDE.md. Explicit invocation works (Test 1 retest passed). Confound noted: inside the agens repo, "this" resolves to a project whose CLAUDE.md already settles the framework. Consider a follow-up trigger test from a neutral project.
- UAT side effect (resolved 2026-07-13): the Test 1 retest ran the delegated skill to completion, writing and committing `02-AI-SPEC.md` (45a0cfe) during acceptance testing. Operator decision: keep the artifact.
- Refusal-template misstatement (Phase 1 behaviour, observed in Test 3): with the vault ungranted, the "no pattern matches / I searched agent-patterns-index.md" refusal fired even though no search happened. The path-does-not-resolve failure needs its own wording so the refusal states the true cause.
- Refuse-then-hint leak (Phase 1 behaviour, observed in Test 3): after the plain refusal, agens named "Knowledge Retrieval / RAG" as a "clear candidate" from model knowledge. The contract is a plain refusal with no candidate named.
- D-09 tail violation (Test 4) — FIXED 2026-07-13 via quick task 260713-tmx (commit 8046ac0): the "Present the target's output as-is" section now states the turn ends at the target's own final line, and the "Wrapping the target's output" anti-pattern bullet covers the tail case. Installed user-level copy synced byte-identical. Retest of the tail behaviour pending next live delegation.
- Stale doc target: the session's reply named "GSD framework selector" as agens' routing target. Locate the Phase 1 wording (likely agens project CLAUDE.md) and update it to `gsd-ai-integration-phase`.
