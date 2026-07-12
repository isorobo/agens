---
phase: 01-agens-read-only-recommend
reviewed: 2026-07-12T00:00:00Z
depth: standard
files_reviewed: 3
files_reviewed_list:
  - .claude/skills/agens/SKILL.md
  - .claude/skills/agens/references/trigger-tests.md
  - .claude/skills/agens/references/agent-authored-convention.md
findings:
  critical: 2
  warning: 3
  info: 2
  total: 7
status: issues_found
---

# Phase 01: Code Review Report

**Reviewed:** 2026-07-12
**Depth:** standard
**Files Reviewed:** 3
**Status:** issues_found

## Summary

Reviewed the three Phase 1 deliverables: the `agens` skill definition (`SKILL.md`),
the trigger/walkthrough acceptance harness, and the agent-authored tagging
convention doc. Both harness and convention files are well-formed documentation
with no executable logic. `SKILL.md` is the load-bearing artifact and it fails
on verification against the actual vault it targets.

Two findings are empirically reproduced against the real vault file at
`C:/Users/Simon/Documents/wiki-agents/30_Concepts/agent-patterns-index.md`, not
theoretical: (1) the vault-relative path convention the skill mandates does not
resolve from the skill's actual working directory, and the `Read` tool's own
contract requires an absolute path — so the citation gate's first check will
fail on every invocation; (2) the citation gate's Grep-anchor construction, as
literally demonstrated in `SKILL.md`, produces zero matches against the real
`**Knowledge Retrieval (RAG)**` entry because the escaping guidance covers only
the bold-markdown asterisks and not the parenthesis metacharacters in that
pattern's own name — verified with a live Grep call against the vault note.

Both defects strike directly at the project's stated Core Value ("Every
recommendation agens gives cites a specific wiki-agents note by path... That
citation discipline must always hold, or agens degrades into asking Claude cold
with extra steps" — `CLAUDE.md`). As specified, the skill cannot pass its own
citation gate for at least the RAG path, and likely cannot resolve the note at
all, so it will either always refuse or silently fall back to
un-gated behaviour if an executing agent "helpfully" works around the broken
path.

## Critical Issues

### CR-01: Vault-relative path never resolves from the skill's actual working directory

**File:** `.claude/skills/agens/SKILL.md:23-26,105-108`

**Issue:** `SKILL.md` mandates that agens "references notes by vault-relative
path only and hard-codes no absolute vault root" (lines 23-26), and Step 2b
instructs: "Read or Glob the vault-relative path
`30_Concepts/agent-patterns-index.md`. If the call returns no content, the path
does not resolve — refuse" (lines 105-108).

This was verified empirically. From the agens project's own working directory
(where the skill actually executes), the bare relative path does not exist:

```
$ pwd
/c/Users/Simon/claude-projects/agens
$ ls "30_Concepts/agent-patterns-index.md"
ls: cannot access '30_Concepts/agent-patterns-index.md': No such file or directory
```

The file only exists under the full vault root,
`C:/Users/Simon/Documents/wiki-agents/30_Concepts/agent-patterns-index.md`.
Additionally, the `Read` tool's own contract in this environment requires an
absolute path ("The file_path parameter must be an absolute path, not a
relative path") — so "vault-relative path only" is not a valid `Read` call
under any circumstance, `--add-dir` grant or not. `Glob`'s `path` parameter
defaults to cwd when omitted, so a bare relative pattern has the same failure
mode.

The project's own research doc (embedded in `PROJECT.md` / `STACK.md`)
anticipated exactly this failure mode and recommended a mitigation the skill
does not implement: "`${CLAUDE_SKILL_DIR}` / `${CLAUDE_PROJECT_DIR}` to reach
the vault | They resolve to the skill dir / project root, not the external
vault — a silent mis-read | An additional-directory grant, **or one
centralised vault-root constant**" (`PROJECT.md`, "What NOT to Use" table).
`SKILL.md` took the additional-directory-grant half of that guidance but
dropped the "centralise the path to one constant" half, leaving no way to turn
the vault-relative path into something either tool can actually open.

As written, Step 2b's first gate check ("Path resolves") will fail on every
invocation, in every session, regardless of whether `--add-dir` was granted.
Since the gate is required before any recommendation, this either makes agens
refuse on every request (defeating its purpose) or invites an executing agent
to improvise an absolute path itself — which reintroduces the exact
hard-coded-vault-root problem the skill explicitly forbids, with no guidance on
what that improvised root should be.

**Fix:** Introduce one centralised, explicit vault-root anchor that the skill
can combine with the vault-relative path before calling `Read`/`Glob`/`Grep` —
for example, state the full expected absolute path pattern in `SKILL.md`
directly (`{vault_root}/30_Concepts/agent-patterns-index.md`) and define
`vault_root` as a single documented constant (e.g. read from an
`--add-dir`-granted directory the user is told to name explicitly, or a value
recorded once in `SKILL.md`/a `references/*.md` file), consistent with the
project's own "one centralised vault-root constant" recommendation. Do not
leave path construction to per-invocation improvisation.

### CR-02: Citation-gate Grep anchor fails against the real `Knowledge Retrieval (RAG)` entry

**File:** `.claude/skills/agens/SKILL.md:90,109-113`

**Issue:** Step 2a maps "Answer questions over data" to the candidate
`Knowledge Retrieval (RAG)` (line 90). Step 2b's citation gate instructs:
"Grep that note ONLY, matching the candidate's bold `**Name**` literal anchored
to the bold form — for example the pattern `\*\*Routing\*\*`... If there is no
anchored hit, the pattern is not a real bold entry — refuse" (lines 109-113).

The only worked example escapes the markdown asterisks and nothing else.
Applying that same treatment to the RAG candidate produces
`\*\*Knowledge Retrieval \(RAG\)\*\*` only if the unescaped parentheses are
also caught and escaped — but the guidance never says to escape parentheses,
periods, or other regex metacharacters, and the one example given contains
none. An executing agent following the literal pattern-construction guidance
by rote (escape only the bold markers) will produce
`\*\*Knowledge Retrieval (RAG)\*\*`, in which the unescaped `(` `)` are treated
as a regex capture group, not literal characters.

Verified empirically against the real vault note
(`C:/Users/Simon/Documents/wiki-agents/30_Concepts/agent-patterns-index.md:39`,
which reads `- **Knowledge Retrieval (RAG)** — retrieve documents...`):

```
Grep pattern: \*\*Knowledge Retrieval \(RAG\)\*\*   → 1 match (line 39)
Grep pattern: \*\*Knowledge Retrieval (RAG)\*\*      → No matches found
```

The second form — the one an agent naturally produces by mirroring the
`\*\*Routing\*\*` example — fails to find an entry that demonstrably exists.
This means the "Answer questions over data" branch, one of only four Goal
options, will incorrectly refuse a request its own vault fully supports,
violating the core citation-gate purpose (a false negative on a real,
resolvable entry) rather than merely a false positive on a near-miss.

**Fix:** State a general escaping rule in Step 2b, not a single example: "Treat
the candidate name as a literal string — escape every regex metacharacter in
it, including `(`, `)`, `.`, `-`, and the bold `**` markers, before using it as
the Grep pattern." Add a second worked example using a parenthesised name (e.g.
`Knowledge Retrieval (RAG)` → `\*\*Knowledge Retrieval \(RAG\)\*\*`) so the
escaping rule is unambiguous for every candidate the mapping table can produce,
including `Model Context Protocol (MCP)` and `Inter-Agent Communication (A2A)`
if the candidate set ever grows.

## Warnings

### WR-01: Case mismatch between the attribution line and its claimed literal search string

**File:** `.claude/skills/agens/references/agent-authored-convention.md:53,57`

**Issue:** The in-note attribution template is defined as:

```
> Agent-authored by agens. This note was written by an agent, not by hand.
```

(line 53, capital "Agent-authored"). The very next paragraph then claims: "The
line makes the agent-authored status visible without parsing frontmatter, and
carries the literal string `agent-authored` for a plain-text search" (line 57,
lowercase "agent-authored"). The actual template text and the claimed literal
search string differ in case. Every other verification mechanism this same
document defines (the frontmatter grep at line 66) is a strict, anchored,
case-sensitive match. If Phase 4 implements body verification the same way —
by mirroring this document's own convention — a case-sensitive search for
`agent-authored` will not match the actual template text `Agent-authored`.

**Fix:** Make the two agree. Either change the template line to start
mid-sentence-case (`Agent-authored` → lowercase where grammatically possible,
e.g. rephrase to "This note is agent-authored by agens...") or change line 57
to state the literal string as `Agent-authored` (matching the template
exactly), and note explicitly whether any future verification grep should be
case-insensitive.

### WR-02: "Path resolves" check describes the wrong failure signal for a missing file

**File:** `.claude/skills/agens/SKILL.md:105-108`

**Issue:** Step 2b bullet 1 says: "If the call returns no content, the path
does not resolve — refuse." Per this environment's own `Read` tool contract,
reading a file that does not exist returns an error, not empty/no content ("It
is okay to read a file that does not exist; an error will be returned."). The
gate's failure-detection language ("returns no content") does not match the
tool's actual failure mode (an error response), which could cause an executing
agent to treat a genuinely present-but-empty note differently from a
missing-note error, or to mishandle the error response because the
documentation never described one.

**Fix:** Restate the check to cover both outcomes explicitly: "If the call
errors, or returns a file that exists but is empty, the path does not
resolve — refuse."

### WR-03: Redundant restatement of the "Other" free-text refusal rule

**File:** `.claude/skills/agens/SKILL.md:71-75,96-98`

**Issue:** The rule "a free-text 'Other' answer on any dimension routes to
refusal" is stated twice, near-verbatim: once in Step 1 (lines 72-75: "Treat
any dimension answered by free-text 'Other' as a non-match: route straight to
the plain refusal in Step 3, and never fabricate a pattern match from
free-text input") and again in Step 2a (lines 96-98: "If any dimension came
back as free-text 'Other', refuse"). This is not contradictory, but it
duplicates the same instruction with slightly different framing in two
locations, adding body length without new information — relevant because the
project's own guidance targets keeping `SKILL.md` lean (under ~500 lines) and
moving repeated material to `references/*.md` once it recurs.

**Fix:** State the "Other" rule once, in Step 2a where the match threshold is
defined, and have Step 1 simply reference it ("see the match threshold in Step
2a for how an 'Other' answer is handled") rather than restating the rule.

## Info

### IN-01: Unresolvable external requirement-ID reference

**File:** `.claude/skills/agens/references/trigger-tests.md:56`

**Issue:** The manual walkthrough checklist item reads: "Each question offers
fixed multiple-choice options, matching the D-03 through D-06 labels." Labels
`D-03` through `D-06` are not defined anywhere in the three reviewed files, nor
in `PROJECT.md`. A reviewer or executor working from this file set alone
cannot verify this checklist item; the reference depends on an external
requirements document not included in scope.

**Fix:** Either inline what D-03 through D-06 refer to (e.g. "the Goal,
Workflow, Sensitivity, Latency dimension labels defined in `SKILL.md` Step 1")
or add a one-line pointer to the source document that defines those IDs.

### IN-02: Goal-to-pattern mapping table overlaps the separately-collected Workflow dimension

**File:** `.claude/skills/agens/SKILL.md:92-93`

**Issue:** The mapping "Automate a multi-step task → Planning, or Prompt
Chaining for a fixed step sequence" (line 93) re-derives essentially the same
distinction the Workflow question already collects directly ("Fixed workflow
(predictable steps)" vs. "Autonomous agent (open-ended, self-directed)", lines
55-57). Nothing in Step 2a states how to reconcile the two signals if they
disagree (e.g. Goal implies "fixed step sequence" phrasing but the user
explicitly selected "Autonomous agent" for Workflow) — Step 2a says the match
threshold requires "the Workflow, Sensitivity, and Latency answers do not
contradict" the candidate entry's Trigger, but leaves "contradict" undefined
for this specific overlap.

**Fix:** Either drop the "or Prompt Chaining for a fixed step sequence" /
"or Routing when..." secondary hints from the Goal-to-family table and let the
Workflow answer alone disambiguate at the Trigger-matching step, or explicitly
state which dimension wins when Goal-implied step-shape and the stated
Workflow answer disagree.

---

_Reviewed: 2026-07-12_
_Reviewer: Claude (gsd-code-reviewer)_
_Depth: standard_
