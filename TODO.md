### 📋 E-Ink LLM Chat 项目执行计划

#### 1. 项目概述
- **项目名称：** E-Ink LLM Chat
- **目标环境：** Android 墨水屏阅读器 + Via 浏览器
- **核心功能：** 轻量级 LLM 对话、API 密钥本地持久化、配置导入/导出
- **技术栈：** 纯原生 HTML/CSS/JS（单文件架构），零依赖、零构建
- **开源要求：** 代码不含任何敏感信息，文档明确声明本地存储安全性

#### 2. 核心技术规范

| 模块 | 技术方案 | 关键约束 |
| :--- | :--- | :--- |
| **主存储** | `localStorage` | 键名: `eink_llm_config`；自动读写；无需权限 |
| **备份存储** | JSON 导入/导出 | 仅用于跨设备迁移与防缓存清除兜底 |
| **网络请求** | 原生 `fetch` | `stream: false`；禁用 SSE 流式输出 |
| **UI 渲染** | Vanilla JS + Block 布局 | 禁用所有 CSS 动画/过渡；避免 Flex/Grid 重绘 |
| **样式适配** | 墨水屏专属 CSS | 纯黑白(#000/#FFF)；字号≥16px；行高≥1.6；直角边框 |
| **安全合规** | 开源最佳实践 | `.gitignore` 排除配置文件；README 声明本地存储 |

#### 3. 功能模块拆解

##### 3.1 配置管理模块
- **自动加载：** 页面初始化时从 `localStorage` 读取配置，填充表单
- **实时保存：** 输入框 `oninput` 事件触发即时写入 `localStorage`
- **导出功能：** 点击按钮 → 序列化配置为 JSON → 触发 `<a download>` 下载
- **导入功能：** `<input type="file">` 选择 JSON → 解析校验 → 写入 `localStorage` → 刷新页面
- **UI 交互：** 设置面板默认隐藏，通过右上角齿轮按钮切换显示

##### 3.2 对话模块
- **消息渲染：** 用户消息右对齐+右侧黑边；AI 消息左对齐+左侧黑边
- **发送逻辑：** 校验密钥存在性 → 追加用户消息 → 调用 API → 追加 AI 回复
- **错误处理：** 网络异常/API 错误以 AI 消息形式展示，不弹窗阻断
- **滚动行为：** 新消息追加后自动滚动到底部（`scrollTo`）
- **触觉反馈：** 发送成功时调用 `navigator.vibrate(50)`（可选，需 try-catch 包裹）

##### 3.3 墨水屏 UI 适配模块
- **全局重置：** `* { transition: none !important; animation: none !important; }`
- **触控优化：** `-webkit-tap-highlight-color: transparent`；按钮最小高度 48px
- **固定底栏：** `position: fixed` 输入栏；内容区预留 `padding-bottom` 防遮挡
- **字体回退：** `font-family: "Droid Sans Fallback", sans-serif`

#### 4. 文件结构与交付物

```
eink-llm-chat/
├── index.html          # 单文件完整应用（HTML+CSS+JS）
├── README.md           # 项目说明 + 安全声明 + 使用指南
├── .gitignore          # 排除 *.local.json, config.private.* 等
└── LICENSE             # 开源许可证
```

#### 5. Code Agent 执行指令

> **请严格按照以下要求生成代码：**
> 1.  输出**单个 `index.html` 文件**，内联所有 CSS 和 JS，不引用任何外部资源
> 2.  CSS 必须包含墨水屏专属重置规则，禁用一切动画与过渡效果
> 3.  `localStorage` 操作必须包裹在 `try-catch` 中，失败时静默降级
> 4.  API 请求必须设置 `stream: false`，禁止使用 EventSource 或 ReadableStream
> 5.  导入/导出功能必须兼容 Android WebView，不使用 File System Access API
> 6.  所有用户可见文本使用中文，注释使用英文
> 7.  代码顶部添加注释说明：本项目为纯前端本地工具，密钥仅存于设备浏览器缓存
> 8.  生成 `README.md`，包含：项目简介、墨水屏使用说明、安全声明、开源贡献指南

#### 6. 验收标准

- 在 Via 浏览器中打开 `index.html`，设置面板可正常展开/收起
- 输入 API Key 后关闭浏览器再打开，密钥依然存在
- 导出 JSON 文件后可成功导入，配置完整恢复
- 发送消息后 AI 回复一次性完整显示，无逐字刷新残影
- 页面无任何灰色文字、圆角、阴影、动画效果
- 代码仓库经 `gitleaks` 扫描无敏感信息泄露
- README 中包含明确的本地存储安全声明

---

### 💡 给 Code Agent 的额外提示

-   **不要引入任何框架**：React/Vue/Angular/Svelte 等均不可用，仅使用原生 Web API
-   **不要使用 CSS 变量**：部分旧版墨水屏 WebView 对 CSS Variables 支持不完整
-   **不要使用 async/await 顶层等待**：确保脚本在 DOMContentLoaded 后执行
-   **测试兼容性**：生成的代码需在 Chrome DevTools 的 "Device Mode" 下模拟 300x400 分辨率验证布局

### 逐步执行顺序
- [x] 构建最基础框架
- [x] 可配置多个模型，每个模型可设置model_id、display_name、baseURL、api_key等等，并且支持对话中切换模型
- [x] 添加配置导入导出功能
- [x] 在模型配置中添加一个可选项"extra_body"用于给模型添加自定义地不符合openai标准协议的参数数值，典型的比如控制模型是否思考
- [ ] 添加可折叠的思维链查看功能