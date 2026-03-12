[English](README.md) | [中文](README.zh-CN.md)

# Skills

A collection of agent skills for automated coding workflows.

## Available Skills

| Skill | Description | Install |
|-------|-------------|---------|
| [code-review](code-review/) | Git diff analysis, SOLID principles, code quality, security scan, dead code detection, best practices. P0-P3 priority with verdict (APPROVE/REQUEST_CHANGES/COMMENT) and interactive fix. Frontend + Backend. | `npx skills add lenslp/skills --path code-review` |

## Structure

Each skill follows the [Anthropic skill-creator](https://github.com/anthropics/skills) convention:

```
skill-name/
├── SKILL.md              # Main instructions (required)
├── agents/
│   └── agent.yaml       # UI metadata
└── references/           # Detailed reference docs loaded on demand
```
