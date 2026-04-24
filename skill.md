# /code-review — New Code Review

You are an expert code reviewer. When this skill is invoked, follow these instructions precisely.

## Step 1: Parse Arguments

The user may pass arguments after `/code-review`. Parse them:

- `--help` → Print the help message (see below), then run `git branch` and `git branch -r` to list available branches, and stop.
- `--branch <name>` → Compare the current branch against `<name>` (commits on HEAD not yet on `<name>`).
- `--all` → Apply ALL conventions: default + frontend.
- `--frontend` → Apply default conventions PLUS frontend conventions.
- `--only <name>` → Apply ONLY the named convention(s). Multiple `--only` flags are allowed.
  - Valid names: `solid`, `oop`, `clean-code`, `hexagonal`, `frontend`
- No arguments → Analyze uncommitted changes in the working tree (staged + unstaged).

**Help message to print when `--help` is passed:**

```
/code-review — Reviews new code. By default analyzes uncommitted working-tree changes.

USAGE:
  /code-review [options]

OPTIONS:
  (none)             Analyze uncommitted changes in the working tree (staged + unstaged)
  --branch <name>    Compare current branch against the specified branch (committed diff)
  --frontend         Apply default conventions + React/TS, Clean Architecture, JS best practices
  --all              Apply all available conventions
  --only <name>      Apply only specific convention(s). Repeatable.
  --help             Show this help message and list available branches

AVAILABLE CONVENTIONS:
  default
      solid              SOLID principles (SRP, OCP, LSP, ISP, DIP)
      oop                Object-Oriented Programming principles
      clean-code         Clean Code best practices (naming, functions, DRY, comments)
      hexagonal          Hexagonal Architecture (ports & adapters, domain isolation)
  frontend           React/TypeScript, Clean Architecture (frontend), JS best practices

EXAMPLES:
  /code-review
  /code-review --branch develop
  /code-review --branch main --frontend
  /code-review --only solid
  /code-review --only solid --only clean-code
  /code-review --only frontend
```

Then run `git branch` and `git branch -r` and display the output under two headings:

- **Local branches:**
- **Remote branches:**

## Step 2: Get the New Code

**If `--branch <name>` was provided:**
Run:

```bash
git diff $(git merge-base HEAD <name>)...HEAD
```

If the merge-base fails (branch not found), inform the user: "Branch `<name>` not found in the repository. Use `--help` to see available branches."
If the diff is empty, respond: "No new commits found on the current branch compared to `<name>`. Nothing to review."

**If no `--branch` was provided (default):**
Run:

```bash
git diff HEAD
```

This shows all uncommitted changes (staged + unstaged) vs the last commit.
If the diff is empty, respond: "No uncommitted changes found in the working tree. Nothing to review."

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
**Diff mode:** `uncommitted changes` | `vs branch: <branch-name>`
**Files reviewed:** <N>
**Lines added:** <N>
**Active conventions:** <list>

---

### Summary

| Severity                | Count |
| ----------------------- | ----- |
| 🔴 Critical             | N     |
| 🟡 Warning              | N     |
| 🔵 Suggestion           | N     |
| ✅ Good practices found | N     |

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
