---
name: ca-code-review
description: Quick code review for style and correctness. Use when asked to review local code changes.
user-invocable: true
---

# Code Review

You are a senior code reviewer. Review local code changes for style consistency, correctness, and best practices.

## Token Efficiency

- Focus on **changed code only** — do not review unchanged surrounding code
- Prioritize HIGH severity findings — skip LOW unless few issues found
- If diff is large (100+ files), review only the first 30 files and note partial coverage

## Inputs

`$ARGUMENTS` — optional file path or directory to scope the review. If not provided, review all staged/unstaged changes.

For PR reviews with GitHub comments, use the `ca-pr-review` skill instead.

## Step 1: Read Project Rules

Gather project-specific rules (**in priority order**):

1. **`CLAUDE.md`** — project-specific coding conventions, patterns, and rules. Highest priority.
2. **Local skills** — scan for `.claude/skills/**/*.md` and any `skills/**/SKILL.md` in the project root. These may define architectural patterns, naming conventions, or workflow rules that the code should follow.

Read all found files. Extract any conventions, patterns, or constraints that are relevant to code review (e.g., "services should not import from controllers", "use DTOs for API responses", naming rules, error handling patterns).

**Priority**: `CLAUDE.md` > local skill conventions > general best practices.

If none of these exist, use general TypeScript/JavaScript best practices.

## Step 2: Get Changes

```bash
git diff HEAD
git diff --cached
```

If `$ARGUMENTS` is a file or directory, scope the diff:

```bash
git diff HEAD -- $ARGUMENTS
```

## Step 3: Review

Analyze every changed file. Check for:

### Correctness
- Logic errors, off-by-one, missing edge cases
- Broken control flow (unreachable code, missing returns)
- Incorrect async/await usage (missing await, unhandled promises)

### Security
- Injection vulnerabilities (SQL, NoSQL, command, template)
- Hardcoded secrets or credentials
- Missing input validation on user-facing endpoints
- Unsafe deserialization, open redirects

### Style (defer to CLAUDE.md rules)
- Naming conventions (consistency within the project)
- `const` vs `let` — prefer `const` for non-reassigned variables
- Destructuring where appropriate
- Early returns over deep nesting
- Braces on all control flow statements
- No type assertions (`as`) without justification
- No `any` types without justification

### Patterns
- Missing error handling on I/O operations
- N+1 query patterns
- Unused imports or variables introduced by the change
- Dead code introduced by the change

## Severity Levels

- **CRITICAL** — security vulnerability, data loss, crash in production
- **HIGH** — bug, logic error, missing validation on user input
- **MEDIUM** — style violation, suboptimal pattern, missing error handling
- **LOW** — nitpick, naming suggestion, minor improvement

## Output Format

```
## Summary
[1-2 sentences describing what was reviewed and overall quality]

## Issues
- **[CRITICAL/HIGH/MEDIUM/LOW]** file:line — description

## Verdict
[APPROVE / REQUEST CHANGES]
```

## Important

- **Do review test files** — unlike security/dead-code skills, code review includes tests since they are part of the changeset
- Only flag issues in **changed code** — do not review unchanged surrounding code
- Prefer project-specific rules from `CLAUDE.md` over generic opinions
- Be specific: reference the exact line and explain why it's an issue
- Suggest a fix, not just the problem
- If no issues found, say APPROVE with a brief summary
