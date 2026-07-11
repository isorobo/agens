# Feature Research

**Domain:** Vault-grounded agent-architecture advisor Skill (Claude Code Skill that interrogates a project, returns a cited recommendation from a personal Obsidian vault, delegates to other tools, and logs its own decisions back into the vault)
**Researched:** 2026-07-11
**Confidence:** MEDIUM-HIGH (Anthropic Skill best practices and RAG citation-grounding research are HIGH; direct prior-art tools for the exact combination are MEDIUM, drawn from two close analogues plus community patterns)

## Prior Art Surveyed

Four categories of comparable tool inform this landscape. None does all of what agens does, but each covers part of it well.

1. **Agent-only Obsidian vaults** ("foundry" pattern, `obsidian-second-brain`). Source-note to concept-note to `log.md` workflows. Directly comparable to the wiki-agents LLM-wiki structure agens reads. Best guidance on decision logging, citation-from-concept-to-source, and the attribution-confusion pitfall.
2. **Citation-backed RAG systems.** The literature on trustworthy citations versus decorative ones. Best guidance on what makes a citation load-bearing rather than cosmetic.
3. **Orchestrator / delegation Skills.** Skills that route to other Skills rather than reimplementing them. Best guidance on the delegation discipline and the "don't orchestrate too early" warning.
4. **Anthropic's own Skill authoring model.** Progressive disclosure, the research-synthesis workflow with a mandatory "verify citations" step, and the plan-validate-execute gate. Best guidance on the human-approval gate shape and auto-trigger descriptions.

## Feature Landscape

### Table Stakes (Users Expect These)

Missing any of these and agens degrades into "asking Claude cold with extra steps" - the exact failure the PROJECT.md Core Value warns against.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Fixed questionnaire before any recommendation | A grounded advisor must gather project shape (goal, workflow shape, data sensitivity, latency/cost) before it can match a pattern. Structured interviews collect the exact inputs the lookup needs. | LOW | Anthropic's "conditional workflow" and "template" patterns fit. Keep it to the four PROJECT.md dimensions; do not let it sprawl into open-ended intake. |
| Recommendation cites a specific vault note by path | This is the Core Value. A recommendation without a `path#heading` citation is indistinguishable from cold model knowledge. | LOW-MEDIUM | The citation must be a real, resolvable vault path. See "citation must resolve" under Differentiators - the trust work is in verifying the citation, not printing one. |
| Read the canonical concept note as the single lookup target | The vault's 6-source `topic/agent-patterns` cluster is consolidated into one concept note (PROJECT.md Phase 0). agens reads that note, not six scattered sources. | LOW | Concept-note-as-single-target mirrors the foundry "compile to `/wiki`" step. Progressive disclosure: SKILL.md points to the concept note, loaded only when triggered. |
| Append-only decision log mirroring the vault's `log.md` | The foundry and second-brain tools both maintain a processing log. agens must record each recommendation append-only, in the vault's own `log.md` style, so the practice is auditable and self-consistent. | LOW | Append-only, never rewrite prior entries. This is a write, but a safe, additive one - distinct from the gated MCP build. |
| Delegate framework-fit questions to `gsd-framework-selector` / `gsd-ai-integration-phase` | The GSD family already owns framework selection. Reimplementing it duplicates installed capability and drifts from it. | LOW-MEDIUM | Handoff/routing pattern. agens classifies the question and routes; it does not answer framework-fit itself. |
| Offer MCP Builder but never auto-run it | Tooling need identified means offer the build behind a human gate. Anthropic's plan-validate-execute pattern and the vault's own MCP Security note both demand this. | MEDIUM | The gate is table stakes precisely because the un-gated version is the self-mutation/rug-pull anti-pattern. |
| Both slash-command and auto-trigger invocation | PROJECT.md confirms both matter. The `description` field drives auto-trigger; the slash command gives explicit control. | LOW | Third-person, key-term-rich description per Anthropic guidance. Trigger on natural project-starting language. |

### Differentiators (Competitive Advantage)

These are where agens earns its existence over cold Claude. They cluster around one theme: making the citation load-bearing, not decorative.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Citation must resolve before the recommendation is emitted | The single biggest trust lever. A hallucinated citation is worse than no citation - the user lowers their guard on seeing a source. agens must confirm the cited path exists and contains the claimed pattern before returning it. | MEDIUM | Directly from RAG-grounding research: "citation relevance swap" and "human auditor replay" tests. Implement as a pre-emit check: does the note at this path exist, and does the recommended pattern appear in it? |
| Refuse to recommend when the vault lacks support | Disciplined refusal beats fluent gap-filling with a nearby-but-irrelevant citation. If no vault note matches the project shape, agens says so rather than citing the closest note as cover. | MEDIUM | The "unsupported query refusal" test. This is what separates a grounded advisor from a decorated guesser. High trust value, moderate cost. |
| Quote the supporting passage, not just the path | Line-level or passage-level traceability lets the user verify the claim from the citation alone, without hunting through the note. | LOW-MEDIUM | "Line-level citation mapping" and "human auditor replay." Return the specific block that supports the pattern choice, not just `note.md`. |
| Decision log links back to the cited note | The log entry records not just the recommendation but the citation that grounded it, as a wikilink. This makes the log itself an audit trail of grounding, not just an activity record. | LOW | Foundry concept notes "cite their sources"; agens' log should too. Turns the log into a reviewable grounding history. |
| Preserve qualifiers from the source | If the vault note says a pattern applies "only for low-latency, stateless workflows," the recommendation must carry that restriction, not flatten it into a blanket endorsement. | MEDIUM | "Qualifier preservation" and "paraphrase fidelity" tests. Prevents the common RAG failure of dropping limiting language and over-generalising. |
| Author attribution in every written note (agens vs. user) | The foundry author's top pitfall was losing track of which notes an agent wrote. Tagging every agens-written log entry as agent-authored keeps the vault's human-written knowledge clean. | LOW | Sentinel-marker / attribution convention. Cheap insurance against the single most-cited vault-writing mistake. |

### Anti-Features (Commonly Requested, Often Problematic)

PROJECT.md already names three (no standalone runtime, no autonomous MCP building, no multi-user). Research surfaces several more that seem helpful but corrode the Core Value.

| Feature | Why Requested | Why Problematic | Alternative |
|---------|---------------|-----------------|-------------|
| Standalone runtime / new agent infrastructure | "Make it a real app." | Duplicates the Claude Code session, tool-calling, and runtime the ecosystem already provides. PROJECT.md Out of Scope. | Ship as a Skill inside Claude Code. |
| Autonomous MCP server construction | "Just build the tool for me." | Exactly the self-mutation/rug-pull pattern the vault's MCP Security note warns against. An agent that builds and trusts its own tools unreviewed. | Offer MCP Builder behind a mandatory human-approval gate. |
| Multi-user / shared-product features | "Make it useful for the team." | Scope explosion; agens is one person's practice tool. PROJECT.md Out of Scope. | Personal, single-user Skill. |
| Reimplementing the pattern taxonomy or framework-selection logic | "Bake the knowledge in so it's self-contained." | Duplicates wiki-agents sources and the GSD family; the copy drifts from the source it was meant to ground against. Defeats citation discipline. | Read the vault; delegate to GSD. agens is the routing layer, not a new store of knowledge. |
| Vector-embedding RAG pipeline over the vault | "Proper RAG needs embeddings and semantic search." | Overkill for one consolidated concept note; adds infrastructure, drifts from the plaintext vault, and reintroduces the very grounding-failure modes (misaligned passages) that citation-verification exists to catch. | Claude reads the markdown note directly and greps it. Anthropic's own knowledge-base pattern rejects the embedding pipeline for exactly this scale. |
| Free-text open-ended intake instead of a fixed questionnaire | "Let the user describe the project however they like." | Loses the structured inputs the lookup needs; makes recommendations non-reproducible and the questionnaire dimension of the tool decorative. | Fixed four-dimension questionnaire (goal, workflow shape, data sensitivity, latency/cost). |
| Confidence scores or star ratings on recommendations | "Show how sure it is." | A number invites the "illusion of groundedness" - it reads as verified certainty while masking whether the citation actually supports the claim. | Show the supporting passage and let the user judge. Refuse when unsupported rather than scoring low. |
| Auto-rewriting or "healing" existing vault notes | Nightly-agent tools do contradiction sweeps and orphan-healing. | agens reads human-authored knowledge; rewriting it destroys the attribution boundary and risks mutating the ground truth it cites against. | Append-only log writes only. Never edit the concept note or any human-authored source. |
| Orchestrating a wide fleet of sub-skills up front | "Wire it into everything." | "The mistake is jumping to orchestration too early." agens has exactly two delegations (GSD, MCP Builder); a general orchestrator is premature. | Two named, deliberate handoffs. Add more only when proven necessary. |
| Citing multiple notes to look thorough | "More citations = more grounded." | Padding with tangential citations is the decorative-citation failure. Each extra weakly-related citation lowers the user's guard further. | One load-bearing citation with its supporting passage. |

## Feature Dependencies

```
Fixed questionnaire
    └──feeds──> Recommendation with resolved citation
                    └──requires──> Canonical concept note (single lookup target, Phase 0)
                    └──requires──> Citation-resolves check (pre-emit)
                                       └──enables──> Refuse-when-unsupported
                    └──produces──> Append-only decision log entry
                                       └──enhances──> Log links back to cited note
                                       └──requires──> Author attribution convention

Recommendation with resolved citation
    └──may-route-to──> gsd-framework-selector / gsd-ai-integration-phase  (framework-fit)
    └──may-offer──> MCP Builder  (tooling need)
                        └──gated-by──> Human-approval gate  (write capability)

Read-only capabilities (questionnaire, recommend, log)
    └──ship-before──> Write capability (gated MCP build)   [PROJECT.md sequencing constraint]

Vector-embedding RAG ──conflicts──> Direct markdown read of concept note
Free-text intake ──conflicts──> Fixed questionnaire
Auto-run MCP build ──conflicts──> Human-approval gate
```

### Dependency Notes

- **Recommendation requires the canonical concept note:** agens has one lookup target by design (PROJECT.md Phase 0). The consolidation must land before the recommend capability is meaningful.
- **Citation-resolves check enables refuse-when-unsupported:** the same mechanism that verifies a citation resolves is what detects the "no supporting note exists" case. Build them together.
- **Log-links-back enhances the plain log:** the append-only log is table stakes; making each entry carry the grounding wikilink is the differentiator layered on top.
- **Read-only ships before write (hard sequencing):** PROJECT.md constraint. Interrogate, recommend, and log earn trust before the gated MCP build is added. This is a phase-ordering constraint, not a preference.
- **Vector RAG conflicts with direct read:** choosing the embedding pipeline abandons the plaintext-vault grounding model and reintroduces retrieval-stage grounding failures. Pick direct markdown read.

## MVP Definition

### Launch With (v1) - Read-Only Spine

Minimum to validate that a vault-grounded advisor beats cold Claude.

- [ ] Fixed four-dimension questionnaire - without structured inputs the lookup is not reproducible
- [ ] Canonical concept note as single lookup target (Phase 0 consolidation) - the thing agens grounds against
- [ ] Recommendation with a resolving citation (path + supporting passage) - the Core Value
- [ ] Citation-resolves pre-emit check - the difference between grounded and decorative
- [ ] Refuse-when-unsupported - disciplined refusal over fluent gap-filling
- [ ] Append-only decision log in `log.md` style, with author attribution - auditable, self-consistent with the vault
- [ ] Both slash-command and auto-trigger invocation - PROJECT.md confirmed both matter
- [ ] Delegate framework-fit to GSD - avoids reimplementing installed capability

### Add After Validation (v1.x)

- [ ] Log entry links back to the cited note as a wikilink - trigger: once the plain log is trusted and used
- [ ] Qualifier-preservation discipline in the recommendation - trigger: first observed over-generalisation of a source restriction
- [ ] Passage-level quoting refinement - trigger: user reports hunting through the cited note to verify

### Future Consideration (v2+) - Gated Write

- [ ] Offer MCP Builder behind human-approval gate - defer: read-only spine must earn trust first (PROJECT.md sequencing); this is the write capability and carries the self-mutation risk
- [ ] Plan-validate-execute intermediate output before any build - defer: only relevant once the gated build exists

## Feature Prioritization Matrix

| Feature | User Value | Implementation Cost | Priority |
|---------|------------|---------------------|----------|
| Fixed questionnaire | HIGH | LOW | P1 |
| Canonical concept note (Phase 0) | HIGH | LOW | P1 |
| Recommendation with resolving citation | HIGH | MEDIUM | P1 |
| Citation-resolves pre-emit check | HIGH | MEDIUM | P1 |
| Refuse-when-unsupported | HIGH | MEDIUM | P1 |
| Append-only decision log + attribution | HIGH | LOW | P1 |
| Slash + auto-trigger invocation | MEDIUM | LOW | P1 |
| Delegate framework-fit to GSD | MEDIUM | LOW-MEDIUM | P1 |
| Log links back to cited note | MEDIUM | LOW | P2 |
| Qualifier preservation | MEDIUM | MEDIUM | P2 |
| Passage-level quoting | MEDIUM | LOW-MEDIUM | P2 |
| Offer MCP Builder behind gate | HIGH | MEDIUM | P3 (sequenced last) |

**Priority key:**
- P1: Must have for launch (read-only spine)
- P2: Should have, add when the spine is trusted
- P3: Gated write capability, deliberately last per PROJECT.md sequencing

## Competitor Feature Analysis

| Feature | Agent-only vault / obsidian-second-brain | Citation-RAG literature | agens approach |
|---------|------------------------------------------|-------------------------|----------------|
| Interrogation | `/create-command` interview flow; `/obsidian-challenge` adversarial questioning | n/a | Fixed four-dimension questionnaire, not adversarial, not open-ended |
| Cited recommendation | Concept notes "must cite sources"; provenance back to exact source block | Grounding = claims materially supported by cited evidence; citation must resolve | Path + supporting passage, verified to resolve before emit |
| Guarding against fake citations | Provenance tagging | 11 tests: relevance swap, qualifier preservation, unsupported-query refusal, human-auditor replay | Pre-emit resolve check + refuse-when-unsupported + qualifier preservation |
| Delegation | Delegates to Perplexity, Gemini, Whisper for research | n/a | Two deliberate handoffs: GSD (framework-fit), MCP Builder (build) |
| Decision logging | `/obsidian-decide` logs decisions; `--formal` writes an ADR | n/a | Append-only `log.md`-style entry, author-attributed, links to citation |
| Write safety | "Never deletes or modifies destructively without confirmation"; sentinel markers preserve user edits | plan-validate-execute for high-stakes ops | Append-only writes only; MCP build behind mandatory human gate; never edits human notes |
| Known mistake to avoid | Attribution confusion (which notes were agent-written); navigation overload | Illusion of groundedness from decorative citations; dropped qualifiers | Author attribution on every write; one load-bearing citation, not many |

## Sources

- Agent-only Obsidian vault ("foundry" source/concept/log workflow, attribution pitfall): https://www.thethinkers.club/p/building-an-agent-only-obsidian-vault
- obsidian-second-brain Skill (decision logging, provenance, interview flow, write-safety confirmations): https://github.com/eugeniughelbur/obsidian-second-brain
- RAG grounding - 11 tests that expose fake citations (relevance swap, qualifier preservation, unsupported-query refusal, human-auditor replay): https://medium.com/@Nexumo_/rag-grounding-11-tests-that-expose-fake-citations-30d84140831a
- Trustworthy RAG with in-text citations (traceability, illusion of groundedness): https://haruiz.github.io/blog/improve-rag-systems-reliability-with-citations
- Why citation-based RAG still hallucinates (partial-hallucination masking): https://yaihq.com/research/citation-based-rag-still-hallucinates
- Anthropic Skill authoring best practices (progressive disclosure, research-synthesis workflow with verify-citations step, plan-validate-execute gate, auto-trigger descriptions): https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices
- Orchestrator Skill / delegation patterns ("don't orchestrate too early", handoff routing): https://www.mindstudio.ai/blog/what-is-orchestrator-skill-claude-code
- Personal knowledge base with Claude Code (markdown-over-embeddings, cite-back-to-wiki-page): https://claudeskills360.com/blog/claude-code-knowledge-base-setup/

---
*Feature research for: vault-grounded agent-architecture advisor Skill*
*Researched: 2026-07-11*
