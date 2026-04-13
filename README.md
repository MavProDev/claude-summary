# Summary

Call /summary at any time during a session or at the end, and it snapshots everything into a wikilinked Obsidian node — diagrams, concept notes, git forensics. Your knowledge graph grows while you work.

A Claude Code skill that turns every coding session into a structured Obsidian knowledge node.

## Install

```bash
npx claudesuite install summary
```

Or manually:

```bash
curl -sL https://raw.githubusercontent.com/ExampleOrgDev/claude-summary/main/SKILL.md \
  --create-dirs -o ~/.claude/skills/summary/SKILL.md
```

> Note: This skill also includes reference files for formatting and concept rules. The `npx` installer handles these automatically.

## Usage

In Claude Code:

```
/summary
/summary "focus on the auth refactor"
```

Invoke at the end of a session, or mid-session for long sessions (Part 1, Part 2). The skill detects everything automatically — no flags required.

## What It Captures

Every session log gets 10 sections:

1. **Summary** — mission, accomplishment, headline result
2. **What Was Done** — detailed breakdown with tables
3. **Key Decisions & Reasoning** — every choice + why
4. **Architecture** — mermaid diagrams, code blocks
5. **Files Modified** — table of every file touched
6. **Next Steps** — prioritized actionable bullets
7. **Git Log** — forensic record
8. **Related** — wikilinks to concepts and projects
9. **Tags** — for vault navigation

## Concept Notes

When the session introduces a new pattern, technology, or insight, the skill creates standalone concept notes with cross-project references. These build a knowledge graph that gets denser with every session.

## Backlink Injection

The skill automatically updates existing concept notes with backlinks to the new session, keeping the graph navigable in both directions.

## Session Index MOC

A maintained Map-of-Content file at the vault root tracks every session by date and project. Statistics, navigation, and discovery all in one place.

## Part of ClaudeSuite

This skill is part of [ClaudeSuite](https://claudesuite.xyz) — a curated collection of open-source Claude Code skills.

## License

MIT
