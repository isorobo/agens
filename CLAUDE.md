<!-- GSD:project-start source:PROJECT.md -->
## Project

**agens**

agens is a Claude Code Skill (or small Skill family) that routes agent-project requests to existing capabilities instead of reimplementing them. It reads the wiki-agents Obsidian vault for agent-pattern and architecture knowledge, delegates framework-fit questions to `gsd-framework-selector`/`gsd-ai-integration-phase`, and delegates MCP server construction to Anthropic's official MCP Builder skill under a mandatory human-approval gate. It is invoked both as an explicit slash command and as an auto-triggered Skill.

**Core Value:** Every recommendation agens gives cites a specific wiki-agents note by path, not general model knowledge. That citation discipline must always hold, or agens degrades into asking Claude cold with extra steps.

### Constraints

- **Ecosystem**: Anthropic Claude Agent SDK / Claude Code Skills only — no LangChain, CrewAI, or other framework as agens' own implementation substrate.
- **Delegation discipline**: MCP construction and framework-selection logic must delegate to existing tools (MCP Builder, `gsd-framework-selector`) rather than be reimplemented inside agens.
- **Human approval gate**: Any capability that writes or builds (MCP server construction) requires explicit human approval before executing — a direct mitigation against the self-mutation/rug-pull risk pattern documented in wiki-agents' MCP Security note.
- **Citation discipline**: Every pattern recommendation must reference a specific wiki-agents note by path.
- **Sequencing**: Read-only capabilities (interrogate, recommend) ship and earn trust before the write capability (build MCP) is added.
<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->
## Technology Stack

## Executive Framing
## Recommended Stack
### Core Technologies
| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| Claude Code Skills (`SKILL.md`) | Claude Code v2.1.196+ | The substrate: a directory with `SKILL.md` becomes a `/command` and an auto-triggerable capability | The project constraint is Anthropic-ecosystem-only. Skills are the native unit. A skill's body loads only when used, so reference material costs almost nothing until needed. |
| YAML frontmatter | Agent Skills open standard | Declares `name`, `description`, invocation control, `allowed-tools` | `description` drives auto-trigger; frontmatter fields give the exact controls agens needs for the gating requirement. Verified field table in official docs. |
| Built-in `Grep` tool | Claude Code current | Search vault note bodies and frontmatter for relevant patterns | ripgrep-backed, regex, frontmatter-aware. At 547 KB / 152 notes this is the correct and sufficient retrieval mechanism. No index needed. |
| Built-in `Glob` tool | Claude Code current | Enumerate notes by path pattern (e.g. `30_Concepts/*.md`, `50_MOCs/MOC - Agent Patterns.md`) | The vault uses a Johnny-Decimal folder taxonomy (`10_Sources`, `30_Concepts`, `50_MOCs`). Glob targets a cluster directly, cheaper than searching everything. |
| Built-in `Read` tool | Claude Code current | Load the canonical concept note and cite it by path | Deterministic file read. The citation-discipline requirement demands reading the actual note, not recalling it. |
| Built-in `Skill` tool | Claude Code current | One skill invokes another (agens → `build-mcp-server`) | This is the sanctioned cross-skill mechanism. Claude invokes any skill lacking `disable-model-invocation: true`. This is how agens hands off to MCP Builder. |
| `AskUserQuestion` tool | Claude Code current | The fixed questionnaire (goal, workflow shape, data sensitivity, latency/cost) and the human-approval gate before MCP Builder | Structured question UI; the natural primitive for both the intake questionnaire and the explicit approval prompt. |
### Supporting Libraries
| Convention | Field / File | Purpose | When to Use |
|---------|---------|---------|-------------|
| Auto-trigger | `description` (+ optional `when_to_use`) | Claude reads this to decide when to load agens automatically | Always. Front-load the natural project-starting language. Combined `description` + `when_to_use` is truncated at 1,536 characters in the skill listing — put the key trigger first. |
| Explicit slash command | Directory name | `.claude/skills/agens/SKILL.md` → `/agens` | Always. The command name comes from the directory, not the `name` field (except plugin-root skills). |
| Both invocation paths | default frontmatter | Leaving `disable-model-invocation` and `user-invocable` unset gives you both `/agens` and auto-trigger | This is the exact requirement: explicit control when wanted, no command to remember for casual mentions. Do not set either field on the top-level agens skill. |
| Pre-approved read tools | `allowed-tools: Read Grep Glob` | Lets agens read and search the vault without a permission prompt each call | On the read-only recommend capability. Keeps the interrogate/recommend loop friction-free while staying read-only. |
| Bundled reference file | `references/*.md` in the skill dir | Move the fixed questionnaire text, citation-format rules, and routing table out of `SKILL.md` | When `SKILL.md` approaches 500 lines. Reference the file from `SKILL.md` so Claude loads it only when needed. |
| Append-only log | `Bash` append (`>>`) or `Read`-then-`Write` | Mirror the vault's LLM-wiki `log.md` pattern for each recommendation decision | Always, per requirement. See "Append-only log" section below. |
| Path portability | `${CLAUDE_SKILL_DIR}`, `${CLAUDE_PROJECT_DIR}` | Reference bundled scripts/files regardless of working directory | Whenever agens references a file it ships with. Avoids brittle absolute paths. Requires v2.1.196+. |
### Development Tools
| Tool | Purpose | Notes |
|------|---------|-------|
| `/skills` menu | Inspect visibility, toggle skill states | Writes `skillOverrides` to `.claude/settings.local.json` for you. |
| Live change detection | Edit `SKILL.md` mid-session and re-trigger | Skills reload during a session; no restart needed to iterate. |
| Existing on-disk skills | Reference implementations | `~/.claude/skills/gsd-ai-integration-phase/SKILL.md` and `~/.claude/skills/find-skills/SKILL.md` are working examples of orchestration and auto-trigger description craft. |
## Where agens Lives, and How It Reads the Vault
### Installation location
### Reading the vault without brittle absolute paths
### Citation format
## When Does a Project Like This Need a Search Index?
| Signal | agens today | Index-warranted threshold |
|--------|-------------|---------------------------|
| Corpus size | 547 KB, 152 notes | Tens of MB, thousands of notes |
| Query type | Keyword/topic lookup against tagged, foldered notes | Fuzzy semantic search where keywords miss |
| Retrieval latency | Single ripgrep pass, sub-second | ripgrep pass becomes slow or returns too many hits to rank |
| Ranking need | Folder taxonomy + `topic/` tags already cluster notes | Relevance ranking across a flat, untagged corpus |
## How One Skill Invokes Another (agens → MCP Builder)
- Plugin: `mcp-server-dev`
- Entry skill: `build-mcp-server` (frontmatter `name: build-mcp-server`, `version: 0.1.0`)
- Invocation: `/mcp-server-dev:build-mcp-server`
- It is the "front door": it interrogates the use case, picks the deployment model
## Conventions for an Append-Only Log
- **Append, never rewrite.** Use `Bash` with `>>` redirection, or `Read` the file then `Write`
- **Timestamp each entry.** Use `${CLAUDE_SESSION_ID}` to correlate a log entry with the
- **One entry per recommendation decision.** Record: the described project, the questionnaire
- **Decide the log's home.** Either inside the skill dir (`~/.claude/skills/agens/log.md`) or
## Installation
# No package manager. agens is markdown files. "Installation" is placing the directory:
# Author SKILL.md (with description tuned for auto-trigger) and supporting files.
# Grant the vault as an additional directory when running Claude Code:
#   claude --add-dir "C:/Users/Simon/Documents/wiki-agents"
# or add it via /add-dir in-session.
# The MCP Builder skill is an installed plugin; confirm it is present:
#   /mcp-server-dev:build-mcp-server   (should appear in the / menu)
## Alternatives Considered
| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|-------------------------|
| Skill (`SKILL.md`) | Standalone Claude Agent SDK app | Never for this project. An SDK app duplicates session/tool machinery the constraint says to reuse. The SDK is the substrate *under* Skills, not a competing choice here. |
| Grep/Glob over the vault | Local embedding index (sqlite-vec, LanceDB) | Only past a few-thousand-note corpus where keyword search misses conceptually relevant notes. Not now. |
| `Skill` tool for delegation | Reimplementing framework-selection logic inline | Never. The constraint forbids reimplementing GSD/MCP-Builder judgement. |
| Single-family or split skills | One monolithic skill | Split `recommend` (read) from `build` (write/gated) if the body grows or the gate needs hard isolation. The sequencing constraint (read ships first) favours a split. |
| Additional-directory access to vault | Hard-coded absolute path in each call | Only as a last resort on a locked single machine; still centralise the path to one constant. |
## What NOT to Use
| Avoid | Why | Use Instead |
|-------|-----|-------------|
| LangChain / CrewAI / any external agent framework | Violates the Anthropic-ecosystem-only constraint; adds a runtime the Skill substrate already provides | Claude Code Skills + built-in tools |
| A bespoke in-skill vector index | Premature against a 547 KB corpus that fits in context; adds build/maintenance burden | Grep/Glob now; a gated MCP-served index later if the corpus explodes |
| `${CLAUDE_SKILL_DIR}` / `${CLAUDE_PROJECT_DIR}` to reach the vault | They resolve to the skill dir / project root, not the external vault — a silent mis-read | An additional-directory grant, or one centralised vault-root constant |
| Relying on MCP Builder's own gating | `build-mcp-server` does not set `disable-model-invocation`; it will run unprompted | agens enforces its own `AskUserQuestion` approval gate before the Skill-tool handoff |
| Rewriting the log file each turn | Risks losing prior entries; not truly append-only | `Bash` `>>` append |
| Citing model knowledge instead of a note path | Destroys the core value — agens degrades into "ask Claude cold with extra steps" | Always `Read` and cite the vault note by relative path |
| Scattering the full vault path across the body | One path change breaks every call site | Centralise to one constant or an added directory |
| A `name` field mismatched to the directory | The command name comes from the directory, not `name`; a mismatch confuses the listing label vs the `/command` | Name the directory `agens`; keep `name: agens` consistent |
## Stack Patterns by Variant
- One `~/.claude/skills/agens/SKILL.md`, no invocation-control fields (both slash + auto-trigger).
- `allowed-tools: Read Grep Glob AskUserQuestion`.
- Because the sequencing constraint says read-only capabilities ship and earn trust first.
- `agens` (read: interrogate + recommend, both invocation paths).
- `agens-build` (write: `disable-model-invocation: true` so only the user can trigger it, and
- Because a write capability that can auto-trigger is the exact self-mutation risk the vault's
- Move it to `references/questionnaire.md` and `references/routing.md`, referenced from
- Because skill body content is a recurring per-turn token cost; keep `SKILL.md` under 500 lines.
## Version Compatibility
| Requirement | Minimum version | Notes |
|-----------|-----------------|-------|
| `${CLAUDE_PROJECT_DIR}` / `${CLAUDE_SKILL_DIR}` substitution | Claude Code v2.1.196 | Also gates the `disable-model-invocation` behaviour that blocks scheduled-task firing. |
| Re-invocation dedupe (no double-append of skill content) | v2.1.202 | Relevant if agens is re-invoked in one session. |
| Skill hidden from Agent SDK / Remote Control on `"off"` | v2.1.199 | Only matters if agens is ever driven headless. |
| `build-mcp-server` (MCP Builder) | plugin `mcp-server-dev`, skill v0.1.0 | Confirm installed via the `/` menu before relying on the handoff. |
## Sources
- `code.claude.com/docs/en/skills` — SKILL.md structure, full frontmatter field table, invocation control (`disable-model-invocation`, `user-invocable`), the `Skill` tool for cross-skill invocation, `${CLAUDE_SKILL_DIR}`/`${CLAUDE_PROJECT_DIR}` substitutions, supporting-file conventions, 1,536-char description cap, 500-line body guidance, additional-directory skill loading, version gates. Retrieved 2026-07-11. Confidence: HIGH (official docs).
- `github.com/anthropics/claude-plugins-official/blob/main/plugins/mcp-server-dev/skills/build-mcp-server/SKILL.md` — MCP Builder canonical name, description, version 0.1.0, absence of `disable-model-invocation`, `/mcp-server-dev:build-mcp-server` invocation. Confidence: HIGH (official repo).
- Local inspection of `~/.claude/skills/gsd-ai-integration-phase/SKILL.md` and `find-skills/SKILL.md` — real frontmatter and auto-trigger description craft. Confidence: HIGH (on-disk).
- Local inspection of `C:/Users/Simon/Documents/wiki-agents` — 152 notes, 547 KB, Johnny-Decimal folders, YAML frontmatter fields, `[[wikilinks]]`/`dataview` blocks, `_memory/recent.md` log style. Confidence: HIGH (on-disk).
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

Conventions not yet established. Will populate as patterns emerge during development.
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Architecture

Architecture not yet mapped. Follow existing patterns found in the codebase.
<!-- GSD:architecture-end -->

<!-- GSD:skills-start source:skills/ -->
## Project Skills

No project skills found. Add skills to any of: `.claude/skills/`, `.agents/skills/`, `.cursor/skills/`, `.github/skills/`, or `.codex/skills/` with a `SKILL.md` index file.
<!-- GSD:skills-end -->

<!-- GSD:workflow-start source:GSD defaults -->
## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:
- `/gsd-quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd-debug` for investigation and bug fixing
- `/gsd-execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->



<!-- GSD:profile-start -->
## Developer Profile

> Profile not yet configured. Run `/gsd-profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->
