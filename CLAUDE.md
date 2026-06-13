# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Goal

开发一套自用的 Claude Code Agent Skill，通过 Unity MCP 桥接让 AI Agent 能直接操控 Unity Editor，实现快速准确地构建 Unity 项目。

## Tech Stack

- **Unity 2022.3.53f1** + **URP 14.0.11**
- **MCP Bridge**: `com.coplaydev.unity-mcp` — 允许 Agent 通过 TCP 与 Unity Editor 实时交互
- **Python + UV** — MCP 相关依赖管理
- **IDE 支持**: Rider, VS, VS Code

## Prerequisites

- Unity Editor 必须保持运行，MCP 桥接才能工作
- MCP 服务器地址：`http://127.0.0.1:8080/mcp`（已注册到 Claude Code）
- 梯子代理 `http://127.0.0.1:10808`（用于安装依赖）

## Git Workflow

- **Ignored**: `Library/`, `Temp/`, `UserSettings/`, `Logs/`, `obj/`, `Build/`
- **Tracked**: `Assets/`, `Packages/manifest.json`, `Packages/packages-lock.json`, `ProjectSettings/`
- 初始提交仅包含 `.gitignore`, `LICENSE`, `README.md`

## Unity Editor Commands

```bash
# Open the project in Unity Editor
"$UNITY_EDITOR_PATH" -projectPath "g:\project\UnitySkillForAgent"
```

## MCP 能力

UnityMCP 让 Agent 可以：

- 查询场景对象和组件
- 创建/修改 GameObject、资源、预制体
- 执行 Editor 命令
- 运行 Play Mode 测试
- 访问项目层级和 Inspector 数据
