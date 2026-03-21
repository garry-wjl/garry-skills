# Reference: 企业级 UI 设计（Ant Design）

## UI 工程存放与 PRD 对应

- **存放位置**：设计出的 UI 工程必须放在**项目根目录下的 `doc/ui` 文件夹**中。若 `doc/ui` 不存在，须先创建。
- **命名规则（与 PRD 一一对应）**：  
  - PRD 文档路径：`doc/产品方案/YYYY-MM-DD_<产品或需求名称>-PRD.md`  
  - 对应 UI 工程目录：`doc/ui/YYYY-MM-DD_<产品或需求名称>/`  
  - 即：与 PRD 文件名去掉 `-PRD.md` 后的部分保持一致（含日期与产品/需求名称）。  
  - 示例：PRD 为 `doc/产品方案/2025-03-16_家庭收支记录系统-PRD.md` 时，UI 工程为 `doc/ui/2025-03-16_家庭收支记录系统/`。
- **一一对应**：每个 PRD 文档对应一个 UI 工程目录；同一日期下多份 PRD 时，每个产品/需求各一个子目录。工程 README 中须写明**对应 PRD 文档路径**（如 `doc/产品方案/2025-03-16_家庭收支记录系统-PRD.md`），便于追溯。

## UI 工程结构建议

设计时创建独立前端工程，与后端分离，且置于 `doc/ui/YYYY-MM-DD_<产品或需求名称>/` 下。**PC 端与移动端必须分为两个独立工程**，不得在同一仓库内用路由混合。

**结构约定（PC + 移动端双端时）：**

```
doc/ui/YYYY-MM-DD_<产品或需求名称>/
├── web/                  # PC 端独立工程：Ant Design Pro（Umi 脚手架 或 Vite+React+antd+@ant-design/pro-components）
│   ├── package.json      # 依赖：antd, @ant-design/pro-components, react, react-router-dom（Umi 方案按脚手架）；若含 Agent 则加 @ant-design/x-sdk
│   ├── vite.config.ts
│   ├── index.html
│   ├── src/
│   │   ├── main.tsx
│   │   ├── App.tsx
│   │   ├── theme/
│   │   ├── routes/
│   │   ├── pages/
│   │   └── components/
│   └── README.md
├── mobile/               # 移动端独立工程：Vite + React + antd-mobile
│   ├── package.json      # 依赖：react, antd-mobile, react-router-dom
│   ├── vite.config.ts
│   ├── index.html
│   ├── src/
│   │   ├── main.tsx
│   │   ├── App.tsx
│   │   ├── theme/
│   │   ├── routes/
│   │   ├── pages/
│   │   └── components/
│   └── README.md
└── README.md             # 总说明 + 对应 PRD 文档路径，并注明 web / mobile 分别的入口与运行方式
```

- **仅需 PC 端**时：只创建 `web/` 一个子工程即可。
- **仅需移动端**时：只创建 `mobile/` 一个子工程即可。
- **PC `web/`**：**Ant Design Pro** — 推荐 **Umi + Pro 官方脚手架**，或 **Vite + React + TypeScript** 并安装 `antd` 与 `@ant-design/pro-components`。**`mobile/`** 安装 `antd-mobile`，两套依赖与构建彼此独立。
- 若含 Agent 会话，在对应端（一般为 `web/`）额外安装 `@ant-design/x-sdk`，并在会话相关页面使用 Ant Design X 的 Hook 与组件。

## Ant Design Pro（PC 端）

企业级 PC 端 **`web/` 工程必须以 Ant Design Pro 为技术栈**：不仅主布局，列表页、表单页、页面容器也应优先采用 ProComponents，与 [Ant Design Pro](https://pro.ant.design) 文档一致。

- **文档**：[Ant Design Pro](https://pro.ant.design)、[ProComponents](https://procomponents.ant.design)
- **推荐组件**  
  - **ProLayout**：主布局（侧栏、顶栏、面包屑、多标签、菜单与路由）。  
  - **PageContainer**：页面级标题、面包屑、操作区。  
  - **ProTable**、**ProForm** 等：后台列表与表单的标准形态。
- **搭建方式（二选一）**  
  1. **官方脚手架**：使用 Ant Design Pro 文档中的 **Umi / create-umi** 流程生成工程，将产出放入 `doc/ui/YYYY-MM-DD_<名称>/web/`，保持与 PRD 及 `doc/ui` 命名约定。  
  2. **手动集成**：在 `web/` 中安装 `antd` 与 `@ant-design/pro-components`，用 **ProLayout** 包裹路由出口，菜单与路由联动；后台类页面用 ProTable、ProForm 等，避免仅用裸 `Layout` + `Table` 拼凑。
- **主题**：通过 antd **ConfigProvider**（或 Umi 主题配置）与技能中的「Ant Design 配色方案」一致；Pro 组件随 antd token 联动。

## Ant Design Mobile（移动端）

当需求包含**移动端界面**（H5、微信公众号内页、小程序等）时，必须使用 **Ant Design Mobile** 在 **独立工程 `mobile/`** 内设计移动端 UI，与 PC 端 `web/`（**Ant Design Pro**）分开，不混在同一工程。

- **文档**：[Ant Design Mobile](https://ant-design-mobile.com)
- **安装**：在 `mobile/` 工程内 `npm install antd-mobile` 或 `pnpm add antd-mobile`
- **使用**：移动端所有页面均在 `doc/ui/YYYY-MM-DD_<名称>/mobile/` 下开发，仅使用 antd-mobile 组件；PC 端仅在 `web/` 下使用 **Ant Design Pro**（`antd` + `@ant-design/pro-components`）。
- **常用组件**：TabBar、NavBar、Capsule、List、Form、Input、Button、Dialog、Toast、Picker、DatePicker、SwipeAction、PullToRefresh、InfiniteScroll 等；布局与导航遵循 Ant Design Mobile 设计规范。
- **主题**：Ant Design Mobile 支持通过 `ConfigProvider` 配置主题与主色，可与 PC 端约定同一品牌主色；在根目录 README 中说明 `web/` 与 `mobile/` 分别使用的库及主题约定。

## Ant Design 配色方案（供用户选择）

以下均来自 Ant Design 官方体系，设计时从中给出选项，由用户选择后再落配置。

- **品牌主色**  
  - 默认蓝：`#1677FF`（Ant Design 默认品牌色）。  
  - 12 色板主色（任选其一作为主色）：薄暮（红）、火山、日暮、金盏花、日出、青柠、极光绿、明青、拂晓蓝、极客蓝、酱紫、法式洋红。  
  - 文档：[Colors - Ant Design](https://ant.design/docs/spec/colors)
- **主题变体**  
  - `theme`: `default` | `compact`（紧凑）| `dark`（暗色），通过 `<ConfigProvider theme={{ algorithm }}>` 或 `theme` 包中的 `defaultAlgorithm`、`compactAlgorithm`、`darkAlgorithm` 切换。
- **自定义主色**  
  - 使用 `@ant-design/colors` 生成衍生色，再通过 ConfigProvider 的 `theme.token.colorPrimary` 注入。  
  - 安装：`npm install @ant-design/colors`

示例：列出 2～3 套方案供用户选，如「默认蓝」「拂晓蓝」「暗色 + 极光绿」。

## Ant Design X（Agent 会话）

当需求包含 **Agent 对话、AI 聊天、多轮会话** 时，在 UI 工程内使用 **Ant Design X**。

- **文档**：[Ant Design X 介绍](https://x.ant.design/x-sdks/introduce-cn/)
- **安装**：`npm install @ant-design/x-sdk --save`
- **核心能力**  
  - **useXChat**：单会话数据管理（消息、状态、操作 API）。  
  - **useXConversations** / **useXAgent**：多会话列表与管理、模型调度。  
  - **Chat Provider**：对接不同 AI 模型（OpenAI 兼容、DeepSeek 等）。  
  - **XRequest / XStream**：请求与流式响应。  
- **界面组件**（按需使用）：Bubble（气泡）、Sender（输入）、Welcome（欢迎）、Conversations（会话列表）、Prompts、ThoughtChain 等。  
- 在工程中新增「会话」或「聊天」页面，使用上述 SDK 与组件实现对话界面，并与后端 Agent 接口对接。

## 设计前与用户确认的选项

- 是否有 **移动端** 需求（H5/公众号等）→ 是则使用 **Ant Design Mobile**，并在 `doc/ui/YYYY-MM-DD_<名称>/` 下单独建 **`mobile/`** 工程，与 **`web/`**（PC 端）分开，两个独立工程；仅 PC 时只建 `web/`，仅移动端时只建 `mobile/`。
- 是否需要 **Agent 会话/聊天** 功能 → 是则引入 Ant Design X 并建会话页。
- **配色/主题**：从上述 Ant Design 配色方案中选 1～2 套（如默认蓝、紧凑、暗色、或指定 12 色板中的主色），确认后写入 theme 与 ConfigProvider。
