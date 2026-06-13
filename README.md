# UnitySkillForAgent

自用 Claude Code Agent Skill 仓库，通过 Unity MCP 桥接让 AI Agent 直接操控 Unity Editor，快速准确地构建 Unity 项目。

## 快速开始

```bash
# 提交项目文件
git add Assets/ ProjectSettings/ Packages/ .gitignore
git commit -m "初始化 Unity 项目结构"
```

## Skills

见 [.claude/skills/README.md](.claude/skills/README.md) — 4 个核心 Skill 覆盖方法论、资产结构、代码结构、工作流。

## 技术栈

- Unity 2022.3.53f1 + URP 14.0.11
- com.coplaydev.unity-mcp（MCP 桥接）
- Claude Code + 4 个自定义 Skill
