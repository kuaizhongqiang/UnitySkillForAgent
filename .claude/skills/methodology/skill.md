# Methodology

Unity 项目开发方法论——流程规范与原则。

## 1. 文档先行

写代码前先写文档。新功能先在 CLAUDE.md 或设计文档中明确：场景结构、Prefab 层级、数据流、关键接口。文档通过再动手。

## 2. 版本控制

### 命令对照

| 操作 | Git | Plastic SCM | SVN |
|------|-----|-------------|-----|
| 检出/克隆 | `git clone <url>` | `cm clone <repo@server>` | `svn checkout <url>` |
| 添加文件 | `git add .` | `cm add -R .` | `svn add * --force` |
| 提交 | `git commit -m ""` | `cm ci -c ""` | `svn commit -m ""` |
| 提交全部 | `git commit -a -m ""` | `cm ci --all --private -c ""` | `svn commit -m ""` |
| 查看状态 | `git status` | `cm status` | `svn status` |
| 查看历史 | `git log` | `cm log` | `svn log --limit N` |
| 更新 | `git pull` | `cm update` | `svn update` |
| 创建分支 | `git branch <name>` | `cm branch <name>` | `svn copy ^/trunk ^/branches/<name>` |
| 切换分支 | `git checkout <branch>` | `cm switch <branch>` | `svn switch ^/branches/<name>` |
| 合并分支 | `git merge <branch>` | `cm merge <branch> --merge` | `svn merge ^/branches/<name>` |
| 撤销本地 | `git reset --hard` | `cm undo -r .` | `svn revert -R .` |
| 暂存 | `git stash` | shelve（Editor 内） | 无直接等价 |
| 加锁文件 | 无原生支持 | `cm lock <file>` | `svn lock <file>` |
| 忽略规则 | `.gitignore` | `.plasticignore` | `svn propset svn:ignore` |

### Unity 规则

- `.meta` 文件必须提交（Unity 依赖 GUID 识别资源）
- `Library/` `Temp/` `UserSettings/` 不提交
- `ProjectSettings/` `Packages/` 完整提交
- 美术资源（模型、贴图、音频等）Agent 直接跳过，不查看、不处理
- Agent 聚焦：`.cs` 脚本、Prefab 结构、场景层级、`ProjectSettings/` 配置、`StreamingAssets/` JSON

## 3. 权责分开

- 一个 MonoBehaviour 一个职责，不做万能脚本
- 数据归数据（ScriptableObject / StreamingAssets JSON）
- 逻辑归逻辑（Manager / Controller）
- 展示归展示（UI / VFX）

## 4. 先膨胀后收敛

快速实现 → 验证流程正确 → 重构收紧 → 固化为规范/工具。避免陷入局部最优。

## 5. 增量验证

改一点 → 验证 → 再改一点。利用 Unity edit → compile → 看效果的短循环，不积攒问题。

## 6. 数据配置

可配置数据放在 `StreamingAssets/` JSON 中，MonoBehaviour 只负责读取和执行。代码不硬编码数值。

**官方文档：** [StreamingAssets](https://docs.unity3d.com/ScriptReference/Application-streamingAssetsPath.html)

## 7. MCP 可回溯

通过 MCP 操作编辑器的关键步骤记录日志（操作人、操作内容、时间、状态），关键变更关联到 commit。
