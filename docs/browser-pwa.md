# 浏览器 + PWA 约束 — Gen4 宝可梦 Godot 4 项目

> 本文档规定 Web 导出、PWA 配置、线程支持、性能约束与测量要求。目标平台是浏览器 + PWA，不是桌面端。

## 1. 渲染器与导出

- Godot 4 Web 导出仅支持 **Compatibility 渲染器（WebGL 2 / GLES3）**；Forward+ 与 Mobile 渲染器不可用。
- **Web 导出不支持 C#/.NET**；全部使用 GDScript。
- 导出后必须通过**本地 Web 服务器**测试（如 `python -m http.server`），直接双击 `index.html` 会因 CORS 限制失败。

## 2. 性能约束（最高优先级）

浏览器端 3D 性能是硬约束，不是事后优化项。Gen4 双屏渲染意味着每帧绘制两个视角，填充率压力翻倍。

- **3D 视口分辨率必须可控**：使用 `3D > Scaling > Scale` 或 `SubViewport` 缩放能力，允许性能不足时降低 3D 渲染分辨率，UI 保持清晰。
- **Draw Call 控制**：合并材质、使用 LOD、避免过度细分场景。
- 每引入一个新的 3D 系统或场景，必须提供浏览器环境下的帧率与 Draw Call 测量数据，作为合并前门槛。

## 3. PWA 配置

Godot 4 Web 导出预设直接支持 PWA。导出时必须确保：

- `Progressive Web App > Enabled: true`。
- 设置图标（144×144、180×180、512×512）、离线页面与显示模式。
- 导出后检查生成的 `index.service.worker.js` 是否存在；以实际文件名为准调整注册路径。

## 4. 线程支持

- 多线程可提升性能，但要求服务器发送：
  - `Cross-Origin-Opener-Policy: same-origin`
  - `Cross-Origin-Embedder-Policy: require-corp`
- Agent 必须与部署环境确认这些头是否可配置；若不可配置，**禁止依赖线程支持**。

## 5. 浏览器/PWA 特有风险

| 风险 | 说明 | 缓解措施 |
| :--- | :--- | :--- |
| WebGL 2 性能天花板 | 双屏渲染叠加浏览器环境，3D 填充率是最大瓶颈。 | 3D 视口降分辨率；LOD；合并材质；早期建立帧率基线并持续监控。 |
| 线程支持依赖服务器头 | `SharedArrayBuffer` 需要 COOP/COEP 头，部分静态托管平台不可配置。 | 启动前确认部署环境头配置能力；不可配置则放弃线程支持。 |
| PWA Service Worker 路径问题 | Godot 4 曾出现 service worker 文件名不匹配导致 PWA 注册失败。 | 导出后检查 `index.service.worker.js` 是否存在，以实际文件名调整注册路径。 |
| Web 导出不支持 C# | Godot 4 Web 导出明确不支持 C#/.NET。 | 引擎层与逻辑层全部使用 GDScript，不引入 C# 依赖。 |