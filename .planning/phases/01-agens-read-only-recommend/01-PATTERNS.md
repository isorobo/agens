# Phase 1: agens (Read-Only Recommend) - Pattern Map

**Mapped:** 2026-07-12
**Files analyzed:** 4 (1 required, 3 conditional)
**Analogs found:** 4 / 4

## Orientation

Phase 1 authors one Claude Code Skill. It builds no runtime, installs no package,
and adds no source language. The only artifact is a `SKILL.md` markdown file (plus
optional `references/*.md`). No in-repo analog exists — Phase 0 was vault
consolidation, not skill-building. The closest analogs are two working skills on
disk outside the repo, plus the vault note the citation check reads.

**Analog files (all read for this map):**
- `C:/Users/Simon/.claude/skills/find-skills/SKILL.md` — read-only discover + recommend + verify + refuse skill. Closest analog by role and data flow.
- `C:/Users/Simon/.claude/skills/gsd-ai-integration-phase/SKILL.md` — frontmatter shape with an `allowed-tools` grant and `AskUserQuestion`; XML-tagged body sections.
- `C:/Users/Simon/.claude/skills/eulogy/SKILL.md` — how a body references bundled `references/*.md` files.
- `C:/Users/Simon/Documents/wiki-agents/30_Concepts/agent-patterns-index.md` — the single citation target and its entry format.

## File Classification

| New/Modified File | Role | Data Flow | Closest Analog | Match Quality |
|-------------------|------|-----------|----------------|---------------|
| `.claude/skills/agens/SKILL.md` | config (skill definition + orchestration body) | request-response (intake → lookup → verify → recommend/refuse) | `find-skills/SKILL.md` (body) + `gsd-ai-integration-phase/SKILL.md` (frontmatter) | role-match (strong) |
| `.claude/skills/agens/references/questionnaire.md` (conditional) | config (reference) | transform (answers → pattern family) | `eulogy/SKILL.md` references pattern | role-match |
| `.claude/skills/agens/references/citation-check.md` (conditional) | config (reference) | request-response (verify gate) | `eulogy/SKILL.md` references pattern | role-match |
| `.claude/skills/agens/references/refusal.md` (conditional) | config (reference) | request-response (refusal template) | `eulogy/SKILL.md` references pattern | role-match |

**Conditional files:** the three `references/*.md` files exist only if `SKILL.md`
nears the 500-line guidance (RESEARCH.md §Recommended Project Structure). Default is
one self-contained `SKILL.md`. The planner should treat the references split as a
size-triggered option, not a required deliverable.

## Pattern Assignments

### `.claude/skills/agens/SKILL.md` (config, request-response)

Two analogs combine: `gsd-ai-integration-phase/SKILL.md` supplies the frontmatter
shape (tool grant, `AskUserQuestion`), and `find-skills/SKILL.md` supplies the body
shape (a read-only recommend skill that verifies before recommending and refuses
plainly when nothing fits).

**Frontmatter — auto-trigger description with natural user phrasings**
Analog: `find-skills/SKILL.md` lines 1-4. Copy the craft: the `description` lists the
phrasings a user *says*, not what the skill *is* (RESEARCH.md Pattern 1, Pitfall 1).
```yaml
---
name: find-skills
description: Helps users discover and install agent skills when they ask questions like "how do I do X", "find a skill for X", "is there a skill that can...", or express interest in extending capabilities. This skill should be used when the user is looking for functionality that might exist as an installable skill.
---
```
For agens, keep `name: agens` matched to the directory, and leave
`disable-model-invocation` and `user-invocable` unset — find-skills omits both, which
is exactly what gives RECOMMEND-01 (`/agens`) and RECOMMEND-02 (auto-trigger)
together. The tuned description text is drafted in RESEARCH.md Pattern 1 (lines
173-186); use that wording as the starting draft.

**Frontmatter — read-only tool grant**
Analog: `gsd-ai-integration-phase/SKILL.md` lines 1-16. Same `allowed-tools` mechanism,
narrowed to the four read-only tools agens needs.
```yaml
---
name: gsd-ai-integration-phase
description: "Generate an AI-SPEC.md design contract for phases that involve building AI systems."
argument-hint: "[phase number]"
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - Agent
  - WebFetch
  - WebSearch
  - AskUserQuestion
  - mcp__context7__*
---
```
For agens, grant only `Read Grep Glob AskUserQuestion` — no `Write`, `Edit`, or `Bash`
(RESEARCH.md Security Domain; the read-only sequencing constraint in PROJECT.md).
CLAUDE.md specifies the space-separated form `allowed-tools: Read Grep Glob AskUserQuestion`;
the YAML-list form above is the equivalent shape if the planner prefers a list.

**Body — "When to Use" phrasing list**
Analog: `find-skills/SKILL.md` lines 10-20. A bullet list of the request shapes that
should load the skill, reinforcing the auto-trigger description in prose.
```markdown
Use this skill when the user:

- Asks "how do I do X" where X might be a common task with an existing skill
- Says "find a skill for X" or "is there a skill for X"
- Asks "can you do X" where X is a specialized capability
- Expresses interest in extending agent capabilities
```

**Body — verify-before-recommend gate (the core pattern)**
Analog: `find-skills/SKILL.md` lines 66-73 ("Verify Quality Before Recommending").
find-skills gates on install count and source reputation; agens gates on the
deterministic Read-then-Grep citation check (RESEARCH.md Pattern 3). Same shape: a
hard check stands between the candidate and the recommendation.
```markdown
### Step 4: Verify Quality Before Recommending

**Do not recommend a skill based solely on search results.** Always verify:

1. **Install count** — Prefer skills with 1K+ installs. Be cautious with anything under 100.
2. **Source reputation** — Official sources ... are more trustworthy than unknown authors.
```
For agens, replace the verify list with the two-step gate from RESEARCH.md Code
Examples (lines 344-352): (1) Read/Glob `30_Concepts/agent-patterns-index.md` — path
resolves; (2) Grep the bold entry `\*\*Pattern Name\*\*` in that note only — pattern
present. Both must pass before any recommendation. Anchor the Grep to the bold form
to avoid the loose-substring false pass (RESEARCH.md Pitfall 3).

**Body — present the recommendation with the cited passage**
Analog: `find-skills/SKILL.md` lines 74-94 ("Present Options to the User"). find-skills
presents name + install count + install command + link. agens presents pattern name +
vault-relative path + the quoted trigger/trade-off sentence (RECOMMEND-04).
```markdown
### Step 5: Present Options to the User

When you find relevant skills, present them to the user with:

1. The skill name and what it does
2. The install count and source
3. The install command they can run
4. A link to learn more at skills.sh
```

**Body — plain refusal when nothing fits**
Analog: `find-skills/SKILL.md` lines 126-142 ("When No Skills Are Found"). Direct
precedent for RECOMMEND-06: state plainly that nothing matched, name what was searched,
offer no dressed-up near-miss.
```markdown
## When No Skills Are Found

If no relevant skills exist:

1. Acknowledge that no existing skill was found
2. Offer to help with the task directly using your general capabilities
3. Suggest the user could create their own skill with `npx skills init`
```
For agens, swap the wording for the fixed refusal template in RESEARCH.md Pattern 4
(lines 222-234), which names the four searched dimensions and refuses to cite a loosely
related pattern.

**Body — XML-tagged section structure (optional style)**
Analog: `gsd-ai-integration-phase/SKILL.md` lines 18-37. If the planner prefers tagged
sections over markdown headings, this is the in-toolchain precedent.
```markdown
<objective>
Create an AI design contract (AI-SPEC.md) for a phase involving AI system development.
Orchestrates gsd-framework-selector → gsd-ai-researcher → ...
</objective>

<process>
Execute end-to-end.
Preserve all workflow gates.
</process>
```
find-skills uses plain `##` headings instead. Either fits; find-skills is the closer
match for a user-facing recommend skill.

---

### `.claude/skills/agens/references/*.md` (config, conditional)

Analog: `eulogy/SKILL.md` — a skill that keeps its body lean and offloads detail to
`references/*.md`, naming each file inline at the point of use and listing them at the
bottom.

**Inline reference at point of use** (`eulogy/SKILL.md` line 60):
```markdown
`references/intake.md` for the file-by-file pipeline (Word, PDF, photographs,
```

**Reference manifest at the bottom** (`eulogy/SKILL.md` lines 148-152):
```markdown
- `references/intake.md` — the file-by-file ingestion pipeline and tools.
- `references/profile-schema.md` — the persona and personage profile schema.
- `references/prompts.md` — the substantive interview prompts and the ...
- `references/rhetoric.md` — eulogy architecture, devices, mistakes, master ...
```
For agens, if `SKILL.md` nears 500 lines, split the questionnaire JSON, the citation
procedure, and the refusal template into `references/questionnaire.md`,
`references/citation-check.md`, and `references/refusal.md`, referenced this way so
Claude loads them on demand (RESEARCH.md §Supporting, §Recommended Project Structure).

## Shared Patterns

### Auto-trigger description craft
**Source:** `find-skills/SKILL.md` lines 1-4
**Apply to:** the `agens` frontmatter `description`
Write the description from the user's phrasing, not the skill's self-image. This single
field owns RECOMMEND-02; the body never loads until it already matched (RESEARCH.md
Pitfall 1). Leave `disable-model-invocation` and `user-invocable` unset — their absence
in find-skills is what yields both invocation paths.

### Read-only tool grant
**Source:** `gsd-ai-integration-phase/SKILL.md` lines 5-16
**Apply to:** the `agens` frontmatter `allowed-tools`
Grant only `Read Grep Glob AskUserQuestion`. Omit `Write`, `Edit`, and `Bash` — Phase 1
writes nothing to the vault, and the narrow grant is the standing mitigation against an
over-broad write capability (RESEARCH.md Security Domain; PROJECT.md sequencing constraint).

### Verify-before-recommend, refuse-plainly
**Source:** `find-skills/SKILL.md` lines 66-73 (verify) and 126-142 (refuse)
**Apply to:** the `agens` body — every recommendation path
A hard check stands between candidate and recommendation; when the check fails, the
skill refuses plainly rather than dressing a near-miss as a fit. This is the exact
shape RECOMMEND-05 and RECOMMEND-06 require, already proven in a read-only recommend
skill.

### Citation target entry format
**Source:** `wiki-agents/30_Concepts/agent-patterns-index.md` lines 26-46
**Apply to:** the Grep pattern and the quoted passage in the `agens` body
Each of the 21 entries is written `- **Pattern Name** — <description>. Trigger: <trigger>. Trade-off: <trade-off>.`
The bold-name literal is the Grep anchor (`\*\*Routing\*\*`); the `Trigger: … Trade-off: …`
sentence is the passage agens quotes (RECOMMEND-04). Example entry:
```markdown
- **Routing** — classify input and direct it to a specialised handler ... Trigger: heterogeneous input types each need different handling. Trade-off: buys specialisation with a misclassification risk at the routing step.
```
Note: the entry format is prose, not a labelled `Trigger:`/`Trade-off:` field pair in
every case — the first pattern (`Prompt Chaining`, line 26) also uses `Trigger: … Trade-off: …`,
so the quotable passage is the two sentences after the em-dash. The planner should
confirm the exact quote span against the matched line at build time.

## No Analog Found

None. Every file has a working analog. The one gap is not a missing analog but a tool
constraint: `AskUserQuestion` always offers a free-text "Other" option that cannot be
suppressed (RESEARCH.md Pitfall 2). No analog skill demonstrates handling this, because
the mitigation is body logic — treat a free-text answer as a non-match and route to the
refusal path. The planner writes this as new orchestration, not copied from an analog.

## Metadata

**Analog search scope:**
- In-repo: `C:/Users/Simon/claude-projects/agens` (no `SKILL.md`, no `.claude/skills/` — confirmed empty via Glob).
- On-disk skills: `C:/Users/Simon/.claude/skills/` (110+ skills enumerated; find-skills, gsd-ai-integration-phase, eulogy read in full or in part).
- Vault: `C:/Users/Simon/Documents/wiki-agents/30_Concepts/agent-patterns-index.md` (citation target, read in full).

**Files scanned:** 4 analogs read; 1 in-repo tree confirmed empty of skill code.
**Pattern extraction date:** 2026-07-12
