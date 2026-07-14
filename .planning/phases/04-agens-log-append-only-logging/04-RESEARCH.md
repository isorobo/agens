# Phase 4: agens-log (Append-Only Logging) - Research

**Researched:** 2026-07-15
**Domain:** Claude Code skill authoring — scoped Bash write grant, append-only file mechanics, vault-convention conformance
**Confidence:** HIGH

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

- **D-01:** Logging is an inline step inside agens' existing `SKILL.md` flow — no separate `agens-log` skill directory exists at all. LOG-02 holds structurally: there is no command, no auto-trigger surface, and no `Skill`-tool target to invoke. Trade-offs accepted: agens' own `allowed-tools` gains the write mechanism (D-02), and `SKILL.md` grows toward the 500-line body budget (currently ~356 lines). Rejected alternatives: a separate skill with `user-invocable: false` (softer enforcement — the `Skill` grant cannot be scoped to one target), and an inline step with a lazy-loaded reference file.
- **D-02:** The write mechanism is a scoped Bash append (`>>` redirection), added narrowly to agens' `allowed-tools`. The log file is never read-modify-rewritten — the mechanical basis of LOG-01's no-overwrite guarantee, matching CLAUDE.md's "What NOT to Use" ruling. Planner should scope the Bash grant as tightly as the frontmatter syntax allows (analogous to `Bash(git:*)`), not grant unrestricted shell.
- **D-03:** A direct conversational request to write, edit, or delete the log outside a recommendation flow gets a fixed, scripted refusal in `SKILL.md` — the Phase 1/2 house style. The refusal states that logging runs only as the tail of a recommendation, is not a command, and agens will not append, edit, or delete entries on request. A fixed string gives verification an exact assertion target and defends against a prompt-injected vault note attempting to trigger a log write.
- **D-04:** Phase 4 edits the project copy (`.claude/skills/agens/`) as the single source of truth and redeploys to the user-level runtime copy (`~/.claude/skills/agens/`) as a built-in phase task. Verification checks the two copies are identical — the drift Phase 2 hit must not recur.

### Claude's Discretion

- Exact wording of the D-03 fixed refusal text — follow the Phase 1 refusal block's register (name what is closed, offer no partial substitute).
- Exact scoping syntax for the D-02 Bash grant — whatever the skill frontmatter's `allowed-tools` specifier support actually allows; verify against current Claude Code behaviour rather than assuming.

### Undecided this session (NOT defaulted — resolve in planning or raise to user)

- **Entry content & vault-state stamp** — what each entry records; what "stamps the vault state it referenced" means concretely; whether refusals and delegations get logged or only successful recommendations.
- **Append mechanism details** — how date headers are managed (one `## YYYY-MM-DD` header per day vs a header per entry) and entry ordering (recent.md reads newest-first; pure `>>` is oldest-first at the tail).
- **Write failure & bounded read-back** — whether a failed log write blocks the recommendation; what "bounded read-back" means in practice; how "never a grounding source" is enforced in SKILL.md text.

This research resolves all three with grounded recommendations (see Open Questions and the body). Each recommendation that reads LOG-01's scope is flagged in the Assumptions Log for user confirmation.

### Deferred Ideas (OUT OF SCOPE)

None deferred this session. **LOG-03** (wikilink each entry to its cited note) was v2-deferred at roadmap creation and must NOT be pulled forward.
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| LOG-01 | Each agens recommendation is recorded as an append-only entry in the vault, in the vault's date-header log convention; no prior entry is ever overwritten. | Append via Git Bash `>>` (verified); one-header-per-day `## YYYY-MM-DD` layout; SHA-256 vault-state stamp using the schema's own `wiki_hash` precedent; anchored `authored_by` grep gate; frontmatter-init-on-first-write pattern. |
| LOG-02 | `agens-log` cannot be triggered directly by conversation — called only by `agens` itself, never a user-facing command. | Satisfied structurally by D-01 (inline step, no skill directory, no command, no `Skill` target). Reinforced by the D-03 fixed refusal against direct write requests and prompt injection. |
</phase_requirements>

## Summary

Phase 4 adds a write tail to an existing read-only skill. The whole capability is one new vault file (`99_Meta/agens-log.md`), a narrowly scoped Bash grant on agens' `allowed-tools`, an inline append step at the end of the Step 3 recommendation branch, a fixed refusal block, a one-line schema update, and a redeploy-and-verify task. No external package, no framework, no new runtime is involved — the mechanism is Git Bash builtins already present on the machine (`printf`, `date`, `sha256sum`, `tail`, `grep`), all verified available.

The load-bearing technical finding concerns the D-02 Bash grant. Current official docs confirm `allowed-tools` accepts a space-separated string and scopes Bash with a `Bash(...)` specifier — but the documented example form is `Bash(git add *)` (command prefix plus glob), not the colon form `Bash(git:*)` that D-02 and CLAUDE.md name. The docs do NOT specify how the permission matcher treats output redirection (`>>`), and agens' append depends on redirection. This is a real, version-sensitive uncertainty that the plan must resolve by empirical test before relying on a friction-free append, with a bundled-script grant as the documented fallback.

The three undecided design areas resolve cleanly against the vault's own precedents: a SHA-256 content hash of the cited note as the vault-state stamp (the schema already defines `wiki_hash` as exactly this), one `## YYYY-MM-DD` header per day with entries appended beneath it (bounded `tail` read to place the header — giving criterion 3's "bounded read-back" a concrete referent), and best-effort logging that never blocks the already-delivered recommendation yet verifies its own write with the anchored `authored_by` grep.

**Primary recommendation:** Ship an inline `>>` append (honouring D-01's minimalism) that (1) creates the file with schema-conformant frontmatter and the `authored_by: agens` marker on first write, (2) places one `## YYYY-MM-DD` header per day via a bounded `tail` read, (3) stamps each entry with a short SHA-256 hash of the cited index note, and (4) verifies the write with the anchored `authored_by` grep. Verify the exact `Bash(...)` grant form in Wave 0; fall back to a bundled-script grant if inline-redirection matching proves unreliable. Update `schema.md` §2 to admit `authored_by` before first use. Log successful recommendations only.

## Architectural Responsibility Map

| Capability | Primary Tier | Secondary Tier | Rationale |
|------------|-------------|----------------|-----------|
| Trigger the log write | agens `SKILL.md` body (skill logic) | — | Runs as the tail of the Step 3 recommendation branch; never a user-facing surface (LOG-02). |
| Perform the append | Git Bash via a scoped `allowed-tools` grant | — | `>>` redirection is the no-overwrite mechanism (D-02). Shell owns file I/O; the skill owns the decision to call it. |
| Store the log | wiki-agents vault, `99_Meta/agens-log.md` (new, vault-owned) | — | 99_Meta already holds tool/system files (schema.md, roadmap.md). A tool-authored log belongs beside them, not among human research notes. |
| Compute the vault-state stamp | Git Bash `sha256sum` over the cited note | — | Deterministic content hash; mirrors the schema's `wiki_hash` field. |
| Enforce append-only / no-overwrite | skill text + shell mechanism (`>>` only, never Write/Edit at runtime) | — | The guarantee is mechanical: no read-modify-write path exists at runtime. |
| Refuse direct log manipulation | agens `SKILL.md` fixed refusal block (D-03) | — | Closes the conversational trigger surface and the prompt-injection surface. |
| Govern the controlled `authored_by` field | `99_Meta/schema.md` §2 (human-authored meta note) | — | Schema is the vault's system of record for controlled vocabulary; it must admit the field before first use. |

## Standard Stack

### Core

| Component | Version | Purpose | Why Standard |
|-----------|---------|---------|--------------|
| Git Bash builtins (`printf`, `date`, `tail`, `grep`, `sha256sum`) | Bundled with Git for Windows (present) | Compose the append, the date header, the bounded read, the stamp, and the verification grep | Zero new dependency. Every command verified available in this environment. `date +%F` yields `2026-07-15`; `sha256sum` yields the content hash. |
| Claude Code `allowed-tools` frontmatter | Claude Code v2.1.196+ (project baseline) | Grants the scoped Bash write without a per-call permission prompt | The sanctioned mechanism for pre-approving a tool while a skill is active. `[CITED: code.claude.com/docs/en/skills]` |
| `${CLAUDE_SKILL_DIR}` / `${CLAUDE_PROJECT_DIR}` substitution | Claude Code v2.1.196+ | Resolve a bundled-script path portably across the two skill copies (fallback grant path) | Documented for the skill body; `${CLAUDE_PROJECT_DIR}` is documented to apply to `allowed-tools` too. `[CITED: code.claude.com/docs/en/skills]` |

### Supporting

| Component | Purpose | When to Use |
|-----------|---------|-------------|
| `disableSkillShellExecution` setting (awareness only) | If `true` in managed settings, replaces the skill's shell command with `[shell command execution disabled by policy]` | Note as an environment precondition. Not set here (personal single-user machine); the append mechanism requires skill shell execution to remain enabled. `[CITED: code.claude.com/docs/en/skills]` |

### Alternatives Considered

| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| Inline `>>` append in `SKILL.md` (D-01-faithful) | Bundled script `scripts/agens-log-append.sh` granted via `Bash(${CLAUDE_SKILL_DIR}/scripts/agens-log-append.sh *)` | The script hides redirection from the permission matcher (sidesteps the undocumented `>>` behaviour), gives the tightest documented grant form, and keeps date-header/hash/verify logic out of the 500-line SKILL.md body. But it adds a bundled file — heavier than D-01's stated lean (which even rejected a lazy-loaded reference file). Use only if Wave 0 shows inline-redirection matching is unreliable, or if body growth threatens the 500-line budget. |
| SHA-256 content hash as the vault-state stamp | Whole-vault `git rev-parse HEAD` | The cited index note is currently **untracked** in the vault's git (verified), so a commit hash would not reflect the note that actually grounded the recommendation. The content hash pins the exact bytes read. |
| SHA-256 content hash | File mtime (`stat -c %y`) | mtime is available but changes on any touch and carries no content meaning; a hash detects real content change and matches the schema's `wiki_hash` precedent. |

**Installation:** None. agens is markdown plus a shell append; the mechanism ships with Git for Windows already installed. The only "install" is placing the edited `SKILL.md` (and any bundled script) into both skill copies (D-04).

## Package Legitimacy Audit

**Not applicable.** Phase 4 installs no external package from any registry. It uses only Git Bash builtins already present on the machine (`printf`, `date`, `tail`, `grep`, `sha256sum`) and Claude Code's native `allowed-tools` mechanism. No npm, PyPI, or crates dependency is added, so slopcheck and registry verification have nothing to check. If planning later introduces a helper package, run the Package Legitimacy Gate before adding it.

## Architecture Patterns

### System Architecture Diagram

```
User describes a new agent project
          │
          ▼
   agens Step 0–2  (unchanged: detect → questionnaire → match → citation gate)
          │
          ▼
   Step 3: recommendation delivered to user  ◀── the recommendation is now OUT
          │
          ▼
   Logging tail (NEW, runs after the recommendation is shown)
          │
          ├─▶ Does 99_Meta/agens-log.md exist?
          │        │ no  ─▶ CREATE file: frontmatter (authored_by: agens, type: meta,
          │        │         status: permanent, topic: topic/meta) + title + attribution line
          │        │ yes ─▶ skip creation
          │        ▼
          ├─▶ Bounded read: tail the file; is the last "## YYYY-MM-DD" header == today?
          │        │ no  ─▶ append "## <today>"    (via >>)
          │        │ yes ─▶ no header needed
          │        ▼
          ├─▶ sha256sum(cited index note) ─▶ short hash  (the vault-state stamp)
          │        ▼
          ├─▶ append entry line (pattern | cited path @ hash | questionnaire answers)  (via >>)
          │        ▼
          └─▶ Verify: grep -Eq '^authored_by:[[:space:]]*agens[[:space:]]*$' agens-log.md
                   │ pass ─▶ done, say nothing further
                   │ fail ─▶ emit fixed log-failure notice (recommendation still stands)

   Direct request to write/edit/delete the log  ─▶  fixed D-03 refusal (no shell call)
   Framework-fit delegation path (D-09 tail rule)  ─▶  NOT logged (see Open Questions)
```

The diagram traces the primary use case: a recommendation flows through the unchanged Phase 1 spine, and the new tail records it without ever reading the log back as a grounding source. The only read of the log is the bounded `tail` that positions the day header — a mechanical file-position check, explicitly not a semantic grounding read.

### Recommended file/asset layout

```
.claude/skills/agens/              # project copy — single source of truth (D-04)
├── SKILL.md                       # + logging tail, + D-03 refusal block, + Bash grant
├── references/
│   ├── agent-authored-convention.md   # enforce as-is (marker, line, grep)
│   └── trigger-tests.md
└── scripts/                       # ONLY if the bundled-script fallback is chosen
    └── agens-log-append.sh
~/.claude/skills/agens/            # runtime copy — redeploy target (D-04); verify identical

wiki-agents/
├── 99_Meta/
│   ├── schema.md                  # + authored_by controlled field (edit BEFORE first use)
│   └── agens-log.md               # NEW — created on first recommendation
```

### Pattern 1: Frontmatter-init-on-first-write, then pure append

**What:** The first-ever recommendation creates `agens-log.md` with full schema-conformant frontmatter, the title, and the visible attribution line. Every subsequent write is a pure `>>` append of a header (if the day is new) and one entry line. No runtime path ever rewrites the file.

**When to use:** Always. This is the mechanical basis of LOG-01's no-overwrite guarantee and of the `authored_by` grep passing (the marker only exists because the init wrote it).

**Example:**
```bash
# Source: verified in this environment (Git Bash on Windows)
LOG="$VAULT_ROOT/99_Meta/agens-log.md"
if [ ! -f "$LOG" ]; then
  printf -- '---\ntype: meta\nstatus: permanent\ncreated: %s\ntopic:\n- topic/meta\nauthored_by: agens\n---\n\n# agens recommendation log\n\n> Agent-authored by agens. This note was written by an agent, not by hand.\n' "$(date +%F)" > "$LOG"
fi
```

### Pattern 2: One date header per day, placed by a bounded tail read

**What:** Before appending an entry, read only the tail of the file and compare the last `## YYYY-MM-DD` header to today. Emit a new header only when the day has changed.

**When to use:** Always. It groups a day's entries under one header (the date-header convention's intent) and gives criterion 3's "bounded read-back" a concrete, testable meaning.

**Example:**
```bash
# Source: verified in this environment
TODAY="$(date +%F)"
LAST_HEADER="$(grep -E '^## [0-9]{4}-[0-9]{2}-[0-9]{2}' "$LOG" | tail -n1)"
[ "$LAST_HEADER" = "## $TODAY" ] || printf -- '\n## %s\n' "$TODAY" >> "$LOG"
```

### Pattern 3: Content-hash vault-state stamp

**What:** Stamp each entry with a short SHA-256 of the cited index note, so the entry pins the exact version of the note that grounded the recommendation.

**When to use:** On every entry. This is the "stamps the vault state it referenced" criterion, grounded in the schema's own `wiki_hash` field (`schema.md` §2.7: "SHA-256 of the normalised note body").

**Example:**
```bash
# Source: verified in this environment
STAMP="$(sha256sum "$VAULT_ROOT/30_Concepts/agent-patterns-index.md" | cut -c1-12)"
printf -- '- **%s** — cited `30_Concepts/agent-patterns-index.md` @ %s — goal=%s workflow=%s sensitivity=%s latency=%s\n' \
  "$PATTERN" "$STAMP" "$GOAL" "$WORKFLOW" "$SENSITIVITY" "$LATENCY" >> "$LOG"
```

### Anti-Patterns to Avoid

- **Reading the log to inform a recommendation.** `agens-log.md` is an output sink, never an input. Step 2 matching reads only `30_Concepts/agent-patterns-index.md`. The bounded `tail` for header placement is a file-position check, not a grounding read — keep the two distinct in SKILL.md text (criterion 3: "never treated as a grounding source").
- **Write or Edit on the log at runtime.** Any read-modify-write breaks the no-overwrite guarantee (LOG-01 / criterion 1). Runtime writes are `>>` only.
- **Logging after a delegation.** D-09's tail rule says the delegated skill speaks last and agens adds nothing after it. A log write after a delegation would violate that rule. Log only the Step 3 recommendation branch (see Open Questions).
- **Granting unrestricted Bash.** D-02 requires the tightest grant the syntax allows. A bare `Bash` or `Bash(*)` grant reopens the write surface LOG-02 exists to close.
- **Skipping the schema update.** Writing `authored_by: agens` before `schema.md` admits the field violates the vault's own rule (a new controlled value requires a schema update first).

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Append without overwrite | A read-file → mutate-in-memory → rewrite-whole-file routine | Shell `>>` redirection | The rewrite path is the exact failure mode LOG-01 forbids; `>>` is atomic-append at the OS level and cannot clobber prior entries. |
| Vault-state fingerprint | A bespoke version counter or timestamp scheme | `sha256sum` (the `wiki_hash` precedent) | The vault already defines a content-hash field; a custom scheme diverges from convention and detects less. |
| Date header | String-building the date by hand | `date +%F` | One command, ISO `YYYY-MM-DD`, matches the convention exactly. |
| Marker verification | Substring search for `authored_by` | Anchored grep `^authored_by:[[:space:]]*agens[[:space:]]*$` | The convention doc pre-specifies the anchored form; a loose substring matches prose and passes falsely. |
| Scoping the write surface | A hand-rolled guard clause in prose only | `allowed-tools` `Bash(...)` grant + D-03 fixed refusal | The frontmatter grant is a real control; prose alone is not. |

**Key insight:** Every primitive this phase needs already exists — in Git Bash, in the vault schema, and in the pre-written attribution convention. The work is composition and conformance, not construction.

## Runtime State Inventory

> Phase 4 creates one new vault file and edits one existing meta note. It is a feature addition, not a rename. This trimmed inventory covers the state that a file-only audit would miss.

| Category | Items Found | Action Required |
|----------|-------------|------------------|
| Stored data | `99_Meta/agens-log.md` does **not exist yet** (verified: `99_Meta/` holds schema.md, roadmap.md, plugins.md, NotebookLM-bridge.md, and two subdirs — no agens-log.md). | First recommendation must CREATE the file with frontmatter; it cannot assume the file is present. |
| Live service config | Controlled-vocabulary state: `schema.md` §2 does **not** define `authored_by` (verified — §2.1–2.7 checked). The vault's own rule requires the schema admit a controlled value before first use. | Add `authored_by` to `schema.md` §2 as a plan task, BEFORE the first log write. This edit is a human-convention update to a human-authored meta note — it does NOT itself carry `authored_by: agens`. |
| OS-registered state | None. No scheduled task, service, or OS registration references the log. Verified — the capability is invoked only from agens' skill body. | None. |
| Secrets/env vars | None. The log records questionnaire answers, a pattern name, a note path, and a content hash — no secret, token, or personal data. Vault root resolves via the existing `--add-dir` grant (no new env var). | None. Confirm no secret ever reaches an entry line (input is the fixed four-option questionnaire, not free text). |
| Build artifacts / deployment | Two skill copies must stay identical (D-04). Currently **identical** (verified: `diff` of project vs `~/.claude` SKILL.md returns IDENTICAL; both carry the `references/` dir). | Redeploy the whole skill directory (SKILL.md + references + any new `scripts/`) and verify with a recursive directory compare, not a single-file diff. |

**Canonical question — after every repo file is updated, what runtime state still carries the old shape?** The log file itself, once created, becomes append-only runtime state: no later phase may rewrite it. The schema's controlled vocabulary is the other piece of live state — it gates whether the `authored_by` marker is valid vault content.

## Common Pitfalls

### Pitfall 1: The Bash grant does not match a redirecting command
**What goes wrong:** The plan grants `Bash(printf *)` expecting the append to run prompt-free, but `printf '...' >> file` carries a `>>` redirection the permission matcher may treat differently — the docs do not specify redirection handling. The append then prompts on every use, or the grant silently fails to match.
**Why it happens:** `printf` is a shell builtin (verified) and redirection is shell syntax, not part of the command token. The matcher's behaviour on redirection is undocumented `[CITED: code.claude.com/docs/en/settings — redirection handling not specified]`.
**How to avoid:** Test the exact grant-plus-command pair in Wave 0. If inline redirection does not match cleanly, adopt the bundled-script fallback — grant `Bash(<script-path> *)` (a single clean token per the documented `Bash(${CLAUDE_PROJECT_DIR}/scripts/lint.sh *)` pattern) and keep `>>` inside the script.
**Warning signs:** A permission prompt appears on the append despite the grant; or the append never runs.

### Pitfall 2: Colon-form grant syntax that the current docs do not use
**What goes wrong:** The plan copies D-02's `Bash(git:*)` analogy literally and the grant behaves unexpectedly.
**Why it happens:** Current official docs example the grant as `Bash(git add *)` — command prefix plus glob, not `command:*` with a colon `[CITED: code.claude.com/docs/en/skills]`. The colon form belongs to an older settings convention.
**How to avoid:** Use the documented `Bash(<prefix> *)` form and verify empirically. Do not assume the colon form from CLAUDE.md/D-02 is current.
**Warning signs:** The grant is present but the command still prompts.

### Pitfall 3: First write lands entries without frontmatter, and the grep gate fails
**What goes wrong:** The append runs before the file is initialised; `agens-log.md` gets entries but no `authored_by` frontmatter, and the anchored grep fails — a failed write per the convention.
**Why it happens:** Pure `>>` appends to the tail and never adds frontmatter; the frontmatter must be written by an explicit create step.
**How to avoid:** Guard the append with the file-exists init (Pattern 1). Run the grep after the write and treat a non-zero exit as a failed write.
**Warning signs:** The log opens without a `---` frontmatter block; the grep returns non-zero.

### Pitfall 4: The bounded read grows into a full-file load
**What goes wrong:** The plan reads the whole log to find today's header, defeating "bounded read-back" as the log grows.
**Why it happens:** `grep ... | tail -n1` still scans the whole file even though it returns one line.
**How to avoid:** Bound the input — `tail -n 50 "$LOG" | grep -E '^## ...' | tail -n1` reads a fixed window. The last header is always near the tail because writes only ever append.
**Warning signs:** Log-append latency scales with file size.

### Pitfall 5: Deployment drift recurs
**What goes wrong:** SKILL.md is redeployed but a new `scripts/` file or a references change is not, so the two copies diverge — the exact failure Phase 2 hit and D-04 exists to prevent.
**Why it happens:** A single-file `diff` misses sibling files.
**How to avoid:** Redeploy and verify the whole skill directory recursively (`diff -r` or a hash of every file), not SKILL.md alone.
**Warning signs:** `diff -r` between the two skill roots reports any difference.

## Code Examples

### Full logging tail (inline reference implementation)
```bash
# Source: composed from primitives verified in this environment (Git Bash on Windows)
# VAULT_ROOT resolved per SKILL.md's existing vault-root resolution rule.
LOG="$VAULT_ROOT/99_Meta/agens-log.md"
TODAY="$(date +%F)"

# 1. init on first write (frontmatter + attribution line)
if [ ! -f "$LOG" ]; then
  printf -- '---\ntype: meta\nstatus: permanent\ncreated: %s\ntopic:\n- topic/meta\nauthored_by: agens\n---\n\n# agens recommendation log\n\n> Agent-authored by agens. This note was written by an agent, not by hand.\n' "$TODAY" > "$LOG"
fi

# 2. bounded read: place today's header only if the day changed
LAST_HEADER="$(tail -n 50 "$LOG" | grep -E '^## [0-9]{4}-[0-9]{2}-[0-9]{2}' | tail -n1)"
[ "$LAST_HEADER" = "## $TODAY" ] || printf -- '\n## %s\n' "$TODAY" >> "$LOG"

# 3. vault-state stamp + entry append
STAMP="$(sha256sum "$VAULT_ROOT/30_Concepts/agent-patterns-index.md" | cut -c1-12)"
printf -- '- **%s** — cited `30_Concepts/agent-patterns-index.md` @ %s — goal=%s | workflow=%s | sensitivity=%s | latency=%s\n' \
  "$PATTERN" "$STAMP" "$GOAL" "$WORKFLOW" "$SENSITIVITY" "$LATENCY" >> "$LOG"

# 4. verify the marker; non-zero exit = failed write
grep -Eq '^authored_by:[[:space:]]*agens[[:space:]]*$' "$LOG" || echo "LOG_WRITE_FAILED"
```

### Documented grant forms to choose between (verify in Wave 0)
```yaml
# Space-separated allowed-tools is supported. [CITED: code.claude.com/docs/en/skills]
# Current example form for git scoping is prefix + glob, not colon:
allowed-tools: Read Grep Glob AskUserQuestion Skill Bash(git add *)

# Fallback: bundled-script grant — the documented path-scoped pattern.
# Verify ${CLAUDE_SKILL_DIR} resolves inside allowed-tools; ${CLAUDE_PROJECT_DIR} is documented to.
allowed-tools: Read Grep Glob AskUserQuestion Skill Bash(${CLAUDE_SKILL_DIR}/scripts/agens-log-append.sh *)
```

### Schema field addition (before first use)
```markdown
<!-- Add to wiki-agents/99_Meta/schema.md §2. Human-convention update; not agent-authored. -->
### 2.8 Agent-authorship marker

| Field | Enum values | Notes |
|---|---|---|
| `authored_by` | `agens` | Present only on notes an agent wrote. Absent on human-authored notes. |
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| `Bash(git:*)` colon-scope form (CLAUDE.md / D-02 analogy) | `Bash(git add *)` prefix-plus-glob form in the skills docs | Current docs (retrieved 2026-07-15) | Use the documented form; verify the grant empirically rather than trusting the colon analogy. |
| `_memory/recent.md` as the assumed vault log convention | `.remember/recent.md` is Claude Code's own session buffer, borrowed as a **format** only; the real target is a new `99_Meta/agens-log.md` | Phase 0 D-10/D-11 | Phase 4 creates a vault-owned file; it does not write into Claude Code's session buffer. |

**Deprecated/outdated:**
- Treating `git rev-parse HEAD` as the vault-state stamp: unreliable while the cited index note is untracked (verified). Use the content hash.

## Assumptions Log

| # | Claim | Section | Risk if Wrong |
|---|-------|---------|---------------|
| A1 | LOG-01's "each agens recommendation" covers successful recommendations only, not refusals or delegations. | Open Questions / Anti-Patterns | If the user wants refusals logged too, the plan needs a second write site. Delegation-logging is separately blocked by D-09's tail rule, so that half is a hard constraint, not merely an assumption. |
| A2 | The vault-state stamp should be a SHA-256 content hash of the cited note (the `wiki_hash` precedent). | Pattern 3 | If the user expects a git revision or a human-readable timestamp instead, the stamp format changes (mechanically trivial to swap). |
| A3 | One `## YYYY-MM-DD` header per day, entries oldest-first beneath it, is the right layout. | Pattern 2 | If the user wants newest-first (recent.md's display order), pure append cannot deliver it without a rewrite — which D-02 forbids. This tension needs a user decision. |
| A4 | Logging is best-effort after delivery and never blocks or retracts the recommendation; a failed write emits a fixed notice. | Open Questions | If the user wants a failed write to be louder or to gate anything, the failure-handling text changes. |
| A5 | `type: meta`, `status: permanent`, `topic: topic/meta` are the correct frontmatter for the log file (schema.md neighbour precedent). | Pattern 1 | If a different type/status is wanted, the init block changes; low risk given the 99_Meta precedent. |

**Not assumed (verified this session):** the two skill copies are identical; the log file does not exist; `schema.md` lacks `authored_by`; the index note is untracked in git; `allowed-tools` accepts a space-separated string with `Bash(...)` scoping; all shell primitives are present.

## Open Questions

1. **Do refusals and delegations get logged, or only successful recommendations?**
   - What we know: LOG-01 says "each agens recommendation." A refusal recommends nothing. A delegation hands off, and D-09's tail rule forbids agens adding anything after the target's output.
   - What's unclear: whether the user reads "each recommendation" narrowly (Step 3 success only) or broadly (every agens outcome).
   - Recommendation: log successful recommendations only. Delegation-logging is ruled out by D-09 regardless. Confirm the refusal case with the user; it is a one-line scope decision, not a design change.

2. **Newest-first or oldest-first entry order?**
   - What we know: `.remember/recent.md` displays newest-first; pure `>>` appends oldest-first at the tail; D-02 forbids any rewrite.
   - What's unclear: whether the user values the newest-first display enough to accept a mechanism other than pure append.
   - Recommendation: oldest-first append (honours D-02, no rewrite). If newest-first is required, raise it — it cannot be met by append alone.

3. **Does `${CLAUDE_SKILL_DIR}` resolve inside `allowed-tools`?**
   - What we know: the docs confirm `${CLAUDE_PROJECT_DIR}` applies to `allowed-tools`; `${CLAUDE_SKILL_DIR}` is documented for the body but not explicitly for the grant.
   - What's unclear: whether the skill-dir variable resolves in the grant, which matters only if the bundled-script fallback is chosen.
   - Recommendation: verify in Wave 0 if the fallback is on the table; otherwise the inline `>>` grant needs no path variable.

## Environment Availability

| Dependency | Required By | Available | Version | Fallback |
|------------|------------|-----------|---------|----------|
| Git Bash (`printf`, `date`, `tail`, `grep`, `sha256sum`) | The append, header, stamp, and verify | ✓ | Bundled with Git for Windows | — |
| Claude Code `allowed-tools` Bash scoping | The scoped write grant (D-02) | ✓ | v2.1.196+ (project baseline) | — |
| wiki-agents vault `--add-dir` grant | Resolving the write path `{vault_root}/99_Meta/agens-log.md` | ✓ | Existing grant (Phase 1 mechanism) | If unresolved, refuse per the existing vault-root rule — do not guess a path. |
| Skill shell execution enabled | Running any Bash from the skill | ✓ (default) | — | If `disableSkillShellExecution: true` in managed settings, the append is blocked by policy. Not set here. |

**Missing dependencies with no fallback:** None.
**Missing dependencies with fallback:** None blocking; the vault-root grant is the only precondition, and its absence routes to the existing plain refusal.

## Security Domain

> `security_enforcement` is absent from config → treated as enabled.

### Applicable ASVS Categories

| ASVS Category | Applies | Standard Control |
|---------------|---------|-----------------|
| V2 Authentication | no | No auth surface; single-user local tool. |
| V3 Session Management | no | No session state beyond the skill turn. |
| V4 Access Control | yes | LOG-02: the log write is reachable only from agens' own body — no command, no `Skill` target, no user-invocable surface. D-03 fixed refusal denies direct manipulation. Fail-closed: an unresolved vault root refuses rather than guessing a write path. |
| V5 Input Validation | yes | Entry fields come from the fixed four-option questionnaire and a verified pattern name, not free text. Confirm no free-text "Other" answer or note content is written verbatim into an entry line. |
| V6 Cryptography | n/a | `sha256sum` is used as a content fingerprint, not for a security property; no key material involved. |

### Known Threat Patterns for this stack

| Pattern | STRIDE | Standard Mitigation |
|---------|--------|---------------------|
| Prompt-injected vault note triggers an unwanted log write or edit | Tampering / Elevation | D-03 fixed refusal; the write runs only as the Step 3 tail, never on request. Log is never read back as input, so a poisoned entry cannot re-enter reasoning. |
| Log overwrite destroys prior entries | Tampering / Repudiation | `>>`-only mechanism; no runtime Write/Edit path; anchored `authored_by` grep confirms each write. |
| Unscoped Bash grant widens the write surface | Elevation of Privilege | Tightest documented `Bash(...)` grant; never bare `Bash`. |
| Path traversal on the write target | Tampering | Write path is the fixed vault-relative `99_Meta/agens-log.md` joined to the resolved vault root; no user input reaches the path. |
| Secret or personal data leaks into an entry | Information Disclosure | Entries carry only questionnaire answers, a pattern name, a note path, and a hash — validate that no free-text field is written verbatim. |

## Sources

### Primary (HIGH confidence)
- `code.claude.com/docs/en/skills` — `allowed-tools` accepts a space-/comma-separated string or YAML list; `Bash(git add *)` and `Bash(${CLAUDE_PROJECT_DIR}/scripts/lint.sh *)` scoping examples; `${CLAUDE_PROJECT_DIR}` applies to `allowed-tools` (v2.1.196+); `Skill(name)` / `Skill(name *)` permission forms; `disableSkillShellExecution`. Retrieved 2026-07-15.
- `wiki-agents/99_Meta/schema.md` — §2.1 required fields, §2.2 `type` enum (no `log`; `meta` is the fit), §2.7 `wiki_hash` = "SHA-256 of the normalised note body" (the stamp precedent), the controlled-vocabulary-before-use rule. Read on disk.
- `.claude/skills/agens/references/agent-authored-convention.md` — the marker, the attribution line, and the anchored verification grep, pre-specified. Read on disk.
- `.claude/skills/agens/SKILL.md` — current `allowed-tools` line, refusal/failure register, the unscopeable-`Skill`-grant anti-pattern. Read on disk (356 lines).
- Environment probes (Git Bash on Windows) — `date +%F`, `sha256sum`, `tail`, append+grep roundtrip, `diff` of the two skill copies (IDENTICAL), vault git state (index note UNTRACKED), absence of `agens-log.md`. Executed this session.

### Secondary (MEDIUM confidence)
- `code.claude.com/docs/en/settings` — confirms `Bash(...)` permission rules exist but does NOT specify redirection (`>>`), pipe, or command-chaining matching semantics. This gap is the basis of Pitfall 1.

### Tertiary (LOW confidence)
- None. No claim in this research rests on an unverified single web source.

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — every primitive verified present; the grant mechanism cited to official docs.
- Architecture: HIGH — grounded in the vault's own precedents (schema `wiki_hash`, 99_Meta neighbours, the pre-written attribution convention) and verified append mechanics.
- Pitfalls: HIGH for the redirection-matching gap (docs explicitly silent, hence flagged for Wave 0 test); HIGH for the rest.

**Research date:** 2026-07-15
**Valid until:** 2026-08-14 (stable domain; re-verify the `allowed-tools` Bash-scoping form if Claude Code minor version advances materially).
