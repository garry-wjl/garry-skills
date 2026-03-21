---
name: design-enterprise-ui-antd
description: Designs enterprise-level application UI with Ant Design Pro for PC (Umi Pro scaffold or Vite/React + @ant-design/pro-components + antd) and Ant Design Mobile for mobile; creates dedicated UI projects under doc/ui with date-based naming aligned to PRD documents; for Agent/conversation features uses Ant Design X; color and style follow Ant Design color system. Use when the user asks to design enterprise UI, admin/console, build a front-end project, Ant Design Pro, or create UI for mobile/Agent chat.
---

# 企业级应用 UI 设计（Ant Design）

**分类**：设计类

**PC 端必须使用 [Ant Design Pro](https://pro.ant.design)** 技术栈搭建企业级界面（官方脚手架或等价：`antd` + `@ant-design/pro-components`，以 **ProLayout** 为主布局，列表/表单优先 **ProTable、ProForm、PageContainer** 等）；移动端使用 **Ant Design Mobile**。设计时**创建独立 UI 工程**，置于 **`doc/ui`** 下并**按日期与 PRD 一一对应**命名；若涉及 **Agent 会话/对话**场景，则使用 **Ant Design X**；色调与风格基于 Ant Design 官方配色方案供用户选择。

## When to Use

- 用户要求「设计企业级 UI」、「Ant Design Pro」、「搭前端项目」、「做管理后台/控制台界面」、「做移动端 H5/公众号界面」、「做 AI 对话/Agent 聊天界面」。
- 需要基于 **Ant Design Pro（PC）** / **Ant Design Mobile（移动）** 产出可落地的 UI 工程与页面结构。

## 设计框架约定

- **PC 端框架**：**Ant Design Pro**（[pro.ant.design](https://pro.ant.design)），**必选**。实现方式二选一（详见 [reference.md](reference.md)「Ant Design Pro（PC 端）」）：(1) **Umi + Ant Design Pro 官方脚手架**生成工程；(2) **Vite + React + TypeScript** 并安装 `antd` 与 `@ant-design/pro-components`，以 **ProLayout** 作为应用主壳，页面级用 **PageContainer**，列表/表单以 **ProTable / ProForm** 等与 Pro 规范一致。**禁止**仅用裸 `antd` Layout 拼凑后台壳子而不引入 Pro 布局与 Pro 组件体系（特例：纯营销落地页等非后台形态可与用户确认后例外）。
- **移动端框架**：**Ant Design Mobile**（[ant-design-mobile.com](https://ant-design-mobile.com)）。设计移动端界面（H5、公众号内页等）时，必须使用 Ant Design Mobile 的组件与规范（如 TabBar、NavBar、Capsule、List、Form、Dialog 等）。**移动端与 PC 端必须分为两个独立工程**（即 `web/` 与 `mobile/` 两个子工程），不得在同一工程内用路由混合，见 [reference.md](reference.md)。
- **UI 工程位置与命名**：设计出的 UI 工程必须放在**项目根目录下的 `doc/ui` 文件夹**中；**按日期命名，且与 PRD 文档一一对应**。对应规则：若 PRD 文档为 `doc/产品方案/YYYY-MM-DD_<产品或需求名称>-PRD.md`，则对应 UI 工程目录为 `doc/ui/YYYY-MM-DD_<产品或需求名称>/`（与 PRD 文件名去掉 `-PRD.md` 后的部分一致）。若该日期下有多份 PRD，则每个产品/需求单独一个子目录。详见 [reference.md](reference.md) 中「UI 工程存放与 PRD 对应」。
- **Agent 会话场景**：若需求涉及 **Agent 对话、AI 聊天、多轮会话**，则在该工程内使用 **Ant Design X**（[x.ant.design](https://x.ant.design)）：`@ant-design/x-sdk` 提供 useXChat、useXConversations、Chat Provider 等；界面组件可使用 Bubble、Sender、Welcome、Conversations 等。详见 [reference.md](reference.md) 中「Ant Design X」小节。
- **色调与风格**：基于 **Ant Design 配色方案**供用户选择，不自行发明色值。可选方向包括：
  - **品牌主色**：Ant Design 默认品牌蓝（#1677FF），或从 Ant Design 12 色板中选一作为主色（薄暮、火山、日暮、金盏花、日出、青柠、极光绿、明青、拂晓蓝、极客蓝、酱紫、法式洋红）。
  - **主题变体**：默认（default）、紧凑（compact）、暗色（dark），通过 ConfigProvider 配置。
  - **自定义主色**：使用 `@ant-design/colors` 或 Ant Design 的 Design Token 生成算法，由用户选定主色后生成整套衍生色。
  设计前可列出 2～3 套配色/主题方案供用户选择，确定后再写入 UI 工程的 theme/ConfigProvider。

## 输出物

- 与 PRD 一一对应的 **UI 产出**，位于 **`doc/ui/YYYY-MM-DD_<产品或需求名称>/`**（PRD 路径：`doc/产品方案/YYYY-MM-DD_<产品或需求名称>-PRD.md`）。**PC 端与移动端必须为两个独立工程**：该目录下包含 **`web/`**（PC 端，**Ant Design Pro** 技术栈）与 **`mobile/`**（移动端，Ant Design Mobile）两个子工程，各自含 package.json、入口、路由、示例页；仅需 PC 时可只建 `web/`，仅需移动端时可只建 `mobile/`。
- 若含 Agent 会话：集成 Ant Design X，提供至少一个会话/聊天示例页。
- **主题/配色**：按用户选择的方案配置 ConfigProvider 或 CSS 变量，并在工程 README 中说明当前采用的配色与如何切换；README 中须注明**对应 PRD 文档路径**，便于追溯。

## 使用完成后的最后一步：更新知识图谱

- **每次使用本技能完成交付后，最后一步须更新知识图谱**。根据本次产出（如新建的 UI 工程路径、对应 PRD），创建或更新 ontology 中的实体与关系（如 Document、Project；`part_of` 等），或调用 **ontology** 技能、或向 `memory/ontology/graph.jsonl` 追加操作记录，使图谱与项目当前状态一致。详见 [ontology/SKILL.md](../ontology/SKILL.md)。

## Reference

- UI 工程存放与 PRD 对应、UI 工程结构、**Ant Design Pro（PC）**、Ant Design Mobile、配色选项、Ant Design X 集成方式见 [reference.md](reference.md)。
