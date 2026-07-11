# Requirements — agens

## v1 Requirements

### Vault Consolidation

- [ ] **SETUP-01**: User's recommendations are grounded against one canonical concept note, not six scattered pattern sources — the wiki-agents `topic/agent-patterns` cluster is consolidated into a single note before any other phase depends on it.

### Recommend (read-only core)

- [ ] **RECOMMEND-01**: User can invoke agens via an explicit slash command.
- [ ] **RECOMMEND-02**: User can invoke agens via auto-trigger, without a command, when conversation naturally describes a new agent project.
- [ ] **RECOMMEND-03**: User answers a fixed four-dimension questionnaire (goal, workflow shape, data sensitivity, latency/cost) before agens gives any recommendation.
- [ ] **RECOMMEND-04**: User receives a pattern recommendation that cites a specific, resolvable wiki-agents note path, with the supporting passage quoted alongside it.
- [ ] **RECOMMEND-05**: agens verifies the cited path exists and the recommended pattern actually appears in it before returning the recommendation to the user.
- [ ] **RECOMMEND-06**: agens refuses to recommend, and says so plainly, when no vault note supports the described project's shape — it does not paper over the gap with a loosely related citation.
- [ ] **RECOMMEND-07**: Every note agens writes into the vault is tagged as agent-authored, distinguishing it from the user's own writing.

### Delegate

- [ ] **DELEGATE-01**: User's framework-fit questions are routed to `gsd-framework-selector`/`gsd-ai-integration-phase` via the `Skill` tool rather than answered inline by agens.
- [ ] **DELEGATE-02**: User sees an explicit, clear failure message — never a silent inline reimplementation — when a delegated Skill is not installed or not found.

### Log

- [ ] **LOG-01**: Each agens recommendation is recorded as an append-only entry in the vault, in the vault's existing date-header log convention, and no prior entry is ever overwritten.
- [ ] **LOG-02**: `agens-log` cannot be triggered directly by conversation — it is called only by `agens` itself, never listed as a user-facing command.

## v2 Requirements (Deferred)

<!-- v1.x tier: add once the v1 spine is shipped and trusted. -->

- [ ] **RECOMMEND-08**: Recommendations preserve qualifiers from the source note (for example "only for low-latency, stateless workflows") rather than flattening them into a blanket endorsement.
- [ ] **RECOMMEND-09**: Passage-level quoting is refined so the user can verify a claim from the citation alone, without hunting through the full note.
- [ ] **LOG-03**: Each log entry links back to its cited note as a wikilink, turning the log into a reviewable grounding history.

<!-- v2+ tier: the gated write capability, deferred until the read-only spine has earned trust per PROJECT.md's sequencing constraint. -->

- [ ] **BUILD-01**: User is offered MCP Builder when a tooling need is identified during a recommendation — agens never auto-runs it.
- [ ] **BUILD-02**: Before any MCP server is installed, the user reviews the actual generated artifact (tool descriptions, network calls, shell-outs, version pins) — not just the initial intent to build — and unpinned `@latest` versions are rejected.

## Out of Scope

- **Standalone runtime or new agent infrastructure** — duplicates the Claude Code session, tool-calling, and runtime the ecosystem already provides.
- **Autonomous MCP server construction** — the self-mutation/rug-pull pattern the vault's own MCP Security note warns against; BUILD-01/02 exist specifically to prevent this.
- **Multi-user or shared-product features** — agens is a personal, single-user practice tool.
- **Reimplementing the pattern taxonomy or framework-selection logic** — duplicates wiki-agents sources and the GSD family; the copy would drift from the source it exists to ground against.
- **Vector-embedding RAG pipeline over the vault** — overkill at 152 notes / 547 KB; reintroduces the grounding-failure modes citation verification exists to catch. Revisit only if the vault grows past a few hundred pages (Karpathy's own stated threshold, already documented in wiki-agents).
- **Free-text, open-ended intake in place of the fixed questionnaire** — loses the structured inputs the lookup needs and makes recommendations non-reproducible.
- **Confidence scores or star ratings on recommendations** — invites an "illusion of groundedness" that masks whether the citation actually supports the claim.
- **Auto-rewriting or "healing" existing vault notes** — agens reads human-authored knowledge; rewriting it destroys the attribution boundary RECOMMEND-07/LOG-02 exist to protect.
- **Orchestrating a wide fleet of sub-skills up front** — agens has exactly two named delegations (GSD, MCP Builder); a general-purpose orchestrator is premature and out of scope until proven necessary.
- **Citing multiple notes per recommendation "to look thorough"** — padding with tangential citations is the decorative-citation failure RECOMMEND-04/05/06 exist to prevent.

## Traceability

<!-- Filled by the roadmapper — maps each REQ-ID to the phase that delivers it. -->

---
*Requirements defined: 2026-07-11*
