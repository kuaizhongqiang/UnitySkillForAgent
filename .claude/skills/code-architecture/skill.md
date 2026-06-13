# Code Architecture

**Service + MVC** 混合架构。数据单向流动：Model → Service → Controller → View。

## 架构总览

```
Service 层（全局服务，DontDestroyOnLoad）
Singleton<T> → GlobalDataMgr / SceneMgr / AudioMgr / AssetMgr / InteractiveMgr

Controller 层（调度 + 数据转换，挂场景空物体）
ControllerBase<TView> → StepControllerBase → 具体模块 Controller

View 层（纯展示，不含业务逻辑）
UIBase → CanvasBase / PanelBase / ElementBase
       → 对象池 Get/Return 管理生命周期

Model 层（纯数据）
[Serializable] POCO → StreamingAssets JSON
TaskDataBase → TaskData_Purpose / Equipment / Training

Dynamic 行为层
DynamicBase<T> : MonoBehaviour, IDynamicBase → 具体任务子类

交互层
IObj → InteractiveObjBase（GlobalInteractiveMgr 统一射线检测）
```

## 命名规范

| 类型 | 规范 | 示例 |
|------|------|------|
| 接口 | `I` 前缀 | `IObj`, `IDynamicBase` |
| 抽象基类 | `Base` 后缀 | `CanvasBase`, `PanelBase` |
| 管理器 | `XxxMgr` | `GlobalDataMgr` |
| 私有字段 | `_camelCase` | `_instance`, `_canvasGroup` |
| 序列化字段 | `[SerializeField] camelCase` | |
| 事件 | `On` 前缀 | `OnStepChanged` |

## 泛型继承

| 模式 | 用途 | 示例 |
|------|------|------|
| 约束自身 | 单例基类 | `Singleton<T> where T : Singleton<T>` |
| 约束数据类型 | 行为基类 | `DynamicBase<T> : IDynamicBase` |
| 接口 + 抽象基类 | 交互对象 | `IObj` → `InteractiveObjBase` |
| 纯虚基类 | UI 层级 | `UIBase → CanvasBase/PanelBase/ElementBase` |

原则：泛型只用在需要类型安全的场景，搭配非泛型接口做类型擦除，继承不超过 3 层。

## 对象池

Panel 和 Dynamic 频繁创建销毁时，用池管理：

```csharp
public class PanelPool
{
    Dictionary<Type, Stack<UIBase>> _pool;
    T Get<T>() where T : UIBase => // 池有就 Pop，没有才 Instantiate
    void Return<T>(T obj) where T : UIBase => // SetActive(false) 入栈，不 Destroy
}
```

Panel 用 CanvasGroup 控制显隐，**外层生命周期**由对象池管。Dynamic 同理，任务切换时 Return 而非 Destroy。

## 交互系统

- 所有 3D 物体统一基类 `InteractiveObjBase : MonoBehaviour, IObj`
- `GlobalInteractiveMgr.Update()` 统一射线检测，不依赖各物体分别检测
- `IObj` 定义 `OnMouseEnter/Exit/Click/Down/Up`，基类默认实现

## 模块间通信

| 方式 | 谁到谁 | 说明 |
|------|--------|------|
| `Service.Instance` | Controller → Service | Controller 读数据 |
| 方法 / UnityEvent | Controller → View | 加工后的数据给 Panel |
| 回调 / UnityEvent | View → Controller | 用户操作通知 |
| `Register/Unregister` | 对象 → Service | 交互对象注册 |

**禁止：** View 直接调 `Service.Instance`。

## 异步加载链路

```
Setup → GlobalManager.DelayInit() → JSON → AB/AA
      → CanvasBase.GetPanel<T>() → Instantiate
      → Controller 取 Service 数据 → 加工 → Panel.SetDisplayData()
      → DynamicBase 创建场景对象
```

## 目录结构

```
Scripts/
├── Controller/         MVC Controller
├── Data/               Model POCO
├── UI/                 View（Panel / ComponentElement / Canvas）
├── GlobalManagement/   Service
├── Dynamic/            行为层
├── Interactive/        交互层
└── Singleton/          单例基类
```

## 核心模板

### ControllerBase

```csharp
public abstract class ControllerBase<TView> : MonoBehaviour where TView : UIBase
{
    protected TView View { get; private set; }
    IEnumerator Start() { yield return BindView(); OnViewBound(); }

    protected virtual IEnumerator BindView()
    {
        while (View == null) { View = CanvasBase.Instance.GetPanel<TView>(); yield return null; }
    }
    protected abstract void OnViewBound();
    protected virtual void OnDestroy() { }
}
```

### StepControllerBase（派生）

```csharp
public abstract class StepControllerBase : ControllerBase<StepPanel>
{
    protected int _currentStep;
    protected void MoveToNext()
    {
        _currentStep++;
        if (_currentStep >= TotalSteps) OnAllComplete();
        else ExecuteStep(_currentStep);
    }
    protected abstract int TotalSteps { get; }
    protected abstract void ExecuteStep(int step);
    protected abstract void OnAllComplete();
}
```

### Singleton Manager

```csharp
public class GlobalXxxMgr : Singleton<GlobalXxxMgr>
{
    public XxxData Data { get; private set; }
    IEnumerator DelayInit() { Data = LoadJson("xxx.json"); IsInit = true; }
}
```

### UI Panel（View）

```csharp
public class XxxPanel : PanelBase
{
    public event System.Action OnAction;
    public void SetDisplayData(DisplayData data) { /* 只展示 */ }
    void Awake() => _btn.onClick.AddListener(() => OnAction?.Invoke());
}
```

## 完整样例：登录 MVC

```
LoginPanel → OnLoginSubmit → LoginController
    ↑                           ↓
    └── ShowResult() ←──  GlobalDataMgr.FindUser()
```

```csharp
// Model
[Serializable] public class UserData { public string username; public string password; }

// Service
public class GlobalDataMgr : Singleton<GlobalDataMgr>
{
    public UserData FindUser(string name) => /* 查 JSON */;
}

// Controller
public class LoginController : ControllerBase<LoginPanel>
{
    void OnViewBound() => View.OnLoginSubmit += HandleLogin;
    void HandleLogin(string user, string pwd)
    {
        var u = GlobalDataMgr.Instance.FindUser(user);
        View.ShowResult(u != null && u.password == pwd, u != null ? "欢迎" : "失败");
    }
}

// View
public class LoginPanel : PanelBase
{
    public event System.Action<string, string> OnLoginSubmit;
    public void ShowResult(bool ok, string msg) { /* 只展示 */ }
}
```

Controller 挂场景空物体上，Panel 动态创建，Controller 通过 `CanvasBase.GetPanel<T>()` 获取。
