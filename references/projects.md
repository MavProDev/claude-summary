# Known Projects

This file is user-customizable. It provides project detection patterns and canonical names so the `/summary` skill can recognize projects in its CLAUDE.md → package.json → git-remote → directory detection chain.

## How to customize

Add your own projects to the table below. The skill will use the canonical name in session log filenames and wikilinks.

## Project Table (template — add your own rows)

| Project | Detection Patterns | Canonical Name | Description |
|---------|-------------------|----------------|-------------|
| FORTRESS | Path contains `FORTRESS` | FORTRESS | Open-source adversarial security audit protocol for Claude Code. |
| ExampleProject | Path contains `ExampleProject` | ExampleProject | Short one-line description of the project and its stack. |

## Detection Priority

1. **CLAUDE.md content** — most reliable, contains explicit project name/description
2. **package.json "name" field** — reliable for Node.js projects
3. **git remote -v** — reliable for GitHub-hosted projects, parse repo name from URL
4. **Directory basename** — fallback, may not match canonical name
5. **No match** — use detected name as-is, capitalize first letter of each word

## Unknown Project Handling

This skill is universal. If the project is not in the table:
- Use the best available name from the detection priority
- Write full session logs and concept notes as normal
- Tag with the project name in kebab-case (e.g., `#new-project-name`)
- The project can be added to this table later for future recognition
