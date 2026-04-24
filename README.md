# /code-review — New Code Review Skill

A Claude Code skill that analyzes **only the new code** introduced in the current branch (vs `main`) and evaluates it against configurable coding conventions.

## Features

- Analyzes only added lines from `git diff` — ignores unchanged and deleted code
- Configurable convention profiles (SOLID, OOP, Clean Code, Hexagonal Architecture, React/TS)
- Categorized output: Critical, Warning, Suggestion, and Good Practices
- Supports Java, JavaScript, and TypeScript/React codebases

## Installation

Copy this folder into your Claude Code skills directory:

```bash
cp -r code-review ~/.claude/skills/
```

Or clone the repo and symlink:

```bash
git clone https://github.com/<your-username>/code-review-skill.git
ln -s $(pwd)/code-review-skill ~/.claude/skills/code-review
```

## Usage

```bash
/code-review                        # Default conventions (SOLID + OOP + Clean Code + Hexagonal)
/code-review --frontend             # Default + React/TS, Clean Architecture, JS best practices
/code-review --all                  # All conventions
/code-review --only solid           # Only SOLID principles
/code-review --only frontend        # Only frontend conventions
/code-review --only solid --only clean-code   # Multiple specific conventions
/code-review --help                 # Show help
```

## Available Conventions

| Name | Description |
|------|-------------|
| `solid` | SOLID principles (SRP, OCP, LSP, ISP, DIP) |
| `oop` | OOP: encapsulation, abstraction, composition over inheritance, polymorphism |
| `clean-code` | Naming, small functions, DRY, no magic numbers, comments, dead code |
| `hexagonal` | Hexagonal Architecture: ports & adapters, domain isolation from infrastructure |
| `frontend` | React hooks rules, TypeScript, Clean Architecture (frontend), JS best practices |

**Default** (no args): `solid` + `oop` + `clean-code` + `hexagonal`

## Report Format

The review produces a structured report with:

- **Summary table** — issue counts by severity
- **Critical** — architecture violations, serious anti-patterns
- **Warning** — bad practices reducing readability or testability
- **Suggestion** — optional improvements
- **Good Practices Found** — positive feedback on what was done well
- **Convention Coverage** — which conventions were active and how many issues per convention

## Customizing Conventions

Edit the files in the `conventions/` directory:

- `conventions/default.md` — SOLID, OOP, Clean Code, Hexagonal Architecture rules
- `conventions/frontend.md` — React/TS, Clean Architecture (frontend), JS rules

You can add new convention files and reference them in `skill.md`.

## Project Structure

```
code-review/
├── skill.md                  # Main skill logic (instructions for Claude)
├── conventions/
│   ├── default.md            # SOLID + OOP + Clean Code + Hexagonal Architecture
│   └── frontend.md           # React/TS + Clean Architecture + JS best practices
├── README.md                 # This file
└── .gitignore
```

## Requirements

- Claude Code CLI
- Git repository with a `main` or `origin/main` branch

## Contributing

1. Fork this repository
2. Add or improve convention rules in `conventions/`
3. Update `skill.md` if adding new convention flags
4. Submit a pull request
