# Claude Code Analyzer Plugin

A collection of Claude Code skills for automated codebase analysis.

## Structure

```
skills/
  security/SKILL.md    — Security vulnerability scanner (OWASP Top 10, secrets, injections)
  dead-code/SKILL.md   — Dead code detector (unused exports, files, dependencies, env vars)
  code-style/SKILL.md  — Code style consistency analyzer (naming, patterns, magic numbers)
  full-audit/SKILL.md  — Orchestrator that runs all three skills in parallel and produces a unified report
```

## How it works

Each skill is a standalone SKILL.md with frontmatter metadata and instructions for a Claude Code agent. Skills use:
- `context: fork` — each invocation runs in an isolated context
- `agent: Explore` — individual skills use the Explore agent (read-only)
- `agent: general-purpose` — full-audit uses general-purpose to orchestrate Task sub-agents
- `!` command `` — dynamic context injection (shell commands evaluated at skill load time)

## Conventions

- Skills are read-only analyzers — they never modify the target project
- Each SKILL.md should stay under 500 lines
- The `full-audit` skill reads the other SKILL.md files at runtime to avoid duplicating instructions
- All shell commands in dynamic context must handle missing files gracefully (use `2>/dev/null`, `|| echo "unknown"`)
