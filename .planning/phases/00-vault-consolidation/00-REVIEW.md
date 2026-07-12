---
phase: 00-vault-consolidation
reviewed: 2026-07-12T00:00:00Z
depth: standard
files_reviewed: 7
files_reviewed_list:
  - C:/Users/Simon/Documents/wiki-agents/30_Concepts/agent-patterns-index.md
  - C:/Users/Simon/Documents/wiki-agents/10_Sources/Books/agentic-design-patterns-gulli-2025.md
  - C:/Users/Simon/Documents/wiki-agents/40_Guides/agentic-design-patterns-plain-english.md
  - C:/Users/Simon/Documents/wiki-agents/10_Sources/Books/agentic-ai-handbook-bar-2025.md
  - C:/Users/Simon/Documents/wiki-agents/30_Concepts/workflow-vs-autonomous-agent.md
  - C:/Users/Simon/Documents/wiki-agents/30_Concepts/react.md
  - C:/Users/Simon/Documents/wiki-agents/10_Sources/Blog/anthropic-building-effective-agents.md
findings:
  critical: 0
  warning: 2
  info: 1
  total: 3
status: issues_found
---

# Phase 00-vault-consolidation: Code Review Report

**Reviewed:** 2026-07-12T00:00:00Z
**Depth:** standard
**Files Reviewed:** 7
**Status:** issues_found

## Summary

Reviewed the new canonical `agent-patterns-index.md` note and the six source notes edited to narrow the `topic/agent-patterns` tag onto it. Verified against the vault's own git history (`git diff`) that every edit is exactly what the plan specifies: a single frontmatter tag line removed per file, one forward-pointer bullet added, and no other content touched. No content loss, no malformed YAML, no broken wikilinks — every wikilink target in the seven files (`react`, `workflow-vs-autonomous-agent`, `the-agent-loop`, `MOC - Agent Patterns`, `MOC - Architectures`, both source notes, `best-practices-index`, `react-yao-2022`, etc.) resolves to an existing file in the vault. The single-lookup-target invariant holds: exactly two files (`agent-patterns-index.md` and the MOC itself) carry the tag, and the MOC's `type != "moc"` dataview clause excludes itself, so the query resolves to the one intended note.

Two quality issues remain, both grounded in the vault's own documented conventions rather than subjective style: a reciprocal-link contract broken by the tag removal (WARNING), and an uncited factual attribution inside the new index note (WARNING). Neither causes data loss or breaks the phase's stated acceptance criteria, but both degrade the vault's navigation and citation guarantees that later phases (RECOMMEND-04/05) depend on.

## Warnings

### WR-01: Tag removal leaves three notes with a one-way link into a MOC that no longer lists them

**File:** `C:/Users/Simon/Documents/wiki-agents/30_Concepts/react.md:100`
**File:** `C:/Users/Simon/Documents/wiki-agents/30_Concepts/workflow-vs-autonomous-agent.md:109`
**File:** `C:/Users/Simon/Documents/wiki-agents/40_Guides/agentic-design-patterns-plain-english.md:216`

**Issue:** `99_Meta/schema.md` section 3 states the vault's linking contract in absolute terms: "Every note links to its topic MOC, and the MOC links back." The MOC's "links back" mechanism is its dataview query (`WHERE contains(topic, "topic/agent-patterns") AND type != "moc"`). Task 2 of this phase's plan removed the `topic/agent-patterns` frontmatter tag from `react.md`, `workflow-vs-autonomous-agent.md`, and `agentic-design-patterns-plain-english.md` (correctly, per D-08), but left each file's existing `[[MOC - Agent Patterns]]` link in its `## See also` section untouched. The result: these three notes still link forward to `MOC - Agent Patterns`, but the MOC's dataview query no longer lists them under "All Notes in This Topic," since they lost the tag that drives that query. The schema's own reciprocal-link guarantee is now broken for exactly these three notes. A reader following the See also link from `react.md` lands on a MOC page that gives no indication `react.md` is related to it.

This is a direct, foreseeable consequence of narrowing the tag (D-08) without auditing pre-existing MOC links in the same files, and it was not caught by the plan's verification (which checked tag removal and forward-pointer presence, but not the now-orphaned MOC references).

**Fix:** Remove the stale `[[MOC - Agent Patterns]]` bullet from these three files' `## See also` sections, since the tag narrowing already established `agent-patterns-index.md` as the sole entry point for this topic — the forward pointer added by this phase supersedes the direct MOC link. For example, in `react.md`:

```diff
 ## See also

 - [[agent-patterns-index]] — consolidated pattern reference
-- [[MOC - Architectures]]
-- [[MOC - Agent Patterns]]
+- [[MOC - Architectures]]
```

Apply the equivalent removal to `workflow-vs-autonomous-agent.md` and `agentic-design-patterns-plain-english.md`.

### WR-02: Anthropic-naming crosswalk claims in the new index note are not backed by a citation in its own Sources list

**File:** `C:/Users/Simon/Documents/wiki-agents/30_Concepts/agent-patterns-index.md:26-32`

**Issue:** The `## Patterns` section makes five specific factual attribution claims to Anthropic — "(name matches Anthropic)" on Prompt Chaining, Routing, and Parallelisation; "Anthropic names the generator-plus-evaluator flavour Evaluator-Optimiser" on Reflection; "Anthropic names the lead-delegates-to-workers coordination Orchestrator-Workers" on Multi-Agent Collaboration — but the note's `## Sources` section (lines 58-61) lists only the Gulli book and the plain-English guide. The specific source that substantiates every one of these Anthropic-naming claims, `10_Sources/Blog/anthropic-building-effective-agents.md`, is not cited anywhere in this note, even though it is one of the six source files this same phase edited to point at this index. A reader (or an automated citation-resolves check per RECOMMEND-04/05) cannot trace the Anthropic-attribution claims back to a source note from within `agent-patterns-index.md` itself.

Per the project's own core-value statement (agens `CLAUDE.md`): "Every recommendation agens gives cites a specific wiki-agents note by path, not general model knowledge. That citation discipline must always hold." This index note is built to be exactly the kind of grounded lookup target that discipline depends on, so uncited factual claims inside it are a direct instance of the failure mode the project exists to prevent.

**Fix:** Add the Anthropic blog post to the `## Sources` list:

```diff
 ## Sources

 - [[10_Sources/Books/agentic-design-patterns-gulli-2025|Agentic Design Patterns (Gulli)]]
 - [[40_Guides/agentic-design-patterns-plain-english|Agentic Design Patterns — Plain English Guide]]
+- [[10_Sources/Blog/anthropic-building-effective-agents|Building Effective Agents]]
```

## Info

### IN-01: `anthropic` frontmatter field on a `type: concept` note is undocumented in schema.md

**File:** `C:/Users/Simon/Documents/wiki-agents/30_Concepts/agent-patterns-index.md:13`

**Issue:** `agent-patterns-index.md` carries `anthropic: false` in its frontmatter. `99_Meta/schema.md` section 2.6 (the field table for `concept`/`architecture`/`pattern`/`anti-pattern` notes) does not list `anthropic` as a valid field — that field is documented only under section 2.4 for `source` notes. This is not a defect introduced by this phase specifically: the plan instructed copying `best-practices-index.md`'s frontmatter verbatim, and that note (and `anti-patterns-index.md`) already carry the same undocumented field, so this note is consistent with existing vault precedent. Flagging as info because it perpetuates a schema-documentation gap rather than fixing it.

**Fix:** Out of scope for this phase to fix in isolation (would require touching two other pre-existing notes plus schema.md), but worth a follow-up: either add `anthropic` to schema.md section 2.6 as a documented optional field for index-type concept notes, or drop it from all three index notes.

---

_Reviewed: 2026-07-12T00:00:00Z_
_Reviewer: Claude (gsd-code-reviewer)_
_Depth: standard_
