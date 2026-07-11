# Phase 0: Vault Consolidation - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-07-12
**Phase:** 0-Vault Consolidation
**Areas discussed:** Note structure & location, Pattern taxonomy scope, Fate of the six source notes, Log-format correction (success criterion #3)

---

## Note structure & location

| Option | Description | Selected |
|--------|-------------|----------|
| New index in 30_Concepts/ | `agent-patterns-index.md`, type: concept, following `best-practices-index.md`/`anti-patterns-index.md` precedent | ✓ |
| Extend the plain-english guide | Broaden the existing Gulli-only guide to cover all six sources | |
| Upgrade the MOC | Disqualified — schema.md forbids MOCs holding transcluded content | |

**User's choice:** New index in 30_Concepts/
**Notes:** Chosen because it's an exact match to existing, already-proven vault convention (two precedents).

| Option | Description | Selected |
|--------|-------------|----------|
| Index only | One index note; no per-pattern stub notes | ✓ |
| Index + stub notes for every pattern | Also create a bare concept note per pattern | |

**User's choice:** Index only
**Notes:** Matches ROADMAP's "Mode: mvp" for this phase.

| Option | Description | Selected |
|--------|-------------|----------|
| Trigger + trade-off pair | Each entry states when to use the pattern and what it costs | ✓ |
| One-liner per entry | Match best-practices-index.md's terse style exactly | |

**User's choice:** Trigger + trade-off pair
**Notes:** Gives Phase 1's citation quoting (RECOMMEND-04) substantive text to quote.

| Option | Description | Selected |
|--------|-------------|----------|
| Gulli as spine, cross-link the rest | Gulli's taxonomy anchors defined_in; existing dedicated notes linked for depth | ✓ |
| Cite all contributing sources per entry | Every entry lists all six sources where relevant | |

**User's choice:** Gulli as spine, cross-link the rest

---

## Pattern taxonomy scope

| Option | Description | Selected |
|--------|-------------|----------|
| All 21 | Complete Gulli taxonomy, no filtering | ✓ |
| Only questionnaire-relevant patterns | Filter to patterns a goal/workflow/sensitivity/latency questionnaire would plausibly select | |

**User's choice:** All 21
**Notes:** Filtering judgement belongs to Phase 1, not Phase 0.

| Option | Description | Selected |
|--------|-------------|----------|
| Leave them out | BDI/SOAR/ACT-R excluded — academic cognitive architectures, not implementation patterns | ✓ |
| Include them alongside the 21 | Full union of every named architecture/pattern across all six sources | |

**User's choice:** Leave them out

| Option | Description | Selected |
|--------|-------------|----------|
| One entry, note the alternate name | Gulli's term as entry name; alternate name noted inline | ✓ |
| Separate entries per source's terminology | One entry per named term regardless of overlap | |

**User's choice:** One entry, note the alternate name

| Option | Description | Selected |
|--------|-------------|----------|
| Restate briefly, link for depth | Trigger/trade-off inline even for patterns with a dedicated note | ✓ |
| Point only, no restatement | Just a one-line pointer for ReAct and Workflow-vs-Agent | |

**User's choice:** Restate briefly, link for depth
**Notes:** Keeps every index entry independently quotable for citation purposes.

---

## Fate of the six source notes

| Option | Description | Selected |
|--------|-------------|----------|
| Narrow to the index only | Remove topic/agent-patterns from the six; only the index keeps the tag | ✓ |
| Leave all six tagged as-is | Six sources plus index all carry the tag | |

**User's choice:** Narrow to the index only
**Notes:** Makes "one canonical concept note" hold in the vault's own dataview tooling, not just in name.

| Option | Description | Selected |
|--------|-------------|----------|
| Add a one-line forward pointer | Each of the six gets a "Consolidated into" line pointing at the index | ✓ |
| Leave the six untouched | Only the tag changes; no other edits | |

**User's choice:** Add a one-line forward pointer

---

## Log-format correction (success criterion #3)

| Option | Description | Selected |
|--------|-------------|----------|
| New vault log, same date-header format | Create a real vault-owned log file borrowing the ## YYYY-MM-DD format as a style choice only | ✓ |
| Defer the decision to Phase 4 | Correct the false premise but leave location/format undecided | |
| Reuse .remember/recent.md as-is | Have agens-log write into Claude Code's session-memory buffer | |

**User's choice:** New vault log, same date-header format
**Notes:** Finding: ROADMAP.md/STATE.md's claim that `_memory/recent.md` exists with a `## YYYY-MM-DD` convention is false. That file/format is actually `.remember/recent.md`, Claude Code's own session-memory buffer for the vault directory — not vault content. Surfaced during codebase scouting, not part of the user's original area selection, but directly affects this phase's stated success criterion #3.

| Option | Description | Selected |
|--------|-------------|----------|
| 99_Meta/agens-log.md, decision only | Matches the vault's other pipeline/system files; Phase 0 records the decision, Phase 4 builds it | ✓ |
| _memory/agens-log.md, decision only | Reuses the existing _memory/ folder name | |

**User's choice:** 99_Meta/agens-log.md, decision only

---

## Claude's Discretion

None — every gray area was resolved to an explicit user choice.

## Deferred Ideas

None — discussion stayed within phase scope.
