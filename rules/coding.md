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

## PuerTS / Unreal Engine API usage rules (TypeScript)

在 Unreal Engine + PuerTS 项目中，TypeScript 代码应基于**确定存在的 UE API**进行调用，
而不是通过“动态探测函数”的方式规避类型系统。

### Hard rules
- 不要使用“函数探测 / 动态调用”模式来访问 UE API，例如：
  - 通过 `getFunction`、`(obj as any)[name]`、`typeof value === "function"` 等方式
  - 通过字符串名称动态查找并调用 UE 对象方法
- 不要把 UE 对象当作“弱结构对象（duck typing）”来使用。

### Rationale
- 在 PuerTS 项目中：
  - UE 对象的真实类型是**可追溯的**
  - API 是否存在应由 **UE 类型系统**保证，而不是 TS 侧“试探”
- 使用函数探测会：
  - 掩盖 UE 类型错误
  - 降低可维护性
  - 破坏 IDE / 静态分析能力

### Preferred pattern (UE-style type check)
当返回类型在 TS 声明中较泛（例如 `UE.Actor`），
但运行时你期望它是某个具体子类时，使用 UE 的运行时类型检查：

```ts
if (spawned.IsA(UE.FeelingAICharacter.StaticClass())) {
    const cha = spawned as UE.FeelingAICharacter;
    cha.SetRoleID(RoleId);
    cha.SetRoleName(RoleName);
} else {
    console.error("spawned is not FeelingAICharacter");
}
```

## TypeScript: 强制声明所有局部变量类型

在 TypeScript 代码中，**所有局部变量（包括函数参数）都必须显式声明类型**，  
即使类型在上下文中已经明显或可以推导，也应明确写出。

### Hard rule
- **所有局部变量（包括 `const` 和 `let`）** 必须声明类型：
  - `const meshComp: UE.SkeletalMeshComponent = this.getMeshComponent(part);`
  - `const tex: UE.Texture = row.Tex?.LoadSynchronous();`
  - `const mats: UE.Material[] = skeletalMesh.GetMaterials();`
- 不允许省略类型推断：
  - ❌ `const meshComp = this.getMeshComponent(part);`
  - ❌ `const tex = row.Tex?.LoadSynchronous();`

### Rationale
- **类型声明增强代码可读性**，提高维护性和类型安全
- 即使 TypeScript 可以推导类型，显式声明会：
  - 帮助 IDE 完成代码补全
  - 减少类型推导过程中的错误
  - 提高团队协作时代码的可理解性
- 在 **PuerTS + UE 项目**中，变量类型通常是已知的，应该显式声明，避免使用隐式类型推断


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

## Type safety (TypeScript / PuerTS)

本项目中的 TypeScript 代码运行在 PuerTS 环境中，类型通常是**可追溯的**，
来源包括：
- Unreal Engine 的反射类型（UObject / UStruct / UEnum 等）
- 项目中明确定义的 TS 接口或类型

### Hard rules
- 禁止使用以下类型逃逸手段作为常规实现：
  - `as any`
  - `unknown as T`
  - `// @ts-ignore`
- 不要因为“类型一时不明确”而放弃类型系统。

### Preferred approaches
当类型不确定或可能为空时，优先使用以下方式（按推荐顺序）：

1. **显式类型守卫（type guard）**
   - 使用 `instanceof`
   - 使用自定义 `isXXX()` 判定函数
   - 使用 `in` / 属性存在性检查

2. **运行时检查 + 类型缩窄**
   - 使用 `if (obj != null)` / `if (obj !== undefined)`
   - 使用明确的条件分支缩窄类型范围
   - 在条件失败时输出 error 级别日志

3. **明确定义接口 / 联合类型**
   - 为 PuerTS 交互对象定义清晰的 TS 接口
   - 使用 union + 判别字段（discriminated union）表达多态

### Rationale
- 在 PuerTS 项目中，大多数对象**本身就有真实来源类型**
- 类型逃逸会掩盖 UE / 脚本交互中的真实错误
- 优先暴露错误，而不是绕过类型系统


## Type safety (C++)
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

## Error handling (TypeScript / PuerTS)

在本项目中，`try/catch` **不应作为常规控制流手段**。

### Hard rules
- 避免在以下场景中使用 `try/catch`：
  - 类型不确定
  - 对象可能为空
  - 属性/方法可能不存在
- 不要使用 `try/catch` 来“兜住一切错误”以维持流程继续。

### Preferred patterns
当存在不确定性时，优先使用：

1. **显式判空 / 判类型**
   - 使用 `if (obj == null)`、`if (!obj)`
   - 使用类型守卫进行分支判断

2. **失败即返回（fail fast）**
   - 条件不满足时：
     - 输出 `error` 级别日志
     - 根据场景直接 `return`
   - 不要在错误状态下继续执行后续逻辑

3. **日志要求**
   - 错误日志必须包含：
     - 对象/资源标识
     - 当前操作意图
     - 失败原因（类型不符 / 对象为空等）

### Allowed usage of try/catch
`try/catch` 仅在以下情况允许使用：
- 明确调用可能抛出异常的底层接口（例如引擎绑定、第三方库）
- 且无法通过 `if` / 类型检查提前规避
- 使用时必须：
  - 在 `catch` 中记录 error 日志
  - 不得静默吞掉异常


## PuerTS / UE: 禁止动态属性访问（bracket notation）调用 UE API

在 Unreal Engine + PuerTS 项目中，UE 对象的方法/属性必须通过**静态可追溯的类型**访问。
禁止使用字符串键进行动态访问来“探测再调用”。

### Hard rules
- 不要对 UE 对象或期望为 UE 对象的值使用以下模式访问方法/属性：
  - `obj["GetMesh"]`
  - `obj["GetPartMeshComponent"]`
  - `obj[name]`（name 为 string）
  - `const fn = obj["Foo"]; if (typeof fn === "function") fn(...)`
- 不要把 UE 对象先降级为 `unknown`/`Record<string, unknown>` 再做“动态反射式调用”。

### Preferred pattern
- 在入口处将对象收窄为明确 UE 类型（或项目自定义类型），后续直接调用确定 API：
  - 使用 `IsA(StaticClass())` + `as UE.XXX`（相当于 C++ Cast<>）
  - 或者在业务层保持 `this.character: UE.XXX` 的强类型字段
- 若确实存在多种角色类型：
  - 用 UE 类型系统分支（IsA）
  - 或用项目自定义的判别字段/union（但不要用字符串键探测 UE 方法）

### Allowed exceptions
仅当你处理的是**纯 JS 对象**（非 UE 对象、非 PuerTS 绑定对象），且字段名来自外部数据（JSON 等）时，才允许 `obj[name]`。

## PuerTS / UE: 入口强类型规则

### Hard rules
- 在业务层（Gameplay/UI 逻辑）中，避免将 UE 对象存为 `unknown` 类型。
- `this.character`、`this.actor` 等字段应声明为明确 UE 类型或明确的项目封装类型：
  - `private character: UE.AFeelingAICharacter`
  - 或 `private character: UE.Actor` + 明确的类型收窄入口（IsA）

### Rationale
- 若入口类型是 `unknown`，实现会被迫走“动态探测”路径，导致：
  - bracket notation
  - typeof function 探测
  - 难维护、难 refactor、难静态分析

## TypeScript / PuerTS: 禁止“条件表达式直接返回失败结果”

在业务代码中，当函数可能失败并返回 `null` / `undefined` / `false` 等结果时，
禁止使用三元表达式或简写形式直接返回，而不进行显式检查和日志输出。

### Hard rules
- 不要写成：
  - `return ok ? value : null`
  - `return value || null`
  - `return cond && value`
- 不要在失败路径上“静默返回”，必须显式处理。

### Preferred pattern
当结果可能为空或失败时，必须：

1. **显式 if 分支检查**
2. **在失败路径打印日志（至少 warn 级别）**
3. **再 return null / return false / return**

例如：

```ts
const ok = UE.DataTableFunctionLibrary.Generic_GetDataTableRowFromName(table, rowName, $ref(outRow));
if (!ok) {
    console.warn(`[tryGetRowValue] failed to get row; table=${table.GetName?.() ?? "<unknown>"}, row=${rowName}`);
    return null;
}
return outRow;
```

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
