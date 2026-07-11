# Project Research Summary

**Project:** agens
**Domain:** Claude Code Skill family - vault-grounded agent-architecture advisor with gated delegation
**Researched:** 2026-07-11
**Confidence:** HIGH

## Executive Summary

agens has no runtime in the conventional sense. It is a small family of Claude Code Skills - markdown files with YAML frontmatter, interpreted by the Claude Code host - that reads the wiki-agents Obsidian vault (152 notes, 547 KB), grounds every recommendation in a specific note citation, and delegates rather than reimplements two neighbouring capabilities: framework-fit judgement (to `gsd-framework-selector`/`gsd-ai-integration-phase`) and MCP server construction (to Anthropic's `mcp-builder`, behind a mandatory human-approval gate). All four researchers converge on the same shape: agens is a routing and grounding layer over capability that already exists, not a new system. At this vault size, Grep and Glob against the filesystem are the correct retrieval mechanism; no embedding index or vector database is warranted now or for a long time.

The recommended approach is a three-Skill family, not one monolith: `agens` (read-only, both slash-command and auto-triggerable), `agens-build` (write-capable, `disable-model-invocation: true`, manual-only, wraps the approval gate before invoking `mcp-builder`), and `agens-log` (append-only, `user-invocable: false`, writes one scoped log file). This split is a runtime requirement, not a style preference: a single `SKILL.md` cannot hold two different invocation policies (auto-triggerable read versus manual-gated write) at once. The three-way split also resolves an open question the Stack researcher raised about whether vault-logging is a second, unaccounted-for write surface: it is, and `agens-log` is the deliberate, narrowly-scoped answer to it - a second gated component alongside `agens-build`, not a loophole in the "one write capability" framing. Each Skill maps to a phase in PROJECT.md's plan, with Phase 0 (consolidating the vault's 6-source pattern cluster into one canonical concept note) as the load-bearing prerequisite every later phase reads from.

The dominant risk is not technical difficulty but discipline erosion under the pressure of "just ship it." Citations can look intact while the substance drifts to general model knowledge (grounding drift); a Skill can auto-trigger unreliably because its `description` reads like documentation instead of trigger phrasing; a human-approval gate can protect the decision to build while leaving the generated artifact itself unreviewed; and agens can slowly absorb the very judgement it was built to delegate, becoming the monolith it exists to avoid. None of these fail loudly during development - they pass a happy-path check and then decay in production. The mitigation for all four is the same discipline: verify, do not assume - verify the citation resolves and the note supports the claim, verify auto-trigger against phrasings the author did not write, verify the artifact (not just the intent) before install, and verify scope against PROJECT.md's Out-of-Scope list at every phase merge.

## Key Findings

### Recommended Stack

agens is markdown files, not a package-managed application. The "stack" is a set of Claude Code Skill conventions: YAML frontmatter fields (`description`, `disable-model-invocation`, `user-invocable`, `allowed-tools`) and four built-in tools - `Grep`/`Glob` for vault search, `Read` for citation-backed note loading, `Skill` for cross-Skill delegation, and `AskUserQuestion` for the fixed questionnaire and the approval gate. No npm or pip dependency exists anywhere in the design.

**Core technologies:**
- Claude Code Skills (`SKILL.md`, v2.1.196+) - the substrate; directory name becomes the `/command`, body loads only when triggered
- Built-in `Grep`/`Glob` - sufficient and correct retrieval at 547 KB / 152 notes; no index needed until the corpus reaches thousands of notes
- Built-in `Skill` tool - the sanctioned mechanism for agens to invoke `gsd-framework-selector` and `mcp-builder` without reimplementing their logic
- `AskUserQuestion` - the fixed four-dimension questionnaire and the mandatory approval gate before any MCP build
- Frontmatter `disable-model-invocation: true` - the structural (not merely instructional) guarantee that a write-capable Skill cannot auto-fire

One version-compatibility risk: `mcp-builder` does not set `disable-model-invocation` itself and will auto-trigger unprompted on phrases like "build an MCP server." agens must enforce the gate itself before handing off - the gate is not inherited.

### Expected Features

Prior art (agent-only Obsidian vaults, citation-backed RAG literature, Anthropic's own Skill-authoring guidance) converges on one theme: the citation must be load-bearing, not decorative. Every table-stakes and differentiator feature exists to make that true.

**Must have (table stakes):**
- Fixed four-dimension questionnaire (goal, workflow shape, data sensitivity, latency/cost) before any recommendation
- Recommendation cites a specific vault note by path - the Core Value
- Canonical concept note (Phase 0) as the single lookup target
- Append-only decision log mirroring the vault's log convention, with author attribution
- Delegate framework-fit questions rather than answering them inline
- Offer `mcp-builder` but never auto-run it
- Both slash-command and auto-trigger invocation

**Should have (competitive differentiators):**
- Citation-resolves pre-emit check - confirm the path exists and the note actually contains the claimed pattern before returning it
- Refuse-when-unsupported - disciplined refusal beats a fluent but weakly-related citation
- Quote the supporting passage, not just the path
- Preserve qualifiers from the source note (do not flatten "only for low-latency workflows" into a blanket endorsement)
- Log entries link back to the cited note as a wikilink

**Defer (v2+):**
- MCP Builder handoff behind the approval gate - sequenced last per PROJECT.md; the read-only spine must earn trust first
- Plan-validate-execute intermediate output before any build - only relevant once the gated build exists
- Confidence scores or star ratings - explicitly rejected; they create an "illusion of groundedness" that masks whether a citation actually supports the claim
- Vector-embedding RAG pipeline - explicitly rejected at this vault size; overkill and reintroduces the grounding-failure modes citation-verification exists to catch

### Architecture Approach

agens is a thin orchestrator over a deterministic scripting layer and the vault's files, split into three Skills by trust boundary rather than by feature. `agens` (read-only, `allowed-tools: Read Grep Glob AskUserQuestion`) runs the questionnaire and recommend flow and can auto-trigger safely. `agens-build` (`disable-model-invocation: true`) wraps the approval gate and invokes `mcp-builder` only on explicit `/agens-build`. `agens-log` (`user-invocable: false`) is a callable helper, not a menu command, that performs one scoped, atomic append. Each Skill calls Python scripts (`vault_search.py`, `read_note.py`, `append_log.py`) for mechanical file work; the model reasons over the JSON the scripts return rather than walking the filesystem or holding the vault in context.

**Major components:**
1. `agens` (recommend) - runs the questionnaire, searches the vault via `vault_search.py`, reads the canonical concept note, emits a citation-backed recommendation
2. `agens-build` (gated MCP) - offers, then on explicit approval delegates MCP construction to `mcp-builder` via the `Skill` tool; never writes server code itself
3. `agens-log` (append-only) - appends one decision record per recommendation to the vault log, atomically, never rewriting prior entries
4. Canonical concept note (Phase 0 artifact) - the single consolidated lookup target that both `vault_search.py` and the grounding-drift check depend on
5. Deterministic script layer - `vault_search.py`, `read_note.py`, `append_log.py`; all file I/O is scripted and reproducible, never ad hoc

### Critical Pitfalls

1. **Trigger description written for a human, not as a trigger signal** - a description that summarises ("recommends agent architectures") instead of listing concrete trigger conditions ("Use when the user describes a new agent project...") causes near-zero auto-trigger. Write third-person, condition-listing descriptions and test against a phrasing set the author did not write, not the happy path.
2. **Grounding drift - citations quietly stop matching the answer** - the visible contract (a path appears) is satisfied long after the invisible one (the note supports the claim) breaks. Retrieve the note before recommending, verify the path resolves and the pattern name appears in it, and refuse rather than staple a plausible citation onto a general-knowledge answer.
3. **Self-maintained decision log rots** - unbounded, unsummarised, and self-contradicting over many sessions. Cap read-back, never treat the log itself as a grounding source, and stamp each entry with the vault state it referenced so staleness is visible.
4. **Silent delegation failure when a delegated Skill is absent or its interface changed** - Skill-to-Skill reference by name has no dependency resolver; a missing target fails silently and agens may reimplement the judgement inline instead, which is the exact anti-goal PROJECT.md forbids. Check presence, fail loudly and explicitly, never substitute.
5. **The approval gate protects the decision but not the artifact** - approving "build an MCP server for X" is not the same as reviewing what `mcp-builder` actually generates (tool descriptions, network calls, shell-outs, `@latest` pins). Gate the artifact, not just the intent: surface the generated content for review and reject `@latest`.

## Cross-Cutting Findings Requiring Resolution

Two findings surfaced independently across research files and must be resolved before Phase 1/4 implementation, not left as loose threads.

### 1. PROJECT.md's `log.md` reference does not point to a real file

Both the Stack and Pitfalls researchers independently confirmed: no file named `log.md` exists anywhere in the wiki-agents vault. The closest analogue is `_memory/recent.md`, which uses `## YYYY-MM-DD` date headers. PROJECT.md's requirement to mirror "the vault's own log.md pattern" describes a documented convention referenced in a source note (the LLM-wiki source/concept/log workflow), not an actual file agens can point to or copy the format of by inspection.

**Resolution:** treat "the vault's `log.md` pattern" as a convention to imitate, not a file to locate. `agens-log` should adopt the `_memory/recent.md` header style (`## YYYY-MM-DD`) for consistency with what the vault actually contains on disk, and PROJECT.md's Active requirement should be understood as satisfied by that adopted convention rather than by any literal `log.md` file. This should be confirmed explicitly in Phase 0 or Phase 1 planning, before `agens-log` is built in Phase 4, so the convention is not invented ad hoc mid-implementation.

### 2. The three-Skill split resolves the "second write surface" question

The Stack researcher flagged an open question: does vault-logging constitute a second, unaccounted-for write capability beyond PROJECT.md's framing of "one write capability" (the gated MCP build)? The Architecture researcher's independent finding answers this directly. A single `SKILL.md` cannot hold two different invocation policies at once - `agens` must stay auto-triggerable and read-only, while any write path needs its own gating. This forces a split into (at least) `agens` and a write-capable sibling, and the write-capable work itself splits further by risk profile: `agens-build` (high-risk, external code generation, `disable-model-invocation: true`, manual-only) and `agens-log` (low-risk, single-file scoped append, `user-invocable: false`, callable only as a helper).

**Resolution:** PROJECT.md's "one write capability" framing should be understood, going into roadmap creation, as two deliberately separate gated components rather than one. This is not scope creep - it is the correct decomposition of the single write concern PROJECT.md already named, split by trust boundary because the runtime cannot express two invocation policies in one Skill. The roadmapper should reflect this as Phase 3 (`agens-build`) and Phase 4 (`agens-log`) remaining distinct phases, each with its own frontmatter gating, rather than merging log-writing into the build phase.

## Implications for Roadmap

Based on combined research, the five-phase structure already sketched in PROJECT.md is sound and independently reconstructed by the Architecture researcher. Research confirms the ordering and adds specific verification gates per phase.

### Phase 0: Vault Consolidation
**Rationale:** Every later phase - search, citation, grounding-drift prevention - depends on one canonical lookup target. Consolidating the 6-source `topic/agent-patterns` cluster into a single concept note is the prerequisite, not an optional cleanup step.
**Delivers:** One canonical concept note collapsing the 6-source pattern cluster.
**Addresses:** FEATURES.md's "canonical concept note as single lookup target" (P1, table stakes).
**Avoids:** Pitfall 2 (grounding drift) - a single, known-current lookup target is what keeps a citation check honest; six scattered sources cannot be kept in sync.

### Phase 1: agens (Read-Only Recommend)
**Rationale:** PROJECT.md's hard sequencing constraint: read-only capabilities ship and earn trust before any write path exists. This is also where the auto-trigger and citation disciplines - the two hardest-to-verify requirements - must be proven before anything else is layered on.
**Delivers:** `agens` Skill with fixed questionnaire, `vault_search.py`/`read_note.py`, citation-backed recommendation, both slash-command and auto-trigger invocation.
**Addresses:** FEATURES.md P1 set in full (questionnaire, resolving citation, citation-resolves check, refuse-when-unsupported, append-only log stub, dual invocation).
**Avoids:** Pitfall 1 (trigger description mismatch - verify against an independent phrasing set, not the author's own wording) and Pitfall 2 (grounding drift - retrieve-then-recommend with a runtime citation-resolves-and-matches check as a named success criterion).

### Phase 2: Delegation Wiring (Framework-Fit)
**Rationale:** Proves the peer-handoff mechanic (the `Skill` tool) on a low-risk delegation before the same mechanic carries the higher-stakes MCP-build handoff in Phase 3. No new write capability is introduced here.
**Delivers:** agens routes framework-fit questions to `gsd-framework-selector`/`gsd-ai-integration-phase` via the `Skill` tool, with explicit presence checking.
**Uses:** The `Skill` tool delegation pattern from ARCHITECTURE.md; both target Skills already installed locally.
**Implements:** ARCHITECTURE.md Pattern 2 (delegation through the Skill tool, not reimplementation).
**Avoids:** Pitfall 4 (silent delegation failure) - test agens with the target Skill absent and confirm an explicit, correct failure message rather than a silent inline reimplementation of framework judgement.

### Phase 3: agens-build (Gated MCP Construction)
**Rationale:** The first write/build capability, and the highest-severity pitfall in the set (Pitfall 5) attaches here. Deliberately sequenced after the delegation mechanic is proven safe in Phase 2 and the read-only spine has earned trust in Phase 1.
**Delivers:** `agens-build` Skill (`disable-model-invocation: true`, manual-only) that offers, gates on explicit `AskUserQuestion` approval, and only then invokes `mcp-builder` via the `Skill` tool.
**Uses:** `AskUserQuestion` (STACK.md); `disable-model-invocation: true` frontmatter (ARCHITECTURE.md Pattern 3).
**Implements:** The human-approval gate as a structural frontmatter guarantee, not prose alone.
**Avoids:** Pitfall 5 (approval gate protects intent, not artifact) - the gate must render the generated tool descriptions, network calls, and shell-outs for review, and reject `@latest` in favour of pinned versions, before install.

### Phase 4: agens-log (Append-Only Decision Logging)
**Rationale:** Depends on recommendations existing to log (Phase 1) and delegation decisions worth recording (Phases 2-3). Lowest-risk of the two write-capable additions - a single scoped file, no external code generation - so it is safe to add last even though it is a write capability.
**Delivers:** `agens-log` Skill (`user-invocable: false`, callable helper only) with `append_log.py` performing atomic, bounded-read-back appends in the `_memory/recent.md`-style date-header convention (see Cross-Cutting Finding 1).
**Addresses:** FEATURES.md's append-only log with author attribution (P1) and log-links-to-citation (P2).
**Avoids:** Pitfall 3 (log rot) - cap read-back to most-recent-N or a rolling summary, stamp each entry with the vault state it referenced, and never treat the log itself as a grounding source for future recommendations.

### Phase Ordering Rationale

- **Dependency chain:** canonical note (Phase 0) -> read/recommend (Phase 1) -> low-risk delegation (Phase 2) -> gated high-risk write (Phase 3) -> scoped low-risk write (Phase 4). Each phase's architecture depends on the one before it; none can be reordered without breaking a dependency FEATURES.md or ARCHITECTURE.md names explicitly.
- **Grouping by trust boundary, not by feature:** the three-Skill split (agens / agens-build / agens-log) exists because a single `SKILL.md` cannot carry two invocation policies at once. Phases 1, 3, and 4 each produce one Skill in that family, in ascending order of write-risk.
- **Pitfall avoidance drove the read-before-write sequencing already in PROJECT.md:** Pitfall 5 (gate-protects-intent-not-artifact) is the costliest failure to recover from (HIGH recovery cost per PITFALLS.md), which is why it is isolated to its own phase (3) with its own named verification, rather than folded into Phase 1 or 4.
- **Pitfall 6 (monolith drift) is a standing check across every phase**, not a single phase - the roadmapper should treat "does this stay a router, not a reimplementation" as a recurring phase-exit gate, sharpest in Phases 2-4 as delegated capabilities are wired.

### Research Flags

Needs research during planning:
- **Phase 3 (agens-build):** MCP Builder's actual output shape (generated tool descriptions, install mechanism, version-pinning behaviour) should be inspected directly before writing the artifact-review gate - PITFALLS.md's five-step audit is a template, not a verified checklist against this specific tool's current output.
- **Phase 1 (agens recommend):** the auto-trigger phrasing test set needs to be constructed deliberately (a list of realistic project-starting phrasings independent of the description's own wording) - this is a verification design task, not a coding task, and should not be skipped as "obviously fine."

Phases with standard, well-documented patterns (research-phase not needed):
- **Phase 0 (vault consolidation):** straightforward content-authoring task with a clear source set (6 notes) and target (1 canonical note); no novel technical risk.
- **Phase 2 (framework-fit delegation):** the `Skill` tool delegation mechanic is already verified against official docs and both target Skills are confirmed installed; this is wiring, not discovery.
- **Phase 4 (agens-log):** atomic file append is a well-understood pattern (`Bash >>` or read-then-write), and the convention question (Cross-Cutting Finding 1) is resolved by this summary rather than left open for that phase.

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | Verified against official Claude Code Skills docs (retrieved 2026-07-11) and the canonical `mcp-builder` repo; cross-checked against two real on-disk Skills and direct inspection of the target vault. |
| Features | MEDIUM-HIGH | Anthropic Skill best practices and RAG citation-grounding research are HIGH confidence; direct prior-art tools for this exact combination (vault-grounded advisor + delegation + gated build) are MEDIUM, drawn from two close analogues plus community patterns rather than an identical precedent. |
| Architecture | HIGH | Verified against official Skills docs frontmatter reference and the Skill tool mechanism; cross-checked against a production Skill (`wiki`) operating on the same vault and the same deterministic-script/progressive-disclosure pattern. |
| Pitfalls | HIGH | Grounded in two wiki-agents source notes agens itself depends on (MCP security attack vectors, Skill authoring lessons) plus current Anthropic Skills documentation; pitfalls are traced to specific, named mechanisms rather than generic risk statements. |

**Overall confidence:** HIGH

### Gaps to Address

- **The `log.md` naming gap (Cross-Cutting Finding 1):** resolved in this summary as "adopt the `_memory/recent.md` convention," but this resolution should be confirmed explicitly, not assumed, when Phase 0 or Phase 1 planning begins - it is a naming/convention decision, not a technical unknown, and costs nothing to confirm early.
- **`mcp-builder`'s actual generated-artifact shape:** PITFALLS.md's artifact-review gate (Pitfall 5) is designed from the vault's general MCP-security note, not from direct inspection of what `mcp-builder` currently outputs. Phase 3 planning should inspect a real `mcp-builder` run before finalising the review checklist.
- **Whether grep-only search will miss semantically-close vault notes:** ARCHITECTURE.md recommends starting with glob+grep and only adding synonym-expansion or a local index if Phase 1 evaluation shows misses. This is a testable question, not a design uncertainty, and should be checked empirically during Phase 1 rather than pre-decided.

## Sources

### Primary (HIGH confidence)
- `code.claude.com/docs/en/skills` - SKILL.md frontmatter reference, invocation control, the Skill tool, path substitutions, version gates. Retrieved 2026-07-11.
- `github.com/anthropics/claude-plugins-official` (`mcp-server-dev` plugin, `build-mcp-server` skill) - canonical MCP Builder identity, invocation, absence of `disable-model-invocation`.
- `platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices` - progressive disclosure, research-synthesis workflow, plan-validate-execute gate, auto-trigger description craft.
- Local inspection of `C:/Users/Simon/Documents/wiki-agents` - 152 notes, 547 KB, folder taxonomy, frontmatter fields, `_memory/recent.md` log style, `.smart-env` index presence.
- Local inspection of `~/.claude/skills/wiki/SKILL.md`, `gsd-ai-integration-phase/SKILL.md`, `find-skills/SKILL.md` - production reference implementations on the same vault.
- wiki-agents: `10_Sources/Blog/mcp-security-attack-vectors.md` - rug pulls, tool-description poisoning, five-step pre-install audit.
- wiki-agents: `10_Sources/Blog/agent-skills-best-practices.md` - description-as-trigger-signal, append-only log memory, silent Skill-dependency failure.

### Secondary (MEDIUM confidence)
- Agent-only Obsidian vault ("foundry" source/concept/log workflow): thethinkers.club - decision-logging and attribution-confusion prior art.
- `obsidian-second-brain` Skill (GitHub) - decision logging, provenance, write-safety confirmation patterns.
- RAG grounding test literature (Medium, haruiz.github.io, yaihq.com) - citation-relevance-swap, qualifier-preservation, illusion-of-groundedness findings.
- Orchestrator Skill / delegation pattern guidance (mindstudio.ai) - "don't orchestrate too early."

### Tertiary (LOW confidence)
- None flagged - all research files rated findings HIGH or MEDIUM-HIGH; no tertiary-only claims carried into this summary.

---
*Research completed: 2026-07-11*
*Ready for roadmap: yes*
