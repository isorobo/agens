# Phase 0: Vault Consolidation - Research

**Researched:** 2026-07-12
**Domain:** Obsidian markdown knowledge-vault editing (wiki-agents); no code
**Confidence:** HIGH

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

- **D-01:** New index note at `30_Concepts/agent-patterns-index.md` (vault-relative), `type: concept`, following the established precedent (`best-practices-index.md`, `anti-patterns-index.md`).
- **D-02:** Phase 0 creates the index note only — no per-pattern stub concept notes. `90_Templates/template - pattern.md` (`type: pattern`) is not adopted. Pattern notes beyond the two that already exist (`react.md`, `workflow-vs-autonomous-agent.md`) are out of scope.
- **D-03:** Each entry states a trigger condition and a trade-off, not a bare one-liner (unlike the terser `best-practices-index.md`), so Phase 1's citation quoting (RECOMMEND-04) has a substantive, independently quotable passage per pattern.
- **D-04:** Gulli's book is the provenance spine (`defined_in`). Entries for patterns with a dedicated concept note (ReAct, Workflow vs Autonomous Agent) restate the trigger/trade-off inline and link out to the fuller note — no entry dead-ends into a second file.
- **D-05:** All 21 of Gulli's patterns get an entry — no filtering to only patterns the four-dimension questionnaire is likely to recommend. That filtering belongs to Phase 1.
- **D-06:** Concepts unique to the non-Gulli sources (Bar's BDI, SOAR, ACT-R cognitive architectures) are left out — academic cognitive architectures, not actionable implementation patterns.
- **D-07:** Where Gulli and Anthropic name the same idea differently (e.g. Orchestrator-Workers), the index carries one entry under Gulli's name with the alternate name noted inline — not a separate entry per source.
- **D-08:** Remove `topic/agent-patterns` from all six source notes once the index exists; each keeps its other topic tag(s). Only `agent-patterns-index.md` carries `topic/agent-patterns` afterward, so the MOC dataview query resolves to exactly one note.
- **D-09:** Each of the six source notes gets a one-line forward pointer (e.g. under "See also": `[[agent-patterns-index]] — consolidated pattern reference`).
- **D-10:** ROADMAP success criterion #3 rests on a false premise. `_memory/recent.md` does not exist as vault content. The `## YYYY-MM-DD` file is `.remember/recent.md`, Claude Code's own session-memory buffer, not hand-maintained vault content.
- **D-11:** `agens-log` (Phase 4, LOG-01) targets a new file at `99_Meta/agens-log.md`. It borrows the `## YYYY-MM-DD` date-header shape as a familiar format choice, not because `.remember/recent.md` is a vault convention.
- **D-12:** Phase 0 only records the log decision (path + format) in CONTEXT.md. Creating `99_Meta/agens-log.md` is Phase 4's job (LOG-01).

### Claude's Discretion
None — every gray area was resolved to an explicit user choice this session.

### Deferred Ideas (OUT OF SCOPE)
None — discussion stayed within phase scope. No scope creep occurred.
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| SETUP-01 | Recommendations are grounded against one canonical concept note, not six scattered sources — the `topic/agent-patterns` cluster is consolidated into a single note before any other phase depends on it. | This research supplies the exact 21-pattern list, the frontmatter/body template lifted from the two existing index precedents, the definitive six-file tag-edit list, the Gulli↔Anthropic crosswalk, and a grep-based single-lookup-target verification. |
</phase_requirements>

## Summary

This phase edits an external Obsidian vault at `C:\Users\Simon\Documents\wiki-agents` (confirmed present, not part of the agens git repo). It writes no code. The work has three parts: create one new index note, narrow one frontmatter tag across six existing notes, and record a log-format decision (already captured in CONTEXT.md).

Every design decision is locked (D-01 through D-12). The research supplies the concrete content the planner needs: the exact 21 pattern names, an exact frontmatter and body template derived from the two existing index notes, the definitive six-file edit list, the Gulli↔Anthropic naming crosswalk, and confirmation of the `## YYYY-MM-DD` date-header format.

The single lookup target is enforced mechanically, not by convention. The MOC dataview query filters `contains(topic, "topic/agent-patterns") AND type != "moc"`. Once the six sources drop the tag and only `agent-patterns-index.md` carries it, that query resolves to exactly one note. Verification is a grep, not an Obsidian render.

**Primary recommendation:** Copy the frontmatter and section skeleton from `best-practices-index.md` verbatim, then fill 21 pattern entries using the trigger/trade-off text already written in `40_Guides/agentic-design-patterns-plain-english.md`. That guide is the vault's own canonical rendering of Gulli's 21 patterns; the entries can be lifted and condensed rather than re-derived.

## Architectural Responsibility Map

| Capability | Primary Tier | Secondary Tier | Rationale |
|------------|-------------|----------------|-----------|
| Canonical pattern lookup | Vault content (`30_Concepts/agent-patterns-index.md`) | MOC dataview query | The index note is the single grounding target every later phase reads. |
| Single-target enforcement | Frontmatter `topic` tag on six sources | Obsidian/Dataview index | Tag narrowing makes the MOC query resolve to one note — tooling enforces the invariant. |
| Cross-navigation from any source | Forward pointer wikilinks (D-09) | schema.md §3 no-orphan rule | A reader landing on any of the six sources still reaches the canonical target. |

## Standard Stack

Not applicable — this phase installs no libraries and writes no code. The tools used are the built-in `Read`, `Edit`/`Write`, `Grep`, and `Glob` capabilities operating on markdown files. [VERIFIED: codebase — no `package.json`, no dependency manifest in phase scope]

## Package Legitimacy Audit

Not applicable — this phase installs no external packages. No registry lookup, no slopcheck run required. [VERIFIED: phase scope is markdown editing only]

## The 21 Gulli Patterns (Exact List for the Index)

The vault's own plain-English guide (`40_Guides/agentic-design-patterns-plain-english.md`) already renders Gulli's 21 patterns with a name, a trigger, and a trade-off. Use these names verbatim as the bold entry titles. Each carries pre-written trigger/trade-off source text the planner can condense into the D-03 entry format. [CITED: wiki-agents/40_Guides/agentic-design-patterns-plain-english.md, lines 46-148] [CITED: wiki-agents/10_Sources/Books/agentic-design-patterns-gulli-2025.md, lines 34-39]

> Naming note: the guide groups the 21 into **four parts** (7+4+3+7); the Gulli source note groups them into **five chapter-clusters** (6+3+4+3+5). Both total 21 and name the same patterns. Use the guide's names — they are the vault-canonical rendering.

| # | Pattern name (bold entry title) | Guide part | Notes for the entry |
|---|--------------------------------|-----------|---------------------|
| 1 | **Prompt Chaining** | One | Anthropic uses the same name (crosswalk). |
| 2 | **Routing** | One | Anthropic uses the same name (crosswalk). |
| 3 | **Parallelisation** | One | Anthropic uses the same name (crosswalk). Sub-modes: Fan-Out/Fan-In, Sectioning, Voting. |
| 4 | **Reflection** | One | Anthropic's "Evaluator-Optimiser" is the generator+evaluator flavour of this (crosswalk). |
| 5 | **Tool Use** | One | Existing note `[[tool-use]]` available as a related link. |
| 6 | **Planning** | One | ReAct is a sub-strategy here → link out to `[[react]]` (D-04). Also Plan-and-Execute, Tree of Thoughts. |
| 7 | **Multi-Agent Collaboration** | One | Anthropic's "Orchestrator-Workers" is a coordination sub-pattern of this (crosswalk). |
| 8 | **Memory Management** | Two | Existing note `[[memory]]` available as a related link. |
| 9 | **Learning and Adaptation** | Two | Reflexion / ACE loop. `[[reflexion]]` available. |
| 10 | **Model Context Protocol (MCP)** | Two | Existing note `[[mcp]]` available as a related link. |
| 11 | **Goal Setting and Monitoring** | Two | Observe-phase comparison against objective; stopping conditions. |
| 12 | **Exception Handling and Recovery** | Three | Fallback chain; error classification. |
| 13 | **Human-in-the-Loop** | Three | HITL / HOTL / HIC tiers on a reversibility framework. |
| 14 | **Knowledge Retrieval (RAG)** | Three | Guide title is "Knowledge Retrieval (RAG)"; Gulli source prose says "retrieval-augmented generation". Use the guide name; note RAG inline. |
| 15 | **Inter-Agent Communication (A2A)** | Four | Guide title "Inter-Agent Communication (A2A)"; Gulli source prose says "agent-to-agent communication". Use the guide name; note A2A inline. |
| 16 | **Resource-Aware Optimisation** | Four | Carries the "agents-versus-workflows distinction" → link out to `[[workflow-vs-autonomous-agent]]` (D-04). |
| 17 | **Reasoning Techniques** | Four | Chain of Thought, Extended Thinking, effort parameter. |
| 18 | **Guardrails and Safety Patterns** | Four | Prompt-injection / memory-poisoning defence in depth. |
| 19 | **Evaluation and Monitoring** | Four | Existing note `[[evaluation]]` available as a related link. |
| 20 | **Prioritisation** | Four | Urgency-importance matrix; dynamic re-prioritisation. |
| 21 | **Exploration and Discovery** | Four | Curiosity-driven queries; bounded exploration budget (~10% token spend). |

**Two patterns link out to existing dedicated notes (D-04):**

| Existing note | `name` frontmatter | Wikilink target | `type` | Natural attach point |
|---------------|-------------------|-----------------|--------|----------------------|
| `30_Concepts/react.md` | ReAct | `[[react]]` | architecture | Pattern #6 Planning (ReAct is its interleaved-reasoning strategy) |
| `30_Concepts/workflow-vs-autonomous-agent.md` | Workflow vs Autonomous Agent | `[[workflow-vs-autonomous-agent]]` | architecture | Pattern #16 Resource-Aware Optimisation (the agents-vs-workflows distinction) |

[VERIFIED: codebase — both target files exist; all other related-link targets (`memory`, `mcp`, `evaluation`, `tool-use`, `the-agent-loop`, `reflexion`, `plan-and-execute`, `planning-and-reasoning`) confirmed present in `30_Concepts/`]

## Gulli ↔ Anthropic Naming Crosswalk (D-07)

Anthropic's "Building Effective Agents" names one building block (the augmented LLM) and **five workflow patterns**: prompt chaining, routing, parallelisation, orchestrator-workers, evaluator-optimiser. [CITED: wiki-agents/10_Sources/Blog/anthropic-building-effective-agents.md, line 51]

Apply D-07 by carrying one entry under Gulli's name and noting the Anthropic term inline where it differs:

| Anthropic term | Gulli pattern (entry name) | Relationship | D-07 inline note to add |
|----------------|---------------------------|--------------|--------------------------|
| Prompt Chaining | #1 Prompt Chaining | Identical | Name matches Anthropic; no alias needed (or note "same term as Anthropic"). |
| Routing | #2 Routing | Identical | Name matches Anthropic. |
| Parallelisation | #3 Parallelisation | Identical | Name matches Anthropic. |
| Evaluator-Optimiser | #4 Reflection | Sub-flavour | Note: "Anthropic names the generator-plus-evaluator flavour *Evaluator-Optimiser*." |
| Orchestrator-Workers | #7 Multi-Agent Collaboration | Sub-pattern | Note: "Anthropic names the lead-delegates-to-workers coordination *Orchestrator-Workers*." |

[CITED: guide lines 61-75 confirm Orchestrator-Workers under Multi-Agent Collaboration and Evaluator-Optimizer under Reflection]

## Architecture Patterns

### The index-note template (lift from `best-practices-index.md`)

Both existing index notes (`best-practices-index.md`, `anti-patterns-index.md`) share one identical shape. Copy it. [CITED: wiki-agents/30_Concepts/best-practices-index.md] [CITED: wiki-agents/30_Concepts/anti-patterns-index.md]

**Exact frontmatter schema for `agent-patterns-index.md`** (fields and order match the precedent; values filled for this note):

```yaml
---
type: concept
status: draft
created: 2026-07-12
name: Agent Patterns Index
slug: agent-patterns-index
topic:
- topic/agent-patterns
tags: [index, design-patterns]
synonyms: [agent design patterns, agentic patterns]
defined_in: "[[10_Sources/Books/agentic-design-patterns-gulli-2025|Agentic Design Patterns (Gulli)]]"
related_concepts: ["[[react]]", "[[workflow-vs-autonomous-agent]]", "[[the-agent-loop]]"]
anthropic: false
---
```

Field rationale, grounded in `99_Meta/schema.md` §2.6 and §2.1: [CITED: wiki-agents/99_Meta/schema.md, lines 43-51, 123-132]
- `type: concept` — matches D-01 and both precedents; `pattern` is deliberately not used (D-02).
- `topic: [topic/agent-patterns]` — this note becomes the **sole** carrier of the tag after D-08. `topic/agent-patterns` is a valid controlled value (schema §2.3, line 63).
- `defined_in` — the Gulli book, the provenance spine (D-04). Wikilink form follows schema §3 source-link convention `[[10_Sources/<Source-Type>/<slug>|Short Title]]`.
- `related_concepts` — schema §3 requires every concept to link to at least three related concepts. The two D-04 notes plus `[[the-agent-loop]]` satisfy the minimum; the planner may widen this.
- `anthropic: false` — the note's spine is Gulli, not Anthropic primary material (contrast the two precedents, which set `anthropic: true`).

**Exact body section skeleton** (headings match the precedent; the entries section is renamed and expanded per D-03):

```markdown
# Agent Patterns Index

> One-line blockquote summary of the consolidated 21-pattern reference.

## Summary
[Short paragraph: this note consolidates the six-source topic/agent-patterns
cluster into one lookup target; Gulli's 21 patterns are the spine.]

## Patterns
[21 entries. Each entry, per D-03, is bold-name + trigger condition + trade-off,
not a bare one-liner. Entry shape below.]

## Detail
[Optional: the four-part grouping, or the agents-vs-workflows framing.]

## Trade-offs and Limits
[Optional short section, matching the precedent.]

## Related
- [[react]]
- [[workflow-vs-autonomous-agent]]
- [[the-agent-loop]]
[≥3 wikilinks per schema §3]

## Sources
- [[10_Sources/Books/agentic-design-patterns-gulli-2025|Agentic Design Patterns (Gulli)]]
- [[40_Guides/agentic-design-patterns-plain-english|Agentic Design Patterns — Plain English Guide]]

## See also
- [[MOC - Agent Patterns]]
```

### Per-pattern entry format (D-03)

The precedent's entries are terse one-liners (`**Small, focused agents** — A narrow scope cuts error compounding...`). D-03 requires more: a trigger AND a trade-off, so Phase 1 (RECOMMEND-04) has an independently quotable passage. The guide's "Verification" section (lines 162-177) already models the trigger/trade-off pair. Recommended entry shape:

```markdown
- **[Pattern name]** — [what it is, one clause]. Trigger: [the symptom that
  says use this now]. Trade-off: [the cost it buys against]. [Optional: See
  [[dedicated-note]] for depth — only for #6 Planning → [[react]] and #16
  → [[workflow-vs-autonomous-agent]] per D-04.]
```

Example lifted from guide source text (Prompt Chaining, guide line 51 + 172-173):

```markdown
- **Prompt Chaining** (Anthropic: same term) — decompose a complex task into a
  sequence of single-objective steps, with gates that validate schema and
  confidence between steps. Trigger: one prompt juggles multiple objectives and
  accuracy drops. Trade-off: buys accuracy with added latency.
```

### System flow (how the pieces connect after this phase)

```
                         topic/agent-patterns tag
   BEFORE:  6 source notes  ───────────────────────►  MOC dataview query
            + MOC (excluded by type != "moc")          returns 7 notes (noise)

   AFTER:   6 source notes  ──(tag removed, D-08)──►   (drop out of query)
            agent-patterns-index.md ──(sole carrier)─► MOC dataview query
                    ▲                                   returns exactly 1 note
                    │  forward pointers (D-09)           = single lookup target
            6 source notes' "See also" ────────────────►│
                                                         ▼
                                          Phase 1 recommend spine reads
                                          this one note for grounded citations
```

### Anti-patterns to avoid
- **Stripping the MOC's own tag.** `MOC - Agent Patterns.md` carries `topic/agent-patterns` (line 7) but is excluded from its own query by `type != "moc"`. Do not remove it; it is not one of the six sources. Removing it is out of scope and pointless.
- **Creating per-pattern stub notes.** D-02 forbids this. Only the index note is created.
- **Adding `type: pattern`.** No note in the vault uses it; D-02 keeps it unused.
- **Re-deriving pattern names from the Gulli source prose.** The guide already provides canonical names — re-deriving risks drift (e.g. "RAG" vs "Knowledge Retrieval").

## Don't Hand-Roll

| Problem | Don't build | Use instead | Why |
|---------|-------------|-------------|-----|
| Index note structure | A bespoke frontmatter/section design | Copy `best-practices-index.md` verbatim | Vault convention is already proven; the user chose "match precedent" in every gray area (CONTEXT §specifics). |
| Pattern trigger/trade-off text | Fresh prose per pattern | Condense the guide's existing per-pattern text (lines 51-148, 162-177) | The vault already has vetted, NZ-English trigger/trade-off descriptions. |
| Single-target enforcement | A convention or a comment | The MOC dataview `type` filter + tag narrowing (D-08) | Tooling enforces the invariant; convention drifts. |
| Verification of "one note" | Manual Obsidian inspection | `grep -rl 'topic/agent-patterns'` count | Deterministic, scriptable, no Obsidian render needed. |

**Key insight:** This phase is a convention-matching exercise, not a design exercise. Every structural question is answered by an existing vault note.

## Runtime State Inventory

This phase narrows a frontmatter tag that a live Obsidian Dataview query depends on — a migration-like operation. The categories below are answered explicitly.

| Category | Items found | Action required |
|----------|-------------|------------------|
| Stored data | The six source notes' `topic:` frontmatter lists each contain `topic/agent-patterns`. Exact lines: gulli L16, plain-english L8, bar L16, workflow-vs-autonomous-agent L9, react L9, anthropic-building-effective-agents L18. | Code/content edit: remove that one list item from each; keep the note's other topic tag(s). |
| Live service config | Obsidian Dataview query in `MOC - Agent Patterns.md` (L26-31) reads the tag live. No external service holds the tag. | None — the query is not edited (D-08). It re-resolves automatically once tags change. |
| OS-registered state | None — verified: no scheduled task, service, or OS registration references `topic/agent-patterns`. | None. |
| Secrets/env vars | None — verified: no secret or env var involved; this is public markdown content. | None. |
| Build artifacts / index cache | Obsidian maintains a metadata/tag cache under `.obsidian/`; schema §2.7 defines auto-stamp fields (`wiki_indexed`, `wiki_hash`) set by a `/wiki apply` pass. | None for Phase 0. Obsidian re-indexes on file change automatically. The `wiki_hash`/`wiki_indexed` stamps re-settle on the next `/wiki apply` run — not a Phase 0 task. |

**The canonical question — after every file is edited, what still holds the old state?** Nothing. Obsidian's Dataview reads tags live and re-runs on file change; there is no external cache to migrate. The single-lookup-target invariant is observable by grep the moment the six edits land.

## Common Pitfalls

### Pitfall 1: Editing the wrong "recent.md"
**What goes wrong:** Treating `_memory/recent.md` (referenced in ROADMAP success criterion #3 and STATE.md) as the format source.
**Why it happens:** ROADMAP names `_memory/recent.md`; that path resolves to no such convention file. The real `## YYYY-MM-DD` file is `.remember/recent.md` (D-10).
**How to avoid:** Success criterion #3 is satisfied by *confirming the format decision in CONTEXT.md* (already done, D-10/D-11/D-12), not by editing any recent.md. Phase 0 creates no log file.
**Warning signs:** A task that writes to `_memory/recent.md` or `.remember/recent.md` — both are wrong for Phase 0.

### Pitfall 2: Query returns two notes, not one
**What goes wrong:** After edits, the dataview query still returns more than the index note.
**Why it happens:** A source note's tag was missed, or the index note's tag was omitted.
**How to avoid:** Verify with grep: exactly one file (the new index) should contain `topic/agent-patterns` and not be `type: moc`. The MOC file also contains the string but is `type: moc` (excluded).
**Warning signs:** `grep -rl 'topic/agent-patterns' | wc -l` returns anything other than 2 (index + MOC).

### Pitfall 3: Broken forward-pointer or related-concept wikilinks
**What goes wrong:** A `[[agent-patterns-index]]` pointer or a related link resolves to nothing.
**Why it happens:** Slug mismatch, or linking to a note that does not exist.
**How to avoid:** The index slug is `agent-patterns-index` (filename `agent-patterns-index.md`). All related-link targets used above are confirmed to exist. Add the forward pointer to all six sources (D-09).
**Warning signs:** Obsidian renders an unresolved (grey) link.

## State of the Art

Not applicable — this is vault-content editing against a fixed, local knowledge base. No fast-moving external ecosystem bears on it. The Gulli (2025) and Anthropic (2024) sources are already captured in the vault.

## Environment Availability

| Dependency | Required by | Available | Version | Fallback |
|------------|-------------|-----------|---------|----------|
| wiki-agents vault | All edits | Yes — confirmed at `C:\Users\Simon\Documents\wiki-agents` | — | None needed |
| `Read`/`Edit`/`Write`/`Grep`/`Glob` tools | Editing + verification | Yes (built-in) | — | None needed |
| Obsidian (app) | Live dataview render | Not required for edits or verification | — | Grep-based verification replaces render |

**Missing dependencies with no fallback:** None.
**Note on vault access:** The vault is an external directory, outside the agens git repo. If execution runs in a sandbox without that path granted, the vault must be added (`--add-dir "C:/Users/Simon/Documents/wiki-agents"`). This session read and verified the path successfully.

## Security Domain

This phase writes no code, handles no user input, and touches no secret. ASVS categories V2-V6 do not apply. [VERIFIED: phase scope]

The one applicable control is file-path confinement (security.md §7): every write must land inside the intended vault directory. Concretely:
- The single new file: `C:\Users\Simon\Documents\wiki-agents\30_Concepts\agent-patterns-index.md`.
- The six edits: only the six source files listed below, only their `topic:` list and a "See also" line.
- No file outside `wiki-agents/` is written. No content in this phase is a secret; all of it is public knowledge-base prose.

## The Definitive Six-File Edit List (D-08 + D-09)

Search result for the exact tag `topic/agent-patterns` across the vault, cross-checked against CONTEXT.md's list of six sources. [VERIFIED: Grep across vault]

| # | File path (vault-relative) | Tag line | Other topic tag(s) that REMAIN | `type` |
|---|----------------------------|----------|--------------------------------|--------|
| 1 | `10_Sources/Books/agentic-design-patterns-gulli-2025.md` | L16 | `topic/architectures` (L17) | source |
| 2 | `40_Guides/agentic-design-patterns-plain-english.md` | L8 | none other — this note has only `topic/agent-patterns` | guide |
| 3 | `10_Sources/Books/agentic-ai-handbook-bar-2025.md` | L16 | (confirm remaining tag on edit; note carries BDI/SOAR/ACT-R material) | source |
| 4 | `30_Concepts/workflow-vs-autonomous-agent.md` | L9 | `topic/architectures` (L8) | architecture |
| 5 | `30_Concepts/react.md` | L9 | `topic/architectures` (L8) | architecture |
| 6 | `10_Sources/Blog/anthropic-building-effective-agents.md` | L18 | `topic/foundations` (L17) | source |

**Flag on file #2 (`agentic-design-patterns-plain-english.md`):** its frontmatter lists `topic/agent-patterns` as its **only** `topic` value (L7-8). Schema §2.1 requires at least one `topic` per note. Removing the tag leaves it with none, violating the schema. The planner must resolve this — options: (a) assign a replacement controlled topic before removing (e.g. `topic/foundations`), or (b) treat the guide as an intended member of the consolidated cluster and decide its topic explicitly. This is a genuine gap not covered by D-01 through D-12. See Open Questions.

**Not on the edit list (do NOT touch):**
- `50_MOCs/MOC - Agent Patterns.md` — carries the tag (L7) but is `type: moc`, excluded from its own query. It stays.
- `99_Meta/schema.md` (L63) and `99_Meta/design/2026-07-10-...md` (L139) — these mention `topic/agent-patterns` only as documentation of the controlled vocabulary, not as a frontmatter tag. They stay.

**D-09 forward pointer to add to each of the six:** a one-line entry, e.g. under a `## See also` section:
```
- [[agent-patterns-index]] — consolidated pattern reference
```

## Log-Format Confirmation (Success Criterion #3)

- `.remember/recent.md` exists and uses `## YYYY-MM-DD` date headers. [VERIFIED: Read — file contains `## 2026-07-10` header]
- `_memory/` exists as a directory, but the `recent.md` the ROADMAP references there is not the format-defining file; the date-header file is `.remember/recent.md` (D-10).
- Success criterion #3 is met by the CONTEXT.md record (D-10/D-11/D-12): the `## YYYY-MM-DD` shape is adopted for the future `99_Meta/agens-log.md` (Phase 4). **Phase 0 creates no log file.** [CITED: CONTEXT.md D-12]

## MOC Dataview Query (do not change)

```dataview
LIST
FROM ""
WHERE contains(topic, "topic/agent-patterns") AND type != "moc"
SORT file.name ASC
```
[CITED: wiki-agents/50_MOCs/MOC - Agent Patterns.md, lines 26-31]

No query rewrite is needed (D-08). The `type != "moc"` clause already excludes the MOC itself. After the six edits, the query resolves to exactly one note: `agent-patterns-index.md`.

## Verification Approach (for the planner's success criteria)

Automated, deterministic, no Obsidian needed:

| Check | Command | Expected |
|-------|---------|----------|
| Single lookup target | count files containing `topic/agent-patterns` | 2 — the new index + the MOC (MOC excluded from query by `type != "moc"`) |
| Index note exists | test `30_Concepts/agent-patterns-index.md` present | pass |
| All 21 patterns named as body text | grep each pattern name in the index | 21 matches (satisfies success criterion #2, and Phase 1's RECOMMEND-05 name-in-body check) |
| Six sources lost the tag | grep `topic/agent-patterns` in each of the six | zero matches in each |
| Forward pointers present | grep `agent-patterns-index` in each of the six | one match in each (D-09) |
| No broken related links | confirm each `[[target]]` in the index maps to an existing file | all resolve |

## Assumptions Log

| # | Claim | Section | Risk if wrong |
|---|-------|---------|---------------|
| A1 | The plain-English guide's 21 names are the correct canonical entry titles (vs. the Gulli source's chapter-cluster phrasing). | The 21 Gulli Patterns | Low — both sources agree on the 21 patterns; only surface phrasing differs (e.g. "Knowledge Retrieval (RAG)" vs "retrieval-augmented generation"). Guide is the vault's deliberate rendering. |
| A2 | The Evaluator-Optimiser↔Reflection and Orchestrator-Workers↔Multi-Agent-Collaboration mappings are the intended D-07 aliases. | Crosswalk | Low — both are stated explicitly in the guide (Evaluator-Optimizer under Reflection L63; Orchestrator-Workers under Multi-Agent Collaboration L72-75). |
| A3 | ReAct attaches to Planning (#6) and Workflow-vs-Autonomous attaches to Resource-Aware Optimisation (#16). | Two patterns link out | Medium — D-04 names both notes but not their exact host entry. The attach points are the strongest textual match; the planner may place them differently. |

## Open Questions (RESOLVED)

1. **RESOLVED — `agentic-design-patterns-plain-english.md` has only `topic/agent-patterns` — removing it leaves zero topics, violating schema §2.1.**
   - What we know: schema §2.1 requires at least one `topic` per note (L50). This guide's sole topic is `topic/agent-patterns` (L7-8).
   - What's unclear: which replacement controlled topic it should carry, or whether D-08 intended this guide to keep the tag as a second consolidated-cluster member.
   - Recommendation: the planner should add a replacement topic before removing the tag. `topic/foundations` fits (the guide is a foundational map). Flag for user confirmation, since it is not covered by D-01 through D-12.
   - Resolution: 00-01-PLAN.md Task 2 adds `topic/foundations` to the guide's frontmatter before removing `topic/agent-patterns`, verified by acceptance criteria against the live vault file.

2. **RESOLVED — Related-concept links beyond the two D-04 notes.**
   - What we know: schema §3 requires ≥3 related concepts; the guide already links to `[[memory]]`, `[[mcp]]`, `[[evaluation]]`, `[[tool-use]]`, `[[the-agent-loop]]` (all confirmed to exist).
   - What's unclear: D-04 explicitly names only ReAct and Workflow-vs-Autonomous as link-out cases. Whether the index should also cross-link the other existing concept notes is unspecified.
   - Recommendation: satisfy the §3 minimum with the two D-04 notes plus `[[the-agent-loop]]`; treat wider cross-linking as optional planner discretion within the schema rule. No user decision required.
   - Resolution: 00-01-PLAN.md Task 1 sets `related_concepts` to exactly these three wikilinks, satisfying the §3 minimum. No further links required.

## Sources

### Primary (HIGH confidence)
- `wiki-agents/40_Guides/agentic-design-patterns-plain-english.md` — canonical 21-pattern names, triggers, trade-offs, four-part grouping.
- `wiki-agents/10_Sources/Books/agentic-design-patterns-gulli-2025.md` — Gulli provenance spine, chapter-cluster grouping.
- `wiki-agents/30_Concepts/best-practices-index.md` + `anti-patterns-index.md` — exact frontmatter + body-section precedent.
- `wiki-agents/10_Sources/Blog/anthropic-building-effective-agents.md` — Anthropic's five workflow-pattern names (crosswalk).
- `wiki-agents/30_Concepts/react.md` + `workflow-vs-autonomous-agent.md` — exact `name`/slug/wikilink targets for the two link-out notes.
- `wiki-agents/99_Meta/schema.md` — frontmatter fields (§2), linking conventions (§3), topic vocabulary (§2.3).
- `wiki-agents/50_MOCs/MOC - Agent Patterns.md` — exact dataview query.
- `wiki-agents/.remember/recent.md` — `## YYYY-MM-DD` date-header format confirmation.
- `wiki-agents/10_Sources/Books/agentic-ai-handbook-bar-2025.md` — BDI/SOAR/ACT-R material (D-06 exclusion confirmed, lines 36, 44-45).
- Grep across vault — definitive six-file `topic/agent-patterns` edit list.

### Secondary (MEDIUM confidence)
- None.

### Tertiary (LOW confidence)
- None. Every claim is grounded in a read vault file or a grep result.

## Metadata

**Confidence breakdown:**
- 21-pattern list: HIGH — lifted from the vault's own canonical guide, cross-checked against the Gulli source.
- Frontmatter/body template: HIGH — copied from two identical existing precedents; validated against schema §2.
- Six-file edit list: HIGH — exact grep result, cross-checked against CONTEXT.md.
- Crosswalk: HIGH — both aliases stated explicitly in vault text.
- Link attach points (A3): MEDIUM — strongest textual match, but D-04 leaves the exact host entry open.
- Guide-topic gap (Open Q1): flagged — genuine schema conflict needing a planner/user decision.

**Research date:** 2026-07-12
**Valid until:** stable — the vault is local and fixed; re-check only if the six source notes' frontmatter changes before planning.
