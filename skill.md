# /code-review — New Code Review

You are an expert code reviewer. When this skill is invoked, follow these instructions precisely.

## Step 1: Parse Arguments

The user may pass arguments after `/code-review`. Parse them:

- `--help` → Print the help message (see below) and stop.
- `--all` → Apply ALL conventions: default + frontend.
- `--frontend` → Apply default conventions PLUS frontend conventions.
- `--only <name>` → Apply ONLY the named convention(s). Multiple `--only` flags are allowed.
  - Valid names: `solid`, `oop`, `clean-code`, `hexagonal`, `frontend`
- No arguments → Apply default conventions (solid + oop + clean-code + hexagonal).

**Help message to print when `--help` is passed:**

```
/code-review — Analyzes only the new code in the current branch.

USAGE:
  /code-review [options]

OPTIONS:
  (none)             Apply default conventions: SOLID, OOP, Clean Code, Hexagonal Architecture
  --frontend         Apply default conventions + React/TS, Clean Architecture, JS best practices
  --all              Apply all available conventions
  --only <name>      Apply only specific convention(s). Repeatable.
  --help             Show this help message

AVAILABLE CONVENTIONS:
  default
      solid              SOLID principles (SRP, OCP, LSP, ISP, DIP)
      oop                Object-Oriented Programming principles
      clean-code         Clean Code best practices (naming, functions, DRY, comments)
      hexagonal          Hexagonal Architecture (ports & adapters, domain isolation)
  frontend           React/TypeScript, Clean Architecture (frontend), JS best practices

EXAMPLES:
  /code-review
  /code-review --frontend
  /code-review --all
  /code-review --only frontend
  /code-review --only frontend --only default
  /code-review --only frontend
```

## Step 2: Get the New Code

Run this bash command to get the diff of only new/changed code in the current branch vs `origin/main`:

```bash
git diff $(git merge-base HEAD origin/main)...HEAD
```

If `origin/main` is not available, fall back to:
```bash
git diff $(git merge-base HEAD main)...HEAD
```

If the diff is empty, respond: "No new code found in this branch compared to main. Nothing to review."

## Step 3: Determine Active Conventions

Based on parsed arguments, load the corresponding convention files from the `conventions/` subdirectory next to this `skill.md`. Read only the convention files that are active.

Convention file mapping:
- `solid`, `oop`, `clean-code`, `hexagonal` → `conventions/default.md`
- `frontend` → `conventions/frontend.md`

Default (no args): read `conventions/default.md`
`--frontend`: read `conventions/default.md` + `conventions/frontend.md`
`--all`: read `conventions/default.md` + `conventions/frontend.md`
`--only <name>`: read the file(s) mapped above, and note which specific sections apply

## Step 4: Analyze the Diff

- Focus ONLY on added lines (lines starting with `+`, excluding the `+++` file header lines).
- Ignore deleted lines (lines starting with `-`).
- For each file in the diff, note the filename and language (Java or JavaScript/TypeScript/React).
- Apply the active convention rules to the added code.
- Group findings by file.

## Step 5: Produce the Report

Output the review using this exact structure:

---

## Code Review Report

**Branch:** `<current-branch-name>`
**Files reviewed:** <N>
**Lines added:** <N>
**Active conventions:** <list>

---

### Summary
| Severity | Count |
|----------|-------|
| 🔴 Critical | N |
| 🟡 Warning | N |
| 🔵 Suggestion | N |
| ✅ Good practices found | N |

---

### 🔴 Critical Issues
> Convention violations that could cause bugs, break architecture boundaries, or introduce serious maintainability problems.

For each issue:
**[FILENAME:LINE_APPROX]** — `<short rule name>`
> <explanation of what's wrong and why it matters>
```
<relevant code snippet>
```
💡 **Fix:** <concrete suggestion>

---

### 🟡 Warnings
> Bad practices that reduce readability, testability, or maintainability.

(same format as Critical)

---

### 🔵 Suggestions
> Optional improvements — good to address but not blocking.

(same format as Critical)

---

### ✅ Good Practices Found
> Acknowledge what was done well in the new code.

- `<FILENAME>` — <what was done correctly and why it's good>

---

### Convention Coverage
List which conventions were active and how many issues were found per convention.

---

If there are no issues in a severity category, write: _None found._

Be constructive, specific, and actionable. Reference the convention that was violated (e.g., "Violates SRP — this class handles both X and Y"). Always include a concrete fix suggestion.
