# Unreal Engine Rules (Global)

仅当仓库与 Unreal Engine/UE 项目相关时适用。

## Safety boundaries
- 未经用户明确要求，不修改 Engine 源码（Engine/）或第三方依赖源码。
- 修改插件/项目代码优先；必要时用“项目侧 workaround”而不是改引擎。

## Build & module hygiene
- 优先最小编译影响：
  - 避免在公共头里引入重头文件
  - 能前置声明就前置声明
- Build.cs 修改要谨慎：
  - 新增依赖必须说明原因与影响
  - 避免随意开启全局宏或禁用警告
- 遵循项目既有 IWYU/Include 风格（不要大范围调整 include 顺序）

## Editor-only vs Runtime
- 严格区分：
  - `WITH_EDITOR`, `WITH_EDITORONLY_DATA`
  - Editor 模块不要被 Runtime 模块依赖
- 不要在 Runtime 路径里调用 Editor-only API。
- 资产/工具链改动要说明对 Cook/Packaging 的影响。

## UObject / GC / Lifetime
- 不要引入悬空 UObject 指针：
  - UObject 成员要用 `UPROPERTY()`（按项目约定）或安全的弱引用
- 跨线程使用 UObject 要极其谨慎：
  - 默认假设 UObject 只能在 GameThread 安全访问
- 回调/Delegate 绑定要注意解绑与生命周期：
  - 防止 lambda 捕获导致延长生命周期或访问已销毁对象

## Async / Threading
- 线程规则：
  - GameThread：UObject、UMG、Gameplay 逻辑
  - AnyThread：纯数据处理（无 UObject 访问）
- 使用 AsyncTask/TaskGraph/QueuedThreadPool 时：
  - 明确线程切换边界
  - 结果回到 GameThread 再触碰 UObject/UMG

## Asset / Packaging / Cook
- 任何涉及资源路径、加载策略、Cook 规则的改动：
  - 必须写清楚影响面（Editor/Runtime/Cook/HotUpdate）
- 默认不建议在运行时频繁同步加载：
  - 优先 AsyncLoad / StreamableManager 等方式（按项目现状）
- 涉及 Pak/IoStore/挂载时：
  - 明确：资源是否已经被 Cook、是否可被运行时加载、是否需要重启（通常不需要，但取决于系统设计）

## PuerTS binding safety (critical)

在 PuerTS 中调用 UE 的 BlueprintFunctionLibrary / UFunction 时，`out` 参数容器的元素类型必须与 UE 侧签名严格一致。
否则会发生内存解释错误，可能导致 PuerTS 侧崩溃。

### Hard rule: Out TArray element type must match exactly
- 如果 UE 签名是 `TArray<FName>&`，TS 侧必须使用 `UE.BuiltinName` 创建容器：
  - ✅ `UE.NewArray(UE.BuiltinName)`
  - ❌ `UE.NewArray(UE.BuiltinString)`（会把内存按 FString 解释，导致崩溃）

### Example
错误示例（可能崩溃）：
```ts
const outRowNames = UE.NewArray(UE.BuiltinString);
UE.DataTableFunctionLibrary.GetDataTableRowNames(table, $ref(outRowNames));
```

正确示例：
```ts
const outRowNames = UE.NewArray(UE.BuiltinName);
UE.DataTableFunctionLibrary.GetDataTableRowNames(table, $ref(outRowNames));
```
### Notes

- 对所有 out / ref / TArray<...> / TSet<...> / TMap<...> 参数同样适用：

  - 必须按 UE 侧真实类型选择对应的 Builtin* 容器类型

- 遇到不确定类型时，优先回到 C++/蓝图签名确认，而不是靠试错。

## Logging & crash safety
- UE 日志优先使用项目既有的 category：
  - `UE_LOG(LogXXX, ...)`
- 不要在 Tick 或高频路径里打印大量日志。

## Output format (UE tasks)
- 给出：
  - 受影响模块/文件
  - 线程/生命周期风险点
  - 验证方式（Editor 复现、打包验证、Cook 验证）
