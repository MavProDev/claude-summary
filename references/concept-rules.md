# Concept Note Rules

## When to Create Concept Notes

Create a standalone concept note for every genuinely new item from these categories:

**DESIGN PATTERNS** — New patterns discovered or applied
- Examples: [[Propose-and-Approve Pattern]], [[Self-Learning Loop]], [[Circuit Breakers]], [[Adaptive Reuse Architecture]], [[Event Sourcing]], [[CQRS]]

**TECHNOLOGIES** — Tools used for the first time or explored in depth
- Examples: [[SARIF Format]], [[CycloneDX SBOM]], [[Drizzle ORM]], [[Hono.js]], [[Phaser 3]], [[Venice.ai]]

**FRAMEWORKS/LIBRARIES** — Newly introduced to the workflow
- Examples: [[shadcn/ui]], [[Prisma]], [[React Query]], [[Tailwind CSS]], [[FastAPI]]

**VULNERABILITY CLASSES** — Security concepts found or discussed
- Examples: [[SQL Injection]], [[Prompt Injection]], [[SSRF]], [[Agent Hallucination in Standards Data]]

**STANDARDS/SPECIFICATIONS** — Referenced during work
- Examples: [[OWASP Agentic 2026]], [[NIST SSDF]], [[DISA STIG]], [[CVSS 4.0 Scoring]], [[MITRE ATT&CK]], [[MITRE ATLAS]]

**ARCHITECTURAL DECISIONS** — Reusable patterns worth documenting
- Examples: [[Adaptive Reuse Architecture]], [[Microservice Boundary Design]], [[Event-Driven Architecture]]

**SIGNIFICANT BUGS** — Bugs that taught a lesson worth preserving
- Examples: [[React2Shell]], [[Markdown Parse Entity Crash]], [[Agent Hallucination in Standards Data]]

**BUSINESS/STRATEGY** — Concepts discussed in strategic context
- Examples: [[Data Monetization]], [[Telemetry Collection]], [[Paper-to-Live Progression]], [[Knowledge Confidence Decay]]

**ALGORITHMS/MATH** — Computational concepts used
- Examples: [[Kelly Criterion]], [[Brier Score]], [[Semantic Similarity Thresholds]], [[Drawdown Containment]]

**PEOPLE/ORGANIZATIONS** — Newly relevant entities
- Examples: [[CrowdStrike]], [[Bellingcat]], [[Anthropic]], [[Intercounty Appliance Co-op]]

## When NOT to Create

**TRIVIAL CONCEPTS** — Things every developer knows. No dedicated page needed.
- NO: [[Variable]], [[For Loop]], [[String]], [[Array]], [[Function]], [[Class]], [[If Statement]]
- YES: [[Circuit Breakers]], [[SARIF Format]], [[Brier Score]], [[Knowledge Confidence Decay]]

**EXISTING NOTES** — Always dedup first (see below). Never create a duplicate.

**THE GRAPH-WORTHINESS TEST:** Would a dedicated page with 15+ lines of substantive content, cross-project references, and 5+ wikilinks be useful to someone encountering this concept 6 months from now? If yes, create it. If no, skip it.

## Deduplication Protocol (Vault-Wide)

Duplicate concept notes fragment the graph and split edges. Always search before creating.

**Search scope:** The ENTIRE vault, not just ClaudeCode Notes/.
**Vault root:** `C:\Users\user\cloud-sync\Documents\Obsidian Notes\Obsidian Vault\`

For each concept note candidate:

1. **Primary search:**
   `Glob "${VAULT_ROOT}/**/*ConceptName*.md"`

2. **Variation searches:**
   - Singular/plural: "Circuit Breaker" AND "Circuit Breakers"
   - Hyphenated/spaced: "Self-Learning Loop" AND "Self Learning Loop"
   - Abbreviated/full: "OWASP" AND "Open Web Application Security Project"
   - If `[[` `]]` in pattern causes issues, use wildcard: `*Concept Name*.md`

3. **Decision matrix:**
   - **Exact match found** — DO NOT create. Link to the existing note in the session log's Related section.
   - **Partial match found** (similar but not identical) — Read the existing note. If it covers the same ground, link to it. If genuinely different, create the new note and cross-link them.
   - **No match found** — Create the concept note.

4. **Expanding existing notes:**
   If the session significantly expanded understanding of an existing concept:
   - Use **Edit** (not Write) to append a `## Session Update — YYYY-MM-DD` section
   - Include what was learned, how it applies, and a backlink to the session note
   - This strengthens the existing node instead of creating a near-duplicate

## Backlink Injection Protocol

Session notes link to concepts via wikilinks. Concepts should link back to sessions that developed them.

**Scope:** Up to 5 most significant concept connections per session. Don't spam-edit 30 notes.

For each selected existing concept note:
1. Read the note
2. Find the `## Related` section
3. Check if the current session note is already listed
4. If NOT listed:
   - Use **Edit** to append: `- [[Session Note Title]] — referenced in YYYY-MM-DD session`
   - If no `## Related` section exists, append one before the bottom tags
5. If already listed: skip

**Selection criteria:**
- Prioritize concepts where the session added substantial new understanding
- Deprioritize casual mentions ("we used [[Python]]" doesn't need a backlink)
- Prioritize cross-project connections

**Safety:**
- Only Edit notes in `ClaudeCode Notes/` or paths confirmed writable
- If Edit fails, log the failure and continue — don't halt the skill

## Graph Intelligence Principles

1. **Nodes without edges are useless.** A concept note with zero outgoing wikilinks is an orphan. Minimum 5 wikilinks per concept note.

2. **Edges > nodes.** Adding a wikilink to an existing note strengthens the graph more than creating a new stub. Prefer enriching existing notes over thin new ones.

3. **Bidirectional > unidirectional.** Session → concept links are automatic (wikilinks). Concept → session backlinks make the graph navigable both ways.

4. **Cross-project links are gold.** A note about [[Rate Limiting]] that references [[ExampleProjectA]], [[ExampleProjectE]], and [[ExampleProject]] creates three cross-project edges — the most valuable connections in the graph.

5. **The MOC is the spine.** The Session Index MOC makes the graph navigable at scale. Always update it.

6. **Dedup before create.** Duplicates fragment the graph. Searching first is not optional.
