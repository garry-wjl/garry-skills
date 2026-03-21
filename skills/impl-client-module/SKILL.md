---
name: impl-client-module
description: Defines DTOs, VOs, Params, and common types used across adapter and application in a six-layer Java project. Client has no dependency on domain, infra, or facade business types. Use when adding API request/response types, unified Result, or client-side constants.
---

# Implement Client Module

**分类**：代码编写类

Guides implementing the **client** module：对外暴露的数据契约与通用类型，供 adapter 与 application 使用。

**参考现有代码**：若仓库中已存在 client 代码，实现前须先阅读现有包结构、Param/DTO/VO 命名与 Result 用法，在现有基础上扩展，保持风格一致。

## When to Use

- 用户要求「定义 XX 请求/响应」、「添加 XX 参数类」、「统一返回格式」。
- 在六层结构中新增或修改 client 层 DTO/VO/Param 或通用类。

## 职责与依赖

- **职责**：定义与**业务无关或与 API 契约相关**的 DTO、VO、Param、枚举、常量、统一响应类（如 `Result<T>`）；可包含调用外部 API 的入参/出参。不包含业务逻辑。
- **依赖**：**不依赖 domain、infra、facade 的业务类型**。仅依赖通用库（lombok、fastjson2、hutool、validation、Knife4j 注解等）；可与 facade 共享「与业务无关」的通用 DTO（按项目约定）。

## 包结构与约定

- 按**能力或资源**划分子包，每包下按用途分子包或直接放类，例如：`client/<能力>/` 下放 Param、DTO、VO；`client/common/` 放 Result、异常类、通用请求基类。
- **命名**：*Param（入参）、*DTO（传输）、*VO（视图/返回）；列表查询可用 *ListParams、*ListVO。
- **统一响应**：若项目约定统一格式为 `{ code, message, data }`，在 client 中定义 `Result<T>`（如 `Result.ok(data)`、`Result.fail(msg)`），adapter 统一返回该类型。

## 规则

1. client 中类仅做数据承载与校验注解（如 `@NotNull`、`@Size`）；不写业务逻辑或调用 domain/infra。
2. 与前端或对接方约定的字段名、枚举值在 client 中统一维护，避免在 adapter/application 散落。
3. 若存在「当前用户」等上下文，通过 adapter 从 request 取后作为参数传入 application，不在 client 中依赖 Servlet 或 Spring 上下文。

## 使用完成后的最后一步：更新知识图谱

- **每次使用本技能完成交付后，最后一步须更新知识图谱**。根据本次产出（如新增/修改的 client 模块、API 契约），创建或更新 ontology 中的实体与关系（如 Project、Task、Document；`part_of`、`has_task` 等），或调用 **ontology** 技能、或向 `memory/ontology/graph.jsonl` 追加操作记录，使图谱与项目当前状态一致。详见 [ontology/SKILL.md](../ontology/SKILL.md)。

## Reference

- 与本 Skill 打包的 Case（Result、Param、VO 示例）：[reference.md](reference.md)。
- **本模块可选第三方依赖**（pom 依赖选择）：[module-dependencies.md](../module-dependencies.md) 中「client」小节。
