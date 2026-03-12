# Skills

A collection of agent skills for automated coding workflows.

## Available Skills

| Skill | Description |
|-------|-------------|
| [code-review](code-review/) | Comprehensive code review on git changes: diff analysis, batch review for large changesets (500+ lines), code quality scan, security vulnerability scan, best practices compliance, and interactive fix with P0-P3 priority system. Supports both frontend and backend codebases. |

## Structure

Each skill follows the [Anthropic skill-creator](https://github.com/anthropics/skills) convention:

```
skill-name/
├── SKILL.md              # Main instructions (required)
├── agents/
│   └── openai.yaml       # UI metadata
└── references/           # Detailed reference docs loaded on demand
```

## Usage

Reference a skill by name when prompting:

```
Use $code-review to review my recent changes.
```
