# Coding Rules (Global)

这些规则用于：命名、类型、错误处理、可维护性，以及避免“AI 常见偷懒”。

## Naming

命名规则需根据语言与技术栈区分适用范围。Codex 在多语言仓库中应按实际代码上下文选择对应规则。

---

### TypeScript / JavaScript / 通用脚本语言

- 避免 1~2 个字母变量名（例如：`g`, `ok`, `tmp`）。
  - 允许的例外：
    - 循环下标：`i`, `j`, `k`
    - 极小作用域且语义明确的数学表达：`x`, `y`（尽量少用）
- 命名应表达“意图 + 单位/语义”：
  - `timeoutMs`, `retryCount`, `assetPath`, `isMounted`
- 布尔命名优先 `is/has/can/should`：
  - `isReady`, `hasError`, `canRetry`, `shouldRebuild`
- 集合命名用复数或明确后缀：
  - `items`, `assetPaths`, `pendingRequests`

---

### Unreal Engine / C++（UE 风格）

遵循 Unreal Engine 官方编码规范与项目既有风格，优先级高于通用 C++ 习惯。

- **类型命名（Types）**
  - `UObject` 派生类：`U` 前缀（`UCharacterComponent`）
  - `AActor` 派生类：`A` 前缀（`APlayerCharacter`）
  - `UStruct`：`F` 前缀（`FCharacterData`）
  - 枚举：
    - 传统 enum：`E` 前缀（`EFeatureElem`）
    - `enum class`：同样使用 `E` 前缀
- **接口**
  - 接口类：`U` + `Interface` 后缀（`UMyFeatureInterface`）
  - 接口实现：`I` 前缀（`IMyFeatureInterface`）

- **函数命名**
  - 使用 PascalCase（大写字母开头）
  - 表达“动作 + 目标”：
    - `LoadAsset`
    - `StartDownload`
    - `ApplyCustomization`
  - Getter / Setter：
    - `GetHealth()`, `SetHealth(float InHealth)`
    - 布尔 Getter 使用 `Is/Has/Can`：
      - `IsValid()`, `HasAuthority()`, `CanInteract()`

- **变量命名**
  - 成员变量使用 PascalCase（遵循 UE 风格）：
    - `Health`, `CurrentState`, `CachedMesh`
  - 布尔成员变量：
    - `bIsReady`, `bHasInitialized`, `bCanRetry`
  - 函数参数：
    - 使用 PascalCase，必要时使用 `In` 前缀：
      - `InDeltaTime`, `InAssetPath`
  - 局部变量：
    - 允许较短但有语义的名字：
      - `Index`, `Count`, `Result`, `Mesh`

- **避免事项**
  - 不要在 UE C++ 中引入 snake_case 或 camelCase 变量命名
  - 不要混用非 UE 风格前缀（例如 `m_`, `_` 前缀）
  - 不要为“看起来像标准 C++”而破坏 UE 项目的一致性

---

### Cross-language consistency

- 同一概念在不同语言中应保持语义一致：
  - C++：`ApplyCustomization`
  - TS：`applyCustomization`
- 避免在绑定层（例如 C++ ↔ TS / Blueprint）中引入不同语义的命名。

## Error handling (strict)
- 禁止宽泛吞错：
  - ❌ `try { ... } catch {}`（空 catch）
  - ❌ `catch (e) { /* ignore */ }`
  - ❌ 只打印日志但继续走导致隐藏故障
- 允许的 catch 形态（至少满足一条）：
  - ✅ 转换成结构化错误返回（Result/enum/error code）
  - ✅ 记录错误并 rethrow（带上下文）
  - ✅ 有明确兜底策略（例如降级路径）且对外显式标记
- 错误信息必须可定位：
  - 包含关键上下文：参数、资源 id、路径、状态、操作阶段
  - 不要只写“failed”，要写“what failed + why + what next”

## Type safety
- TypeScript：
  - 避免 `as any`、`// @ts-ignore`、`unknown as T` 这种逃逸。
  - 优先：
    - 类型守卫（type guard）
    - 运行时检查 + 缩窄
    - 明确定义接口/联合类型
- C++：
  - 避免不必要的 `reinterpret_cast`。
  - 尽量用明确的所有权语义（RAII、智能指针/引用规则）并保持一致。
- 如果必须做不安全处理：
  - 隔离在单独函数/模块
  - 写清楚“为什么必要、风险、如何验证”

## Code structure
- 优先清晰胜过“炫技”：
  - 少用过度抽象、魔法常量、隐式副作用
- 控制函数复杂度：
  - 超过 ~60 行通常应拆分（除非确有理由）
  - 早返回（guard clauses）优于深层嵌套
- 保持单一职责：
  - 解析/验证/执行/记录分开，避免一坨大函数

## Logging / Observability
- 日志应能帮助定位问题，但不要刷屏：
  - error/warn：包含上下文
  - debug：默认不要过多（尤其循环内）
- 不要把异常当作“正常流程控制”。

## Batch editing policy
- 一次性完成相关修改：
  - 改 API 同时改调用点
  - 改数据结构同时改序列化/反序列化
  - 改行为同时补验证/测试
- 避免“改一行试一次”的碎片化提交建议。

## Output expectations
- 给出明确的验证步骤：
  - build/test 命令
  - 运行路径/复现步骤
- 如果无法验证：
  - 明确说明限制，并给出用户侧验证 checklist。
