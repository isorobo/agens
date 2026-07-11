# Architecture Research

**Domain:** Claude Code Skill family — vault-grounded agent-architecture recommender with gated delegation
**Researched:** 2026-07-11
**Confidence:** HIGH

## Standard Architecture

agens is not a service or an application. It is a Skill family that lives inside the Claude Code runtime. The runtime already supplies the loop, the tool-calling machinery, and the session. agens supplies a citation-disciplined workflow and the connective routing between existing capabilities. The architecture below reflects that: thin Skills that orchestrate, deterministic scripts that do file work, and delegation to installed Skills through the Skill tool.

### System Overview

```
┌──────────────────────────────────────────────────────────────────────┐
│                       Claude Code runtime (host)                       │
│   Skill listing (names + descriptions) · Skill tool · permissions      │
├──────────────────────────────────────────────────────────────────────┤
│                         agens Skill family                             │
│  ┌────────────────┐   ┌────────────────┐   ┌────────────────┐          │
│  │ agens          │   │ agens-build    │   │ agens-log      │          │
│  │ (recommend)    │──▶│ (gated MCP)    │   │ (append-only)  │          │
│  │ read-only      │   │ write, /manual │   │ write, scoped  │          │
│  └───────┬────────┘   └───────┬────────┘   └───────┬────────┘          │
│          │                    │                    │                   │
├──────────┼────────────────────┼────────────────────┼───────────────────┤
│          │  invokes via Skill tool / spawns via context:fork           │
│          ▼                    ▼                    ▼                   │
│  ┌──────────────┐   ┌──────────────────┐   ┌──────────────────┐        │
│  │ Skill tool → │   │ Skill tool →     │   │ (no delegation)  │        │
│  │ gsd-frame-   │   │ mcp-builder      │   │ writes log file  │        │
│  │ work-selector│   │ (Anthropic)      │   │                  │        │
│  └──────────────┘   └──────────────────┘   └──────────────────┘        │
├──────────────────────────────────────────────────────────────────────┤
│                    Deterministic layer (scripts)                       │
│  ┌──────────────────┐   ┌──────────────────┐   ┌──────────────────┐    │
│  │ vault_search.py  │   │ read_note.py     │   │ append_log.py    │    │
│  │ grep/glob/index  │   │ frontmatter+body │   │ atomic append    │    │
│  └────────┬─────────┘   └────────┬─────────┘   └────────┬─────────┘    │
├───────────┼──────────────────────┼──────────────────────┼──────────────┤
│                         Data stores (files)                            │
│  ┌──────────────────┐   ┌──────────────────┐   ┌──────────────────┐    │
│  │ wiki-agents vault│   │ canonical concept│   │ decision log.md  │    │
│  │ 148 notes/26 MOC │   │ note (Phase 0)   │   │ (append-only)    │    │
│  │ .smart-env index │   │ single lookup    │   │ mirrors LLM-wiki │    │
│  └──────────────────┘   └──────────────────┘   └──────────────────┘    │
└──────────────────────────────────────────────────────────────────────┘
```

### Component Responsibilities

| Component | Responsibility | Typical Implementation |
|-----------|----------------|------------------------|
| `agens` (recommend) | Run the fixed questionnaire, search the vault, recommend a pattern with a path citation | `SKILL.md`, `allowed-tools: Read Grep Glob Bash`, auto-triggerable |
| `agens-build` (gated MCP) | Offer, then on approval delegate MCP construction to mcp-builder | `SKILL.md`, `disable-model-invocation: true`, manual only |
| `agens-log` (append-only) | Append each recommendation decision to the vault log | `SKILL.md` or a helper invoked by `agens`; writes one file only |
| `vault_search.py` | Deterministic search: glob candidate notes, grep for terms, optionally query the local index | Python script, exits 0/non-zero, returns JSON of path + snippet |
| `read_note.py` | Read a note, split frontmatter from body, return structured text | Python script, deterministic |
| `append_log.py` | Append a decision record atomically, never rewrite prior entries | Python script, deterministic |
| Canonical concept note | The single consolidated lookup target for the 6-source pattern cluster | One Markdown note authored in Phase 0 |
| wiki-agents vault | Source of truth for pattern knowledge; already carries a `.smart-env` BGE-micro-v2 index | Obsidian vault at `C:\Users\Simon\Documents\wiki-agents` |

## Recommended Project Structure

Skills are directories under `~/.claude/skills/`, and the directory name is the slash-command name (verified: `.claude/skills/deploy-staging/SKILL.md` → `/deploy-staging`). Keep each `SKILL.md` under 500 lines; push reference material and scripts into the skill directory.

```
~/.claude/skills/
├── agens/                       # /agens — the read-only recommender (Phase 1)
│   ├── SKILL.md                 # questionnaire + recommend flow, ≤500 lines
│   ├── references/
│   │   ├── questionnaire.md     # the fixed 4-question set (goal, shape, sensitivity, latency/cost)
│   │   ├── citation-rules.md    # every recommendation cites a note by path — the core discipline
│   │   └── vault-map.md         # where pattern knowledge lives (canonical note path, MOC paths)
│   └── scripts/
│       ├── vault_search.py      # deterministic search over the vault
│       └── read_note.py         # frontmatter/body split
├── agens-build/                 # /agens-build — gated MCP construction (Phase 3)
│   ├── SKILL.md                 # disable-model-invocation: true; approval gate; delegates to mcp-builder
│   └── references/
│       └── approval-gate.md     # the human-approval protocol before any write
└── agens-log/                   # append-only decision logging (Phase 4)
    ├── SKILL.md                 # user-invocable: false; called by agens after a recommendation
    └── scripts/
        └── append_log.py        # atomic append to vault log
```

### Structure Rationale

- **A small family, not a monolith.** Split by trust boundary, not by feature. `agens` is read-only and safe to auto-trigger. `agens-build` writes and builds, so it carries `disable-model-invocation: true` and runs only on an explicit `/agens-build`. `agens-log` writes one scoped file. This split mirrors the existing `wiki` skill's own separation of a dry-run `scan` from a writing `apply`, and it directly serves PROJECT.md's Key Decision to "split recommend (read-only) from build (write) into two gated capabilities". A single SKILL.md cannot carry two different invocation policies (`allowed-tools`, `disable-model-invocation`) at once, so the split is a runtime requirement, not a preference.
- **`references/` holds progressive-disclosure material.** The questionnaire text, citation rules, and vault map load only when the flow needs them, keeping the always-loaded description small. This copies the `wiki` skill's "do not load all references on every invocation" discipline.
- **`scripts/` holds deterministic work.** File I/O, glob, grep, and log appends run in Python and return JSON. The model does semantic work (pattern judgement, citation selection); the script does mechanical work (finding candidate notes). This is the exact division the `wiki` skill already uses (`scan_vault.py` is "100% deterministic"; the LLM proposes tags).

## Architectural Patterns

### Pattern 1: Orchestrator Skill over deterministic scripts

**What:** The Skill body is a thin orchestrator. It calls a Python script via a Bash tool to do file work, receives JSON, then reasons over the result. The script never reasons; the Skill never walks the filesystem by hand.
**When to use:** Any Skill that touches many files or must be reproducible. The vault holds ~148 notes; loading them into context is wasteful and unreliable.
**Trade-offs:** Adds a script dependency and a Python runtime assumption. Buys determinism, testability, and low token cost. The `wiki` skill validates this pattern in production in the same vault.

**Example:**
```markdown
## Workflow — recommend
1. Ask the fixed questionnaire (references/questionnaire.md).
2. Run: python scripts/vault_search.py --vault "$VAULT" --terms "<pattern terms>" --out results.json
   Script returns candidate note paths + snippets. No LLM work yet.
3. Read the top candidate via scripts/read_note.py; select the pattern.
4. Emit the recommendation WITH the note path as citation (references/citation-rules.md).
```

### Pattern 2: Delegation through the Skill tool, not reimplementation

**What:** One Skill hands off to another installed Skill by invoking it through the host's **Skill tool** (`Skill(name)` in permission syntax). Claude sees the target Skill's name and description in the always-loaded listing and calls it. This is the mechanism behind PROJECT.md's delegation requirements to `gsd-framework-selector`/`gsd-ai-integration-phase` and to Anthropic's mcp-builder.
**When to use:** When the judgement or capability already exists in an installed Skill. Do not copy framework-selection logic or MCP-scaffolding logic into agens.
**Trade-offs:** The target Skill must be installed and visible in the listing. Delegation is a soft handoff (Claude chooses to call the tool), not a hard function call, so the orchestrating Skill must state the handoff as a standing instruction and check the result.

**Example:**
```markdown
When tooling is identified as a need, do NOT build the server here.
Offer /agens-build. On explicit approval, that skill invokes the mcp-builder
skill via the Skill tool and passes the tool spec. agens itself never writes.
```

Note the two composition directions the docs define. A Skill with `context: fork` runs its own body as a task in a subagent of a chosen `agent` type. A subagent declared with a `skills` field preloads named Skills as reference. agens uses the first form for isolated vault research and the Skill tool for peer handoff; it does not need a custom subagent.

### Pattern 3: Trust-graded invocation via frontmatter

**What:** Encode the human-approval gate in frontmatter rather than in prose alone. Read-only `agens` stays auto-triggerable. Write-capable `agens-build` sets `disable-model-invocation: true` so only the user can start it, and the model can never "decide the code looks ready" and build unprompted. `agens-log` sets `user-invocable: false` so it is a callable helper, not a menu command.
**When to use:** Whenever a Skill has side effects. The docs name exactly this case: use `disable-model-invocation: true` for `/deploy`-like actions "You don't want Claude deciding to deploy because your code looks ready."
**Trade-offs:** More Skills to maintain. Buys a structural, not merely instructional, guarantee against the self-mutation/rug-pull risk PROJECT.md flags.

**Example:**
```yaml
---
name: agens-build
description: Build an MCP server for an agens-recommended tool, behind human approval.
disable-model-invocation: true
allowed-tools: Read Grep Glob Bash
---
```

## Data Flow

### Request Flow (recommend — Phase 1)

```
User: "I want to build an agent that triages support tickets"
    ↓ (auto-trigger via description match, or explicit /agens)
agens SKILL.md loads
    ↓ ask fixed questionnaire (goal, workflow shape, data sensitivity, latency/cost)
User answers
    ↓ python vault_search.py --terms "<derived terms>"  → results.json (paths + snippets)
    ↓ python read_note.py <canonical concept note>       → structured pattern text
Claude selects pattern
    ↓ emit recommendation + MANDATORY citation: note path
    ↓ (Phase 4) invoke agens-log → append_log.py appends decision to vault log.md
```

### Delegation Flow (the load-bearing mechanic)

```
Framework-fit question arises
    ↓ agens does NOT answer from model knowledge
    ↓ Skill tool → Skill(gsd-framework-selector) or Skill(gsd-ai-integration-phase)
    ↓ result returns to agens conversation, folded into the recommendation

Tooling need identified (Phase 3)
    ↓ agens offers /agens-build (never auto-runs it)
    ↓ user runs /agens-build  →  HUMAN APPROVAL GATE
    ↓ on approval: Skill tool → Skill(mcp-builder) with the tool spec
    ↓ mcp-builder scaffolds server; agens-build never writes server code itself
```

### Key Data Flows

1. **Vault search (read):** Skill → `vault_search.py` → glob+grep (and optional index query) over the vault → JSON candidates → Skill reads the winner → citation. Data flows one way into the recommendation; the vault is never mutated on this path.
2. **Delegation (handoff):** Skill → Skill tool → installed peer Skill → result back into the same conversation. No MCP call, no HTTP, no direct file reference to the other Skill's directory. Delegation is a named-tool invocation the host resolves against the Skill listing.
3. **Decision log (append-only write):** Skill → `append_log.py` → atomic append to one log file in the vault, mirroring the LLM-wiki `log.md` pattern. Prior entries are never rewritten. This is the only write path on the recommend side, and it is scoped to a single file.

### Vault search — how to search 148 notes reliably without loading the vault

The vault already carries a Smart Connections index at `.smart-env/` (BGE-micro-v2 embeddings, confirmed in `smart_env.json`), so a vector search substrate exists. Recommendation, graded by cost:

- **Start with glob + grep (Phase 1 default).** 148 notes across 26 MOCs is small. Deterministic `grep` for pattern terms plus `glob` on the `50_MOCs/` and canonical-note paths resolves most lookups. This is what `vault_search.py` should do first. It needs no index, no embedding model at query time, and is fully reproducible. The canonical concept note (authored in Phase 0) collapses the 6-source pattern cluster into one file, so most recommendations resolve to a single known path — grep is enough.
- **Add index query only if grep proves insufficient.** A BM25 or vector query becomes worthwhile when term-matching misses semantically-close notes (user describes "a loop that critiques its own output" but the note says "reflection pattern"). The vault's `.smart-env` is an Obsidian-plugin index, not a CLI, so agens cannot query it directly at runtime. Two clean options: (a) let the Skill's search terms be model-expanded synonyms before grep, which closes most of the gap at zero infrastructure cost; or (b) build a small local index (BM25 via a Python library, or a pre-computed embedding table the script reads) that `vault_search.py` queries and returns the same JSON shape. Prefer (a) first; reach for (b) only if Phase 1 evaluation shows grep missing correct notes.
- **Never load the whole vault into context.** The script returns paths and short snippets; the Skill reads at most one or two full notes. This keeps token cost flat regardless of vault growth and preserves the citation discipline — a recommendation cites the note the script surfaced, not a general impression.

## Scaling Considerations

This is a single-user personal tool (PROJECT.md: "personal tool for the user's own agent-development practice"). Scale means vault growth and Skill-family growth, not concurrent users.

| Scale | Architecture Adjustments |
|-------|--------------------------|
| ~148 notes (today) | glob + grep in `vault_search.py`; no index needed |
| ~500+ notes | Add synonym-expanded grep; consider a BM25 index the script reads |
| Many overlapping topics | Rely on the canonical concept note as the single lookup target; add more consolidated notes rather than more search cleverness |

### Scaling Priorities

1. **First bottleneck: recall, not speed.** grep is fast; the risk is missing a semantically-close note. Fix with model-expanded search terms before grep, then a local BM25 index if needed.
2. **Second bottleneck: Skill-listing crowding.** Many installed Skills shorten descriptions in the listing and can strip trigger keywords. Keep agens' description keyword-dense and front-loaded (the docs truncate `description` + `when_to_use` at 1,536 characters).

## Anti-Patterns

### Anti-Pattern 1: One monolithic SKILL.md for recommend and build

**What people do:** Put questionnaire, recommendation, and MCP-build in a single Skill.
**Why it's wrong:** A single Skill has one invocation policy. It cannot be both auto-triggerable (good for recommend) and manual-only with an approval gate (required for build). Merging them either exposes the write path to auto-trigger or blocks the read path behind a manual gate. It also breaks PROJECT.md's explicit recommend/build split.
**Do this instead:** Separate `agens` (read-only, auto) from `agens-build` (`disable-model-invocation: true`, manual).

### Anti-Pattern 2: Reimplementing framework selection or MCP scaffolding inside agens

**What people do:** Copy framework-fit heuristics or MCP boilerplate into agens because "it's just a bit of logic."
**Why it's wrong:** It duplicates installed capability (`gsd-framework-selector`, mcp-builder), drifts out of sync, and violates PROJECT.md's delegation discipline. agens degrades into "asking Claude cold with extra steps."
**Do this instead:** Delegate through the Skill tool. agens routes; it does not reason about frameworks or generate server code.

### Anti-Pattern 3: Walking the vault by loading notes into context

**What people do:** Read many notes into the conversation to "understand the vault" before recommending.
**Why it's wrong:** Token cost scales with vault size, results are non-reproducible, and citations become vague impressions rather than a specific path.
**Do this instead:** A deterministic `vault_search.py` returns candidate paths + snippets; the Skill reads one note and cites its path.

## Integration Points

### External Services

| Service | Integration Pattern | Notes |
|---------|---------------------|-------|
| wiki-agents vault | Local filesystem via Python scripts | Read for recommend; append-only for log. Never rewrite note bodies (matches `wiki` skill rule). |
| Anthropic mcp-builder skill | Skill tool invocation behind approval gate | Install via `npx skills add anthropics/mcp-builder`. If absent, it will not appear in the listing and the delegation cannot resolve — agens must detect and report this, not silently proceed. |
| gsd-framework-selector / gsd-ai-integration-phase | Skill tool invocation | Already installed in this environment (confirmed in `~/.claude/skills/`). Peer handoff, not a subprocess. |

### Internal Boundaries

| Boundary | Communication | Notes |
|----------|---------------|-------|
| `agens` ↔ `vault_search.py` | Bash tool call → JSON stdout | Deterministic; script exits non-zero on failure and the Skill stops rather than guessing |
| `agens` ↔ `agens-build` | Offer + user runs the slash command | Deliberately not an automatic call — the human gate is the boundary |
| `agens` ↔ `agens-log` | Skill tool (or direct script call) | `agens-log` is `user-invocable: false`; a callable helper, not a menu command |
| `agens-build` ↔ mcp-builder | Skill tool after approval | agens-build orchestrates; mcp-builder writes the server |

### Delegation failure modes (explicit)

Delegation resolves against the host's Skill listing. Failure modes and required handling:

- **Target Skill not installed:** Its name is absent from the listing, so `Skill(name)` cannot resolve. agens must check for the capability and report "mcp-builder is not installed — run `npx skills add anthropics/mcp-builder`" rather than falling back to reimplementing the work. Failing closed here preserves the delegation discipline.
- **Target Skill denied by permissions:** A `Skill(deploy *)` deny rule or a `Skill` global deny blocks invocation. Surface the permission error; do not work around it.
- **Delegation is soft, not a function call:** Claude chooses to invoke the Skill tool from a standing instruction. Write the handoff as a non-negotiable rule in the SKILL.md body and verify the returned result before folding it into a recommendation.
- **Auto-trigger of a write Skill:** Prevented structurally by `disable-model-invocation: true` on `agens-build`, not by prose alone.

## Suggested Build Order

The five-phase plan in PROJECT.md is sound and the architecture reinforces it. One insertion (Phase 0) is already a Key Decision. Ordering by trust and dependency:

| Phase | Builds | Architectural dependency |
|-------|--------|--------------------------|
| **0** | Consolidate the 6-source `topic/agent-patterns` cluster into one canonical concept note | Nothing depends on it, but every later phase reads it. Do first — it is the single lookup target `vault_search.py` and the recommend flow both assume. |
| **1** | `agens` Skill: questionnaire + `vault_search.py`/`read_note.py` + citation-backed recommendation, read-only | Depends on Phase 0's canonical note. Ships and earns trust before any write path exists (PROJECT.md sequencing constraint). |
| **2** | Delegation wiring to `gsd-framework-selector`/`gsd-ai-integration-phase` via the Skill tool | Depends on Phase 1's recommend flow being the place delegation slots into. Adds the peer-handoff pattern with no new write capability. |
| **3** | `agens-build`: gated MCP construction delegating to mcp-builder behind approval | Depends on the Phase 2 delegation mechanic being proven, plus a separate Skill carrying `disable-model-invocation: true`. First write/build capability — deliberately last-but-one. |
| **4** | `agens-log`: append-only decision logging via `append_log.py`, mirroring LLM-wiki `log.md` | Depends on recommendations existing to log (Phase 1) and the delegation decisions worth recording (Phases 2-3). Scoped single-file write; safe to add last. |

The order follows the read-before-write sequencing constraint and the dependency chain: canonical note → read/recommend → delegate → gated build → log. Phases 3 and 4 are the two write-capable additions and both come after the read-only core has earned trust.

## Sources

- Claude Code Skills documentation — SKILL.md frontmatter reference (`context`, `agent`, `disable-model-invocation`, `user-invocable`, `allowed-tools`), the Skill tool and `Skill(name)` permission syntax, `context: fork` subagent execution, and the two-direction skill/subagent composition table. https://code.claude.com/docs/en/skills [HIGH]
- Anthropic MCP Builder skill — install (`npx skills add anthropics/mcp-builder`) and invocation-when-task-is-MCP-development. https://github.com/anthropics/skills/tree/main/skills/mcp-builder [HIGH]
- Local evidence: `~/.claude/skills/wiki/SKILL.md` — production pattern for deterministic-script + progressive-disclosure + dry-run/apply split on the same vault [HIGH]
- Local evidence: `~/.claude/skills/gsd-ai-integration-phase/SKILL.md` and `get-shit-done/workflows/ai-integration-phase.md` — orchestration-over-sub-capabilities pattern and `allowed-tools` usage [HIGH]
- Local evidence: `C:\Users\Simon\Documents\wiki-agents\.smart-env\smart_env.json` — vault carries a BGE-micro-v2 Smart Connections index (Obsidian-plugin, not a runtime CLI) [HIGH]
- Local evidence: `C:\Users\Simon\Documents\wiki-agents\50_MOCs\MOC - Agent Patterns.md` — the pattern cluster to consolidate in Phase 0 [HIGH]

---
*Architecture research for: Claude Code Skill family — vault-grounded agent-architecture recommender*
*Researched: 2026-07-11*
