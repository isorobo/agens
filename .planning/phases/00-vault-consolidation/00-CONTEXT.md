# Phase 0: Vault Consolidation - Context

**Gathered:** 2026-07-12
**Status:** Ready for planning

<domain>
## Phase Boundary

Collapse the wiki-agents vault's six-source `topic/agent-patterns` cluster into one canonical concept note that becomes the single lookup target every later phase (starting with Phase 1's citation-grounded recommend spine) reads from. This phase writes vault content — it does not touch agens' own Skill code, which doesn't exist yet.

</domain>

<decisions>
## Implementation Decisions

### Note structure & location
- **D-01:** New index note at `30_Concepts/agent-patterns-index.md` (vault-relative path), `type: concept`, following the established index precedent already in the vault: `30_Concepts/best-practices-index.md` and `30_Concepts/anti-patterns-index.md`.
- **D-02:** Phase 0 creates the index note only — no per-pattern stub concept notes. The vault's `90_Templates/template - pattern.md` (`type: pattern`) exists but is unused by any current note; this phase does not adopt it. Individual pattern notes beyond the two that already exist (`react.md`, `workflow-vs-autonomous-agent.md`) are out of scope.
- **D-03:** Each entry states a trigger condition and a trade-off, not a bare one-liner (unlike the terser `best-practices-index.md` style) — gives Phase 1's citation quoting (RECOMMEND-04) a substantive, independently quotable passage per pattern.
- **D-04:** Gulli's book (`10_Sources/Books/agentic-design-patterns-gulli-2025.md`) is the provenance spine (`defined_in`). Entries for patterns that already have a dedicated concept note (ReAct, Workflow vs Autonomous Agent) restate the trigger/trade-off briefly inline and link out to the fuller note for depth — no entry is citation-dead-ended into a second file.

### Pattern taxonomy scope
- **D-05:** All 21 of Gulli's patterns get an entry — no filtering down to only patterns agens' four-dimension questionnaire (goal, workflow shape, data sensitivity, latency/cost) is likely to recommend. That filtering judgement belongs to Phase 1, not Phase 0.
- **D-06:** Concepts unique to the non-Gulli sources (Bar's BDI, SOAR, ACT-R cognitive architectures) are explicitly left out of the index — they are academic cognitive architectures, not actionable implementation patterns in the sense the other five sources use the word. Including them would blur "pattern" into "any concept mentioned anywhere in the cluster."
- **D-07:** Where Gulli and Anthropic name the same underlying idea differently (e.g. Orchestrator-Workers appears in both, described the same way), the index carries one entry under Gulli's name, with the alternate name noted inline — not a separate entry per source's terminology.

### Fate of the six source notes
- **D-08:** Remove `topic/agent-patterns` from all six source notes once the index exists; each keeps whatever other topic tag(s) already apply (e.g. `topic/architectures`, `topic/foundations`). Only `agent-patterns-index.md` carries `topic/agent-patterns` afterward, so `MOC - Agent Patterns.md`'s dataview query resolves to exactly one note — the literal "single lookup target" the phase goal states, enforced by the vault's own tooling, not just by convention.
- **D-09:** Each of the six source notes gets a one-line forward pointer (e.g. under "See also": `[[agent-patterns-index]] — consolidated pattern reference`) so a reader who lands on any of the six directly still finds the canonical target. Matches the vault's existing cross-linking discipline (schema.md §3: no orphan notes).

### Log-format correction (success criterion #3)
- **D-10:** ROADMAP.md's success criterion #3 rests on a false premise. `_memory/recent.md` does not exist as vault content. The file matching that description — `## YYYY-MM-DD` date headers — is `.remember/recent.md`, Claude Code's own session-memory buffer for the vault directory, not vault-authored content the user maintains by hand.
- **D-11:** `agens-log` (Phase 4, LOG-01) targets a new, real vault-owned file at `99_Meta/agens-log.md`. It borrows the `## YYYY-MM-DD` date-header shape purely as a familiar, already-proven format choice — not because `.remember/recent.md` is an actual vault convention. `99_Meta/` already holds the vault's other pipeline/system files (`schema.md`, `roadmap.md`, `watchlist.md`), making it the natural home for a tool-authored log distinct from human research notes.
- **D-12:** Phase 0 only records this decision (path + format) in this CONTEXT.md. Creating and writing `99_Meta/agens-log.md` is Phase 4's job (LOG-01), not Phase 0's — Phase 0's own success criteria only require confirming the log format, not building the log.

### Claude's Discretion
None — every gray area presented was resolved to an explicit user choice this session; no "you decide" selections occurred.

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Vault schema & structural precedent
- `wiki-agents/99_Meta/schema.md` — frontmatter field rules, folder purpose table, `type` enum, `topic` controlled vocabulary, linking conventions. §3 ("MOCs collect wikilinks, not transcluded content") is the rule that disqualified upgrading `MOC - Agent Patterns.md` in place of a new index note.
- `wiki-agents/30_Concepts/best-practices-index.md` — structural precedent for the new index note: `type: concept`, bold-named entries with a description + wikilink, `defined_in`/`related_concepts` frontmatter, Sources/See also sections.
- `wiki-agents/30_Concepts/anti-patterns-index.md` — second precedent, identical shape.
- `wiki-agents/90_Templates/template - pattern.md` — the vault's `type: pattern` template. Considered and not used (D-02); no note in the vault currently uses `type: pattern`.

### The six sources being consolidated
- `wiki-agents/10_Sources/Books/agentic-design-patterns-gulli-2025.md` — Gulli's 21-pattern taxonomy; the provenance spine for the new index (D-04).
- `wiki-agents/40_Guides/agentic-design-patterns-plain-english.md` — existing plain-English map of Gulli's 21 patterns (Gulli-only scope; not extended, per D-01).
- `wiki-agents/10_Sources/Books/agentic-ai-handbook-bar-2025.md` — Bar's broader agentic-AI survey; source of the BDI/SOAR/ACT-R material explicitly excluded (D-06).
- `wiki-agents/30_Concepts/workflow-vs-autonomous-agent.md` — existing dedicated concept note; index entry links here for depth (D-04).
- `wiki-agents/30_Concepts/react.md` — existing dedicated concept note; index entry links here for depth (D-04).
- `wiki-agents/10_Sources/Blog/anthropic-building-effective-agents.md` — source of Anthropic's five-workflow-pattern terminology, cross-referenced where it names the same idea as Gulli under a different term (D-07).

### MOC and log target
- `wiki-agents/50_MOCs/MOC - Agent Patterns.md` — existing MOC whose dataview query (`WHERE contains(topic, "topic/agent-patterns")`) becomes a true single-note lookup once the six sources' tags are narrowed (D-08).
- `wiki-agents/.remember/recent.md` — the actual `## YYYY-MM-DD` date-header file; referenced for its format only (D-11), not as a vault convention (D-10).
- `wiki-agents/99_Meta/agens-log.md` — target path for the new vault-owned log Phase 4 will create; does not exist yet (D-11, D-12).

### Project-level
- `.planning/PROJECT.md` — Core Value (citation discipline), Constraints, Key Decisions table entry for Phase 0 consolidation.
- `.planning/REQUIREMENTS.md` — SETUP-01, this phase's sole requirement.
- `.planning/ROADMAP.md` §Phase 0 — success criteria this phase must satisfy, including the corrected criterion #3 (D-10, D-11).

</canonical_refs>

<code_context>
## Existing Vault Insights

### Reusable Assets
- `best-practices-index.md` / `anti-patterns-index.md` structure — directly reusable as the template shape for `agent-patterns-index.md` (frontmatter fields, entry format, Related/Sources/See also sections).

### Established Patterns
- Vault linking convention (`schema.md` §3): every concept links to at least three related concepts; every note links to its topic MOC and the MOC links back; no orphan notes.
- `defined_in` frontmatter field points to the source establishing a note's canonical definition — used here to anchor the index to Gulli's book.

### Integration Points
- `MOC - Agent Patterns.md`'s dataview query is the mechanical link between the new index and the vault's existing navigation — no query rewrite needed once the tag-narrowing decision (D-08) is executed.
- Phase 1's citation-resolves check (RECOMMEND-05) will read `agent-patterns-index.md` looking for pattern names as body text — the index's bold-name + trigger/trade-off entry format (D-03) is what that check matches against.

</code_context>

<specifics>
## Specific Ideas

The user consistently chose the option that matched existing vault precedent (the index-note structure, Gulli-as-spine provenance, tag-narrowing to enforce a literal single lookup target) over a from-scratch design in every gray area discussed. Treat "match established vault convention first" as the default lens for later phases too, not just this one.

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope. No scope creep occurred; every gray area discussed was either explicitly in this phase's success criteria or a direct correction to one of them (the log-format finding).

</deferred>

---

*Phase: 0-Vault Consolidation*
*Context gathered: 2026-07-12*
