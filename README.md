# sinnoh-godot

用 **Godot 4 + GDScript** 从零重写 Gen4 宝可梦（参考《白金 / 心金魂银》）的同人技术项目：**3D 场景 + 2D 精灵**的混合视觉，加上下双屏异步交互。

**唯一目标平台是浏览器 + PWA**，桌面端只用于开发调试。

> 当前仓库状态：**文档先行阶段**。已落地的是架构契约、数据管线、浏览器约束与启动前验证清单，游戏代码尚未开始编写。下一步是按 `docs/startup-checklist.md` 逐项验证技术路线。

## 技术路线（已锁定）

| 项 | 决策 |
| :--- | :--- |
| 引擎 | Godot 4（唯一选项，桌面端仅用于调试） |
| 脚本 | GDScript（Godot 4 Web 导出不支持 C#/.NET） |
| 渲染器 | Compatibility（WebGL 2 / GLES3）；Forward+ 与 Mobile 在浏览器不可用 |
| 双屏 | `SubViewport` + `SubViewportContainer`：上屏 3D 世界，下屏 2D UI 与触摸 |
| 逻辑层 | 纯 GDScript 类，不继承 `Node`、不操作 `SceneTree`；渲染层只读状态、只做表现 |
| 模型 | `apicula` 离线转换为 glTF/GLB，**禁止运行时解析 NSBMD** |

浏览器端 3D 性能是硬约束而非事后优化：每引入新的 3D 系统或场景，都要给出帧率与 Draw Call 测量数据（双屏目标 ≥30fps）。

## 文档索引

| 文档 | 内容 |
| :--- | :--- |
| [`AGENTS.md`](AGENTS.md) | 项目入口与硬约束清单，改代码前先读 |
| [`docs/architecture.md`](docs/architecture.md) | 引擎路线、逻辑与渲染分离、双屏范式、数据访问、资产分发 |
| [`docs/data-pipeline.md`](docs/data-pipeline.md) | romkit / Uxie 数据提取，apicula 模型转换，2D 资产处理 |
| [`docs/browser-pwa.md`](docs/browser-pwa.md) | Web 导出、PWA 配置、线程与 COOP/COEP、性能测量要求 |
| [`docs/startup-checklist.md`](docs/startup-checklist.md) | 大规模写逻辑代码前的验证清单与全局风险表 |

## 目录结构

```
AGENTS.md              项目入口与硬约束
README.md
docs/                  架构、数据管线、浏览器约束、启动清单
data/                  结构化数据包（由工具链从 ROM 生成，不入库）
assets/                模型、精灵、UI 资产（同样不入库）
```

## 资产与法律声明

- 本仓库**只包含代码、工具脚本与文档**，不提交任何从 ROM 提取的精灵、模型、文本等原版素材。
- 使用本项目需要你**自行提供合法拥有的 Gen4 ROM**，并通过工具链自行生成资产。
- 项目定位为私有 / 教育 / 技术学习用途，非盈利。公开分发包含原版素材的构建存在法律风险，请自行评估。
- Pokémon 及相关名称为任天堂 / Game Freak / Creatures Inc. 的商标与版权财产；本项目与其无任何关联，也未获其背书。

## 数据与模型工具链

- [romkit](https://github.com/platinvm/romkit) — 批量提取物种、招式、文本库、战斗精灵、盒子图标
- [Uxie](https://github.com/KalaayPT/uxie) — 地图头信息、脚本符号映射、全局脚本表查询
- [apicula](https://github.com/scurest/apicula) — NSBMD/NSBTX/NSBCA 转 glTF/GLB（`-fglb`）

## 本地运行（后续）

Godot 4 Web 导出产物必须经本地 Web 服务器访问，直接双击 `index.html` 会因 CORS 失败：

```bash
python -m http.server 8000
```

若需要线程支持（`SharedArrayBuffer`），服务器须发送 `Cross-Origin-Opener-Policy: same-origin` 与 `Cross-Origin-Embedder-Policy: require-corp`；无法配置时不要依赖线程。

## 许可

代码以 [MIT](LICENSE) 授权。该许可不涵盖任何第三方素材或原版资产。
