# Workflow

Unity 开发工作流——Prefab 驱动、编辑器工具、CI 构建。

## 1. Prefab 驱动开发

先在场景里搭好层级，再写脚本。

1. 在场景中搭建 GameObject 层级结构
2. 挂载需要的组件
3. 导出为 Prefab 到 `Prefabs/` 目录
4. 编写 MonoBehaviour 脚本挂到对应节点
5. 通过 Prefab 变体迭代

要点：场景中的结构即架构图，每个 Prefab 职责单一，嵌套 Prefab 代替深度继承。

## 2. 编辑器工具先行

凡重复三次以上的操作，就应该写一个 Editor 工具。

适用场景：批量资源处理（重命名/移动/格式转换）、自定义 Inspector、构建/打包自动化、项目初始化。

工具放在 `Assets/Editor/` 下，按功能分目录。

## 3. CI / Build

### 方案一：本机手动 Build

在 `Assets/Editor/` 下创建构建脚本：

```csharp
using UnityEditor;
using UnityEditor.Build.Reporting;

public static class BuildScript
{
    public static void BuildWindows()
    {
        BuildPlayerOptions opt = new()
        {
            scenes = EditorBuildSettings.scenes,
            locationPathName = "Builds/Windows/Game.exe",
            target = BuildTarget.StandaloneWindows64,
            options = BuildOptions.None
        };
        BuildPipeline.BuildPlayer(opt);
    }
}
```

命令行调用：

```bash
"$UNITY_PATH" -quit -batchmode -projectPath "g:\project\UnitySkillForAgent" -executeMethod BuildScript.BuildWindows -logFile build.log
```

**官方文档：**
- [BuildPlayerOptions](https://docs-alpha.unity3d.com/2022.3/Documentation/ScriptReference/BuildPlayerOptions.html)
- [BuildPipeline.BuildPlayer](https://docs.unity.cn/2022.1/Documentation/ScriptReference/BuildPipeline.BuildPlayer.html)

### 方案二：团结云开发

Unity 中国云 CI/CD 平台（[devops.unity.cn](https://devops.unity.cn)），项目根目录 `.workflows/` YAML 定义流水线。

```yaml
# .workflows/build-android.yaml
name: Devops Build
on: [push]

jobs:
  build:
    name: Build Android
    runs-on: windows-server-2022-unity-2021.3.41f1c1-8c-16g
    steps:
      - uses: actions/checkout-plasticscm@v1
        with:
          path: tjcloudbuild
      - uses: actions/tj-builder@v3
        id: build-action
        with:
          targetPlatform: Android          # 可选: StandaloneWindows64 / Android / WebGL / WeixinMiniGame / OpenHarmony
          projectPath: ./tjcloudbuild
      - uses: actions/tj-upload-artifact@v2
        with:
          name: Build
          path: ${{ steps.build-action.outputs.buildsPath }}
```

可用 Actions：`checkout-plasticscm@v1`（签出代码）、`tj-builder@v3`（构建）、`tj-upload-artifact@v2`（上传产物）、`tj-cache`（缓存加速）。

### 注意事项

- CI 服务器需安装对应平台的 Build Support 模块
- 许可证激活是 CI 构建最常见的失败原因
- 构建产物路径用 `-logFile` 指定，方便 Agent 排查错误
