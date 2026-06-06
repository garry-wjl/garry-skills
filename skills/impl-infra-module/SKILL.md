---
name: impl-infra-module
description: Implements the infrastructure module in a six-layer Java project: Lombok @Data entities with field comments; RepositoryImpl must inject only this aggregate's MyBatis Mappers (never Gateway, DomainEventPublisher, or any other bean); existence checks by business num; FactoryImpl injects only repository. Infra depends on domain and facade. Use when adding or modifying infra, persistence, or integrations.
---

# Implement Infra Module

**分类**：代码编写类

Guides implementing the **infra** module：基础设施层，实现 domain 定义的仓储/工厂/网关接口，提供持久化、外部调用、领域事件发布等。

**参考现有代码**：若仓库中已存在 infra 或该领域实现（Entity、Mapper、RepositoryImpl 等），实现前须先阅读现有包结构、表与实体映射、已有实现风格，在现有基础上扩展或修改，保持与仓库一致。

## When to Use

- 用户要求「实现 infra 模块」、「写 XX 仓储实现」、「XX 表对应的 Entity/Mapper」。
- 在六层结构中新增或修改 infra 层代码。

## 职责与依赖

- **职责**：实现 domain 的 **Repository**、**Factory**、**Gateway** 接口；定义与表对应的 **Entity**（PO，**`@Data`** + 字段注释）与 **Mapper**；实现 **DomainEventPublisher**（如基于 Spring `ApplicationEventPublisher`，放在 **common/event**）；提供通用工具、常量、异常类（如 `infra/common`）。**RepositoryImpl 仅允许注入 Mapper**（规则 8）；**FactoryImpl** 见规则 7。
- **依赖**：**仅依赖 domain**（及 facade，因 domain 依赖 facade）。不依赖 application、adapter、client。

## 包结构与约定

### 规则 1：必须存在 common 子包

- **infra 下必须存在 `common` 子包**，用于存放**跨领域**的常量、事件、异常、工具等，不与具体业务领域绑定。
- 建议子目录：
  - **common/constant**：跨领域常量（如逻辑删除标记、锁 key 前缀等）。
  - **common/event**：领域事件发布实现（如 `CommonDomainEventPublisher`）。
  - **common/exception**：基础设施层异常（如 `LockException` 等）。
  - **common/util**：跨领域工具类（如 JSON 解析、日期等）。
  - **common/client**：**client 放在 infra 的 common 子包下面**（即 `infra/common/client`）。调用外部第三方 HTTP 接口或 API 时在此完成——在 common/client 下定义**接口**，接口的**参数对象**放在 **client/param**，**API 专用的数据传输对象**（请求/响应 DTO）放在 **client/dto**；实现类也可放在 client 包下，领域 gateway 注入该接口并调用。

### 规则 2：其余子包按业务领域划分

- **除 common 外**，其余子包按**业务领域**与 domain 聚合一一对应，**一个领域一个子包**（如 conversation、message、task、user、report）。
- **按业务领域划分的子包中没有 client 子包**；client 仅存在于 infra 的 common 下（infra/common/client）。
- 每个领域子包下**仅包含**以下目录：
  - **entity**：与表结构对应的 PO，MyBatis Plus 注解；含审计字段与 is_deleted；**必须使用 Lombok `@Data`**，**禁止手写 getter/setter**；**每个字段须有注释**（类内行注释或字段 Javadoc），说明含义、单位或约束。
  - **mapper**：MyBatis Plus 的 BaseMapper<Entity> 及自定义 SQL（XML/注解）。
  - **repository**：实现 domain 的 Repository 接口；Entity↔领域实体转换；**构造器与字段仅允许本聚合表对应的 `BaseMapper<?>` 实现类**（含本聚合子表 Mapper）；**绝对禁止**注入 **任何形式的** `DomainEventPublisher`、`ApplicationEventPublisher`、**任何**领域 **Gateway**、其他聚合 Mapper、其他 Repository、Application Service 等——**除 Mapper 外不得出现任何可注入依赖**；判断「是否存在」时**必须用业务编码 `num` 查库**，**查到即存在，查不到即不存在**；**事务与分布式锁**：在 **application** 层声明（如 `@Transactional`），**禁止**在 RepositoryImpl 注入 `PlatformTransactionManager`、`TransactionTemplate`、Lock 客户端等非 Mapper Bean。
  - **factory**：实现 domain 的 Factory 接口；**构造器/字段仅注入本聚合的 Repository**，**不得**注入 Gateway、DomainEventPublisher；`createByNum` 等应委托 `Repository.build*By(num)`。若领域聚合根仍依赖 Gateway/Publisher：**由应用层**在工厂/仓储返回后补全依赖，或**先重构领域模型/构造方式**使加载路径不依赖在 infra 仓储内持有上述 Bean；**禁止**指望 RepositoryImpl 代为注入或组装。
  - **gateway**：实现 domain 的 Gateway 接口（如生成唯一编号）；若需调用外部 API，gateway 实现类注入 **common/client** 下定义的接口并调用。

### 各目录职能摘要

**common 子包下各目录（跨领域）**

| 目录 | 职能 |
|------|------|
| **common/constant** | 跨领域常量（如逻辑删除标记、锁 key 前缀等）。 |
| **common/event** | 领域事件发布实现（如 CommonDomainEventPublisher 实现 facade 的 DomainEventPublisher，委托 Spring ApplicationEventPublisher）。 |
| **common/exception** | 基础设施层异常（如 LockException 等）。 |
| **common/util** | 跨领域工具类（如 JSON 解析、日期等）。 |
| **common/client** | 外部第三方 HTTP/API 调用：接口定义、实现类；**client/param** 放接口参数对象，**client/dto** 放 API 专用请求/响应 DTO；领域 gateway 注入该接口并调用。 |

**按业务领域划分的子包下各目录**

| 目录 | 职能 |
|------|------|
| **entity** | 表映射 PO；`@Data`；字段全注释；@TableName/@TableId/@TableField；审计字段 + is_deleted；无业务逻辑。 |
| **mapper** | MyBatis Plus 的 BaseMapper<Entity>；自定义 SQL 仅在此或 XML。 |
| **repository** | **仅** Mapper（本表+本聚合子表）；**零** Gateway / 事件发布器 / 其他 Bean；存在性只按 `num`；Entity↔领域转换；不暴露 Entity 给上层。 |
| **factory** | 仅注入本聚合 Repository；不注入 Gateway、DomainEventPublisher。 |
| **gateway** | 实现 domain 的 Gateway 接口（如生成 ID）；调用外部 API 时注入 common/client 下接口。 |

## 规则

1. **common 子包必须存在**，且包含跨领域的 constant、event、exception、util 等；不在 common 中放领域专属代码。
2. **领域子包**与 domain 聚合对应，一领域一子包；每子包下**仅**包含 entity、factory、gateway、mapper、repository，**没有 client 子包**；外部 HTTP/API 调用统一在 **common/client**（接口 + param + dto）完成。
3. 表与审计字段命名与项目数据库规范一致（如 create_no、update_no、create_time、update_time、is_deleted）；不使用外键约束。
4. 领域实体与 Entity 的转换仅在 infra 的 Repository/Factory 内进行，不把 Entity 暴露给 application 或 domain。
5. 若 domain 新增某聚合的 Repository/Factory/Gateway 接口，再在 infra 中新增对应领域子包及实现；保持「domain 定义、infra 实现」的单一方向。
6. **Entity**：必须使用 **`@Data`**，**禁止**手写 getter/setter；**每一个字段**须有注释（含义、单位、枚举取值或业务约束）。
7. **FactoryImpl**：**仅**注入本聚合 **Repository**；**禁止**注入 **Gateway**、**DomainEventPublisher**。
8. **RepositoryImpl（硬约束）**：
   - **唯一允许的依赖类型**：本聚合（含子表）的 **MyBatis-Plus `BaseMapper` 实现类** 对应的 Spring Bean。
   - **绝对禁止**（构造器注入、字段注入、`@Resource`/`@Autowired`、Setter 注入，以及其他形式）：`DomainEventPublisher`、`ApplicationEventPublisher`、**任意** `*Gateway`、其他领域 **Mapper**、**其他 Repository**、`ApplicationContext` 为装配用途二次取 Bean、以及任何「仅为给领域根塞 Gateway/Publisher」的非 Mapper Bean。
   - `toDomain` / `toEntity` **只做** 字段级映射与简单常量；**不得**在仓储类内 `new` 或引用 Gateway/Publisher 来构造完整领域根（若领域模型仍要求二者，改应用层装配或改 domain 构造策略）。
   - 判断是否存在：**必须**通过业务编码 **`num`** 查库（`selectOne`/`selectCount` 条件含 `num`），有行即存在。
9. 若存量代码违反规则 7、8，**新增与修改一律按本 skill 执行**；重构时去掉仓储内对 Gateway/Publisher 的注入，将装配移到 **application** 或 domain 层策略。

## 使用完成后的最后一步：更新知识图谱

- **每次使用本技能完成交付后，最后一步须更新知识图谱**。根据本次产出（如新增/修改的 infra 实现、表结构），创建或更新 ontology 中的实体与关系（如 Project、Task、Document；`part_of`、`has_task` 等），或调用 **ontology** 技能、或向 `memory/ontology/graph.jsonl` 追加操作记录，使图谱与项目当前状态一致。详见 [ontology/SKILL.md](../ontology/SKILL.md)。

## Reference

- 与本 Skill 打包的 Case（包结构、common 常量/异常/事件/**client 接口+param+dto**、Entity、FactoryImpl、RepositoryImpl、CommonDomainEventPublisher）：[references/infra-implementation-cases.md](references/infra-implementation-cases.md) 和 [references/infra-package-structure.md](references/infra-package-structure.md)。
- **本模块可选第三方依赖**（pom 依赖选择）：[module-dependencies.md](../module-dependencies.md) 中「infra」小节。
