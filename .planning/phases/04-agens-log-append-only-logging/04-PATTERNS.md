# Phase 4: agens-log (Append-Only Logging) - Pattern Map

**Mapped:** 2026-07-15
**Files analyzed:** 5 (2 modified, 1 runtime-created, 2 conditional/deployment)
**Analogs found:** 5 / 5

## Orientation

Phase 4 adds a write tail to a read-only skill. It installs no package. The work
is composition and conformance, not construction: every primitive already exists
in Git Bash, in the vault schema, in the pre-written attribution convention, and
in agens' own Phase 1/2 body. RESEARCH.md line 214 states this outright — "the
work is composition and conformance, not construction."

Five analog classes cover the phase:

1. **agens' own SKILL.md body** — the file D-01's inline step lands in. Its
   gate-before-showing citation check (Step 2b, lines 144-163), its
   resolve-first-refuse-plainly path rule (lines 30-39), and its two fixed
   refusal blocks (Step 3 lines 180-191, Delegate part 4 lines 311-324) are the
   in-repo precedents for the logging tail's verification gate, the log write
   path resolution, and the D-03 fixed refusal.
2. **CLAUDE.md's append ruling** — repo root `§Conventions for an Append-Only
   Log` and `§What NOT to Use` name `Bash >>` as the mechanism and reject
   rewrite-per-turn. This is the in-repo authority D-02 executes.
3. **The vault schema's field-table format** — `99_Meta/schema.md` §2.4-2.6 give
   the exact table shape the new `authored_by` field (§2.8) must copy.
4. **The 99_Meta neighbour frontmatter** — `schema.md` and `roadmap.md` share one
   frontmatter shape (`type: meta`, `status: permanent`, `created`, `topic:
   - topic/meta`). The new `agens-log.md` init block copies it, plus the
   `authored_by: agens` marker.
5. **The pre-written attribution convention** — `references/agent-authored-convention.md`
   supplies the marker, the visible attribution line, and the anchored
   verification grep verbatim. Phase 4 executes it; it does not redesign it.

**Analog files (all read for this map):**
- `C:/Users/Simon/claude-projects/agens/.claude/skills/agens/SKILL.md` — the file under edit; source of the gate, path-resolution, and fixed-refusal shapes (356 lines, near the 500-line budget).
- `C:/Users/Simon/claude-projects/agens/CLAUDE.md` — `§Conventions for an Append-Only Log`, `§What NOT to Use` (`Bash >>` named, rewrite rejected, 500-line guidance).
- `C:/Users/Simon/claude-projects/agens/.claude/skills/agens/references/agent-authored-convention.md` — the three-element marker convention and the anchored grep, pre-specified.
- `C:/Users/Simon/Documents/wiki-agents/99_Meta/schema.md` — §2.4-2.6 field-table format; §2.7 `wiki_hash` (the vault-state-stamp precedent); the controlled-value-before-use rule.
- `C:/Users/Simon/Documents/wiki-agents/99_Meta/roadmap.md` (frontmatter only) + `schema.md` frontmatter — the 99_Meta meta-note frontmatter shape the log-file init copies.
- `C:/Users/Simon/Documents/wiki-agents/.remember/recent.md` — the `## YYYY-MM-DD` date-header format precedent (format only, per Phase 0 D-10).

## File Classification

| New/Modified File | Role | Data Flow | Closest Analog | Match Quality |
|-------------------|------|-----------|----------------|---------------|
| `.claude/skills/agens/SKILL.md` (modified) | config (skill body + inline logging tail) | request-response + file-I/O (append) | agens' own Phase 1/2 body (gate, refusal, path resolution) | role-match (strong, same file) |
| `wiki-agents/99_Meta/schema.md` (modified) | config (controlled-vocabulary meta note) | transform (add one field row) | schema.md §2.4-2.6 field tables (self-analog) | exact |
| `wiki-agents/99_Meta/agens-log.md` (runtime-created) | data (append-only log sink) | file-I/O (append-only) | 99_Meta neighbour frontmatter + `.remember/recent.md` header format | role-match |
| `.claude/skills/agens/scripts/agens-log-append.sh` (conditional new) | utility (bundled shell script) | file-I/O (append) | none in repo (no `scripts/` dir exists) | no analog |
| `~/.claude/skills/agens/` (redeploy target) | deployment (runtime copy) | batch (directory sync) | Phase 2 D-04 deployment-sync history | role-match |

**Conditional file:** `scripts/agens-log-append.sh` exists only if Wave 0's
empirical test shows inline `>>` redirection does not match the `Bash(...)` grant
cleanly (RESEARCH.md Pitfall 1, Alternatives Considered line 86). The default,
D-01-faithful path is an inline append in SKILL.md with no bundled script. Treat
the script as a test-triggered fallback, not a required deliverable.

**Runtime-created file:** the planner does not write `agens-log.md` directly. The
skill body creates it on the first recommendation via the Pattern 1 init block.
The "analog" here is the frontmatter template the init block must emit, not a file
the plan authors.

## Pattern Assignments

### `.claude/skills/agens/SKILL.md` (config, request-response + file-I/O append)

Four edits land in this file. Each maps to a concrete analog excerpt below.

---

**Edit 1 — Frontmatter: extend `allowed-tools` with a scoped Bash grant** (agens SKILL.md line 14)

Current grant, to be extended:
```yaml
allowed-tools: Read Grep Glob AskUserQuestion Skill
```
Target: append one scoped `Bash(...)` specifier. RESEARCH.md line 293 gives the
documented prefix-plus-glob form to verify in Wave 0:
```yaml
allowed-tools: Read Grep Glob AskUserQuestion Skill Bash(git add *)
```
Copy the *form*, not the `git add` command — the phase needs a `printf`/`date`/
`sha256sum`/`tail`/`grep` append, scoped as tightly as the syntax allows.

**Analog for scoping discipline:** agens' own anti-pattern at SKILL.md lines
347-356 already documents that `allowed-tools` cannot scope the `Skill` tool to
one target. The same least-privilege reasoning governs the Bash grant: never a
bare `Bash` or `Bash(*)` (RESEARCH.md Anti-Patterns line 201, Security Domain
line 381). D-02 and CLAUDE.md name a `Bash(git:*)` colon analogy — RESEARCH.md
Pitfall 2 (line 238) and State of the Art (line 314) correct this: the current
docs use `Bash(git add *)`, not the colon form. Verify empirically before
relying on a friction-free append; the bundled-script grant
`Bash(${CLAUDE_SKILL_DIR}/scripts/agens-log-append.sh *)` is the documented
fallback (RESEARCH.md line 297). This is a required edit.

---

**Edit 2 — New logging tail section** (insert after Step 3's recommendation branch, before `## Delegate`, around current line 192)

The tail runs AFTER the recommendation is shown, on the Step 3 success branch
only. It composes four sub-patterns; each is verified in RESEARCH.md Code
Examples (lines 265-287) and carries an in-repo shape analog.

**2a. Frontmatter-init-on-first-write — analog: 99_Meta neighbour frontmatter.**
The first recommendation creates `agens-log.md` with schema-conformant
frontmatter, the title, and the visible attribution line. The frontmatter copies
the `schema.md` / `roadmap.md` meta-note shape:
```yaml
---
type: meta
status: permanent
created: 2026-07-15
topic:
- topic/meta
authored_by: agens
---
```
The attribution line copies `agent-authored-convention.md` §2 verbatim:
```
> Agent-authored by agens. This note was written by an agent, not by hand.
```
Guard the append behind a file-exists check (RESEARCH.md Pattern 1, Pitfall 3):
the marker exists only because the init wrote it, so the verification grep can
only pass on an initialised file.

**2b. One date header per day via bounded tail read — analog:
`.remember/recent.md` header format.** The format precedent is `## YYYY-MM-DD`
(Phase 0 D-10: format only, not a vault convention). Read only a fixed tail
window to place the header (RESEARCH.md Pattern 2, Pitfall 4):
```bash
LAST_HEADER="$(tail -n 50 "$LOG" | grep -E '^## [0-9]{4}-[0-9]{2}-[0-9]{2}' | tail -n1)"
[ "$LAST_HEADER" = "## $TODAY" ] || printf -- '\n## %s\n' "$TODAY" >> "$LOG"
```
This bounded `tail` is a file-position check, NOT a grounding read. Keep the
distinction explicit in SKILL.md text — criterion 3: the log is "never treated as
a grounding source" (RESEARCH.md Anti-Patterns line 198).

**2c. Content-hash vault-state stamp — analog: schema.md §2.7 `wiki_hash`.** The
schema already defines a content hash ("SHA-256 of the normalised note body",
schema.md line 141). Stamp each entry with a short SHA-256 of the cited index
note (RESEARCH.md Pattern 3):
```bash
STAMP="$(sha256sum "$VAULT_ROOT/30_Concepts/agent-patterns-index.md" | cut -c1-12)"
```
A git revision does not serve — the cited index note is untracked in the vault's
git (RESEARCH.md line 87, verified). The content hash pins the exact bytes read.

**2d. Verify the marker — analog: `agent-authored-convention.md` §3 grep,
verbatim.** The convention doc pre-specifies the anchored form (line 66); copy it
exactly, and treat a non-zero exit as a failed write:
```bash
grep -Eq '^authored_by:[[:space:]]*agens[[:space:]]*$' "$LOG" || echo "LOG_WRITE_FAILED"
```
The anchor `^authored_by:` binds to the frontmatter field at line start; a loose
substring matches prose and passes falsely (RESEARCH.md Don't Hand-Roll line
211). Run this grep as a gate in the Phase 1 gate-before-showing style — the same
"run the check, non-zero routes to the failure path" shape as Step 2b (SKILL.md
lines 144-163).

**Vault-root resolution — analog: SKILL.md lines 30-39.** The write path
`{vault_root}/99_Meta/agens-log.md` resolves the same way Phase 1 resolves the
read path: determine the vault root from the granted additional directories, and
refuse plainly if no granted directory contains the vault. No new path mechanism
is needed (CONTEXT.md Reusable Assets). An unresolved vault root fails closed to a
refusal, never guesses a write path (RESEARCH.md Security Domain line 371).

---

**Edit 3 — D-03 fixed refusal block** (add to the logging section or the anti-patterns region)

A direct conversational request to write, edit, or delete the log outside a
recommendation flow gets a fixed, scripted refusal.

**Analog: agens' two existing fixed-refusal blocks.** Step 3's plain refusal
(SKILL.md lines 180-191) and the Delegate part 4 failure message (lines 311-324)
set the register: fixed text, name what is closed, offer no partial substitute.
```
No pattern in the wiki-agents vault matches this shape.

I searched agent-patterns-index.md for a pattern whose trigger fits:
- goal: {goal}
...
None resolved. I will not cite a loosely related pattern to fill the gap.
```
For D-03, keep the discipline and swap the body: state that logging runs only as
the tail of a recommendation, is not a command, and agens will not append, edit,
or delete entries on request (CONTEXT.md D-03). A fixed string gives verification
an exact assertion target and defends against a prompt-injected vault note
attempting to trigger a write (RESEARCH.md Security Domain line 379). Wording is
Claude's discretion, in the Phase 1 refusal register (CONTEXT.md Claude's
Discretion).

---

**Edit 4 — Extend `## Anti-patterns`** (agens SKILL.md lines 326-356)

The existing entries model the form: a bold name, then a sentence naming the
failure and why it destroys the skill's value.
```markdown
- **Un-gated model-knowledge recommendation.** Never recommend a pattern without
  the Step 2b Read+Grep gate passing. Recommending from model recall alone
  destroys agens' core value — a grounded, verifiable citation every time.
```
Add the Phase 4 anti-patterns in the same form (RESEARCH.md Anti-Patterns lines
196-202): reading the log to inform a recommendation, Write/Edit on the log at
runtime, logging after a delegation (D-09 tail rule), granting unrestricted Bash,
and skipping the schema update.

---

### `wiki-agents/99_Meta/schema.md` (config, transform — one field row)

**Analog:** `schema.md` §2.4-2.6, the file's own field-table format (self-analog).
Add `authored_by` as a new subsection BEFORE first use, per the schema's own rule
that a new controlled value requires a schema update first (schema.md lines
13-14). RESEARCH.md line 307 gives the row:
```markdown
### 2.8 Agent-authorship marker

| Field | Enum values | Notes |
|---|---|---|
| `authored_by` | `agens` | Present only on notes an agent wrote. Absent on human-authored notes. |
```
This edit is a human-convention update to a human-authored meta note. It does NOT
itself carry `authored_by: agens` (RESEARCH.md Runtime State Inventory line 223).
Match the existing table columns exactly (`Field | Enum values | Notes`), the
same three-column shape §2.1 and §2.8 share.

---

### `wiki-agents/99_Meta/agens-log.md` (data, file-I/O append-only — runtime-created)

**Analog:** the 99_Meta neighbour frontmatter (`schema.md`, `roadmap.md`) for the
init block, and `.remember/recent.md` for the `## YYYY-MM-DD` header layout. The
planner does not author this file; the SKILL.md init block (Edit 2a) emits it on
first write. The pattern to copy is the frontmatter template above. 99_Meta is the
correct home — it already holds tool/system files (schema.md, roadmap.md,
plugins.md), so a tool-authored log belongs beside them, not among human research
notes (RESEARCH.md Architectural Responsibility Map line 60).

---

### `.claude/skills/agens/scripts/agens-log-append.sh` (utility, conditional)

**No in-repo analog.** No `scripts/` directory exists under the skill. Author only
if Wave 0 shows inline `>>` does not match the grant, or if body growth threatens
the 500-line budget (RESEARCH.md Alternatives Considered line 86). The script
would hold the full logging tail (RESEARCH.md Code Examples lines 265-287) behind
a single clean grant token `Bash(${CLAUDE_SKILL_DIR}/scripts/agens-log-append.sh *)`.
See No Analog Found below.

## Shared Patterns

### Gate-before-acting, verify-then-continue
**Source:** `agens/SKILL.md` lines 144-163 (Step 2b citation gate)
**Apply to:** the logging tail's `authored_by` verification grep (Edit 2d)
Phase 1 proves the shape: run the check, and a failure routes to a defined path.
The log's post-write grep reuses it — a non-zero exit is a failed write and emits
the fixed log-failure notice, while the already-delivered recommendation stands
(RESEARCH.md Assumption A4, best-effort logging). This is the same gate idiom, not
a new one (CONTEXT.md Established Patterns).

### Fixed, plain, scripted text for every closed path
**Source:** `agens/SKILL.md` lines 180-191 (Step 3 refusal) and 311-324 (Delegate failure)
**Apply to:** the D-03 direct-manipulation refusal (Edit 3) and the log-failure notice
agens never improvises a refusal. Both existing blocks name what is closed and
offer no substitute. D-03 copies the register: logging is not a command, and agens
will not append, edit, or delete on request. A fixed string is the verification
assertion target (CONTEXT.md D-03, Reusable Assets).

### Resolve-first, refuse-on-uncertainty path handling
**Source:** `agens/SKILL.md` lines 30-39 (vault-root resolution rule)
**Apply to:** the log write path `{vault_root}/99_Meta/agens-log.md`
Resolve the write path against the granted additional directory, exactly as Phase
1 resolves the read path; hard-code no absolute vault root. An unresolved root
fails closed to a refusal, never a guessed write path (RESEARCH.md Security Domain
line 371). No new path mechanism is introduced (CONTEXT.md Reusable Assets).

### Append-only via shell `>>`, never read-modify-write
**Source:** `CLAUDE.md` §Conventions for an Append-Only Log, §What NOT to Use
**Apply to:** every runtime write to the log (Edit 2)
CLAUDE.md names `Bash >>` as the mechanism and rejects rewrite-per-turn. The
guarantee is mechanical: no Write or Edit path touches the log at runtime, so no
prior entry can be clobbered (RESEARCH.md Don't Hand-Roll line 208, LOG-01). This
is the in-repo authority D-02 executes.

### Content-hash as the vault-state stamp
**Source:** `wiki-agents/99_Meta/schema.md` §2.7 `wiki_hash` (line 141)
**Apply to:** every log entry's stamp (Edit 2c)
The schema already defines "SHA-256 of the normalised note body." The stamp reuses
this precedent rather than inventing a version scheme; a git revision fails
because the cited index note is untracked (RESEARCH.md lines 87, 209).

### Agent-authorship marker, pre-specified
**Source:** `references/agent-authored-convention.md` §1-3
**Apply to:** the log init frontmatter (Edit 2a), the attribution line, and the verify grep (Edit 2d)
The marker (`authored_by: agens`), the visible attribution line, and the anchored
grep are pre-written. Phase 4 executes them verbatim; it does not redesign them
(CONTEXT.md Canonical References). The schema must admit the field first (schema.md
edit above).

### Redeploy-and-verify the whole skill directory
**Source:** Phase 2 D-04 deployment-sync history; RESEARCH.md Pitfall 5 (line 256)
**Apply to:** the `~/.claude/skills/agens/` redeploy task (D-04)
Phase 2 hit deployment drift and reconciled it. Verify the two skill copies with a
recursive directory compare (`diff -r` or a per-file hash), not a single-file
`diff` — a single-file check misses a new `scripts/` file or a references change
(RESEARCH.md Pitfall 5). Both copies currently carry `SKILL.md` + `references/`
and are identical (verified this map).

## No Analog Found

Files or concerns with no close in-repo match. The planner draws these from
RESEARCH.md and the vault, not from a copied file.

| File / Concern | Role | Data Flow | Reason no analog | Where it comes from |
|----------------|------|-----------|------------------|---------------------|
| `scripts/agens-log-append.sh` (conditional) | utility | file-I/O | No `scripts/` directory exists under any skill in this repo; agens has never bundled an executable | RESEARCH.md Code Examples lines 265-287 (the full tail) + line 297 (the grant form); author only if Wave 0 demands it |
| The `Bash(...)` grant form itself | config | — | No prior agens phase granted Bash; the docs' redirection-matching behaviour is undocumented | RESEARCH.md Pitfall 1-2, State of the Art line 314 — verify empirically in Wave 0, do not copy the CLAUDE.md `Bash(git:*)` colon analogy |
| Refusal/delegation logging scope | — | — | LOG-01 says "each agens recommendation"; whether refusals/delegations are logged is undecided (not defaulted) | CONTEXT.md "Not discussed this session"; RESEARCH.md Open Question 1 / Assumption A1 — recommendation: log successful recommendations only; confirm the refusal case with the user |
| Newest-first vs oldest-first entry order | — | — | `.remember/recent.md` displays newest-first; pure `>>` is oldest-first; D-02 forbids the rewrite newest-first would need | RESEARCH.md Open Question 2 / Assumption A3 — recommendation: oldest-first append; raise to user if newest-first is required |

## Metadata

**Analog search scope:**
- In-repo: `C:/Users/Simon/claude-projects/agens/.claude/skills/agens/` (SKILL.md under edit + `references/`) and `CLAUDE.md` (append ruling).
- Vault: `C:/Users/Simon/Documents/wiki-agents/99_Meta/` (schema.md field tables + frontmatter, roadmap.md frontmatter) and `.remember/recent.md` (header format).
- Deployment: `C:/Users/Simon/.claude/skills/agens/` — runtime copy confirmed present with `SKILL.md` + `references/`.

**Verified this map:** the runtime copy exists and mirrors the project copy's structure; `agens-log.md` does not yet exist in `99_Meta/`; `schema.md` lacks `authored_by`; the `## YYYY-MM-DD` header format precedent is `.remember/recent.md`; SKILL.md is 356 lines (near the 500-line budget).

**Files scanned:** 6 analogs read (SKILL.md, CLAUDE.md, agent-authored-convention.md, schema.md, roadmap.md frontmatter, recent.md); 1 directory listing of the runtime copy.
**Pattern extraction date:** 2026-07-15
