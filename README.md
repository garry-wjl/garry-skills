# garry-skills

garryAi 技能仓库 — Claude AI 技能集合，帮助开发者高效完成 Java 企业级应用的**产品设计**和**代码实现**。

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)

---

## 📖 项目简介

garry-skills 是一套基于 Claude AI 的技能模块，围绕 **DDD（Domain-Driven Design）六层架构**，将企业应用开发划分为三个层次：

1. **需求设计** — 产出结构化的产品需求文档（PRD）
2. **技术方案** — 依据 PRD 产出可落地的技术方案
3. **代码实现** — 按 DDD 分层逐一实现各模块代码

技能之间按依赖顺序串联，形成从需求到代码的完整工作流。

---

## 🧩 技能列表

### 设计类

| 技能 | 说明 |
|------|------|
| [design-prd](skills/design-prd/SKILL.md) | 编写 PRD（产品需求文档），产出结构化需求 |
| [design-technical-solution](skills/design-technical-solution/SKILL.md) | 编写技术方案文档，含模块变更清单、实现顺序、接口契约 |
| [design-enterprise-ui-antd](skills/design-enterprise-ui-antd/SKILL.md) | 企业级 UI 设计（Ant Design Pro / Ant Design Mobile / Ant Design X） |

### 代码实现类（DDD 六层）

| 技能 | 层 | 职责 |
|------|:--:|------|
| [impl-facade-module](skills/impl-facade-module/SKILL.md) | Facade | 通用组件（DomainEntity、DTO、Result、事件等） |
| [impl-client-module](skills/impl-client-module/SKILL.md) | Client | 客户端契约（Param、DTO、VO、API 定义） |
| [impl-domain-module](skills/impl-domain-module/SKILL.md) | Domain | 业务逻辑（聚合、实体、Repository/Factory/Gateway 接口） |
| [impl-infra-module](skills/impl-infra-module/SKILL.md) | Infra | 持久化实现（Entity、Mapper、Repository/Factory/Gateway 实现） |
| [impl-application-module](skills/impl-application-module/SKILL.md) | Application | 应用服务（CommandService、QueryService） |
| [impl-adapter-module](skills/impl-adapter-module/SKILL.md) | Adapter | 外部接口（Controller、Listener、Config） |

### 编排与集成

| 技能 | 说明 |
|------|------|
| [impl-from-technical-solution](skills/impl-from-technical-solution/SKILL.md) | 根据技术方案按实现顺序编排所有 impl-*-module，一站式完成代码 |
| [impl-third-party-sdk](skills/impl-third-party-sdk/SKILL.md) | 选择并引入第三方 SDK（pom 依赖配置） |
| [create-java-project-structure](skills/create-java-project-structure/SKILL.md) | 创建六层 DDD 多模块 Java 项目脚手架 |

### 其他

| 技能 | 说明 |
|------|------|
| [ontology](skills/ontology/SKILL.md) | 知识图谱本体论，记录项目实体与关系 |

---

## 🚀 使用方式

通过 `npx skills` 命令安装并使用：

```bash
# 安装单个技能
npx skills add https://github.com/garry-wjl/garry-skills --skill <技能名称>

# 示例：安装技术方案设计技能
npx skills add https://github.com/garry-wjl/garry-skills --skill design-technical-solution
```

### 完整安装命令

```bash
# 设计类
npx skills add https://github.com/garry-wjl/garry-skills --skill design-prd
npx skills add https://github.com/garry-wjl/garry-skills --skill design-technical-solution
npx skills add https://github.com/garry-wjl/garry-skills --skill design-enterprise-ui-antd

# 代码实现类（按依赖顺序）
npx skills add https://github.com/garry-wjl/garry-skills --skill impl-facade-module
npx skills add https://github.com/garry-wjl/garry-skills --skill impl-client-module
npx skills add https://github.com/garry-wjl/garry-skills --skill impl-domain-module
npx skills add https://github.com/garry-wjl/garry-skills --skill impl-infra-module
npx skills add https://github.com/garry-wjl/garry-skills --skill impl-application-module
npx skills add https://github.com/garry-wjl/garry-skills --skill impl-adapter-module

# 编排与集成
npx skills add https://github.com/garry-wjl/garry-skills --skill impl-from-technical-solution
npx skills add https://github.com/garry-wjl/garry-skills --skill impl-third-party-sdk
npx skills add https://github.com/garry-wjl/garry-skills --skill create-java-project-structure

# 其他
npx skills add https://github.com/garry-wjl/garry-skills --skill ontology
```

---

## 🏗️ DDD 六层架构

```
┌─────────────────────────────────────────────────┐
│                   Adapter                        │  HTTP 控制器、MQ 监听器
├─────────────────────────────────────────────────┤
│                 Application                      │  用例编排、事务边界
├─────────────────────────────────────────────────┤
│     ┌───────────┐  ┌───────────┐                │
│     │  Domain   │  │   Infra   │                │  领域逻辑 + 持久化实现
│     └───────────┘  └───────────┘                │
├─────────────────────────────────────────────────┤
│                    Client                        │  对外接口契约
├─────────────────────────────────────────────────┤
│                    Facade                        │  通用组件/事件
└─────────────────────────────────────────────────┘
```

### 依赖规则

```
adapter → application → domain ← infra
                            ↓
                        client ← facade
```

- **禁止逆向依赖**：domain 不能依赖 application，infra 不能依赖 adapter
- **跨层依赖**：application 可依赖 domain、infra、client；adapter 可依赖 application、client

---

## 📁 项目结构

```
garry-skills/
├── README.md                      # 本文件
├── CLAUDE.md                      # Claude AI 项目指南
├── LICENSE                        # Apache 2.0
├── skills/
│   ├── README.md                  # 技能分类索引
│   ├── module-dependencies.md     # 模块依赖清单
│   ├── design-prd/
│   ├── design-technical-solution/
│   ├── design-enterprise-ui-antd/
│   ├── impl-facade-module/
│   ├── impl-client-module/
│   ├── impl-domain-module/
│   ├── impl-infra-module/
│   ├── impl-application-module/
│   ├── impl-adapter-module/
│   ├── impl-from-technical-solution/
│   ├── impl-third-party-sdk/
│   ├── create-java-project-structure/
│   └── ontology/
└── doc/                           # 产出的技术方案文档（运行时生成）
```

每个技能目录包含：
- `SKILL.md` — 技能说明文档（规则、约定、使用方式）
- `references/` — 参考资源（代码示例、模板、规范详情）

---

## 📋 推荐工作流

```
需求阶段    ──→  design-prd（编写 PRD）
                         │
设计阶段    ──→  design-technical-solution（编写技术方案）
                         │
实现阶段    ──→  impl-from-technical-solution（一站式编排实现）
                  ├── impl-third-party-sdk（引入依赖）
                  ├── impl-facade-module（通用组件）
                  ├── impl-client-module（接口契约）
                  ├── impl-domain-module（领域逻辑）
                  ├── impl-infra-module（持久化实现）
                  ├── impl-application-module（应用服务）
                  └── impl-adapter-module（外部接口）
                         │
UI 设计     ──→  design-enterprise-ui-antd（Ant Design 前端）
```

---

## 📄 许可证

[Apache 2.0](LICENSE)