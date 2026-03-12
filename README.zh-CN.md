[English](README.md) | [中文](README.zh-CN.md)

# Skills

一组用于自动化编码工作流的 Agent Skills。

## 可用 Skills

| Skill | 说明 | 安装 |
|-------|------|------|
| [code-review](code-review/) | Git diff 分析、SOLID 原则、代码质量、安全扫描、死代码检测、最佳实践。P0-P3 优先级 + 合并建议（APPROVE/REQUEST_CHANGES/COMMENT）+ 交互式修复。支持前后端。 | `npx skills add lenslp/skills --path code-review` |

## 目录结构

每个 skill 遵循 [Anthropic skill-creator](https://github.com/anthropics/skills) 规范：

```
skill-name/
├── SKILL.md              # 主指令文件（必需）
├── agents/
│   └── agent.yaml        # UI 元数据
└── references/           # 按需加载的详细参考文档
```
