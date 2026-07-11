# Pitfalls Research

**Domain:** Vault-grounded, delegation-based Claude Code Skill (agent-architecture advisor)
**Researched:** 2026-07-11
**Confidence:** HIGH

This file catalogues what commonly goes wrong for a Skill of agens' exact shape: a read-mostly router that grounds every answer in an Obsidian vault, delegates framework-fit and MCP construction to other tools, logs its own decisions, and gates one write capability behind human approval. Findings are grounded in the wiki-agents notes agens itself depends on (`mcp-security-attack-vectors.md`, `agent-skills-best-practices.md`) and the current Anthropic Skills documentation. Phase numbers reference PROJECT.md's existing plan (Phase 0 consolidation through Phase 4).

## Critical Pitfalls

### Pitfall 1: Trigger description written for a human, not as a trigger signal

**What goes wrong:**
agens ships, works when invoked by slash command, and never fires on its own. The user mentions "I want to build an agent that watches a queue and calls a tool" and agens stays silent, because its description reads like a summary ("Recommends agent architectures grounded in the wiki-agents vault") rather than a list of the phrases a user actually uses to start an agent project.

**Why it happens:**
The `description` field is model-visible trigger metadata pre-loaded into the system prompt, not documentation. The single most common reason a Skill goes unused is a description written for a human reader. A description that names a topic ("agent architecture advice") triggers on everything or nothing; one that lists concrete conditions ("Use when the user describes a new agent or automation project and wants a pattern recommendation, mentions choosing between workflow and agent designs, or asks which agent pattern fits a task") fires reliably. Reported gap is roughly 50% baseline versus 80%+ with a condition-listing, third-person description.

**How to avoid:**
Write the description in the third person as "Use when the user..." plus explicit trigger phrases drawn from how the user actually phrases project starts. Because agens is dual-invoked, keep the slash command as the deterministic fallback and treat auto-trigger as best-effort. Test auto-trigger against a written set of realistic user phrasings before declaring the requirement met, not against the phrasing the author had in mind while writing the description.

**Warning signs:**
Low or zero auto-trigger rate in the invocation log while slash-command use is normal. The author can only make it fire by quoting the description back. Every real invocation is a slash command.

**Phase to address:**
Phase 1 (the read-only recommend capability, where auto-trigger is first wired). Verification: a phrasing test set, not a single happy-path check.

---

### Pitfall 2: Grounding drift — citations quietly stop matching the answer

**What goes wrong:**
agens keeps producing a citation on every recommendation, so the citation discipline looks intact, but the substance of the answer drifts back to the model's general knowledge while the citation degrades into decoration. The path still resolves; the note no longer supports the specific claim, or the note moved and the path now points at nothing, or agens paraphrases training-data pattern advice and staples on the nearest vault path.

**Why it happens:**
Grounding is a discipline, not a data type. A model asked to "recommend a pattern and cite the vault" satisfies the visible contract (a path appears) far more easily than the invisible one (the cited note actually contains the recommendation). Three drift mechanisms compound: (1) the canonical concept note evolves in the vault while agens' behaviour is fixed, so citation and content diverge; (2) vault reorganisation renames or moves notes and citations rot into dead paths; (3) the model, under pressure to answer, generates a general-knowledge answer first and retrofits a citation. PROJECT.md names this exactly — without grounding, agens "degrades into asking Claude cold with extra steps."

**How to avoid:**
Make the citation load-bearing, not cosmetic. Retrieve the note content before recommending, and derive the recommendation from retrieved text rather than citing after the fact. Verify at runtime that the cited path resolves and that the recommended pattern name appears in the retrieved note; refuse or downgrade to "no grounded answer available" when it does not. Pin agens to the single consolidated concept note from Phase 0 so there is one lookup target to keep honest, not six. Prefer an explicit "I could not find a grounded answer in the vault" over an ungrounded answer with a plausible citation.

**Warning signs:**
Recommendations that read like generic pattern advice with a citation appended. Cited paths that no longer exist or point at unrelated notes. The same recommendation for materially different described projects. Answers that would be identical with the vault removed.

**Phase to address:**
Phase 0 (consolidate to one canonical note) and Phase 1 (retrieve-then-recommend, with a runtime citation-resolves-and-matches check as a success criterion).

---

### Pitfall 3: Self-maintained decision log rots — stale, unbounded, and self-contradicting

**What goes wrong:**
agens logs each recommendation append-only, mirroring the vault's log pattern, and the log becomes a liability rather than memory. It grows without bound and starts consuming context on read-back; it accumulates decisions that contradict each other with no reconciliation; and old entries citing since-moved notes make the log itself a source of stale, misleading grounding when agens reads it back.

**Why it happens:**
Append-only logs are given to a Skill as cheap memory, read back on the next invocation. The failure is that "append-only" has no forgetting, no summarisation, and no conflict resolution built in. Over many sessions the log develops three defects: staleness (entries reference a vault state that no longer holds), unbounded growth (read-back cost rises until the log crowds out the actual task), and contradiction (session A recommended pattern X for a described shape, session B recommended Y for the same shape, and nothing reconciles them). If agens then treats its own past log entries as grounding, drift (Pitfall 2) compounds through its own history.

**How to avoid:**
Decide up front what the log is *for* and bound it to that. Treat the log as an audit trail of what agens did, not as a grounding source it trusts — grounding comes from the vault, never from prior log entries. Cap read-back (most-recent-N, or a rolling summary) so growth cannot crowd out the task. Store enough per entry to detect contradiction (described-project shape, recommended pattern, cited note) but do not attempt automatic reconciliation; surface conflicts to the user instead. Record the vault note version or date alongside the citation so a stale entry is visibly stale.

**Warning signs:**
Log read-back growing turn over turn. Contradictory recommendations for equivalent inputs sitting unremarked in the log. agens citing its own log rather than the vault. Log entries whose cited paths no longer resolve.

**Phase to address:**
Phase 1 or 2 (whichever introduces the decision log). Verification: a bounded read-back cost and a rule that grounding never sources from the log.

---

### Pitfall 4: Silent delegation failure when a delegated tool is absent or its interface changed

**What goes wrong:**
agens delegates framework-fit to `gsd-framework-selector`/`gsd-ai-integration-phase` and MCP construction to MCP Builder. When one of those is not installed, or its invocation interface has changed, the delegation fails silently — MCP has no native Skill-to-Skill dependency mechanism, and referencing another Skill by name invokes it only if present and fails silently if not. Worse than a clean error, agens may then fill the gap itself: it reimplements framework-selection judgement inline, the exact anti-goal PROJECT.md forbids, or it produces a confident answer as if delegation had succeeded.

**Why it happens:**
Delegation by name is a soft reference, not a contract. There is no dependency resolver and no interface version check. A missing target produces no exception the calling Skill can catch by default, so absence reads as "nothing happened" and the model routes around it. Interface drift is worse: the target Skill is present but now expects different inputs, so delegation appears to run but the handoff is malformed.

**How to avoid:**
Make delegation explicit and fail loudly. Before delegating, check the target Skill is installed and name the exact fallback when it is not. Design the graceful failure deliberately: state plainly that the delegated capability is unavailable, name the missing Skill, and stop — never silently substitute agens' own reimplementation of framework-selection logic, since that violates the delegation-discipline constraint. Treat "delegated tool missing" and "delegated tool present but returned nothing usable" as distinct, each with its own user-facing message. Pin or document the expected interface of each delegated Skill so drift is detectable rather than silent.

**Warning signs:**
agens answers framework-fit questions itself with no citation to a delegated tool. No user-visible message when a delegation target is absent. Behaviour that is identical whether or not `gsd-framework-selector` is installed.

**Phase to address:**
Phase 2 (framework-fit delegation) and Phase 3/4 (MCP Builder delegation). Verification: run agens with each delegated Skill uninstalled and confirm an explicit, correct failure message rather than a silent reimplementation.

---

### Pitfall 5: The approval gate protects the human decision but not the tool boundary

**What goes wrong:**
agens correctly refuses to build an MCP server without explicit human approval, so the "did the human say yes" gate holds. The gate still fails at its boundary because the thing the human approved is not the thing that runs. The user approves "build an MCP server for X"; MCP Builder then generates and installs a server whose tool descriptions carry model-directed instructions, whose code shells out or reads credential paths, or which pins `@latest` and rug-pulls later. Approval was granted for an intent; execution happened on unreviewed generated artifacts.

**Why it happens:**
MCP's trust model is unusual: a tool description is a model-visible prompt, not documentation a human reads, so malicious or simply wrong instructions can sit in text the approver never sees. MCP "shipped composability first and integrity verification basically never" — the operator is the verification layer, not the ecosystem. A human-approval gate placed at the *decision* ("shall we build?") does nothing about the *content* of what gets built. Two further failure surfaces exist at the boundary itself: a generated or referenced server pinned to `@latest` passes review once and mutates later (rug pull); and a project-checked-in Skill can grant itself broad tool access through `allowed-tools` after the workspace trust dialog is accepted, so the trust decision and the capability grant are separated in time.

**How to avoid:**
Gate the artifact, not just the intent. Put the human approval after generation and before install, on the concrete output — surface the generated tool descriptions verbatim (the model-visible text), the outbound network destinations, any shell execution, and any credential-path reads, so the approver reviews what agens' own vault security note tells them to review. Apply that five-step pre-install audit to MCP-Builder output as a matter of course. Refuse `@latest`; require pinned versions so a reviewed artifact cannot silently mutate. Keep agens itself narrowly scoped in `allowed-tools` and never let the build capability grant itself write or network tools as a side effect of being trusted. Preserve PROJECT.md's sequencing: read-only capabilities earn trust first; the build capability is added last and stays behind this artifact-level gate.

**Warning signs:**
An approval prompt that shows intent ("build server for X?") but not the generated code or tool descriptions. Generated servers installed via `@latest`. agens or its build sub-skill acquiring network or filesystem tools it did not previously hold. Approval granted once, then multiple installs proceeding under it.

**Phase to address:**
Phase 3/4 (the MCP Builder integration and its gate). This is the highest-severity pitfall in the set and the direct target of PROJECT.md's self-mutation/rug-pull constraint. Verification: the approval prompt renders the concrete artifact, and a pinned-version rule blocks `@latest`.

---

### Pitfall 6: agens becomes the monolith it was built to avoid

**What goes wrong:**
Each phase adds "just one more thing" inline — a bit of framework judgement here, a pattern taxonomy restated there, a hardcoded shortcut instead of a delegation — until agens is a single large Skill spanning several of the nine functional Skill types, and the routing-layer thesis is lost. A Skill that spans multiple categories confuses the model about when to fire it, which loops straight back into Pitfall 1.

**Why it happens:**
Reimplementing is faster in the moment than wiring a delegation, and restating known taxonomy feels helpful. PROJECT.md's own analysis is that the bespoke work is the connective routing layer, not a new system — but every phase presents a local temptation to absorb a neighbouring capability rather than route to it. Cramming everything into one document also forfeits progressive disclosure and burns context budget repeating what the model already knows.

**How to avoid:**
Hold the routing-layer scope as a hard boundary at every phase. Fit agens to one functional type (it is a router/advisor, not a taxonomy and not a framework selector). Delegate rather than restate: framework-fit goes to GSD, taxonomy lives in the vault, MCP construction goes to MCP Builder. Keep `SKILL.md` to decision-critical routing instructions and push reference material to on-demand subfolders. Review each phase against the Out-of-Scope list in PROJECT.md before merging.

**Warning signs:**
`SKILL.md` growing past routing logic into pattern content or framework judgement. Sections restating vault taxonomy the model could retrieve. A description that has to cover several unrelated trigger conditions because the Skill now does several jobs.

**Phase to address:**
Every phase (a standing scope check), with the sharpest risk in Phases 2-4 as delegated capabilities are wired.

---

## Technical Debt Patterns

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| Cite the vault after generating the answer instead of retrieving first | Faster to build; answer always has a citation | Grounding drift (Pitfall 2); citation becomes decoration | Never — this defeats agens' Core Value |
| Reference delegated Skills by bare name with no presence check | Less wiring | Silent delegation failure (Pitfall 4); inline reimplementation | Never for the two named delegations; tolerable only for a truly optional enhancement |
| Append-only log with no read-back cap | Simple memory, mirrors vault pattern | Unbounded growth and contradiction (Pitfall 3) | MVP only, if a cap is a named Phase-2 follow-up |
| Gate MCP build on intent ("build server?") not on the generated artifact | Simpler prompt | The rug-pull / poisoned-description surface stays open (Pitfall 5) | Never |
| Restate a bit of framework or taxonomy logic inline "just for this case" | Avoids a delegation round-trip | Monolith drift (Pitfall 6); violates delegation discipline | Never for the two delegated domains |

## Integration Gotchas

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|------------------|
| wiki-agents vault (read) | Assuming a cited path is stable; citing without confirming the note contains the claim | Retrieve note content, verify path resolves and pattern name appears, before recommending |
| `gsd-framework-selector` / `gsd-ai-integration-phase` | Bare-name reference that fails silently when absent; reimplementing on failure | Presence check, explicit unavailable message, no inline substitution |
| MCP Builder | Trusting generated server output because the human approved the intent | Artifact-level review of tool descriptions, network calls, shell-outs, and file reads before install |
| agens' own decision log | Reading past entries back as grounding | Log is audit trail only; grounding always sources from the vault |
| Skill `allowed-tools` frontmatter | Letting the build sub-skill self-grant broad tool access after trust | Keep scope narrow; review project skills before trusting the repository |

## Security Mistakes

| Mistake | Risk | Prevention |
|---------|------|------------|
| Approving MCP build intent rather than the generated artifact | Poisoned tool description or exfiltrating code installed under a valid approval | Move the gate after generation; render the concrete artifact for review |
| Allowing `@latest` for any generated or referenced server | Rug pull: reviewed-clean server mutates later | Require pinned exact versions; reject `@latest` |
| Broad filesystem or network tool grant on the build capability | Credential-path reads (`.ssh`, `.env`, `.aws`) or exfiltration | Scope `allowed-tools` narrowly; no self-grant of write/network tools |
| Treating "N clean tools + 1 approved build" as mostly-safe | Cross-server hijacking; trust is multiplicative, not additive | Audit each generated server independently against the five-step checklist |
| Trusting a project-checked-in agens without reviewing its `allowed-tools` | A skill grants itself broad access on trust-dialog acceptance | Review project skills before accepting the workspace trust dialog |

## UX Pitfalls

| Pitfall | User Impact | Better Approach |
|---------|-------------|-----------------|
| agens fires on unrelated requests due to an over-broad description | Noise; the user disables auto-trigger | Condition-listing description scoped to project-start phrasings |
| agens never fires (over-narrow or human-worded description) | User forgets it exists; falls back to asking Claude cold | Test against realistic phrasings; keep slash command as deterministic fallback |
| Silent delegation makes agens answer as if it had authority it lacks | User acts on ungrounded framework advice | Explicit "delegated capability unavailable" message; stop rather than guess |
| Recommendation with a citation the user cannot verify | Erosion of the trust the citation was meant to build | Cite a resolvable path; ideally quote the supporting line |

## "Looks Done But Isn't" Checklist

- [ ] **Auto-trigger:** Often "works" only for the author's phrasing — verify against an independent phrasing test set
- [ ] **Citation discipline:** Often present but cosmetic — verify the cited path resolves and the note actually contains the recommendation
- [ ] **Delegation:** Often untested with the target absent — verify an explicit failure message when each delegated Skill is uninstalled
- [ ] **Decision log:** Often unbounded — verify read-back is capped and grounding never sources from the log
- [ ] **Approval gate:** Often gates intent not artifact — verify the prompt renders the generated tool descriptions and code before install
- [ ] **Version pinning:** Often defaulted to `@latest` — verify pinned versions are required and `@latest` is rejected

## Recovery Strategies

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|----------------|
| Trigger description mismatch (P1) | LOW | Rewrite description as third-person conditions; re-test against phrasing set |
| Grounding drift (P2) | MEDIUM | Add retrieve-then-recommend and runtime citation check; re-pin to the canonical note |
| Log rot (P3) | LOW | Cap read-back; add version stamps; stop treating the log as grounding |
| Silent delegation (P4) | MEDIUM | Add presence checks and explicit failure messages; remove any inline reimplementation |
| Approval-gate boundary breach (P5) | HIGH | Move gate to artifact; audit and pin any already-installed generated servers; narrow tool grants |
| Monolith drift (P6) | HIGH | Extract absorbed logic back to delegation; trim `SKILL.md` to routing; refit description to one job |

## Pitfall-to-Phase Mapping

| Pitfall | Prevention Phase | Verification |
|---------|------------------|--------------|
| P1 Trigger description mismatch | Phase 1 | Auto-trigger fires on an independent phrasing test set |
| P2 Grounding drift | Phase 0 + Phase 1 | Cited path resolves and pattern name appears in the retrieved note |
| P3 Decision-log rot | Phase 1/2 (log introduction) | Read-back is capped; grounding never sources from the log |
| P4 Silent delegation failure | Phase 2 + Phase 3/4 | agens run with each delegated Skill absent yields an explicit message, no reimplementation |
| P5 Approval-gate boundary breach | Phase 3/4 | Approval prompt renders the generated artifact; `@latest` rejected |
| P6 Monolith drift | Every phase | Each merge checked against PROJECT.md Out-of-Scope; `SKILL.md` stays routing-only |

## Sources

- wiki-agents: `10_Sources/Blog/mcp-security-attack-vectors.md` — Shirley, "MCP Security: 6 Attack Vectors and a 5-Step Audit", AI Builder Club, 11 June 2026 (rug pulls, tool-description poisoning, cross-server hijacking, five-step pre-install audit). HIGH.
- wiki-agents: `10_Sources/Blog/agent-skills-best-practices.md` — Jason Zhou, "Anthropic's 300+ Claude Code Skills: Lessons Learned", AI Builder Club, 9 June 2026 (description-as-trigger-signal, append-only log memory, silent Skill-to-Skill dependency failure, gotchas sections). HIGH.
- Anthropic, "Extend Claude with skills", Claude Code Docs, https://code.claude.com/docs/en/skills (`disable-model-invocation`, `allowed-tools` self-grant on trust, invocation control). HIGH.
- Anthropic, "Agent Skills", Claude Platform Docs, https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview (description pre-loaded as trigger metadata; specificity and reliability). MEDIUM-HIGH.
- agens `.planning/PROJECT.md` — stated risks (monolith, unreviewed MCP build), delegation discipline, citation discipline, sequencing constraints. HIGH.

---
*Pitfalls research for: vault-grounded delegation-based Claude Code Skill*
*Researched: 2026-07-11*
