# Roadmap: agens

## Overview

agens ships as a routing and grounding layer over capability that already exists, not a new system. The journey starts by collapsing the vault's six-source pattern cluster into one canonical lookup target (Phase 0), then builds the read-only recommend spine that grounds every answer in a specific vault citation (Phase 1). Delegation wiring proves the peer-handoff mechanic on a low-risk framework-fit route (Phase 2), and append-only decision logging records each recommendation in the vault's own convention (Phase 4). The gated MCP-build capability is deferred to v2, sequenced last per the read-before-write constraint once the read-only spine has earned trust.

## Phases

**Phase Numbering:**

- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [x] **Phase 0: Vault Consolidation** - Collapse the six-source `topic/agent-patterns` cluster into one canonical concept note
 (completed 2026-07-12)

- [ ] **Phase 1: agens (Read-Only Recommend)** - Citation-grounded pattern recommendation via questionnaire, dual invocation, and a runtime citation-resolves check
- [ ] **Phase 2: Delegation Wiring** - Route framework-fit questions to the GSD skills via the `Skill` tool, with explicit presence checking
- [ ] **Phase 4: agens-log (Append-Only Logging)** - Record each recommendation as an append-only, attributed entry in the vault's date-header convention

## Phase Details

### Phase 0: Vault Consolidation

**Goal**: One canonical concept note becomes the single lookup target every later phase reads from.
**Mode:** mvp
**Depends on**: Nothing (first phase)
**Requirements**: SETUP-01
**Success Criteria** (what must be TRUE):

  1. One canonical concept note exists that collapses the six `topic/agent-patterns` sources into a single lookup target.
  2. The consolidated note names each pattern in text a later citation check can match against by pattern name.
  3. The `_memory/recent.md` date-header convention (`## YYYY-MM-DD`) is confirmed as the log format agens-log will adopt, replacing PROJECT.md's non-existent `log.md` reference.

**Plans**: 1 plan

Plans:

- [x] 00-01-PLAN.md — Create the canonical agent-patterns index note, narrow the topic tag across six sources, and verify the single lookup target

### Phase 1: agens (Read-Only Recommend)

**Goal**: agens gives a citation-grounded pattern recommendation from a fixed questionnaire, invoked either by command or auto-trigger, and refuses when the vault does not support the described project.
**Mode:** mvp
**Depends on**: Phase 0
**Requirements**: RECOMMEND-01, RECOMMEND-02, RECOMMEND-03, RECOMMEND-04, RECOMMEND-05, RECOMMEND-06, RECOMMEND-07
**Success Criteria** (what must be TRUE):

  1. User invokes agens by an explicit slash command, and agens also auto-triggers when the conversation describes a new agent project in phrasings the author did not write.
  2. agens asks the fixed four-dimension questionnaire (goal, workflow shape, data sensitivity, latency/cost) before giving any recommendation.
  3. User receives a pattern recommendation that cites a resolvable wiki-agents note path with the supporting passage quoted alongside it, and agens verifies the path exists and the pattern name appears in it before returning the answer.
  4. agens refuses plainly, with no loosely related citation, when no vault note supports the described project's shape.
  5. Any note agens writes into the vault is tagged as agent-authored, distinguishing it from the user's own writing.

**Plans**: 3 plans
**UI hint**: no

Plans:
**Wave 1**

- [x] 01-01-PLAN.md — Recommend/refuse spine: dual-invocation SKILL.md, four-question questionnaire, Read+Grep citation gate, plain refusal
- [x] 01-02-PLAN.md — Define the agent-authored tagging convention for Phase 4 to enforce

**Wave 2** *(blocked on Wave 1 completion)*

- [ ] 01-03-PLAN.md — Human checkpoint: verify dual invocation, questionnaire, recommendation, and refusal live

### Phase 2: Delegation Wiring

**Goal**: agens routes framework-fit questions to the GSD skills through the `Skill` tool rather than answering inline, and fails loudly when a target skill is absent.
**Mode:** mvp
**Depends on**: Phase 1
**Requirements**: DELEGATE-01, DELEGATE-02
**Success Criteria** (what must be TRUE):

  1. User's framework-fit question is routed to `gsd-framework-selector`/`gsd-ai-integration-phase` via the `Skill` tool, never answered inline by agens.
  2. User sees an explicit, clear failure message when a delegated skill is not installed or not found, never a silent inline reimplementation of framework judgement.
  3. agens stays a router at this boundary: with the target skill absent, agens declines rather than absorbing the delegated judgement.

**Plans**: TBD

Plans:

- [ ] 02-01: TBD

### Phase 4: agens-log (Append-Only Logging)

**Goal**: Each recommendation is recorded as an append-only, attributed entry in the vault's date-header convention, callable only by agens itself.
**Mode:** mvp
**Depends on**: Phase 2
**Requirements**: LOG-01, LOG-02
**Success Criteria** (what must be TRUE):

  1. Each agens recommendation is recorded as an append-only entry in the vault's `## YYYY-MM-DD` date-header convention, and no prior entry is ever overwritten.
  2. `agens-log` cannot be triggered directly by conversation: it is called only by agens itself and never listed as a user-facing command.
  3. Read-back is bounded, each entry stamps the vault state it referenced, and the log is never treated as a grounding source for future recommendations.

**Plans**: TBD

Plans:

- [ ] 04-01: TBD

## Deferred to v2

**Phase 3: agens-build (Gated MCP Construction)** carries only BUILD-01 and BUILD-02, both v2-deferred in REQUIREMENTS.md per PROJECT.md's read-before-write sequencing constraint. It stays out of the v1 milestone. When promoted, it inserts as Phase 3, between Phase 2 and Phase 4, so the write-capable path arrives only after the read-only spine and the delegation mechanic have earned trust. Its gate must review the generated artifact (tool descriptions, network calls, shell-outs, version pins), not just the intent to build, and reject unpinned `@latest` versions.

## Standing Check (all phases)

Monolith drift is a recurring phase-exit gate, not a single phase: at every phase merge, confirm agens stays a router and a grounding layer, not a reimplementation of the pattern taxonomy or framework-selection logic it exists to delegate.

## Progress

**Execution Order:**
Phases execute in numeric order: 0 → 1 → 2 → 4

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 0. Vault Consolidation | 1/1 | Complete   | 2026-07-12 |
| 1. agens (Read-Only Recommend) | 2/3 | In Progress|  |
| 2. Delegation Wiring | 0/TBD | Not started | - |
| 4. agens-log (Append-Only Logging) | 0/TBD | Not started | - |
