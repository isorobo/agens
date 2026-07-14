# Phase 4: agens-log (Append-Only Logging) - Context

**Gathered:** 2026-07-15
**Status:** Ready for planning

<domain>
## Phase Boundary

Phase 4 delivers agens' recommendation log: each recommendation is recorded as an append-only, attributed entry in a new vault-owned file at `99_Meta/agens-log.md`, using the `## YYYY-MM-DD` date-header convention. The logging capability is callable only by agens itself — it is never a user-facing command and cannot be triggered directly by conversation (LOG-02). Read-back is bounded, each entry stamps the vault state it referenced, and the log is never treated as a grounding source for future recommendations (ROADMAP.md Phase 4 success criterion 3). It does not touch pattern recommendation logic (Phase 1), delegation wiring (Phase 2), or the deferred MCP-build capability (v2).

</domain>

<decisions>
## Implementation Decisions

### Packaging & LOG-02 enforcement
- **D-01:** Logging is an inline step inside agens' existing `SKILL.md` flow — no separate `agens-log` skill directory exists at all. LOG-02 holds structurally: there is no command, no auto-trigger surface, and no `Skill`-tool target to invoke. The trade-offs accepted with this choice: agens' own `allowed-tools` gains the write mechanism (D-02), and `SKILL.md` grows toward the 500-line body budget (currently ~357 lines). The rejected alternatives were a separate skill with `user-invocable: false` (softer enforcement — the `Skill` grant cannot be scoped to one target, per the platform limitation already documented in SKILL.md's anti-patterns) and an inline step with a lazy-loaded reference file.
- **D-02:** The write mechanism is a scoped Bash append (`>>` redirection), added narrowly to agens' `allowed-tools`. The log file is never read-modify-rewritten — this is the mechanical basis of LOG-01's no-overwrite guarantee, and it matches CLAUDE.md's own "What NOT to Use" ruling (rewrite-per-turn rejected; `Bash >>` named as the append mechanism). Planner should scope the Bash grant as tightly as the frontmatter syntax allows (analogous to `Bash(git:*)`), rather than granting unrestricted shell.
- **D-03:** A direct conversational request to write, edit, or delete the log outside a recommendation flow gets a fixed, scripted refusal in `SKILL.md` — the Phase 1/2 house style (fixed refusal block, fixed failure message). The refusal states that logging runs only as the tail of a recommendation, is not a command, and agens will not append, edit, or delete entries on request. A fixed string gives verification an exact assertion target, and it also defends against a prompt-injected vault note attempting to trigger a log write.
- **D-04:** Phase 4 edits the project copy (`.claude/skills/agens/`) as the single source of truth and redeploys to the user-level runtime copy (`~/.claude/skills/agens/`) as a built-in phase task. Verification checks the two copies are identical — the drift Phase 2 hit and reconciled must not recur.

### Claude's Discretion
- Exact wording of the D-03 fixed refusal text — follow the Phase 1 refusal block's register (name what is closed, offer no partial substitute).
- Exact scoping syntax for the D-02 Bash grant — whatever the skill frontmatter's `allowed-tools` specifier support actually allows; verify against current Claude Code behaviour rather than assuming.

### Not discussed this session
Three gray areas were presented but the user chose to discuss only "Packaging & LOG-02 enforcement" this session. These are **undecided**, not defaulted — do not assume a trivial answer:
- **Entry content & vault-state stamp** — what each entry records (questionnaire answers, recommended pattern, cited path?), what "stamps the vault state it referenced" means concretely (index-note modified date, content hash, git revision?), and whether refusals and delegations get logged or only successful recommendations. LOG-01 says "each agens recommendation"; whether that extends to refusal/delegation outcomes is open.
- **Append mechanism details** — how date headers are managed (one `## YYYY-MM-DD` header per day with entries appended beneath, vs a header per entry), and entry ordering. Note the tension: the format precedent `.remember/recent.md` reads newest-first, but pure `>>` append is oldest-first at the file tail. D-02 locks the append mechanism; the header/ordering convention within it is unresolved.
- **Write failure & bounded read-back** — whether a failed log write blocks the recommendation or logging is best-effort; what "bounded read-back" means in practice (read nothing before appending? tail only, to find today's header?); and how "the log is never a grounding source" is enforced in SKILL.md text.

Research and planning should resolve these against REQUIREMENTS.md (LOG-01/LOG-02), ROADMAP.md Phase 4's three success criteria, and the vault conventions below — or raise them back to the user if the answer materially changes the plan.

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Project-level
- `.planning/PROJECT.md` — Core Value (citation discipline), Constraints (human-approval gate, citation discipline), Key Decisions table.
- `.planning/REQUIREMENTS.md` — LOG-01, LOG-02 (this phase's full requirement set); LOG-03 (wikilinks from entries to cited notes) is v2-deferred and must NOT be pulled forward.
- `.planning/ROADMAP.md` §Phase 4 — goal and the three success criteria, including the bounded read-back / vault-state stamp / never-a-grounding-source criterion no requirement ID carries.
- `.planning/phases/00-vault-consolidation/00-CONTEXT.md` — D-10/D-11/D-12: the log-format correction. The log target is `99_Meta/agens-log.md` (new, vault-owned); the `## YYYY-MM-DD` date-header shape is borrowed from `.remember/recent.md` purely as a proven format, not as a vault convention. Creating the file is this phase's job.
- `.planning/phases/02-delegation-wiring/02-CONTEXT.md` — D-09 (the target speaks last / tail rule) and the deployment-sync history D-04 responds to.

### The write target and its conventions
- `wiki-agents/99_Meta/agens-log.md` — target path; does not exist yet. Phase 4 creates it.
- `wiki-agents/99_Meta/schema.md` — vault frontmatter rules. §2 must gain the `authored_by` field BEFORE first use, per the schema's own rule that a new controlled value requires a schema update first.
- `wiki-agents/.remember/recent.md` — the `## YYYY-MM-DD` date-header format precedent (format only, not a vault convention — Phase 0 D-10).

### Attribution convention (already fully specified — enforce, do not redesign)
- `.claude/skills/agens/references/agent-authored-convention.md` — the three-element convention Phase 4 enforces: `authored_by: agens` frontmatter marker, the visible in-note attribution line below the title, and the anchored verification grep (`^authored_by:[[:space:]]*agens[[:space:]]*$`). A failed grep is a failed write.

### This project's own code
- `.claude/skills/agens/SKILL.md` — the file D-01's inline step lands in. Current `allowed-tools: Read Grep Glob AskUserQuestion Skill` (line 14) is what D-02 extends. The Phase 1 refusal block and Phase 2 fixed failure message are the register for D-03's refusal text. The anti-patterns section documents the unscopeable-`Skill`-grant platform limitation that motivated D-01.
- `CLAUDE.md` (repo root) §Conventions for an Append-Only Log and §What NOT to Use — `Bash >>` append named as the mechanism; rewrite-per-turn rejected; 500-line SKILL.md body guidance.

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- `.claude/skills/agens/SKILL.md`'s vault-root resolution rule (resolve via granted additional directories, refuse plainly if unresolvable) — the log write path `{vault_root}/99_Meta/agens-log.md` resolves the same way; no new path mechanism needed.
- Phase 1's fixed refusal block and Phase 2's fixed failure message — direct templates for D-03's scripted refusal.
- `references/agent-authored-convention.md` — the attribution marker, in-note line, and verification grep are pre-written; Phase 4 executes them.

### Established Patterns
- Gate-before-showing (Phase 1's citation gate, Phase 2's combined pre-flight) — any verification around the log write (e.g. the attribution grep) should run as a gate in the same style.
- Fixed, plain, scripted text for every closed path — agens never improvises a refusal.
- Match existing vault convention over inventing new structure (Phase 0 `<specifics>`, consistently reaffirmed) — the log file's frontmatter must follow `schema.md`, extended with `authored_by` per the convention doc.

### Integration Points
- The logging step sits at the tail of agens' recommendation path (after Step 3's recommendation output). Interaction with Phase 2's D-09 tail rule needs care on the delegation path: D-09 says the delegated skill's output ends the turn with nothing after it — whether/how delegations are logged is one of the undecided areas above.
- agens' `allowed-tools` frontmatter (SKILL.md line 14) gains the scoped Bash grant.
- Deployment: project copy → `~/.claude/skills/agens/` copy step + identity check (D-04).

</code_context>

<specifics>
## Specific Ideas

No specific references beyond the decisions above — the discussion stayed at the mechanism level. The standing preference holds: match existing conventions (vault schema, agens house style, CLAUDE.md's append ruling) over inventing new structure.

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope. No scope creep occurred. (LOG-03 — wikilinking each entry to its cited note — was already v2-deferred at roadmap creation and stays there.)

</deferred>

---

*Phase: 4-agens-log (Append-Only Logging)*
*Context gathered: 2026-07-15*
