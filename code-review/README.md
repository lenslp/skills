[English](README.md) | [中文](README.zh-CN.md)

# Code Review

Comprehensive code review skill for AI agents. Analyzes git diff, checks SOLID principles, scans code quality and security, detects dead code, and verifies best practices. Issues classified by P0-P3 priority with an overall verdict (APPROVE / REQUEST_CHANGES / COMMENT). Review-only by default — no code changes until you confirm. Supports both frontend and backend.

## Installation

```bash
npx skills add lenslp/skills --path code-review
```

## What It Checks

| Category | Checks |
|----------|--------|
| **SOLID & Architecture** | SRP, OCP, LSP, ISP, DIP violations, code smells, refactor suggestions |
| **Code Quality** | Correctness, boundary conditions, readability, type safety, API design |
| **Security** | Injection, auth/authz, data exposure, input validation, dependency CVEs, frontend security |
| **Dead Code** | Unused/redundant code, removal candidates with safe-delete vs defer plan |
| **Best Practices** | Architecture, error handling, testing, performance |
| **Frontend** | Component design, rendering performance, state management, a11y, bundle size, UX robustness |

## Priority Levels

| Priority | Definition | Action |
|----------|-----------|--------|
| **P0** | Security vulnerabilities, data loss, crash | Must fix immediately |
| **P1** | Logic errors, missing error handling, broken contracts | Fix before merge |
| **P2** | Code quality, readability, test coverage gaps | Should fix |
| **P3** | Style nits, naming, minor optimizations | Nice to have |

## Workflow

```
git diff → Scope → SOLID → Quality → Security → Dead code → Best practices → Frontend → Report → Verdict → Confirm
```

1. **Analyze Diff** — Run `git diff --stat`, assess scope, build summary table
2. **Batch Strategy** — Over 500 lines? Split by module, review each batch
3. **SOLID & Architecture** — Check SRP/OCP/LSP/ISP/DIP, flag code smells
4. **Code Quality** — Correctness, boundary conditions, readability, type safety
5. **Security Scan** — Injection, auth, data exposure, frontend security, language-specific
6. **Dead Code** — Identify removal candidates (safe-delete vs defer-with-plan)
7. **Best Practices** — Architecture, error handling, testing, performance
8. **Frontend Scan** — Component design, state, a11y, bundle size, UX (auto-detected)
9. **Report** — All issues in P0-P3 tables with verdict
10. **Interactive Fix** — User chooses what to fix (review-only until confirmed)

## Verdict

| Verdict | Condition |
|---------|-----------|
| ✅ **APPROVE** | No P0 or P1 issues |
| ⚠️ **REQUEST_CHANGES** | Any P0 or P1 issues exist |
| 💬 **COMMENT** | No issues, or informational observations only |

## Usage in Cursor

The skill is automatically registered after installation. Use it in Cursor Chat (Agent mode).

### Auto Trigger

The skill activates when your prompt contains keywords like:

- "review my code"
- "check this PR"
- "scan for security issues"
- "check code quality"

### Manual Trigger

Example prompts:

```
Review my recent code changes.
```

```
Review the changes on this branch against main.
```

```
Check my staged code for security issues.
```

### After Review

The report lists all issues sorted by P0-P3 priority with a verdict. Choose how to fix:

- By priority: `fix P0`, `fix P0 and P1`
- By module: `fix src/auth/`
- By issue number: `fix #1, #3, #5`
- Fix everything: `fix all`
- Skip: `skip`

## Files

```
code-review/
├── SKILL.md                          # Main workflow
├── agents/agent.yaml                 # UI metadata
└── references/
    ├── solid-checklist.md            # SOLID principles, code smells, refactor heuristics
    ├── security-checklist.md         # Security checks by category & language
    ├── best-practices.md             # Backend + frontend best practices
    ├── frontend-checklist.md         # Frontend-specific checks (50+ items)
    └── removal-plan.md              # Dead code removal plan template
```
