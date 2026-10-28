# 数据管线 — Gen4 宝可梦 Godot 4 浏览器/PWA 项目

> 本文档规定从合法拥有的 Gen4 ROM 提取数据、转换 3D 模型、处理 2D 资产与资产分发的规范。Agent 不得自行猜测数据格式，必须基于工具实际输出。

## 1. 数据提取（从合法拥有的 Gen4 ROM）

| 工具 | 链接 | 用途 |
| :--- | :--- | :--- |
| **romkit** | https://github.com/platinvm/romkit | 批量提取 `species.json`、`moves.json`、文本库、80×80 战斗精灵、32×32 盒子图标等。 |
| **Uxie** | https://github.com/KalaayPT/uxie | 精细查询地图头信息、脚本符号映射、文本库、全局脚本表。 |

**产出要求**：结构化 JSON 数据包（宝可梦、招式、物品、文本），存入项目 `data/` 目录。数据格式以 romkit/Uxie 实际输出为准。

## 2. 3D 模型转换

| 工具 | 链接 | 用途 |
| :--- | :--- | :--- |
| **apicula** | https://github.com/scurest/apicula | 读取 NSBMD/NSBTX/NSBCA，转换为 glTF/GLB（使用 `-fglb` 参数输出 `.glb`）。 |

**关键限制**：`apicula` 的 glTF 转换器**不支持会改变材质纹理的动画**（如某些形态变化）。选用模型前必须逐个验证。

**工作流**：`apicula` 离线转换为 glTF/GLB → 导入 Godot → GDScript 控制动画与渲染。**禁止**在运行时解析 NSBMD。

## 3. 2D 资产

- 战斗精灵、图标、训练家精灵、物品图标：romkit 输出 PNG，在 Godot 中作为 `Sprite2D` 或 `Sprite3D`（Billboard）使用。
- 地图图块与 UI：按 Godot 的 2D 资源范式组织，不依赖 NDS 图块格式。

## 4. 资产分发策略

- 仓库代码开源，但**不提交从 ROM 提取的精灵、模型、文本等资产**。
- README 中声明：使用者需自行提供合法拥有的 ROM，并通过工具链生成资产。
- 项目定位优先为私有/教育/技术学习；公开分发含原版素材的构建需谨慎评估法律风险。