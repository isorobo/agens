# agens

A Claude Code Skill that turns an agent-knowledge Obsidian vault into a working design assistant — one that recommends agent architectures, delegates framework choices, and builds tooling on request, always citing the vault note behind its answer.

## The problem

Agent design moves fast. Papers, courses, and framework releases outpace anyone's ability to read them all, and a new project rarely starts from a blank map — it starts from whatever pattern you half-remember from three sources ago. agens exists to close that gap: point it at a curated vault, describe a project, and it answers from the vault instead of from a model's general training data.

## What agens does

1. **Interrogates.** A short, fixed questionnaire — project goal, expected workflow shape, data sensitivity, latency and cost constraints — replaces an open-ended "tell me about your idea" with the same four questions every time.
2. **Recommends.** It matches the answers against the vault's own pattern taxonomy and returns a specific recommendation with a citation, not a paraphrase. If the vault says nothing relevant, agens says so, rather than filling the gap from general knowledge.
3. **Delegates.** Framework-fit questions go to whatever framework-selection tooling the host project already has. MCP server construction goes to Anthropic's own MCP Builder skill. agens does not reimplement either.
4. **Builds, on approval only.** The one capability that writes anything — constructing a bespoke MCP server — sits behind an explicit human approval gate. agens never builds a tool and trusts it unreviewed.
5. **Remembers.** Every recommendation is logged append-only, mirroring the same log-file pattern the vault itself uses to stay current. The tool that reads the vault also feeds it.

## Why it is not another agent framework

agens is deliberately thin. It does not run agents, does not manage tool execution, and does not ship its own reasoning loop — Claude Code already provides all three. The two temptations a project like this usually falls into are reinventing infrastructure that already exists (a new agent runtime, a new MCP toolchain) and letting a "smart assistant" quietly become an unreviewed system that edits its own capabilities. agens is scoped against both: it is a routing layer over existing tools, not a new one, and its one write capability is gated, not autonomous.

## How it works

```
Your project description
        │
        ▼
  Fixed questionnaire (goal · workflow shape · data sensitivity · latency/cost)
        │
        ▼
  Vault lookup — grep/search across your topic-tagged pattern notes
        │
        ├─► Pattern recommendation, cited by file path
        ├─► Framework-fit question → delegated to your framework-selection tooling
        └─► MCP tooling need identified → offered, not run → your approval → MCP Builder
        │
        ▼
  Decision logged back to the vault, append-only
```

agens assumes a vault with a controlled topic vocabulary and one note per source or concept — the shape this project's own reference vault, `wiki-agents`, already follows. It is not tied to that specific vault's content; any Obsidian vault built on the same pattern (source notes with frontmatter topics, a pattern-taxonomy concept note, MOCs for navigation) should work.

## Status

Early. This repository currently holds the project's planning documents — `PROJECT.md` and a working `config.json` — produced through a structured discussion process. Phase 0 is complete; the Skill itself has not been built yet. The v1 milestone executes in this order: 0 → 1 → 2 → 4.

| Phase | Goal | Status |
|---|---|---|
| 0 | Consolidate the vault's pattern-taxonomy sources into one canonical lookup target | Complete |
| 1 | Define the fixed questionnaire and the citation-backed recommendation format | Complete |
| 2 | Wire delegation to existing framework-selection tooling and MCP Builder — no reimplementation | In progress |
| 4 | Close the loop — agens logs its own decisions back into the vault | Not started |

**Phase 3 (agens-build, the gated MCP-build capability) is deferred to v2.** It is not part of this milestone's execution order. When promoted, it inserts between Phase 2 and Phase 4, so the one write-capable path lands only after the read-only spine and the delegation mechanic have earned trust.

### Proposed: execution-substrate guidance

The vault  documents Claude Code dynamic workflows as a run-time. A future change may surface a cited substrate line in agens recommendations. agens' own machinery stays interactive and never runs as a workflow.

## Design principles

- **One Skill, one job.** Interrogation, recommendation, delegation, and building stay separable, not one monolithic prompt.
- **Cite or say nothing.** A recommendation without a source-note citation is not a recommendation agens should give.
- **Read before write.** The recommendation half ships and earns trust before the build half is added.
- **Human approval on every write.** Building a bespoke MCP server never happens unreviewed — the same discipline the wider MCP ecosystem names as the defence against a tool that turns on its own user.
- **Delegate, don't duplicate.** If a capability already exists elsewhere in the ecosystem, agens routes to it instead of rebuilding it.

## License

Licenced under MIT , matching the wider Agent Skills ecosystem this project builds on.
---

This project is early and opinionated, built for one person's vault first — you are welcome to clone, fork, and use this repo in your vaults.
