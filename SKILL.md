---
name: summary
description: >
  Writes comprehensive Obsidian session summary notes and concept notes at the
  end of every Claude Code session. Produces wikilink-dense, factually precise
  session logs with tables, mermaid diagrams, git logs, and architecture
  breakdowns. Creates standalone concept notes for new patterns, technologies,
  vulnerabilities, standards, and architectural insights — expanding the user's
  Obsidian knowledge graph with every session. Maintains a running ClaudeCode
  Session Index MOC and injects bidirectional backlinks into existing concept
  notes for graph growth. Use this skill whenever the user says any of: "write
  the session note", "document this session", "time for the session summary",
  "save the session notes", "write the notes", "end of session", "session
  summary", "obsidian note", or invokes /summary. Also use when the user
  appears to be wrapping up a session and mentions documentation or notes.
  ALWAYS execute immediately without asking permission. This skill applies to
  ALL projects universally — it detects the project automatically.
---

# /summary — Obsidian Session Summary Skill

## Live Session Context (auto-injected)

**Working Directory:** !`pwd`
**Current Date:** !`date +%Y-%m-%d`
**Current Branch:** !`git branch --show-current 2>/dev/null || echo "not a git repo"`
**Recent Commits (last 25):**
```!
git log --oneline -25 2>/dev/null || echo "no git history"
```
**Files Changed (uncommitted):**
```!
git diff --stat 2>/dev/null || echo "no changes"
```
**Git Status:**
```!
git status --short 2>/dev/null || echo "not a git repo"
```
**CLAUDE.md (first 50 lines, if exists):**
```!
head -50 CLAUDE.md 2>/dev/null || echo "no CLAUDE.md found"
```

---

## Path Constants

Configure these before using the skill. The skill resolves paths in this order:

1. **Environment variables** (recommended):
   - `CLAUDE_SUMMARY_VAULT_ROOT` — absolute path to your Obsidian vault root
   - `CLAUDE_SUMMARY_NOTES_DIR` — absolute path where ClaudeCode session notes live (defaults to `${VAULT_ROOT}/ClaudeCode Notes`)

2. **Config file** (fallback): `~/.config/claude-summary/config.json` with `{ "vaultRoot": "...", "notesDir": "..." }`

3. **Home-dir auto-detect** (last resort): `~/Documents/Obsidian Vault/ClaudeCode Notes`

All notes — session logs AND concept notes — are written to NOTES_DIR.

**Path safety (platform-agnostic, Bash-compatible):**
- Forward slashes in all Bash commands (works on Windows Git Bash and Unix)
- Double-quote paths that may contain spaces or apostrophes
- Write atomically — full content in a single Write call (cloud-synced folders like OneDrive / Dropbox / iCloud handle syncing in background)
- `[[` `]]` and `—` (em dash) in filenames are valid on Windows NTFS and Linux/macOS ext4/APFS, and work in Obsidian
- **Cloud-sync race awareness:** if the vault lives on OneDrive / Dropbox / iCloud, files can be modified between your Read and Edit calls by the sync agent, by Obsidian itself, or by your own prior Edits to the same file earlier in the session. If you get `File has been modified since read`, re-Read and retry. This is normal; don't panic.
- **Write requires prior Read for existing files.** If a file already exists, you MUST Read it before any Edit or Write — Write will error on an unread existing file. Always Glob FIRST to discover whether a file exists, then Read-then-Edit for appends (never Write-over-existing).

---

## Arguments

If the user typed `/summary "focus on the auth refactor"`, use $ARGUMENTS to guide emphasis.
If no arguments, proceed normally with full documentation.

---

## The Workflow

Execute immediately. No confirmation prompts. No "would you like me to?" Just write.

### Step 1: Setup & Project Detection

1. Verify NOTES_DIR exists. If not: `mkdir -p "${NOTES_DIR}"`
2. Determine the project:
   - Read the DCI-injected CLAUDE.md content (already in context above)
   - Read `${CLAUDE_SKILL_DIR}/references/projects.md` for the known project table
   - Match using detection priority: CLAUDE.md → package.json name → git remote → directory basename
   - If known project: use canonical name. If unknown: use detected name, title-cased.
3. Determine the model: State which model you are currently running as (you know this — it's self-knowledge).
4. Estimate duration: Use git timestamps if available (`git log --format="%ai"` earliest today vs latest). If no commits, estimate from conversation depth (short Q&A = ~15min, deep build session = ~2-4h).

### Step 2: Gather What Changed

1. Review the DCI-injected git data (already in context above):
   - Recent commits, uncommitted changes, working tree status
2. If commits exist, determine session boundary:
   - Identify commits belonging to THIS session (today's date, or after the last known session)
   - Run: `git diff --stat HEAD~N` for aggregate stats (N = session commit count)
   - Run: `git diff HEAD~N --name-status` for file-level changes
3. If no git repo: note it. Check file modification times as fallback: `find . -maxdepth 3 -mmin -360 -type f 2>/dev/null | head -30`
4. Count: total commits, files changed, lines added, lines removed
5. Get git log for the note: `git log --format="%h %s" -20`

### Step 3: Review Session Conversation

This is the most critical step. Scan the conversation history (available because this skill runs inline) and extract everything below.

**Compaction awareness:** If the conversation was long enough for Claude Code to compress earlier messages, some granular detail (exact error messages, specific reasoning discussions) may have been lost. When this has happened:
- Acknowledge it in the Summary: "Note: early session context was compacted; git data supplements."
- Lean heavily on the DCI-injected git data (commits, diffs, file changes) — this is compaction-proof and proves what changed even if the conversation about *why* was compressed.
- For really long sessions, consider recommending mid-session `/summary` invocations in the Next Steps section so future sessions capture everything at full fidelity.

Extract:

1. **THE HEADLINE** — The single most important thing that happened. Specific and data-driven.
2. **KEY DECISIONS** — Every significant decision + reasoning. "We chose X because Y."
3. **BUGS & FIXES** — Symptoms, root cause, fix. Include actual error messages in code blocks.
4. **ARCHITECTURE** — System design changes, data flow mods, component relationships → mermaid diagram candidates.
5. **NEW TECHNOLOGIES** — Frameworks, tools, patterns introduced → concept note candidates.
6. **RESEARCH** — What was searched, found, concluded. Agent dispatches and their findings.
7. **PROBLEMS & SOLUTIONS** — Integration issues, design challenges, performance problems.
8. **STRATEGIC CONTEXT** — Business decisions, roadmap changes, prioritization shifts.
9. **ERROR MESSAGES & LOGS** — Actual terminal output in fenced code blocks.
10. **METRICS** — Line counts, file counts, commit counts, test results, benchmarks.

### Step 4: Identify & Deduplicate Concept Note Candidates

1. From Steps 2-3, compile a list of concept note candidates.
2. Read `${CLAUDE_SKILL_DIR}/references/concept-rules.md` for creation criteria and the graph-worthiness test.
3. For EACH candidate, search the vault for existing notes:
   - `Glob "${VAULT_ROOT}/**/*ConceptName*.md"` (search entire vault, not just ClaudeCode Notes/)
   - Check variations: singular/plural, hyphenated/spaced, abbreviated/full
4. Decision for each:
   - **Exists** → link to it in the session log's Related section. If the session significantly expanded understanding, Read the file first, then Edit to append a `## Session Update — YYYY-MM-DD` section. Do not use Write — it will error on the unread existing file and would clobber existing content even if it didn't.
   - **Doesn't exist** → mark for creation in Step 6.
5. **Same-day multi-scope guard.** If the existing note already has a `## Session Update — <today>` section (e.g., from an earlier session on a different project the same day), add a scope suffix to disambiguate: `## Session Update — 2026-04-18 (ClaudeSuite integration)` vs `## Session Update — 2026-04-18 (SolScore ship)`. Do NOT merge sections — they're distinct sessions with distinct contexts and future-you needs to tell them apart.
6. **Runbook-adjacent concepts.** If the session produced an in-repo procedural runbook (e.g., `docs/runbooks/<X>.md`), create a corresponding concept note `[[<X> Runbook]]` that describes the concept + motivation + links to the runbook file location — but does NOT duplicate the runbook's step-by-step content. The concept note serves graph growth; the runbook serves the repo. They are complementary, not redundant.
7. Record results: which already existed, which are genuinely new.

### Step 5: Write Session Log

1. Read `${CLAUDE_SKILL_DIR}/references/formatting.md` for the complete format, section templates, and gold standard examples.
2. Construct filename: `[[ProjectName — Topic Summary — YYYY-MM-DD]].md`
   - Em dash ` — ` (space-emdash-space), NEVER hyphen
   - Topic: 3-8 words, the headline from Step 3
   - Multiple sessions today? Glob `*ProjectName*YYYY-MM-DD*` in NOTES_DIR to check. Append `Session 2` if needed.
3. Build ALL 10 required sections IN ORDER:
   - H1 + metadata (Date, Project wikilink, Session # if obvious, Duration, Model)
   - `---` then `## Summary` (2-5 sentences, mission + accomplishment + headline)
   - `---` then `## What Was Done` (detailed breakdown, tables, sub-sections)
   - `---` then `## Key Decisions & Reasoning`
   - `---` then `## Architecture / Technical Details` (mermaid diagrams, code blocks)
   - `---` then `## Files Modified` (table: File | Status | Purpose)
   - `---` then `## Next Steps` (prioritized bullets)
   - `---` then `## Git Log` (fenced code block)
   - `---` then `## Related` (wikilinked concepts + sessions + project)
   - `---` then tags: `#claudecode #session-log #project-tag #topic-tags`
4. Write the session log to NOTES_DIR using a single Write call with the full content.

**Quality standards (built in, not a separate checklist):**
- 20+ unique `[[wikilinks]]` — every concept, tool, project, framework, library, pattern, standard, named entity
- At least one table
- Mermaid diagram if architecture/flow/state was discussed
- Git log section if any commits were made
- No filler. Every sentence carries factual, data-driven information.
- Exact numbers, exact names, actual error messages quoted in code blocks.

### Step 6: Write Concept Notes

1. For each NEW concept (passed dedup in Step 4):
   - Filename: `[[Concept Name]].md` (title case)
   - Structure per formatting.md: YAML frontmatter → H1 → opening paragraph → Core Idea → Implementations (with project sub-headers) → Why It Matters → Related → bottom tags
   - Minimum 15 lines of real content. Minimum 5 wikilinks.
   - Tags at BOTH top (frontmatter) and bottom.
   - Write to NOTES_DIR.
2. If 8+ concept notes needed: use the Agent tool to dispatch parallel sub-agents for speed.

### Step 7: Backlinks & Session Index

**Backlink injection (up to 5 most significant):**
1. For the 5 concepts where this session added the most understanding:
   - Read the existing concept note
   - If no backlink to this session exists in its `## Related` section:
     - Use Edit to append: `- [[Session Note Title]] — referenced in YYYY-MM-DD session`
     - If no `## Related` section exists, add one before the bottom tags
   - If Edit fails, log it and continue.

**Session Index MOC (`[[ClaudeCode Session Index]].md` in NOTES_DIR):**
- If exists: Read it, then Edit to:
  - Add a new row to the TOP of the `## By Date` table
  - Add/update the project section under `## By Project`
  - Update `## Stats` counts
- If doesn't exist: Write it fresh using the MOC template from formatting.md, with this session as the first entry.

### Step 8: Report

Print a clean summary:

```
Session log written: [[filename]]
  Location: full/path
  Sections: 10/10 | Wikilinks: N | Tables: N | Diagrams: N

Concept notes created: N
  [[Concept 1]]
  [[Concept 2]]

Backlinks injected: N
  Added session reference to: [[Existing Concept 1]], [[Existing Concept 2]]

Session Index MOC: updated (or created)
  Total sessions tracked: N

Graph growth: +N new nodes, +N new edges this session
```

---

## Edge Cases

Handle all of these without breaking or asking for guidance:

**Very short sessions (<15 min, just Q&A):**
- Still write a session log, but it can be brief
- Minimum viable: Summary + What Was Discussed + Related + Tags
- Skip sections that don't apply (no Git Log if no commits, no Files Modified if nothing changed)
- Concept notes only if something genuinely new emerged
- Session index still gets updated

**Multi-project sessions:**
- 80/20 split → one log for primary project, mention secondary in Related
- 50/50 split → two separate session logs
- Many projects touched briefly → one "Multi-Project" session note

**No code changes (research/strategy/planning):**
- Still write the log — document discussions, decisions, plans
- Files Modified: "No files modified this session"
- Git Log: "No commits this session"
- These sessions often produce the richest concept notes

**Continuation sessions:**
- Reference prior session: "Continuation of [[ProjectName — Prior Topic — Date]]"
- Don't repeat prior context — focus on what's new
- Glob `*ProjectName*` in NOTES_DIR to find prior notes

**First session in a new project:**
- More context in Summary since no prior session to reference
- Create a project concept note if one doesn't exist in the vault

**Mid-session invocation (recommended for long sessions):**
- Append "(Part 1)" to the filename topic
- Document everything so far with full fidelity (this captures pre-compaction detail)
- If invoked again later in the same session:
  - Write a SEPARATE "Part 2" note — do NOT rewrite or supersede Part 1
  - Part 2 covers only what happened AFTER Part 1
  - Part 2's Related section links to Part 1: `- [[Part 1 Note Title]] — first half of this session`
  - Part 1 already has full-fidelity detail from before compaction; Part 2 captures the rest
  - Two detailed notes with full context > one note built from compacted memory
  - Both parts get their own Session Index MOC entries

**Non-git directories:**
- DCI commands already use `2>/dev/null || echo "fallback"`
- Document what was discussed/built without git history
- Use file modification times as a proxy

---

## NEVER Rules

1. **NEVER ask "Want me to write the note?"** — `/summary` IS the instruction. Just write.
2. **NEVER write stub concept notes** — Under 15 lines of real content = don't create it at all.
3. **NEVER skip concept notes** — Every non-trivial session adds at least one. They're what grows the graph.
4. **NEVER put notes in `.claude/projects/*/memory/` or the project directory** — Notes go in NOTES_DIR. Claude Code memory is separate from the Obsidian vault.
5. **NEVER use hyphens where em dashes belong** — Filenames use ` — ` not ` - `.
6. **NEVER forget `[[wikilinks]]`** — Every named entity gets double brackets. 20+ per session log, 5+ per concept note.
7. **NEVER write filler** — "Great session" = DELETE. "Built 4,982 lines across 15 commits" = KEEP.
8. **NEVER duplicate existing concept notes** — Always vault-wide search first. Link to existing, or Edit to expand.
9. **NEVER create trivial concept notes** — `[[For Loop]]` = no. `[[Circuit Breakers]]` = yes. The test: would 15+ lines of content with cross-project refs be useful in 6 months?
10. **NEVER omit Git Log if commits were made** — It's the forensic record.
11. **NEVER omit the Related section** — Every note links outward. Orphan nodes are useless.
12. **NEVER omit tags** — `#claudecode #session-log` always. Plus project and topic tags.
13. **NEVER write opinion as fact** — If uncertain, say so. Never fabricate specs or stats.
14. **NEVER overwrite existing concept notes** — Link to them, or Read-then-Edit to append. Never Write over them. The Write tool will error on an unread existing file; that error is a feature, not a bug — it forces you to see existing content before attempting change. When you hit it, Glob → Read → Edit instead.
15. **NEVER prescribe actions that conflict with the project's `CLAUDE.md` never-touch rules** — before writing Next Steps, check CLAUDE.md. Example: if CLAUDE.md says session prompts are gitignored, don't suggest `git add docs/sessionN-next-prompt.md` in Next Steps. Project-specific conventions override the skill's defaults.
