---
name: design-technical-solution
description: Produces technical solution documents from PRD input, structured for use with impl-*-module skills. Requires asking at least 10 design-related questions before writing; for complex PRDs, decomposes into low-coupling sub-requirements with reference to existing code. Use when the user provides a PRD and asks for technical design or implementation plan for the six-layer Java architecture.
---

# 技术方案文档（对接代码编写类技能）

**分类**：设计类

**输入**：PRD 文档（产品需求文档）。PRD 可由 **design-prd** 技能产出，或由用户直接提供。以 PRD 为输入，产出**便于按层实现**的技术方案文档：文档结构与各 **impl-*-module**（代码编写类）skill 一一对应，实现时可按章节或清单逐层调用对应 skill。

## 产出文档存放与命名

- **存放位置**：产出的技术方案文档必须放在**项目根目录**下的 **`doc/技术方案/`** 文件夹中。若该路径不存在，则先创建 `doc` 与 `doc/技术方案` 目录再写入。
- **命名规则**：文件名须带**日期前缀**，格式为 `YYYY-MM-DD_<功能或需求名称>-技术方案.md`。例如：`2025-03-16_会话关闭-技术方案.md`。日期为文档编写或定稿日期。若由复杂 PRD 拆解为多份子需求方案，每份单独文件，均放在 `doc/技术方案/` 下并加日期前缀。

## 前置步骤：设计前必问至少 10 个问题

**在开始编写技术方案之前，必须向使用者提出至少 10 个与设计技术方案相关的问题**，并基于使用者的回答再开始工作。不得在未完成提问与回答的前提下直接输出完整技术方案。

- 问题应覆盖：业务边界与优先级、涉及模块/领域、与现有功能的关系、接口与数据约定、非功能要求（性能、安全、兼容）等。**必须让用户在部署架构处理方式中三选一**：① 重新设计部署架构；② 部署架构不变，无需设计；③ 部署架构要调整（此时必须要求用户给出调整思路）。若用户选择「部署架构不变，无需设计」，方案的「部署架构」章节默认只写明「部署架构不变，无新增部署组件，复用现有部署架构」，不展开详细部署设计。可参考 [design-questions.md](references/design-questions.md) 中的「技术方案设计前必问问题清单」。
- 得到回答后，将共识要点简要写入技术方案的「目标与范围」或「其他」中，再展开模块变更清单与实现顺序。

## 复杂 PRD：拆解为低耦合子需求

**若 PRD 较复杂**（多模块、多流程、多角色、多端等）：

1. **结合现有代码**：先阅读代码仓库中相关模块/领域的现有实现（包结构、已有实体与接口），从技术实现视角理解可复用部分与扩展点。
2. **拆解子需求**：将 PRD 拆解为**耦合度较低**的子需求（如按领域拆、按用例拆、按「先读后写」拆），使每个子需求可独立产出技术方案并对应一次「实现顺序」闭环。
3. **逐子需求产出方案**：为每个子需求单独产出技术方案（含模块变更清单、实现顺序），子需求之间尽量复用已有或前序子需求产出的接口与数据契约。
4. **覆盖 PRD 全部内容**：确保所有子需求并集覆盖 PRD 中的全部功能与非功能要求，最终完成所有 PRD 内容。

## 参考现有代码

**若代码仓库中已存在相关模块或领域代码**，编写方案前应先查看现有代码：包结构、领域/聚合命名、已有接口与分层。方案中的领域名、模块变更清单、接口与数据契约须与现有代码风格一致，避免与既有实现冲突或重复造轮。

## When to Use

- 用户提供 **PRD 文档**，并要求「写技术方案」、「出实现方案」或「按 PRD 做技术设计」。
- 在六层架构项目中，需要基于 PRD 先产出可执行的技术文档再落码。

## 文档结构（必选章节）

1. **目标与范围**  
   简要说明要解决的问题、业务目标、涉及的系统/模块与不涉及的范围。

2. **架构设计**  
   架构设计必须分为 **应用架构** 与 **部署架构** 两大部分，分别描述代码/模块视角与运行/部署视角。详见 [application-architecture-experience.md](references/application-architecture-experience.md) 与 [deployment-architecture-experience.md](references/deployment-architecture-experience.md)。

   **2.1 应用架构（必选）**  
   应用架构描述本需求在系统内部的模块、分层、包结构、依赖关系与调用关系，必须包含「代码结构」部分。代码结构需**根据业务情况**体现模块与包结构，使用**表格**表示，表格分为四列：**层**、**领域**、**包**、**职责**。  
   - **层**：六层之一（facade / client / domain / infra / application / adapter）。  
   - **领域**：业务领域或子域名（与各层领域子包一致，如 user、book、record）。  
   - **包**：该层下具体包路径或包名（如 `com.xxx.adapter.rest.record`）。  
   - **职责**：该包在本方案中的职责说明（一句话）。  
   应用架构还应说明关键模块间调用关系、同步/异步边界、核心依赖方向；adapter 层除 HTTP Controller 外，还应承载定时任务、事件监听等外部触发入口；模块调用关系必须明确区分**命令类控制/调用**（CommandController / CommandService，负责新增、修改、删除、触发动作）与**查询类控制/调用**（QueryController / QueryService，负责查询、列表、详情、统计）；若无法判断某个接口或用例属于命令类还是查询类，必须向用户确认，不得自行猜测；若涉及现有系统改造，须说明复用与扩展点。

   **2.2 部署架构（必选）**  
   部署架构描述本需求上线后的**应用与前端运行拓扑**，不设计数据库/缓存/MQ/对象存储等基础设施本身的部署方式（默认使用云服务）。至少说明：后端应用服务、前端应用（Web/H5/管理端等）的部署方式、部署入口、实例数量、网络访问关系、配置项、环境差异（dev/test/prod）、扩缩容与高可用考虑；依赖的云服务只说明用途与连接关系，不展开其基础设施部署。  
   若用户选择**部署架构不变，无需设计**，则本节默认只写明「部署架构不变，无新增应用/前端部署实例，复用现有部署架构」，不展开详细部署设计；若用户选择**重新设计部署架构**或**部署架构要调整**，则必须展开应用/前端运行拓扑、实例数量、访问入口、配置项与环境差异。

3. **Facade 层设计（必选，当方案涉及 facade 层或通用契约变化时）**  
   Facade 层用于设计跨模块通用基础类型与通用契约，便于与 **impl-facade-module** 对齐落码。若本需求不涉及 facade 层变化，也须明确写出「本次无 Facade 层变更」。详见 [facade-layer-design-experience.md](references/facade-layer-design-experience.md)。  
   **设计内容**：  
   - **通用领域基类**：如 `DomainEntity`、审计字段、领域校验方法、保存/删除抽象方法等是否需要新增或调整。  
   - **通用事件契约**：如 `DomainEventDTO`、`DomainEventPublisher`、事件基础字段、事件发送约定等。  
   - **通用返回/请求对象**：如 `Result`、分页对象、通用请求上下文等。  
   - **通用常量/异常/枚举**：跨模块复用且不归属于具体业务领域的类型。  
   **输出形式**：使用表格列出类/接口/枚举名、包路径、职责、关键字段/方法、是否新增或修改、对应实现 Skill。

4. **领域层设计（必选）**  
   遵循**先按业务分层级，再按业务领域分节、每领域下依次设计领域模型、领域动作、领域规则、领域工厂、领域网关、领域事件**的原则。须按 DDD 规范设计，便于与 **impl-domain-module** 对齐落码，并满足**领域模型基本属性与必备动作**约定。详见 [domain-layer-design-experience.md](references/domain-layer-design-experience.md) 与 [domain-model-design.md](references/domain-model-design.md)。  
   **文档目录结构**：  
   - **二级目录**：按**业务领域**划分（如 4.2 用户（user）、4.3 家庭（family）、4.4 账本（book）、4.5 账户（account））；业务领域与 4.1 业务层级划分一致，每个领域一节。  
   - **三级目录**：每个业务领域下固定六个子节——**领域模型**、**领域动作**、**领域规则**、**领域工厂**、**领域网关**、**领域事件**。  
   - 示例：`4.2 用户（user）` → `4.2.1 领域模型`、`4.2.2 领域动作`、`4.2.3 领域规则`、`4.2.4 领域工厂`、`4.2.5 领域网关`、`4.2.6 领域事件`；`4.3 家庭（family）` → `4.3.1 领域模型`、`4.3.2 领域动作`、`4.3.3 领域规则`、`4.3.4 领域工厂`、`4.3.5 领域网关`、`4.3.6 领域事件`；依此类推。

   **各三级目录内容要求**（在每一业务领域下分别填写）：

   - **领域模型**（如 4.x.1）  
     - **基本属性（强制）**：聚合根与实体须有 **id**、**num**、**create_no**、**update_no**；聚合根还必须持有领域协作依赖属性：**Repository（仓储）**、**Gateway（领域网关）**、**DomainEventPublisher（领域事件发布器）**；值对象无需。  
     - **软删除标记约束（强制）**：软删除标记（如 `is_deleted`、`deleted`、`isDeleted`）不得出现在聚合根或领域实体中；软删除属于持久化实现细节，只能出现在数据库表与 infra Entity 中。  
     - **必备动作（强制）**：聚合根及可持久化实体须有 **save(operatorId)**、**delete(operatorId)**；**所有领域动作须带操作人参数**。  
     - **表现形式**：**领域类图 + 表格**。类图表达类名、属性（含类型）、关联；表格体现领域结构（对象、类型、属性、与其它对象关系）。  
     - **聚合与边界**：本领域聚合边界、聚合根及聚合内实体/值对象；一致性边界；跨聚合仅 ID 引用；Repository/Factory/Gateway 方法清单或类图。

   - **领域动作**（如 4.x.2）  
     - 使用**表格**列出本领域聚合根/实体的领域动作，列：聚合/实体、领域动作、职责、前置条件、后置条件/规则、领域事件。  
     - **每个领域动作须配有一张时序图**（Mermaid sequenceDiagram 或等价图），表达步骤顺序、Repository/Gateway 调用、事件发布时机。

   - **领域规则**（如 4.x.3）  
     - 使用**表格**展示本领域的聚合内不变性与关键业务规则，列：聚合/对象、规则类型、规则描述、违反时表达。

   - **领域工厂**（如 4.x.4）  
     - **强制设计领域工厂（Factory）接口**：每个聚合根必须设计对应 `*Factory`，由 domain 层定义接口，infra 层实现。  
     - **对象构建约束**：聚合根、实体等领域对象的创建、加载、重建必须通过领域工厂完成；application 层不得直接 `new` 领域对象，也不得直接调用领域对象静态 `create` 方法构建对象。  
     - **方法清单**：使用表格列出工厂方法，列：Factory、方法名、入参、返回值、职责、依赖。领域工厂**只能包含两个方法**：`create(...)`（根据创建该领域对象时用户可能填写的业务字段构建新的领域对象）、`createByNum(...)`（根据业务编码 num 构建既有领域对象，方法内部通过 Repository 获取并构建领域对象）。不得设计 `createById(...)`、`rebuild(...)` 或其他工厂方法。  
     - **create 参数约束**：`create(...)` 的入参只能是创建该领域对象时用户可能填写的字段；不得包含 `operatorId`、创建人/更新人、状态、审计字段、系统生成编号、默认值、流程流转字段等用户不可见且用于内部流转的内容。  
     - **时序说明**：`create(...)` 与 `createByNum(...)` 都用于构建领域对象；`createByNum(...)` 的时序须体现通过 Repository/仓储获取数据并构建领域对象。

   - **领域网关**（如 4.x.5）  
     - **定义位置**：领域网关（Gateway）接口必须定义在 domain 层，用于表达领域层需要的外部能力或跨边界协作能力。  
     - **实现位置**：领域网关实现在 infra 层，由基础设施适配第三方服务、编号生成、外部系统调用、读模型协作等具体能力。  
     - **方法清单**：使用表格列出 Gateway、方法名、入参、返回值、职责、外部依赖、失败策略。  
     - **依赖约束**：domain 只依赖 Gateway 接口，不依赖 infra 实现；application 不直接调用 GatewayImpl。

   - **领域事件**（如 4.x.6）  
     - 使用**表格**列出本领域涉及的领域事件，列：事件名、触发时机、载荷要点、可订阅方/用途。

   在「领域层设计」章首可保留 **4.1 业务层级划分**（表格列出层级与业务/子域说明），再按业务领域分节。详见 [domain-model-design.md](references/domain-model-design.md) 中「领域模型设计：领域类图与领域动作」及「**领域对象设计规范（专业版）**」小节。

5. **基础设施层设计（必选，当方案涉及 infra 层时）**  
   基础设施层用于实现 domain 定义的 Repository / Factory / Gateway 接口，以及数据库 Entity、Mapper、外部客户端、事件发布器等基础设施能力，便于与 **impl-infra-module** 对齐落码。若本需求不涉及 infra 层变化，也须明确写出「本次无基础设施层变更」。详见 [infra-layer-design-experience.md](references/infra-layer-design-experience.md)。  
   **设计内容**：  
   - **Entity / Mapper**：按数据库设计列出需要新增/修改的 Entity、Mapper、XML/注解 SQL；Entity 字段须与数据库表结构一致。  
   - **RepositoryImpl**：实现 domain Repository 接口，仅负责 Entity ↔ 领域对象转换与持久化；RepositoryImpl 不作为 application 查询入口。  
   - **FactoryImpl**：实现 domain Factory 接口，只能实现 `create(...)` 与 `createByNum(...)` 两个方法；两者都用于构建领域对象，`createByNum(...)` 委托 Repository `findByNum(num)` 获取数据后构建领域对象。  
   - **GatewayImpl**：实现 domain Gateway 接口，如编号生成、第三方服务调用、读模型协作等。  
   - **common 能力**：如事件发布实现、外部 client、常量、异常、工具类等。  
   **输出形式**：按业务领域使用表格列出类型（Entity/Mapper/RepositoryImpl/FactoryImpl/GatewayImpl/common）、类名、包路径、职责、依赖、对应表/外部服务、是否新增或修改。

6. **应用层设计（必选，当方案涉及 application 层时）**  
   与**领域层设计**一致：**先根据业务层级/业务模块划分，再按具体模块编写**；模块与「领域层设计」中的业务领域（如 4.2 用户、4.3 家庭、4.4 账本）一一对应，便于与 impl-application-module 落码对齐。详见 [application-layer-design-experience.md](references/application-layer-design-experience.md)。  
   **应用层依赖硬约束**：application 层**禁止注入或调用任何 domain Repository 接口**，**禁止注入或调用任何领域 Gateway 接口**；Repository 与 Gateway 仅供领域对象、领域工厂及 infra 实现内部使用。CommandService 如需读取/校验其他数据，必须调用对应领域的 **QueryService** 方法；查询需求统一由对应领域的 QueryService 提供，QueryService 可通过 infra Mapper 做只读查询并返回 client DTO。  
   **文档目录结构**：  
   - **二级目录**：先 **6.1 业务模块划分**（表格列出本方案涉及的 application 模块，与 4.1 业务层级/领域对应），再按**业务模块**分节（如 6.2 用户（user）、6.3 家庭（family）、6.4 账本（book）、6.5 账户（account）；若认证单独成模块可为 6.2 认证（auth）等）。  
   - **三级目录**：每个业务模块下包含——**Service 方法清单**（本模块的 Service、方法签名、职责、入参/出参）、**方法时序逻辑**（本模块关键方法的执行步骤或时序图）。  
   - 示例：`6.2 用户（user）` → `6.2.1 Service 方法清单`、`6.2.2 方法时序逻辑`；`6.3 家庭（family）` → `6.3.1 Service 方法清单`、`6.3.2 方法时序逻辑`；跨模块或公共的 Service（如导入、统计、公众号绑定）可单独成节（如 6.x 导入（import）、6.x 统计（stats））。  
   - **方法时序逻辑**内容要求：**设计出的所有 Service 方法都须有时序图**描述实现逻辑；每个方法配一张 Mermaid sequenceDiagram（或等价图），表达步骤顺序。CommandService 步骤通常为：1. 参数校验 2. 获取 Redis 分布式锁（锁 key 基于业务唯一标识，如 num）3. 解析操作人/鉴权 4. 如需查询则调用对应领域 QueryService 5. 通过 Factory 创建/加载领域对象 6. 调用领域动作（领域对象内部通过其持有的 Gateway/Repository/DomainEventPublisher 完成外部协作与事件发布）7. 释放 Redis 分布式锁 8. 返回。禁止出现 application 直接调用 Repository 或 Gateway 的步骤。事务边界须内嵌在锁区域内（先获取锁，再开启事务）。跨 Service 调用须标明。  
   详见 [implementation-layers.md](references/implementation-layers.md) 中「**应用层设计：Service 方法与时序逻辑**」小节。

7. **Adapter 层设计（必选，当方案涉及 adapter 层时）**  
   与**领域层设计**一致：**先根据业务层级/业务模块划分，再按具体模块编写**；模块与「领域层设计」及「应用层设计」中的业务模块对应，便于 **impl-adapter-module** 落码。adapter 层是外部触发入口层，除 HTTP Controller 外，还包括**定时任务入口**（如 Scheduler/Job）与**事件监听入口**（如 MQ Listener、Domain/Application Event Listener）。详见 [adapter-layer-design-experience.md](references/adapter-layer-design-experience.md)。  
   **接口约定（强制）**：  
   - **HTTP 方法仅允许两种**：**GET**（查询）、**POST**（增删改）；不得使用 PUT、DELETE、PATCH 等。  
   - **入参必须使用 `*Param` 类**：Controller 方法的入参必须使用 `*Param` 类封装，定义在 client 层，以 JSON 或 query 方式传递；禁止使用 `DTO`、领域对象或零散原始类型作为 Controller 入参。  
   - **返回值必须使用 `*VO` 类**：Controller 返回的业务数据（除 `Result` 等全局包装结构外）必须使用 `*VO` 类封装，定义在 client 层；返回格式为 `Result<*VO>` 或 `Result<List<*VO>>`。若应用层返回的是 `DTO` 类型，控制器须在接口时序逻辑中说明 `DTO` → `VO` 的转换步骤。  
   - **入参与返回值**：每个 HTTP 接口须用 **JSON** 方式描述入参（`*Param`）和返回值（`Result<*VO>`）的 JSON 示例或 schema，便于与 impl-client-module、impl-adapter-module 对齐。  
   - **定时任务入口**：须描述任务名称、触发周期/cron、幂等策略、并发控制、调用的 Application Service、失败重试与告警。  
   - **事件监听入口**：须描述监听来源（MQ topic/tag、事件类型等）、消费幂等、重试/DLQ、并发消费、调用的 Application Service、异常处理与告警。  
   **文档目录结构**：  
   - **二级目录**：先 **7.1 业务模块划分**（表格列出本方案涉及的 Controller / Scheduler / Listener / Config 模块，与 6.x 应用层模块对应），再按**业务模块**分节（如 7.2 认证（auth）、7.3 家庭（family）等）。  
   - **三级目录**：每个业务模块下按实际入口类型包含——**Controller 接口清单**（本模块的 GET/POST、路径、入参/出参 **JSON 描述**、职责）、**定时任务清单**、**事件监听清单**、**接口/任务/监听时序逻辑**（每个入口须配有时序图描述实现逻辑）。  
   - **接口时序逻辑**内容要求：**设计出的所有 adapter 入口都须有时序图**描述处理逻辑；HTTP 接口配 Mermaid sequenceDiagram（Client→Controller→Service）；定时任务配 Scheduler→Application Service；事件监听配 MQ/Event→Listener→Application Service。步骤含：接收触发、鉴权/解析上下文（如有）、参数校验、调用 Application Service、封装返回或确认消费；异常与错误码/重试/DLQ/告警映射可一并说明。  
   详见 [implementation-layers.md](references/implementation-layers.md) 中「**控制器设计：接口与时序逻辑**」小节。

8. **数据库设计（必选）**  
   须包含**数据库设计**模块，明确持久化方案，便于 **impl-infra-module** 落表与 Mapper。详见 [database-design-experience.md](references/database-design-experience.md)。
   - **主键 id（强制）**：每张业务表的主键 **`id` 必须为 `BIGINT` 类型，且使用数据库自增**（如 MySQL：`BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY`）。**禁止**将主键 `id` 设计为 `VARCHAR/UUID` 等非整型自增方案（对外业务编号仍用 **`num`** 等独立字段 + 唯一索引）。领域层/Java 实体中与之对应的标识类型建议 **`Long`**。详见 [implementation-layers.md](references/implementation-layers.md)「数据库设计模块」。
   - **创建/更新时间（强制）**：须具备 **`create_time`（创建时间）**、**`update_time`（最后更新时间）** 两列；**禁止**使用 `created_at`、`updated_at` 等其它命名。时间类型**必须精确到毫秒**（如 MySQL：`DATETIME(3)`；PostgreSQL：`TIMESTAMP(3)`；DDL 与表结构描述须一致）。详见 [implementation-layers.md](references/implementation-layers.md)「数据库设计模块 → 时间字段约定」。
   - **表结构**：列出主要表名、字段（含类型、是否必填、索引）、主键与外键或逻辑关联；若有枚举/字典表一并列出。
   - **DDL 语句（强制）**：须**生成对应的 DDL 语句**（如 CREATE TABLE、CREATE INDEX 等），可直接执行建表；与表结构描述一致；**DDL 须体现上述 `id` 约定及 `create_time`/`update_time` 毫秒精度**。
   - **刷数/数据迁移**：若方案涉及**刷数**（历史数据迁移、初始化数据、枚举数据等），须**生成对应的 DML 语句**（如 INSERT、UPDATE）；注明执行顺序与前置条件。
   - **与领域对应**：标明表与领域聚合/实体的对应关系。
   - 可选：关键查询场景与索引建议、分表分库策略。详见 [implementation-layers.md](references/implementation-layers.md) 中「数据库设计模块」小节。

9. **模块变更清单**  
   **核心**：按层列出变更，且每条变更标明对应的**代码编写类** skill，便于实现时直接选用。
   - **facade**：新增/修改 DomainEntity、DomainEventDTO、DomainEventPublisher、CommonRequest、Result 等；**须与「Facade 层设计」一致** → 使用 **impl-facade-module**。
   - **client**：新增/修改 ParamDTO、DTO、VO、Result、常量 → 使用 **impl-client-module**。
   - **domain**：新增/修改聚合、实体、Repository/Factory/Gateway 接口、值对象、领域事件常量；**须与「领域层设计」中的领域类图、领域动作一致** → 使用 **impl-domain-module**。
   - **infra**：新增/修改 Entity、Mapper、Repository/Factory/Gateway 实现、common 下常量/异常/事件发布；**须与「基础设施层设计」和「数据库设计」一致** → 使用 **impl-infra-module**。
   - **application**：新增/修改 CommandService、QueryService、StreamService 或 Agent 相关 config/hook/interceptor/service/tool；**须与「应用层设计」一致** → 使用 **impl-application-module**。
   - **adapter**：新增/修改 Controller、Scheduler/Job、Listener、或 config 下全局组件；**须与「Adapter 层设计」一致** → 使用 **impl-adapter-module**。

10. **代码分支命名（必选）**  
   技术方案中须**明确规定**本需求/修复对应的代码仓库分支名，便于实现时统一创建分支。  
   **命名规则**：  
   - **需求类（新功能、需求迭代）**：`feature-年月日-需求名称(英文)`。年月日为 8 位数字 `YYYYMMDD`，需求名称使用英文短横线连接（如多个词用 `-`），小写。示例：`feature-20250319-family-invite`、`feature-20250319-account-export`。  
   - **BUG 修复类**：`hotfix-年月日-bug名称(英文)`。年月日同上，bug 名称使用英文短横线连接，小写。示例：`hotfix-20250319-login-timeout`、`hotfix-20250319-amount-calculation`。  
   在方案中须写出**本条方案对应的具体分支名**（一条方案一个分支名），日期取方案文档日期或计划开发日期。

11. **实现顺序与依赖**  
   建议实现顺序（与技能使用顺序一致）：  
   **facade → client → domain → infra → application → adapter**。  
   若某需求只涉及部分层，则只列出涉及的层及顺序；跨层需求按文档结构和依赖关系推进，通常先确定 facade/client/domain 契约，再设计 infra、application，最后 adapter；数据库设计须与领域层和基础设施层相互校验。

12. **接口与数据契约（可选）**  
   列出关键 API（路径、**仅 GET/POST**、入参与返回值 **JSON 描述**）及主要 DTO/VO 名称，便于与 **impl-client-module** 和 **impl-adapter-module** 对齐。

13. **其他（可选）**  
   外部依赖、配置项、非功能需求等，按需简短列出。

## 模块级自检门禁

在技术方案设计过程中，**每完成一个模块/章节后，必须立即进行该模块自检；自检通过后，才能继续设计下一模块/章节**。不得在未完成当前模块自检的情况下直接进入下一步。

模块级自检要求：

1. **架构设计完成后自检**：确认应用架构、部署架构、模块调用关系、命令/查询归属、部署架构处理方式均已明确。
2. **Facade 层设计完成后自检**：确认是否有 Facade 层变更；若无变更，已明确写出「本次无 Facade 层变更」。
3. **领域层设计完成后自检**：确认领域模型、领域动作、领域规则、领域工厂、领域网关、领域事件均已完整；Factory、Gateway、Repository 等强规约已满足。
4. **基础设施层设计完成后自检**：确认 Entity、Mapper、RepositoryImpl、FactoryImpl、GatewayImpl、common 能力与领域层/数据库设计一致。
5. **应用层设计完成后自检**：确认 CommandService / QueryService 分工、ParamDTO 入参、DTO 返回、Repository 禁用约束、Service 时序图均满足要求。
6. **Adapter 层设计完成后自检**：确认 Controller、定时任务、事件监听等入口清单和时序图完整。
7. **数据库设计完成后自检**：确认表结构、DDL、必要 DML、主键、业务编码、审计字段、时间字段和索引设计完整。
8. **模块变更清单完成后自检**：确认每条变更都能对应唯一实现 Skill，且与前面各层设计一致。

若某模块自检发现遗漏或冲突，必须先补齐或修正当前模块，再继续下一模块。

## 友好性自检

- [ ] 每个「模块变更」都能对应到**唯一**的代码编写类 skill（impl-facade-module、impl-client-module 等）。
- [ ] **架构设计**已包含 **应用架构** 与 **部署架构** 两部分；应用架构包含代码结构四列表格（**层**、**领域**、**包**、**职责**）及关键模块调用关系；adapter 入口已覆盖 Controller、定时任务、事件监听等外部触发方式；模块调用关系已明确区分命令类控制/调用（CommandController / CommandService）与查询类控制/调用（QueryController / QueryService），无法判断时已向用户确认；部署架构处理方式已让用户三选一，若选择不变则默认写明「部署架构不变，无新增应用/前端部署实例，复用现有部署架构」，若选择重新设计或调整则已展开应用/前端部署设计并明确实例数量。
- [ ] **Facade 层设计**已说明本次是否涉及通用基础类型与通用契约变化；若无变化，已明确写出「本次无 Facade 层变更」。
- [ ] **领域层设计**按**业务领域为二级目录**（如 4.2 用户、4.3 家庭、4.4 账本…），每个业务领域下包含**六个三级目录**：**领域模型**、**领域动作**、**领域规则**、**领域工厂**、**领域网关**、**领域事件**；各节内容符合上条（类图+表格、动作表+每动作一时序图、规则表、工厂方法表、网关方法表、事件表）；与模块变更清单中 domain 层描述一致。
- [ ] 每个聚合根已设计对应 **Factory 接口**，且 Factory 只包含 `create(...)` 与 `createByNum(...)` 两个方法；两个方法都用于构建领域对象；application 层不直接 new 领域对象、不直接调用领域对象静态 create 构建对象。
- [ ] 领域网关 Gateway 接口已定义在 domain 层，并明确由 infra 层实现 GatewayImpl。
- [ ] 所有聚合根与实体已具备**基本属性** id、num、create_no、update_no；聚合根已具备 Repository、Gateway、DomainEventPublisher 三类领域协作依赖属性；聚合根与领域实体中未出现 is_deleted/deleted/isDeleted 等软删除标记；均具备 **save**、**delete** 领域动作；**所有领域动作均包含操作人参数**（如 operatorId）。
- [ ] **基础设施层设计**已覆盖 Entity、Mapper、RepositoryImpl、FactoryImpl、GatewayImpl 与 common 能力；RepositoryImpl 不作为 application 查询入口；与数据库设计和 domain 接口一致；Spring Bean 注入统一使用 `@Resource`，未使用 `@Autowired`、构造器注入、Setter 注入或 `ApplicationContext` 手动获取；若无 infra 变化，已明确写出「本次无基础设施层变更」。
- [ ] **数据库设计**已包含表结构、**DDL 语句**（可直接执行），若涉及刷数则含 **DML 语句**；与领域对应关系及模块变更清单中 infra 层描述一致；**各表主键 `id` 为 `BIGINT` 且自增**；**`create_time`/`update_time` 命名且时间类型毫秒精度**（见 §8 / reference 数据库模块）。
- [ ] **代码分支命名**已填写：需求类为 `feature-YYYYMMDD-英文名`，BUG 修复类为 `hotfix-YYYYMMDD-英文名`；方案中写出本条对应的具体分支名。
- [ ] 实现顺序与六层依赖一致，不出现「先写 adapter 再写 domain」等逆依赖。
- [ ] 领域/聚合命名在各层一致（如 conversation、message、auth），便于各 skill 的包结构约定对齐。
- [ ] 若涉及 Agent，在 application 与 adapter 的变更中明确是 Agent 子包结构（config/hook/interceptor/service/tool、单一 AgentService）。
- [ ] **设计出的所有方法/入口都须有时序图**：领域动作、应用层每个 Service 方法、Adapter 层每个 HTTP 接口/定时任务/事件监听入口均配有时序图描述实现逻辑。
- [ ] **应用层设计**先**业务模块划分**（6.1），再**按业务模块为二级目录**，每模块下含 **Service 方法清单**与**方法时序逻辑**（**每个方法一张时序图**）；与领域层设计中的业务领域对应；application 中未注入/调用任何 Repository 或 Gateway，查询需求均通过对应领域 QueryService 提供；CommandService 写操作已设计 Redis 分布式锁；Spring Bean 注入统一使用 `@Resource`。
- [ ] **Adapter 层设计**覆盖 HTTP Controller、定时任务、事件监听等外部触发入口；HTTP 仅使用 **GET（查询）**与 **POST（增删改）**；Controller 入参使用 `*Param` 类，出参使用 `*VO` 类（除 `Result` 等全局包装外），若应用层返回 DTO 则说明 DTO → VO 转换步骤；每个接口/任务/监听入口均有时序图；定时任务说明 cron、幂等、并发与告警；事件监听说明 topic/tag/事件类型、幂等、重试/DLQ 与告警；与应用层、领域模块对应；Spring Bean 注入统一使用 `@Resource`。

## 使用完成后的最后一步：更新知识图谱

- **每次使用本技能完成交付后，最后一步须更新知识图谱**。根据本次产出（如新建/修改的技术方案文档），创建或更新 ontology 中的实体与关系（如 Document、Project、Task；`part_of`、`has_task` 等），或调用 **ontology** 技能、或向 `memory/ontology/graph.jsonl` 追加操作记录，使图谱与项目当前状态一致。详见 [ontology/SKILL.md](../ontology/SKILL.md)。

- 完整模板与「变更类型 → skill」映射表见 [implementation-layers.md](references/implementation-layers.md)。

## Reference

- **设计前问题清单**：[design-questions.md](references/design-questions.md)
- **应用架构设计经验**：[application-architecture-experience.md](references/application-architecture-experience.md)
- **部署架构设计经验**：[deployment-architecture-experience.md](references/deployment-architecture-experience.md)
- **门面层设计经验**：[facade-layer-design-experience.md](references/facade-layer-design-experience.md)
- **领域层设计经验**：[domain-layer-design-experience.md](references/domain-layer-design-experience.md)
- **领域模型设计规范**：[domain-model-design.md](references/domain-model-design.md)
- **基础设施层设计经验**：[infra-layer-design-experience.md](references/infra-layer-design-experience.md)
- **应用层设计经验**：[application-layer-design-experience.md](references/application-layer-design-experience.md)
- **适配层设计经验**：[adapter-layer-design-experience.md](references/adapter-layer-design-experience.md)
- **数据库设计经验**：[database-design-experience.md](references/database-design-experience.md)
- **分层实现参考**：[implementation-layers.md](references/implementation-layers.md)
