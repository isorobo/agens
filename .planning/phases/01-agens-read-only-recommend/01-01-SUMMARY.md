---
phase: 01-agens-read-only-recommend
plan: 01
subsystem: agens-skill
tags: [claude-code-skill, recommend, citation-gate, refusal]
requires: []
provides:
  - "agens read-only recommend/refuse spine (SKILL.md)"
  - "trigger-tests acceptance harness for the Plan 03 human check"
affects:
  - ".claude/skills/agens/"
tech-stack:
  added: []
  patterns:
    - "Dual-invocation Claude Code Skill (no invocation-control fields)"
    - "Single four-question AskUserQuestion intake"
    - "Read+Grep deterministic citation gate before any recommendation"
    - "Fixed plain-refusal template on any non-match"
key-files:
  created:
    - ".claude/skills/agens/SKILL.md"
    - ".claude/skills/agens/references/trigger-tests.md"
  modified: []
decisions:
  - "Match threshold: one bold entry whose Trigger the four answers satisfy; else refuse"
  - "Surface exactly one pattern; no multi-pattern output (REQUIREMENTS Out of Scope)"
  - "Goal question carries the four D-03 labels with no catch-all bucket; tool's Other routes to refusal"
  - "Grep is a post-hoc bold-**Name** check of an LLM-selected pattern, not a lexical search of goal wording"
metrics:
  duration: "~5 minutes"
  completed: 2026-07-12
---

# Phase 1 Plan 01: agens Read-Only Recommend/Refuse Spine Summary

Authored the agens read-only recommend/refuse spine as one Claude Code Skill: a
dual-invocation `SKILL.md` that asks four fixed questions in a single
AskUserQuestion call, gates every recommendation behind a Read+Grep citation
check against `30_Concepts/agent-patterns-index.md`, and refuses plainly when no
bold entry fits.

## What Was Built

- **`.claude/skills/agens/references/trigger-tests.md`** — the behavioural oracle,
  authored test-first before the skill. Holds a 10-item should-trigger set and an
  8-item should-not-trigger set, both worded independently of the SKILL.md
  description (per the STATE.md blocker), plus a manual walkthrough checklist for
  the Plan 03 human check.
- **`.claude/skills/agens/SKILL.md`** — the recommend/refuse spine:
  - Frontmatter with `name: agens`, an auto-trigger-tuned `description` that
    front-loads natural project-starting phrasings, and
    `allowed-tools: Read Grep Glob AskUserQuestion`. Neither
    `disable-model-invocation` nor `user-invocable` is set, so both `/agens` and
    auto-trigger work (RECOMMEND-01 + RECOMMEND-02).
  - A When-to-Use list and a single four-question AskUserQuestion call carrying
    the locked D-03 through D-06 labels under the headers Goal, Workflow,
    Sensitivity, and Latency (each ≤12 characters) (RECOMMEND-03).
  - A goal-bucket to pattern-family mapping, a single-entry match threshold, and
    a two-step citation gate: Read/Glob the vault-relative path, then anchored
    Grep of the bold `**Name**` literal (RECOMMEND-04, RECOMMEND-05).
  - A one-pattern recommendation quoting the matched Trigger and Trade-off, and a
    fixed plain refusal that names the four searched dimensions and cites nothing
    (RECOMMEND-06). Anti-pattern notes forbid decorative citation and un-gated
    model recall.

## Task Commits

| Task | Name | Commit |
| ---- | ---- | ------ |
| 1 | Trigger-tests acceptance harness | d7f56c0 |
| 2 | SKILL.md frontmatter, When-to-Use, four-question intake | 5deb6e8 |
| 3 | Citation gate, recommendation output, plain refusal | 30f7c00 |

## Requirements Satisfied

- RECOMMEND-01: `agens` directory yields `/agens`; `name` matches the directory.
- RECOMMEND-02: description front-loads natural phrasings; neither invocation-control field is set.
- RECOMMEND-03: one AskUserQuestion call carries the four fixed questions before any recommendation.
- RECOMMEND-04: a recommendation cites the vault-relative path with the Trigger/Trade-off quoted.
- RECOMMEND-05: the Read+Grep gate runs before any recommendation.
- RECOMMEND-06: a non-match refuses plainly, names the four dimensions, and cites nothing.

## Deviations from Plan

None — plan executed exactly as written. The three resolved open questions and the
grep-recall resolution were applied as specified in the plan's
`<resolved_open_questions>` block.

## Threat Model Coverage

All five `mitigate`-disposition threats are addressed in the skill body:
- T-01-01 (spoofed grounding) — no recommendation without the Read+Grep gate.
- T-01-02 (free-text tampering) — "Other" on any dimension routes to refusal.
- T-01-03 (prompt injection) — matched passage quoted as data; read-only tool grant.
- T-01-04 (path traversal) — fixed vault-relative path; no user-controlled segment.
- T-01-05 (privilege) — allowed-tools limited to Read Grep Glob AskUserQuestion.

## Known Stubs

None. The recommend/refuse spine is fully wired against the live vault note. Full
behavioural confirmation (live auto-trigger, live questionnaire, live
recommendation, live refusal) is deferred to the Plan 03 human checkpoint, as the
plan specifies.

## Verification

- `.claude/skills/agens/SKILL.md` exists (150 lines) with frontmatter,
  questionnaire, gate, recommendation, and refusal sections.
- `.claude/skills/agens/references/trigger-tests.md` exists (62 lines) with
  should-trigger, should-not-trigger, and walkthrough sections.
- All per-task automated greps passed.

## Self-Check: PASSED

- FOUND: .claude/skills/agens/SKILL.md
- FOUND: .claude/skills/agens/references/trigger-tests.md
- FOUND commit d7f56c0, 5deb6e8, 30f7c00
