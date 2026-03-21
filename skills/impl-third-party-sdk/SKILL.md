---
name: impl-third-party-sdk
description: Selects and introduces third-party SDKs for a six-layer Java (Maven) project. Lists available SDKs, versions, and which module may use each; adds dependencyManagement in root and dependencies in the correct module. Use when the user asks to add a library, choose SDK version, introduce 某某依赖, or 引入三方包.
---

# 三方 SDK 版本选择与引入

**分类**：代码编写类

根据用户需求，从预置的 SDK 清单中选出合适的三方包，并按**引入方式**在根 pom 与对应模块中完成依赖配置。

**参考现有代码**：若仓库中已有根 pom 与各模块 pom，引入新依赖前须先查看现有 dependencyManagement、properties 与各模块已引用的依赖，在现有基础上追加，避免重复或冲突。

## When to Use

- 用户说「引入 XX」「加一个 XX 依赖」「用什么版本」「选一个 XX 的 SDK」等。
- 新增或调整项目中的第三方库（持久化、缓存、文档、JSON、工具等）。

## 流程

1. **理解需求**：根据用户描述（如「数据库」「缓存」「API 文档」「参数校验」「邮件」）确定需要的 SDK 类型。
2. **查清单**：在 [reference.md](reference.md) 的「可选 SDK 清单」中查找匹配项，确认**名称、版本、可引入模块**。
3. **根 pom**：若该 SDK 尚未在根 pom 的 `dependencyManagement` 中声明，则在根 pom 的 `properties` 中增加版本变量，在 `dependencyManagement` 中增加对应 `<dependency>`（含版本）。
4. **模块 pom**：在**允许使用该 SDK 的模块**的 `dependencies` 中增加 `<dependency>`（**不写 version**，由根统一管理）。

## 引入方式约定

| 引入方式 | 说明 |
|----------|------|
| **根 pom 统一管理** | 所有第三方 SDK 的版本均在根 pom 的 `dependencyManagement` 中声明；根 pom 的 `properties` 中定义版本变量（如 `lombok.version`）。 |
| **通用包由根直接引入** | 若项目约定 lombok、fastjson2、hutool 由根 pom 的 `<dependencies>` 引入、子模块继承，则各模块不再重复声明。 |
| **模块按需引用** | 其他 SDK 仅在**允许使用该 SDK 的模块**的 `dependencies` 中引用，不写 `<version>`。 |

## 模块与 SDK 对应关系（摘要）

- **facade / domain**：仅通用包（若未由根继承则显式写 lombok、fastjson2、hutool）。
- **client**：通用包 + spring-boot-starter-validation、knife4j（可选）。
- **infra**：通用包 + mybatis-plus、mysql-connector-j、redisson、knife4j、javax.mail、Spring AI 等（按需）。
- **application**：通用包 + spring-security-crypto、flexmark-docx（按需）。
- **adapter**：通用包 + spring-boot-starter-web、spring-boot-starter-webflux、knife4j（按需）。

禁止：在 domain/facade 中引入 MyBatis、Redis、Spring Web；在 client 中引入 domain/infra。

## 使用完成后的最后一步：更新知识图谱

- **每次使用本技能完成交付后，最后一步须更新知识图谱**。根据本次产出（如新增的依赖、模块），创建或更新 ontology 中的实体与关系（如 Project、Document；`part_of` 等），或调用 **ontology** 技能、或向 `memory/ontology/graph.jsonl` 追加操作记录，使图谱与项目当前状态一致。详见 [ontology/SKILL.md](../ontology/SKILL.md)。

## Reference

- **可选 SDK 清单**（名称、用途、版本、可引入模块、Maven 片段）：[reference.md](reference.md)。
- 与各模块依赖约定一致的总表可另见 [module-dependencies.md](../module-dependencies.md)。
