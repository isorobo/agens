# agens

## What This Is

agens is a Claude Code Skill (or small Skill family) that routes agent-project requests to existing capabilities instead of reimplementing them. It reads the wiki-agents Obsidian vault for agent-pattern and architecture knowledge, delegates framework-fit questions to `gsd-framework-selector`/`gsd-ai-integration-phase`, and delegates MCP server construction to Anthropic's official MCP Builder skill under a mandatory human-approval gate. It is invoked both as an explicit slash command and as an auto-triggered Skill.

## Core Value

Every recommendation agens gives cites a specific wiki-agents note by path, not general model knowledge. That citation discipline must always hold, or agens degrades into asking Claude cold with extra steps.

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] User can invoke agens via an explicit slash command
- [ ] User can invoke agens via auto-trigger (Skill description matches natural project-starting language)
- [ ] agens asks a fixed questionnaire (goal, workflow shape, data sensitivity, latency/cost) for a described project
- [ ] agens recommends an agent pattern grounded in a specific wiki-agents citation
- [ ] agens offers, but does not auto-run, MCP Builder when tooling is identified as a need
- [ ] agens delegates framework-fit questions to `gsd-framework-selector`/`gsd-ai-integration-phase` rather than reimplementing that judgement
- [ ] agens logs each recommendation decision append-only, mirroring the vault's own LLM-wiki `log.md` pattern
- [ ] wiki-agents' `topic/agent-patterns` cluster (6 sources) is consolidated into one canonical concept note as agens' single lookup target

### Out of Scope

- Standalone application or new agent runtime — agens is a Skill inside the existing Claude Code / Anthropic ecosystem, not new infrastructure.
- Autonomous MCP server construction — MCP Builder only runs behind an explicit human approval gate; agens never builds a server unreviewed.
- Reimplementing pattern taxonomy or framework-selection logic already covered by wiki-agents sources or the GSD skill family.
- Multi-user or shared-product features — agens is a personal tool for the user's own agent-development practice.

## Context

- Built on top of the wiki-agents Obsidian vault (`C:\Users\Simon\Documents\wiki-agents`), which already documents roughly 148 notes across 26 MOCs, including a pattern taxonomy (Gulli's 21 patterns, Anthropic's Building Effective Agents workflow/agent split, the agentic AI handbook) and a graduated `topic/agent-skills` cluster (Anthropic's Claude Code Skills playbook, `google/skills`, `last30days-skill`, `davidondrej/skills`, MCP Builder, Claude API Skill).
- The user has committed to specialising in the Anthropic ecosystem (a "one tool a week" learning strategy that landed on the Claude Agent SDK and Skills).
- The user already runs a GSD skill family (`gsd-framework-selector`, `gsd-ai-integration-phase`, and others) for general software-project phase planning. agens is scoped specifically to agent-pattern selection, not a replacement for that family.
- The prior strategic discussion that produced this project concluded agens substantially overlaps with three existing things — Anthropic's Skills ecosystem, the user's own GSD skill family, and published pattern taxonomies already in the vault — and that the genuinely bespoke work is the connective, routing layer between them, not a new system.
- A candidate reference to a Boris Cherny "five levels" framework was raised during the strategic discussion and later withdrawn by the user as a mistake. Out of scope; do not chase it.

## Constraints

- **Ecosystem**: Anthropic Claude Agent SDK / Claude Code Skills only — no LangChain, CrewAI, or other framework as agens' own implementation substrate.
- **Delegation discipline**: MCP construction and framework-selection logic must delegate to existing tools (MCP Builder, `gsd-framework-selector`) rather than be reimplemented inside agens.
- **Human approval gate**: Any capability that writes or builds (MCP server construction) requires explicit human approval before executing — a direct mitigation against the self-mutation/rug-pull risk pattern documented in wiki-agents' MCP Security note.
- **Citation discipline**: Every pattern recommendation must reference a specific wiki-agents note by path.
- **Sequencing**: Read-only capabilities (interrogate, recommend) ship and earn trust before the write capability (build MCP) is added.

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Build as a Claude Code Skill, not a standalone system | The Anthropic ecosystem already provides the runtime, session, and tool-calling machinery a bespoke system would otherwise duplicate | — Pending |
| Split "recommend" (read-only) from "build" (write) into two gated capabilities | An agent that builds its own tools and trusts them unreviewed is the exact self-mutation/rug-pull pattern the vault's own MCP Security note warns against | — Pending |
| Delegate framework-fit questions to `gsd-framework-selector`/`gsd-ai-integration-phase` rather than reimplementing | Avoids reproducing existing, already-installed GSD capability | — Pending |
| Invoke agens via both slash command and auto-trigger | User confirmed both matter: explicit control when wanted, no command to remember for casual mentions | — Pending |
| Consolidate the vault's 6-source `topic/agent-patterns` cluster into one canonical concept note in Phase 0 | User decided now rather than deferring to research — gives agens one lookup target instead of six | — Pending |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd-transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd:complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-07-11 after initialization*
