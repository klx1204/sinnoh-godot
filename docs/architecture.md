# 架构契约 — Gen4 宝可梦 Godot 4 浏览器/PWA 项目

> 本文档规定引擎路线、逻辑与渲染边界、双屏范式、数据访问与资产分发策略。Agent 在设计战斗、事件、菜单、双屏或数据层前必须阅读本文档。

## 1. 引擎与技术路线（已锁定）

| 决策 | 内容 |
| :--- | :--- |
| 引擎 | **Godot 4**（唯一选项） |
| 脚本 | **GDScript**。Godot 4 Web 导出不支持 C#/.NET。 |
| 渲染器 | Web 导出自动使用 **Compatibility（WebGL 2 / GLES3）**。 |
| 双屏 | **`SubViewport`**：上屏 3D 世界 + 下屏 2D UI/触摸，由 `SubViewportContainer` 组合。 |
| 目标平台 | **浏览器 + PWA**。桌面端仅用于开发调试。 |

## 2. 逻辑层策略

- 不直接移植 `pret` 的 C 代码。`pret` 代码面向 NDS 硬件，包含大量寄存器操作。
- 将 `pret` 作为“逻辑字典”：确认战斗公式、技能判定顺序、事件触发条件时阅读其 C 函数，然后用 GDScript 重新表达。
- 战斗系统核心：**状态机 + 字节码脚本虚拟机（VM）**。逻辑层实现为不依赖 Godot 节点的纯 GDScript 类，渲染层仅做表现。
- 战斗系统架构参考基线：
  https://forum.godotengine.org/t/building-an-entire-extensible-pokemon-like-battle-system-in-godot/126991

## 3. 逻辑与渲染分离

- 战斗、事件、菜单状态机实现为**纯 GDScript 类**：不继承 `Node`，不直接操作 `Node` 或 `SceneTree`。
- 渲染层通过信号或显式查询读取逻辑状态并播放表现。
- 逻辑层应可在 Godot 编辑器外用 `gdscript` 单元测试。

## 4. 双屏范式

- 主场景结构：`SubViewportContainer`（上屏，3D）+ `SubViewportContainer`（下屏，2D）。
- 触摸输入映射到**下屏**的 `SubViewport`，需处理坐标变换。
- 浏览器端不得依赖多窗口 API 替代 `SubViewport`。

## 5. 数据访问

- 启动时加载 JSON 数据并构建内存查询表（`Dictionary`），或使用异步加载避免阻塞主线程。
- 若 JSON 解析耗时过长，需转向预编译二进制格式或分块异步加载。
- Agent 在实现数据层前必须提供加载耗时测量（浏览器环境）。

## 6. 资产分发策略

- 仓库代码开源，但**不提交从 ROM 提取的精灵、模型、文本等资产**。
- README 中声明：使用者需自行提供合法拥有的 ROM，并通过工具链生成资产。
- 项目定位优先为私有/教育/技术学习；公开分发含原版素材的构建需谨慎评估法律风险。

## 7. 变更规则

每次引入新系统前，先更新本文档中该系统的接口边界；若涉及 3D 渲染或双屏，附浏览器环境下的帧率与 Draw Call 测量数据。