# Agent-Authored Tagging Convention

This file defines how agens marks any note it writes into the wiki-agents vault
as agent-authored. It is documentation only. It records a convention for a later
phase to enforce; it performs no write itself.

## Phase 1 writes NOTHING to the vault

Phase 1 (read-only recommend) writes nothing to the vault. agens reads and cites
notes; it creates and edits no vault file. RECOMMEND-07 reads as a conditional
guard — "IF agens writes a note, THEN the note is tagged agent-authored." The
antecedent is false in Phase 1, so the requirement holds vacuously.

The first and only vault write is Phase 4's `99_Meta/agens-log.md` (LOG-01).
That write, and every later agens write, carries the marker defined below.

## Why a new field is needed

The vault schema (`99_Meta/schema.md` §2) defines `type`, `status`, `created`,
`topic`, `tags`, the source and person fields, and the auto-generated stamps in
§2.7. None marks a note as authored by an agent. The `anthropic:` boolean marks
Anthropic source material, not agent authorship, so it does not serve here.

A new field is required. It follows the schema's own frontmatter pattern rather
than inventing a competing structure, consistent with the vault owner's standing
preference for existing convention over new form.

## The three elements of the convention

### 1. Frontmatter marker

Every note agens writes carries this controlled frontmatter field:

```yaml
authored_by: agens
```

`authored_by` is a new string field. It collides with no field in `schema.md`
§2 (checked against §2.1 required, §2.2–2.6 typed, §2.7 auto-generated). Its
one controlled value in Phase 4 is `agens`. The field is absent from a
human-authored note, so its presence alone identifies an agent-authored note.

When Phase 4 lands, add `authored_by` to `schema.md` §2 before first use, per the
schema's own rule that a new controlled value requires a schema update first.

### 2. In-note attribution line

The frontmatter marker is invisible in rendered reading views. So agens also adds
one explicit attribution line to the body of any note it writes, immediately below
the note title:

```
> Agent-authored by agens. This note was written by an agent, not by hand.
```

The line makes the agent-authored status visible without parsing frontmatter, and
carries the literal string `agent-authored` for a plain-text search.

### 3. The verification grep Phase 4 will run

Phase 4 (LOG-01) enforces the marker with an anchored grep of the frontmatter
field. The check confirms a written note carries the exact marker rather than a
near-miss:

```bash
grep -Eq '^authored_by:[[:space:]]*agens[[:space:]]*$' "99_Meta/agens-log.md"
```

The anchor `^authored_by:` binds the match to the frontmatter field at line start,
not to the string appearing elsewhere in prose. The trailing `[[:space:]]*$` allows
trailing whitespace and rejects a longer value. A non-zero exit means the note lacks
the marker, and Phase 4 treats that as a failed write.

## Scope guard

This plan creates, edits, and writes no file under
`C:/Users/Simon/Documents/wiki-agents`. The convention above is the deliverable;
its enforcement belongs to Phase 4.
