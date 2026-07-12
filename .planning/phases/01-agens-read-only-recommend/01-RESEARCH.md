# Phase 1: agens (Read-Only Recommend) - Research

**Researched:** 2026-07-12
**Domain:** Claude Code Skill authoring — auto-trigger description craft, `AskUserQuestion` intake, deterministic citation verification, plain refusal, agent-authorship tagging
**Confidence:** HIGH

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

**Questionnaire delivery**
- **D-01:** All four dimensions (goal, workflow shape, data sensitivity, latency/cost) are asked in a single `AskUserQuestion` turn — one multi-question call — not sequential single-dimension turns and not an infer-then-confirm flow. Keeps the "same four questions every time" guarantee (RECOMMEND-03) visibly literal to the user.
- **D-02:** Every dimension uses fixed multiple-choice options, not free text. Answers must be reproducible and directly matchable against the vault's pattern trigger/trade-off text — RECOMMEND-05's citation-resolves check needs a resolvable match, not prose to interpret.

**Dimension option sets**
- **D-03:** Goal is bucketed by capability shape, mirroring the pattern families already in `agent-patterns-index.md` — e.g. "Summarise/transform content", "Answer questions over data", "Automate a multi-step task", "Orchestrate multiple specialists" — so the goal answer maps close to a pattern family instead of needing separate free-text interpretation.
- **D-04:** Workflow shape: "Fixed workflow (predictable steps)" / "Autonomous agent (open-ended, self-directed)" / "Not sure" — mirrors the vault's own `workflow-vs-autonomous-agent.md` concept note directly.
- **D-05:** Data sensitivity: "Public" / "Internal" / "Sensitive or regulated".
- **D-06:** Latency/cost: "Real-time (sub-second)" / "Interactive (seconds)" / "Batch (minutes+, cost-optimised)".

### Claude's Discretion
- Whether agens echoes the four answers back for confirmation before running the vault lookup, or proceeds straight to matching — not decided this session. Default: proceed straight to matching without a separate echo step. The `AskUserQuestion` UI already shows the user's selections before submission; a redundant confirm adds a turn without adding information. Revisit if research surfaces a reason confirmation matters (e.g. users frequently mis-click).
- Exact final wording of the goal-bucket options beyond the four examples in D-03 (including whether a catch-all "something else" bucket is needed) — left to research/planning, guided by the pattern-family language already in `agent-patterns-index.md`.

### Not Decided This Session (undecided, not defaulted — do not assume a trivial answer)
- **Match & refusal threshold** — the concrete rule for what counts as a citation-resolves match (RECOMMEND-05) vs. a "loosely related" citation agens must refuse (RECOMMEND-06).
- **RECOMMEND-07 write scope** — whether Phase 1 writes anything to the vault at all, or whether RECOMMEND-07 only sets the agent-authored tagging convention that Phase 4's logging actually uses.
- **Recommendation output format** — how much of the matched entry gets quoted back, and whether more than one pattern can be surfaced when several genuinely fit.

### Deferred Ideas (OUT OF SCOPE)
None — discussion stayed within phase scope. No scope creep occurred.
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| RECOMMEND-01 | User can invoke agens via an explicit slash command | Directory `.claude/skills/agens/` yields `/agens`; command name comes from directory, not `name` field [CITED: code.claude.com/docs/en/skills]. See Standard Stack + Pattern 1. |
| RECOMMEND-02 | User can invoke agens via auto-trigger, without a command | `description` (plus optional `when_to_use`) drives auto-trigger; leave `disable-model-invocation`/`user-invocable` unset. See Pattern 1 + Pitfall 1. |
| RECOMMEND-03 | User answers a fixed four-dimension questionnaire before any recommendation | Single `AskUserQuestion` call carries exactly 4 questions — inside the tool's 1-4 limit [CITED: code.claude.com/docs/en/agent-sdk/user-input]. See Pattern 2. |
| RECOMMEND-04 | Recommendation cites a resolvable vault note path with the supporting passage quoted | `agent-patterns-index.md` bold-name + trigger + trade-off entry format supplies the quotable passage. See Pattern 3. |
| RECOMMEND-05 | agens verifies the cited path exists and the pattern appears in it before returning | Two-step Read-then-Grep verification. See Pattern 3 + Code Examples. |
| RECOMMEND-06 | agens refuses plainly, with no loosely related citation, when nothing supports the shape | Fixed refusal template; match threshold rule. See Pattern 4 + Open Questions. |
| RECOMMEND-07 | Every note agens writes into the vault is tagged as agent-authored | Phase 1 writes nothing to the vault; requirement holds vacuously and defines the convention Phase 4 enforces. See "RECOMMEND-07 Scope Resolution". |
</phase_requirements>

## Summary

Phase 1 authors one Claude Code Skill: a directory `.claude/skills/agens/` holding `SKILL.md`, using only built-in tools. No package installs, no runtime, no build step. The four mechanics the phase must get right are all supported and documented in current official docs: dual invocation through the `description` field, a single four-question `AskUserQuestion` call, a deterministic Read-then-Grep citation check, and a fixed refusal template. Every mechanic verified against the Claude Code Skills docs and the Agent SDK user-input docs, both retrieved 2026-07-12.

Two findings change the plan. First, `AskUserQuestion` always offers the user a free-text "Other" option that the skill cannot suppress at the tool level; D-02's "fixed multiple-choice, no free text" is not fully enforceable, so agens must handle a free-text answer by refusing or re-mapping rather than assuming a clean bucket. Second, the vault schema carries no agent-authorship field today, so RECOMMEND-07's "tagged as agent-authored" convention does not yet exist and must be defined — but Phase 1 writes nothing to the vault, so the requirement holds vacuously and its real job is to fix the convention Phase 4's `agens-log` will enforce.

The single lookup target — `wiki-agents/30_Concepts/agent-patterns-index.md` — exists (verified on disk, 8,111 bytes, 21 bold pattern entries, `- **Name** — trigger… Trade-off: …` shape). Claude Code 2.1.207 is installed, clearing the v2.1.196 floor for `${CLAUDE_SKILL_DIR}` and the `disable-model-invocation` scheduled-task behaviour.

**Primary recommendation:** Author `agens` as a single top-level Skill with an auto-trigger-tuned `description`, no invocation-control fields, and `allowed-tools: Read Grep Glob AskUserQuestion`. Ask all four dimensions in one `AskUserQuestion` call, then gate every recommendation behind a Read (path resolves) + Grep (bold pattern name present) check, quoting the matched trigger/trade-off sentence. Refuse from a fixed template when no bold entry matches. Write nothing to the vault this phase.

## Architectural Responsibility Map

The tiers below adapt the standard multi-tier frame to a single-Skill capability. Each "tier" is a conceptual stage inside the one `SKILL.md`, not a separate process.

| Capability | Primary Tier | Secondary Tier | Rationale |
|------------|-------------|----------------|-----------|
| Explicit `/agens` command (RECOMMEND-01) | Invocation surface (Claude Code skill loader) | — | Command name derives from the skill directory name; the loader owns it, not skill body logic. |
| Auto-trigger on natural phrasing (RECOMMEND-02) | Invocation surface (`description` matching) | — | Claude matches the request against the `description` listing before the body loads; body logic cannot influence whether it triggers. |
| Fixed four-dimension intake (RECOMMEND-03) | Intake (`AskUserQuestion` tool) | Skill body (renders the call) | The tool owns the structured-question UI; the body only specifies the four questions and their options. |
| Vault lookup + match (RECOMMEND-04) | Retrieval (Grep/Read over `agent-patterns-index.md`) | Skill body (maps answers → pattern family) | ripgrep over one 8 KB note is the correct, sufficient retrieval mechanism at this corpus size. |
| Citation verification (RECOMMEND-05) | Verification (Read path + Grep pattern name) | Skill body (gates output on the result) | A deterministic filesystem+content check, not model judgement — belongs to the tool layer, invoked by the body. |
| Plain refusal (RECOMMEND-06) | Skill body (fixed template) | Verification (supplies the no-match signal) | The refusal wording is body logic; the trigger for it is the verification step returning no match. |
| Agent-authorship tagging (RECOMMEND-07) | Skill body (convention definition) | — | No vault write happens this phase; the body records the convention for Phase 4. |

## Standard Stack

### Core

| Component | Version | Purpose | Why Standard |
|-----------|---------|---------|--------------|
| `SKILL.md` in `.claude/skills/agens/` | Claude Code 2.1.207 (installed) | The skill substrate — directory becomes `/agens` and an auto-triggerable capability | Project constraint is Anthropic-ecosystem-only; a Skill is the native unit. [CITED: code.claude.com/docs/en/skills] |
| YAML frontmatter (`description`, `allowed-tools`) | Agent Skills open standard | Declares auto-trigger text and pre-approved tools | `description` drives auto-trigger; `allowed-tools` removes the per-call permission prompt. [CITED: code.claude.com/docs/en/skills] |
| Built-in `Grep` | Claude Code current | Match a bold pattern name inside `agent-patterns-index.md` | ripgrep-backed, regex, sufficient at 8 KB / 21 entries. No index needed. |
| Built-in `Read` | Claude Code current | Confirm the vault-relative path resolves; read the matched passage to quote it | Deterministic file read — the citation-discipline requirement demands reading the actual note. |
| Built-in `Glob` | Claude Code current | Confirm a path exists without loading the whole file | Cheap existence check for the "path resolves" half of RECOMMEND-05. |
| Built-in `AskUserQuestion` | Claude Code current | The fixed four-dimension questionnaire in one call | The native structured-question primitive; supports 1-4 questions, 2-4 options each. [CITED: code.claude.com/docs/en/agent-sdk/user-input] |

### Supporting

| Convention | Field / File | Purpose | When to Use |
|------------|--------------|---------|-------------|
| Auto-trigger tuning | `description` (+ optional `when_to_use`) | Front-load natural project-starting phrasings the author did not write | Always. Combined `description` + `when_to_use` capped at 1,536 characters in the listing; put the key trigger first. [CITED: code.claude.com/docs/en/skills] |
| Both invocation paths | leave `disable-model-invocation` and `user-invocable` unset | Gives `/agens` and auto-trigger together | Always for the top-level agens skill. Setting either breaks one of RECOMMEND-01/RECOMMEND-02. [CITED: code.claude.com/docs/en/skills] |
| Pre-approved read tools | `allowed-tools: Read Grep Glob AskUserQuestion` | Read and search the vault, and ask the questionnaire, without a permission prompt each call | On this read-only recommend capability. Keeps the loop friction-free while staying read-only. |
| Bundled reference file | `references/*.md` in the skill dir | Move questionnaire text, citation-format rules, and refusal template out of `SKILL.md` if it nears 500 lines | Only if the body grows; reference the file so Claude loads it on demand. [CITED: code.claude.com/docs/en/skills] |

### Alternatives Considered

| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| Single top-level `agens` skill | Split `agens` (read) + `agens-build` (gated write) | The split matters at Phase 3 (BUILD-01/02), not Phase 1. Phase 1 ships read-only; a split now adds structure with no write capability to isolate. |
| Grep/Glob over the vault | Local embedding index (sqlite-vec, LanceDB) | Overkill at 547 KB / 152 notes; the lookup target is a single 8 KB note. Out of scope per REQUIREMENTS.md. |
| `description`-driven auto-trigger | `paths:` glob activation | `paths:` triggers on file edits, not on conversation describing a new agent project; wrong mechanism for RECOMMEND-02. |

**Installation:** No package manager. agens is markdown. "Installation" is placing `.claude/skills/agens/SKILL.md` and granting the vault directory:
```bash
# In the agens project, grant read access to the vault:
claude --add-dir "C:/Users/Simon/Documents/wiki-agents"
# or /add-dir in-session. A .claude/skills/ inside an --add-dir directory loads automatically;
# permissions.additionalDirectories in settings.json does NOT load skills. [CITED: code.claude.com/docs/en/skills]
```

**Version verification:** Claude Code 2.1.207 installed (`claude --version`, verified 2026-07-12). Clears the v2.1.196 floor for `${CLAUDE_SKILL_DIR}`/`${CLAUDE_PROJECT_DIR}` substitution and the `disable-model-invocation` scheduled-task guard. [VERIFIED: claude --version]

## Package Legitimacy Audit

**N/A — no external packages.** agens is authored as markdown `SKILL.md` files driven by built-in Claude Code tools (Read, Grep, Glob, AskUserQuestion). No npm, PyPI, or crates dependency is installed in this phase. slopcheck, `npm view`, and postinstall checks do not apply. The only external artefacts agens depends on are the installed Claude Code binary (2.1.207, verified) and the wiki-agents vault (verified on disk).

## Architecture Patterns

### System Architecture Diagram

```
User request
  │
  ├─ types "/agens"  ─────────────┐        (RECOMMEND-01: directory name → command)
  │                               │
  └─ describes a new agent        │
     project in conversation ─────┤        (RECOMMEND-02: description match → auto-load)
                                   ▼
                          [ agens SKILL.md loads ]
                                   │
                                   ▼
                 [ AskUserQuestion: 4 questions, 1 call ]   (RECOMMEND-03)
                 goal · workflow shape · data sensitivity · latency/cost
                                   │  answers (labels)
                                   ▼
                 [ Map answers → candidate pattern family ]
                                   │
                                   ▼
        ┌──────────── [ Verification gate ] ────────────┐
        │  A. Read/Glob the note path → resolves?        │  (RECOMMEND-05)
        │  B. Grep bold "**Pattern**" in the note → hit? │
        └───────────────────────────────────────────────┘
              │ both pass                    │ either fails / no match
              ▼                              ▼
   [ Recommendation ]              [ Plain refusal ]        (RECOMMEND-06)
   path + quoted trigger/          no citation, names
   trade-off passage               what was searched
   (RECOMMEND-04)
                                   │
                                   ▼
              [ Phase 1 writes NOTHING to the vault ]       (RECOMMEND-07 vacuous;
               convention defined for Phase 4 agens-log)     Phase 4 = LOG-01)
```

### Recommended Project Structure
```
.claude/skills/agens/
├── SKILL.md                    # frontmatter (auto-trigger description) + body
└── references/                 # optional, only if SKILL.md nears 500 lines
    ├── questionnaire.md         # the four dimensions + option sets
    ├── citation-check.md        # the Read-then-Grep verification procedure
    └── refusal.md               # the fixed refusal template
```

### Pattern 1: Dual-invocation skill with auto-trigger description
**What:** One top-level skill invocable by `/agens` and auto-loaded when conversation describes a new agent project.
**When to use:** RECOMMEND-01 and RECOMMEND-02 together.
**How:**
- The directory name `agens` produces `/agens`. Keep `name: agens` consistent with the directory (a mismatch only relabels the listing, but avoid it). [CITED: code.claude.com/docs/en/skills]
- Do NOT set `disable-model-invocation` (would kill auto-trigger, breaking RECOMMEND-02) and do NOT set `user-invocable: false` (would kill the `/agens` command, breaking RECOMMEND-01). The default — both unset — is exactly the requirement. [CITED: code.claude.com/docs/en/skills]
- Auto-trigger quality is entirely a function of the `description` text. Model it on `find-skills`, whose description lists natural user phrasings ("how do I do X", "find a skill for X", "is there a skill that can…") rather than the skill's own internal vocabulary. [VERIFIED: ~/.claude/skills/find-skills/SKILL.md]

```yaml
# Source: code.claude.com/docs/en/skills (frontmatter reference) + find-skills description craft
---
name: agens
description: >-
  Recommends a citation-grounded agent design pattern from the wiki-agents vault.
  Use when the user is planning, scoping, or starting a new agent, assistant, bot,
  or automation project and asks which approach or pattern fits — for example
  "I want to build an agent that…", "how should I structure this automation",
  "what pattern fits a bot that answers questions over our docs", or "help me
  design an assistant that…".
allowed-tools: Read Grep Glob AskUserQuestion
---
```

**Verification note (per STATE.md blocker):** the auto-trigger phrasing test set must be built from realistic project-starting phrasings independent of the description's own wording. The skill-creator plugin automates a should-trigger / should-not-trigger comparison and proposes description edits when the skill fires on the wrong requests. [CITED: code.claude.com/docs/en/skills] This is a verification-design task, not a coding task.

### Pattern 2: Single four-question AskUserQuestion call
**What:** All four dimensions asked in one tool call (D-01).
**When to use:** RECOMMEND-03.
**Constraints (verified):** `questions` is an array of 1-4 questions; each has `question` (text), `header` (max 12 characters), `options` (2-4, each `label` + `description`), and `multiSelect`. [CITED: code.claude.com/docs/en/agent-sdk/user-input] Four dimensions fit the 1-4 limit exactly.

Header-length check (max 12 chars) against the locked option sets:
| Dimension | Proposed header | Length | Verdict |
|-----------|-----------------|--------|---------|
| Goal (D-03) | `Goal` | 4 | OK |
| Workflow shape (D-04) | `Workflow` | 8 | OK |
| Data sensitivity (D-05) | `Sensitivity` | 11 | OK (`Data sensitivity` = 16 → too long, do not use) |
| Latency/cost (D-06) | `Latency` | 7 | OK (`Latency/cost` = 12 is at the limit; `Latency` is safer) |

Option-count check: goal 4 options (D-03), workflow 3 (D-04), sensitivity 3 (D-05), latency 3 (D-06) — all inside 2-4. All four fit.

**Anti-pattern this pattern avoids:** sequential single-dimension turns (violates D-01) and infer-then-confirm (violates D-01).

### Pattern 3: Deterministic citation verification (Read path + Grep pattern name)
**What:** A two-step, checkable gate run before any recommendation is shown.
**When to use:** RECOMMEND-04 and RECOMMEND-05.
**Procedure:**
1. **Path resolves (existence).** Resolve the vault-relative path `30_Concepts/agent-patterns-index.md` against the granted vault root and Read (or Glob) it. If Read errors or Glob returns empty, the path does not resolve — stop, do not cite. "Verifies the path exists" = a Read/Glob call that returns content, not model recall.
2. **Pattern name appears (content).** Grep the note for the bold pattern-name literal, e.g. `\*\*Routing\*\*`. The index entry format is `- **Pattern Name** — <trigger>. Trade-off: <trade-off>.` A Grep hit confirms the recommended pattern is present as an actual bold entry, not a hallucination. [VERIFIED: wiki-agents/30_Concepts/agent-patterns-index.md]
3. **Quote the passage.** Read the matched line and quote its trigger + trade-off sentence verbatim as the supporting passage alongside the path (RECOMMEND-04).

Only when steps 1 and 2 both pass does agens present the recommendation. Either failure routes to Pattern 4 (refusal).

### Pattern 4: Plain refusal from a fixed template
**What:** When no bold entry matches, agens refuses plainly and cites nothing.
**When to use:** RECOMMEND-06.
**Standard:** State that no vault note supports the described shape. Name the dimensions searched. Offer no citation and no "closest" pattern dressed as a fit. A "loosely related citation" is any reference to `agent-patterns-index.md` that is not a specific bold entry whose trigger the answers satisfy.

```
# Fixed refusal template (wording standard for RECOMMEND-06)
No pattern in the wiki-agents vault matches this shape.

I searched agent-patterns-index.md for a pattern whose trigger fits:
- goal: {goal}
- workflow: {workflow}
- data sensitivity: {sensitivity}
- latency/cost: {latency}

None resolved. I will not cite a loosely related pattern to fill the gap.
You could refine one of the four answers, or add a pattern note to the vault.
```

### Anti-Patterns to Avoid
- **Decorative citation:** citing the index note or a near-miss pattern "to look thorough." REQUIREMENTS.md Out of Scope names this the exact failure RECOMMEND-04/05/06 exist to prevent.
- **Citing from model knowledge:** recommending a pattern without the Read+Grep gate. Destroys the core value (CLAUDE.md).
- **Setting `disable-model-invocation` or `user-invocable` on top-level agens:** each breaks one required invocation path.
- **Assuming the questionnaire answer is always one of the fixed labels:** the tool always offers a free-text "Other" (see Pitfall 2).

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Structured four-dimension intake | A custom numbered-prompt parser in the body | `AskUserQuestion` (one call, 4 questions) | Native structured UI; shows selections before submission; handles multi-select and free text. [CITED: code.claude.com/docs/en/agent-sdk/user-input] |
| Vault search | A bespoke tokeniser or embedding index | `Grep` over the one index note | ripgrep is correct and sufficient at 8 KB / 21 entries. |
| Path-exists check | String-matching a path against a hard-coded list | `Read`/`Glob` returning content | A real filesystem call is the checkable evidence RECOMMEND-05 demands. |
| Auto-trigger logic | Keyword detection inside the body | The `description` field | The body never loads until the description already matched; body logic cannot decide triggering. [CITED: code.claude.com/docs/en/skills] |

**Key insight:** every mechanic this phase needs is a built-in tool or a frontmatter field. The skill body is orchestration and wording, not machinery. Custom machinery here would duplicate the substrate the constraint says to reuse.

## RECOMMEND-07 Scope Resolution

The phase title ("Read-Only Recommend") and success criterion 5 ("Any note agens writes into the vault is tagged as agent-authored") appear to conflict. They do not, once read as a conditional guard.

**Findings:**
- The vault schema defines no agent-authorship field. `schema.md` §2 lists `type`, `status`, `created`, `topic`, `tags`, and source/person fields; §2.7 lists auto-generated stamps (`wiki_role`, `wiki_hash`, etc.). None marks a note as authored by an agent. The `anthropic:` boolean marks Anthropic *source material*, not agent authorship. [VERIFIED: grep of wiki-agents/99_Meta/schema.md — "no agent-authorship field in schema.md"]
- Phase 0 (D-11/D-12) established that the only vault file agens writes — `99_Meta/agens-log.md` — is created and written in Phase 4 (LOG-01), not before. [CITED: 00-CONTEXT.md D-11, D-12]
- CONTEXT.md lists "RECOMMEND-07 write scope" as undecided.

**Recommendation (for the planner; the write-scope decision itself is an Open Question below):**
Phase 1 writes nothing to the vault. RECOMMEND-07 reads as "IF agens writes a note, THEN it is tagged agent-authored." In Phase 1 the antecedent is false, so the requirement holds vacuously and success criterion 5 is satisfied by writing no notes. Phase 1's actual RECOMMEND-07 deliverable is to *define* the agent-authored tagging convention that Phase 4's `agens-log` will enforce — mirroring how the four questionnaire dimensions are agens' own frame, not a vault taxonomy. Treat any actual vault write in Phase 1 as out of scope for the phase name and the sequencing constraint (read-only ships and earns trust first, per PROJECT.md).

## Common Pitfalls

### Pitfall 1: Auto-trigger tuned to the skill's own words, not the user's
**What goes wrong:** the `description` describes what agens *is* ("citation-grounded pattern recommender") but not the phrasings a user *says* when starting an agent project, so RECOMMEND-02 fails silently.
**Why it happens:** authors write descriptions from the tool's perspective.
**How to avoid:** front-load natural project-starting phrasings (see Pattern 1). Build the phrasing test set independently of the description wording (STATE.md blocker), and tune with skill-creator's should-trigger/should-not-trigger loop.
**Warning signs:** the skill triggers on `/agens` but never auto-loads from conversation; `/doctor` shows the description truncated below its key phrases.

### Pitfall 2: Assuming AskUserQuestion answers are always fixed labels
**What goes wrong:** D-02 mandates "fixed multiple-choice, not free text," but the tool always offers the user an "Other" free-text option that the skill cannot suppress. A user can return arbitrary text as any dimension's answer.
**Why it happens:** the tool's contract is fixed — "Users always see an Other option for custom text input" [CITED: code.claude.com/docs/en/agent-sdk/user-input].
**How to avoid:** the skill body must treat a free-text answer as a non-match by default — either map it to a fixed bucket if it clearly corresponds, or route to the plain refusal (Pattern 4). Do not fabricate a pattern match from free text. Document this as the intended degradation so D-02's reproducibility guarantee survives contact with the tool's real behaviour.
**Warning signs:** a recommendation issued on an answer that is not one of the D-03/D-04/D-05/D-06 labels.

### Pitfall 3: Grep matching the pattern name as loose substring
**What goes wrong:** grepping `Routing` matches "Routing" inside prose in unrelated notes, or inside "re-routing," producing a false "pattern appears" pass.
**Why it happens:** an unanchored substring search over the wrong file scope.
**How to avoid:** scope the Grep to `agent-patterns-index.md` only, and match the bold-entry form `\*\*Routing\*\*`, which is how each of the 21 patterns is written. [VERIFIED: agent-patterns-index.md entry format]
**Warning signs:** verification passes for a pattern name that has no `- **Name** —` entry.

### Pitfall 4: Vault path drift breaking every citation
**What goes wrong:** the vault root path is duplicated across the body; one change breaks every Read/Grep call.
**Why it happens:** hard-coding `C:/Users/Simon/Documents/wiki-agents` at each call site.
**How to avoid:** rely on the `--add-dir` grant and reference vault notes by their vault-relative path (`30_Concepts/agent-patterns-index.md`). Do NOT use `${CLAUDE_SKILL_DIR}`/`${CLAUDE_PROJECT_DIR}` to reach the vault — they resolve to the skill dir / project root, not the external vault (CLAUDE.md "What NOT to Use").
**Warning signs:** citation checks fail after moving the vault or the skill.

## Code Examples

### Ask the four dimensions in one call (RECOMMEND-03, D-01/D-02)
```json
// Source: code.claude.com/docs/en/agent-sdk/user-input (question format).
// Illustrative option sets from D-03..D-06; goal-bucket wording to be finalised in planning.
{
  "questions": [
    {
      "question": "What is the agent's core job?",
      "header": "Goal",
      "options": [
        { "label": "Summarise/transform content", "description": "Take input, produce a rewritten or condensed output" },
        { "label": "Answer questions over data",   "description": "Retrieve and answer from a body of documents or records" },
        { "label": "Automate a multi-step task",   "description": "Run a sequence of steps to a known end state" },
        { "label": "Orchestrate multiple specialists", "description": "Coordinate several sub-agents or tools" }
      ],
      "multiSelect": false
    },
    {
      "question": "How is control held?",
      "header": "Workflow",
      "options": [
        { "label": "Fixed workflow (predictable steps)", "description": "Code sequences the LLM calls through predefined paths" },
        { "label": "Autonomous agent (open-ended, self-directed)", "description": "The model directs its own process and tool use" },
        { "label": "Not sure", "description": "Undecided between the two" }
      ],
      "multiSelect": false
    },
    {
      "question": "How sensitive is the data?",
      "header": "Sensitivity",
      "options": [
        { "label": "Public", "description": "No confidentiality concern" },
        { "label": "Internal", "description": "Business-internal, not public" },
        { "label": "Sensitive or regulated", "description": "Personal, regulated, or high-stakes data" }
      ],
      "multiSelect": false
    },
    {
      "question": "What latency and cost profile fits?",
      "header": "Latency",
      "options": [
        { "label": "Real-time (sub-second)", "description": "Interactive, tight latency budget" },
        { "label": "Interactive (seconds)", "description": "A few seconds is acceptable" },
        { "label": "Batch (minutes+, cost-optimised)", "description": "Throughput and cost over speed" }
      ],
      "multiSelect": false
    }
  ]
}
```

### Verify a citation before recommending (RECOMMEND-04/05)
```text
// Source: procedure derived from agent-patterns-index.md entry format (verified on disk)
1. Glob "30_Concepts/agent-patterns-index.md"     → non-empty? else refuse (path unresolved)
2. Read the note                                   → success? else refuse
3. Grep "\*\*Routing\*\*" in that note only        → hit? else refuse (pattern absent)
4. Read the matched line; quote its "Trigger: … Trade-off: …" text
5. Present: pattern name + vault-relative path + quoted passage
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| `.claude/commands/*.md` custom commands | Merged into Skills; a command and a same-named skill both create `/name` | Current Claude Code | Author agens as a Skill, not a command file — Skills add the directory, frontmatter invocation control, and auto-load. [CITED: code.claude.com/docs/en/skills] |
| Every skill re-invocation appends a full copy | Identical re-invocation adds a short "already loaded" note | v2.1.202 | Re-triggering agens in one session does not bloat context. [CITED: code.claude.com/docs/en/skills] |

**Deprecated/outdated:** none material to this phase.

## Assumptions Log

| # | Claim | Section | Risk if Wrong |
|---|-------|---------|---------------|
| A1 | RECOMMEND-07 is a conditional guard satisfied vacuously in Phase 1; agens writes nothing to the vault this phase | RECOMMEND-07 Scope Resolution | If the user intends a Phase 1 write, the plan omits a write task and its agent-authored tagging. Flagged as an Open Question. |
| A2 | The agent-authored tagging convention (undefined in the vault) should be defined in Phase 1 for Phase 4 to enforce; a concrete form (e.g. a frontmatter marker plus an explicit in-note "agent-authored" line) is not yet chosen | RECOMMEND-07 Scope Resolution | Choosing a convention the user rejects wastes a Phase 4 rework. Needs user confirmation. |
| A3 | The match threshold — a match requires a bold entry whose trigger the four answers satisfy; anything less refuses | Pattern 4, Open Questions | Too loose → decorative citations slip through (RECOMMEND-06 fails); too strict → over-refusal. User-facing behaviour, undecided in CONTEXT. |
| A4 | Goal-bucket wording beyond D-03's four examples (and whether a catch-all bucket exists) mirrors `agent-patterns-index.md` family language | Code Examples, Open Questions | Wrong buckets misroute the lookup. Marked Claude's Discretion; confirm final wording in planning. |
| A5 | Header `Latency` (not `Latency/cost`) and `Sensitivity` (not `Data sensitivity`) used to stay inside the 12-char header cap | Pattern 2 | Over-cap header is rejected or truncated by the tool. Low risk — the cap is verified. |

## Open Questions

1. **RECOMMEND-07 write scope (undecided in CONTEXT.md).**
   - What we know: no agent-authorship field exists in the schema; Phase 4 owns the only vault write (`agens-log.md`, LOG-01); the phase is named read-only.
   - What's unclear: whether Phase 1 writes anything at all, or only fixes the convention.
   - Recommendation: treat Phase 1 as writing nothing; define the tagging convention for Phase 4. Confirm with the user before planning any Phase 1 vault write.

2. **Match & refusal threshold (undecided in CONTEXT.md).**
   - What we know: a match must resolve to a specific bold entry; a "loosely related" citation must be refused.
   - What's unclear: the exact rule mapping four answers to a pattern-trigger fit, and how many near-miss dimensions still count as a match.
   - Recommendation: adopt A3 (match = a bold entry whose trigger the answers satisfy; else refuse) as the default, and confirm it with the user, since it sets the RECOMMEND-05/06 boundary.

3. **Recommendation output format (undecided in CONTEXT.md).**
   - What we know: RECOMMEND-04 requires the path plus a quoted supporting passage; REQUIREMENTS.md forbids citing multiple notes "to look thorough."
   - What's unclear: how much of the matched entry to quote, and whether more than one pattern may be surfaced when several genuinely fit.
   - Recommendation: quote the matched entry's trigger + trade-off sentence; surface one pattern by default. Confirm whether multi-pattern output is wanted.

4. **Grep-only recall (STATE.md concern).**
   - What we know: the lookup target is one 8 KB note with 21 bold entries.
   - What's unclear: whether grep-only search misses a semantically close pattern when the goal bucket does not lexically match a trigger.
   - Recommendation: confirm empirically with the phrasing/answer test set before adding any synonym expansion or index — do not pre-build an index (out of scope).

## Environment Availability

| Dependency | Required By | Available | Version | Fallback |
|------------|------------|-----------|---------|----------|
| Claude Code binary | The Skill substrate, all built-in tools | ✓ | 2.1.207 | — (exceeds v2.1.196 floor) |
| wiki-agents vault | Every citation lookup (RECOMMEND-04/05/06) | ✓ | — | — (verified at C:/Users/Simon/Documents/wiki-agents) |
| `agent-patterns-index.md` | The single lookup target | ✓ | 8,111 bytes, 21 bold entries | — (created in Phase 0) |
| `AskUserQuestion` tool | Fixed questionnaire (RECOMMEND-03) | ✓ | current | — |
| Read / Grep / Glob | Lookup + verification | ✓ | current | — |
| Vault `--add-dir` grant | Read access to the external vault | Runtime step | — | `permissions.additionalDirectories` does NOT load skills — must use `--add-dir`/`/add-dir` |

**Missing dependencies with no fallback:** none.
**Missing dependencies with fallback:** none; the only runtime action is granting the vault via `--add-dir` at invocation time.

## Security Domain

`security_enforcement` is absent from config.json (= enabled). agens is a read-only skill over a local vault; the attack surface is small, but two controls matter.

### Applicable ASVS Categories

| ASVS Category | Applies | Standard Control |
|---------------|---------|-----------------|
| V2 Authentication | no | No auth surface; local skill. |
| V3 Session Management | no | No sessions. |
| V4 Access Control | yes (light) | Read-only tool grant (`allowed-tools: Read Grep Glob AskUserQuestion`); no Write/Bash. Confine reads to the granted vault dir. |
| V5 Input Validation | yes | Questionnaire answers (including free-text "Other") are untrusted; a free-text answer must not fabricate a citation — route to refusal (Pitfall 2). Grep pattern-name match anchored to bold form (Pitfall 3). |
| V6 Cryptography | no | No secrets, no crypto in this phase. |

### Known Threat Patterns for a read-only vault skill

| Pattern | STRIDE | Standard Mitigation |
|---------|--------|---------------------|
| Hallucinated / decorative citation | Spoofing (of grounding) | The Read+Grep verification gate — no recommendation without a resolved path and a present bold entry (RECOMMEND-05). |
| Free-text answer smuggling an unsupported claim | Tampering | Treat "Other"/free-text as non-match; refuse rather than infer (Pitfall 2). |
| Over-broad tool grant enabling a write | Elevation of privilege | Grant only Read/Grep/Glob/AskUserQuestion; no Write, Edit, or Bash in Phase 1. |

## Sources

### Primary (HIGH confidence)
- code.claude.com/docs/en/skills — SKILL.md frontmatter reference (full field table), invocation control (`disable-model-invocation`, `user-invocable`), how the command name derives from the directory, 1,536-char description cap, `${CLAUDE_SKILL_DIR}`/`${CLAUDE_PROJECT_DIR}` (v2.1.196), skills from `--add-dir` directories, re-invocation dedupe (v2.1.202), skill-creator eval loop. Retrieved 2026-07-12.
- code.claude.com/docs/en/agent-sdk/user-input — AskUserQuestion question format (`question`, `header` max 12 chars, `options` 2-4, `multiSelect`), the always-present free-text "Other", the 1-4 questions / 2-4 options limits. Retrieved 2026-07-12.
- Local: `~/.claude/skills/find-skills/SKILL.md` — auto-trigger description craft (natural user phrasings). Verified on disk.
- Local: `~/.claude/skills/gsd-ai-integration-phase/SKILL.md` — real frontmatter (`allowed-tools`, `argument-hint`) shape. Verified on disk.
- Local: `wiki-agents/30_Concepts/agent-patterns-index.md` — 21 bold pattern entries in `- **Name** — trigger. Trade-off: …` form; the citation-check target. Verified on disk.
- Local: `wiki-agents/99_Meta/schema.md` — confirms no agent-authorship field exists. Verified via grep.
- `claude --version` → 2.1.207. Verified 2026-07-12.

### Secondary (MEDIUM confidence)
- WebSearch (claudelog, atcyrus, smartscope summaries) — AskUserQuestion 1-4 questions / 2-4 options / 12-char header / auto "Other". Cross-verified against the primary Agent SDK user-input docs above, which raised it to HIGH.

### Tertiary (LOW confidence)
- None relied upon.

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — every component is a built-in tool or documented frontmatter field, verified against current official docs and the installed 2.1.207 binary.
- Architecture / patterns: HIGH — dual invocation, the four-question call, and the Read+Grep gate are all documented and match the verified index-note format.
- Pitfalls: HIGH — the "Other" free-text finding and the header-cap finding come directly from the Agent SDK docs; the grep-anchoring pitfall from the verified note format.
- Open questions: the three undecided CONTEXT items and the grep-recall concern are genuine gaps for the planner or user to resolve.

**Research date:** 2026-07-12
**Valid until:** 2026-08-11 (30 days; Skills and AskUserQuestion docs are stable, but Claude Code ships often — re-verify frontmatter fields if the plan slips past this window).
