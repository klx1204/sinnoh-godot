# AGENTS.md — Gen4 宝可梦 Godot 4 浏览器/PWA 重写项目

> 本文件是项目入口与硬约束清单。任何 Agent 在编写代码或生成资源前必须先读本文件，并按“必读索引”加载对应详情文档。

## 项目目标

使用 **Godot 4 + GDScript**，从零重写 Gen4 宝可梦游戏（参考《白金 / 心金魂银》），实现 **3D 场景 + 2D 角色/宝可梦精灵** 的混合视觉与双屏异步交互。**唯一目标平台是浏览器 + PWA**，桌面端仅用于开发调试。

## 硬约束（不可违反）

1. **目标平台是浏览器 + PWA，不是桌面端。** 所有架构与性能决策以浏览器运行时为准。
2. **引擎锁定 Godot 4，脚本锁定 GDScript。** Godot 4 的 Web 导出不支持 C#/.NET，禁止引入 C# 依赖。
3. **Web 导出使用 Compatibility 渲染器（WebGL 2 / GLES3）。** Forward+ 与 Mobile 渲染器在浏览器中不可用。
4. **双屏必须用 `SubViewport` 实现。** 上屏 3D 世界 + 下屏 2D UI/触摸，由 `SubViewportContainer` 组合。浏览器端不得依赖多窗口 API。
5. **逻辑层与 Godot 节点解耦。** 战斗、事件、菜单状态机应为纯 GDScript 类，不继承 `Node`，不直接操作 `SceneTree`；渲染层仅读取状态并播放表现。
6. **3D 性能是硬约束。** 每引入新的 3D 系统或场景，必须提供浏览器环境下的帧率与 Draw Call 测量数据。

## 禁止事项

- 不提交从 ROM 提取的精灵、模型、文本等原版资产。
- 不在运行时解析 NSBMD/NSBTX 二进制；模型必须走 `apicula` 离线转换管线。
- 不直接移植 `pret/pokeplatinum` 或 `pret/pokeheartgold` 的 C 代码；`pret` 仅作为“逻辑字典”参考。
- 不使用 Pokémon Essentials / RPG Maker XP 作为主引擎。
- 不基于 NDS 模拟器运行原版 ROM 作为实现方式。

## 必读索引（动对应模块前先读）

| 文档 | 何时读 |
| :--- | :--- |
| `docs/architecture.md` | 设计战斗系统、事件系统、菜单状态机、双屏结构、数据访问层之前。 |
| `docs/data-pipeline.md` | 使用 romkit / Uxie 提取数据、使用 apicula 转换模型、处理 2D 资产与资产分发之前。 |
| `docs/browser-pwa.md` | 配置 Web 导出、PWA、线程支持、性能测量与优化之前。 |
| `docs/startup-checklist.md` | 大规模生成游戏逻辑代码之前；每次引入新系统前复核。 |

## 战斗系统参考基线

Godot 论坛教程（以 GDScript 构建类宝可梦战斗系统，强调软件架构与关注点分离）：
https://forum.godotengine.org/t/building-an-entire-extensible-pokemon-like-battle-system-in-godot/126991

## 给 Agent 的工作指令摘要

1. 先读本文件，再按索引读对应详情文档；与硬约束冲突的方案必须先提出并等待确认。
2. 数据层优先：在渲染完整场景前，先跑通 romkit 数据加载与查询原型。
3. 逻辑层保持纯 GDScript 类；渲染层只做表现。
4. 双屏使用 `SubViewport`；触摸输入映射到下屏并处理坐标变换。
5. 模型转换走 `apicula` 离线管线，禁止运行时解析 NSBMD。
6. 仓库只保留代码、工具脚本与文档；不提交原版资产。
7. 每次引入新系统前，先更新 `docs/architecture.md` 的接口边界，并附浏览器性能测量（如适用）。