---
name: impl-facade-module
description: Implements the facade module with business-agnostic, portable and reusable DTOs and interface definitions in a six-layer Java project. Ensures DomainEntity, DomainEventDTO, and DomainEventPublisher exist before adding other facade content. Use when implementing the facade layer or when domain/infra need facade contracts.
---

# Implement Facade Module

**分类**：代码编写类

Guides implementing the **facade** module: mainly **business-agnostic, portable and reusable** DTOs and interface definitions shared by **domain** and **infra**.

## 参考现有代码

若代码仓库中已存在 facade 模块代码，实现前须先阅读现有包结构、已有类（DomainEntity、Result 等）与命名风格，在现有基础上扩展或修改，保持与仓库一致。

## When to Use

- User asks to "实现 facade 模块"、"补全 facade"、"添加领域基类/事件接口"等.
- Adding new domain-facing interfaces or DTOs in facade.
- domain 或 infra 代码依赖 facade 中的类（如 DomainEntity、DomainEventPublisher），但当前项目尚未定义这些类。

## Rule 1: Create Domain Base Types First

**在向 facade 添加业务相关接口或 DTO 之前，必须先确保以下三个基础类型存在：**

| 类型 | 说明 | 若不存在则 |
|------|------|-------------|
| **DomainEntity** | 领域实体抽象基类，含审计字段与 validate/save/delete 抽象方法 | 在 `facade/domain` 下新建 |
| **DomainEventDTO** | 领域事件传输对象（id, type, data, time, sender） | 在 `facade/domain` 下新建 |
| **DomainEventPublisher** | 领域事件发布接口（`void send(DomainEventDTO eventDTO)`） | 在 `facade/domain` 下新建 |

**流程：** 检查 `facade/domain/` 下是否已有上述三个类；若任一不存在，先按 [references/facade-core-classes.md](references/facade-core-classes.md) 创建，再继续其他 facade 内容。

### Lombok 使用约定（本模块必选）

| 类 | 注解 | 说明 |
|----|------|------|
| **Result** | `@Getter`、`@Setter` | 统一响应结果，使用 Lombok 生成 getter/setter |
| **DomainEntity** | `@Getter`、`@Setter` | 领域实体基类，使用 Lombok 生成 getter/setter |
| **DomainEventDTO** | `@AllArgsConstructor`、`@NoArgsConstructor`、`@Builder`、`@Getter`、`@Setter` | 领域事件 DTO，支持构造器与 Builder |
| **CommonRequest** | `@Data` | 通用请求基类，使用 @Data 生成 getter/setter/equals/hashCode 等 |

创建或修改上述类时须严格按上表使用注解，模板见 [references/facade-core-classes.md](references/facade-core-classes.md)。

**兼容性说明**：若项目未启用 Lombok 注解处理器（例如与 Java 21 的兼容性问题导致 `TypeTag::UNKNOWN` 等），可暂时手写 getter/setter、全参/无参构造器及 Builder，保持与上述注解生成的 API 一致，以便 domain/infra 等依赖方正常编译。

## Rule 2: Create Request / Result / Exception When Missing

**编写 facade 模块时，若以下子包或类不存在，则自动创建：**

| 子包 / 类 | 说明 | 若不存在则 |
|-----------|------|-------------|
| **request** 子包 + **CommonRequest** | 通用请求基类（如 operatorId），供入参 DTO 继承 | 在 `facade/request/` 下新建 `CommonRequest.java` |
| **Result** 类 | 统一响应结果（code, msg, data/rows；ok/fail 静态方法） | 在 `facade/common/` 下新建 `Result.java` |
| **exception** 子包 + **BusinessException** | 业务异常（code + message），供全局异常处理返回 Result | 在 `facade/exception/` 下新建 `BusinessException.java` |

**流程：**

1. 检查是否存在 `facade/request/CommonRequest.java`；不存在则创建 request 包与 CommonRequest。
2. 检查是否存在 `facade/common/Result.java`；不存在则创建 common 包与 Result。
3. 检查是否存在 `facade/exception/BusinessException.java`；不存在则创建 exception 包与 BusinessException。

创建时使用 [references/facade-common-classes.md](references/facade-common-classes.md) 和 [references/facade-core-classes.md](references/facade-core-classes.md) 中「Request / Result / Exception 模板」节，将 `BASE_PACKAGE` 替换为项目包名。

## Facade 的职责与依赖

- **职责**：主要定义与**业务无关、可移植复用**的各类 DTO、接口定义等（如领域实体基类、事件 DTO、事件发布接口）；不包含业务逻辑实现，也不包含业务特有的模型。
- **依赖**：facade 仅依赖通用库（lombok、fastjson2、hutool），不依赖 domain、application、infra、adapter、client。
- **使用方**：domain 层实体继承 `DomainEntity`，通过 `DomainEventPublisher` 发送 `DomainEventDTO`；infra 层实现 `DomainEventPublisher`（如基于 Spring `ApplicationEventPublisher`），并在工厂/仓储中注入该实现后传给 domain 实体。

## 审计字段与项目规范

- 模板中 `DomainEntity` 使用 `createId`、`updateId`（String）。若项目规范使用 `create_no`/`update_no` 或其它命名，创建时改为与项目 server 规则一致（如 `.cursor/rules/server-constraints.mdc` 中的审计字段约定）。

## 使用完成后的最后一步：更新知识图谱

- **每次使用本技能完成交付后，最后一步须更新知识图谱**。根据本次产出（如新增/修改的 facade 模块、领域类型），创建或更新 ontology 中的实体与关系（如 Project、Task、Document；`part_of`、`has_task` 等），或调用 **ontology** 技能、或向 `memory/ontology/graph.jsonl` 追加操作记录，使图谱与项目当前状态一致。详见 [ontology/SKILL.md](../ontology/SKILL.md)。



## Spring Bean 注入约束

- **必须使用 `@Resource` 注入**：在 Spring 中注入 Bean 时，必须使用 `@Resource` 注解进行字段注入；禁止使用 `@Autowired`、构造器注入、Setter 注入或通过 `ApplicationContext` 手动获取 Bean。若某模块规则进一步限制可注入类型（如 RepositoryImpl 仅允许注入 Mapper），则同时遵守该模块的更严格约束。

## 后端基础工具优先级

- **Hutool 优先**：在后端开发过程中，凡涉及字符串判断、集合判空、对象判空、日期处理、类型转换、JSON 辅助、加解密、随机值、ID 生成等基础工具操作，务必优先使用 **Hutool**（如 `StrUtil`、`CollUtil`、`ObjectUtil`、`DateUtil`、`Convert` 等）。只有在 Hutool 中找不到合适能力或无法满足业务/性能/安全要求时，才考虑 JDK 原生工具、Spring 工具类或其他第三方方案。

## Reference

- 参考内容与本 Skill 打包：领域基础类型与 Request/Result/Exception 的完整代码模板（按 BASE_PACKAGE 替换包名）见 [references/facade-core-classes.md](references/facade-core-classes.md) 和 [references/facade-common-classes.md](references/facade-common-classes.md)。
- **本模块可选第三方依赖**（pom 依赖选择）：[module-dependencies.md](../module-dependencies.md) 中「facade」小节。
