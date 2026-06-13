# Asset Structure

Assets 目录规范——知道从哪找，知道往哪放。

## 目录结构

```
Assets/
├── Animation/           模型/UI 动画
├── Audio/BGM/SFX/Voices/  音频
├── Documentation/        项目文档
├── Editor/              编辑器工具
├── Fonts/               字体
├── Model/               原始模型（FBX 等）
├── Prefabs/             预制体
├── Resources/           Resources 资源
├── Scenes/              场景
├── Scripts/             脚本（按模块分二级目录）
├── Settings/            项目设置（InputActions / PipelineAssets / ScriptableObjects）
├── Shaders/             着色器
├── Sprites/             精灵图
├── StreamingAssets/     运行时数据（JSON 配置 / AB 包等）
├── VFX/                 特效（Graphs / Textures）
└── Video/               视频
```

## 资产整理

- 所有资源按类型归入顶层目录，二级按功能细分
- 路径即规范：新来的资源直接放对应位置，不用想
- 合规检查：扫描 Assets，发现放错位置的文件并提示移动

## Addressables（AA 包）

- 资源通过 Addressables key 或 AssetReference 加载，不直接拖拽引用
- 资源移动、替换、分包不依赖代码
- 支持热更新和自动依赖管理

**官方文档：** [Addressables](https://docs.unity3d.com/Packages/com.unity.addressables@latest)

## AssetBundle（AB 包）

- 分组规则：按功能模块或更新频率分
- 打包的资源放在独立目录（如 `Prefabs/ModelsAB/`、`Sprites/AB/`），与普通资源分开管理
- 压缩方式：LZ4 或 LZMA 按场景选
