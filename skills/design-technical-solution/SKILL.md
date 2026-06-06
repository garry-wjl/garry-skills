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

- 问题应覆盖：业务边界与优先级、涉及模块/领域、与现有功能的关系、接口与数据约定、非功能要求（性能、安全、兼容）等。可参考 [design-questions.md](references/design-questions.md) 中的「技术方案设计前必问问题清单」。
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
   **仅包含「代码结构」部分**。代码结构需**根据业务情况**体现模块与包结构，使用**表格**表示，表格分为四列：**层**、**领域**、**包**、**职责**。  
   - **层**：六层之一（facade / client / domain / infra / application / adapter）。  
   - **领域**：业务领域或子域名（与各层领域子包一致，如 user、book、record）。  
   - **包**：该层下具体包路径或包名（如 `com.xxx.adapter.rest.record`）。  
   - **职责**：该包在本方案中的职责说明（一句话）。  
   不在此处展开其它架构描述（如六层依赖关系可在「目标与范围」或「其他」中简述）。

3. **领域模型设计（必选）**  
   遵循**先按业务分层级，再按业务领域分节、每领域下依次设计领域模型、领域规则、领域动作、领域事件**的原则。须按 DDD 规范设计，便于与 **impl-domain-module** 对齐落码，并满足**领域模型基本属性与必备动作**约定。  
   **文档目录结构**：  
   - **二级目录**：按**业务领域**划分（如 3.2 用户（user）、3.3 家庭（family）、3.4 账本（book）、3.5 账户（account））；业务领域与 3.1 业务层级划分一致，每个领域一节。  
   - **三级目录**：每个业务领域下固定四个子节——**领域模型**、**领域规则**、**领域动作**、**领域事件**。  
   - 示例：`3.2 用户（user）` → `3.2.1 领域模型`、`3.2.2 领域规则`、`3.2.3 领域动作`、`3.2.4 领域事件`；`3.3 家庭（family）` → `3.3.1 领域模型`、`3.3.2 领域规则`、`3.3.3 领域动作`、`3.3.4 领域事件`；依此类推。

   **各三级目录内容要求**（在每一业务领域下分别填写）：

   - **领域模型**（如 3.x.1）  
     - **基本属性（强制）**：聚合根与实体须有 **id**、**num**、**create_no**、**update_no**；值对象无需。  
     - **必备动作（强制）**：聚合根及可持久化实体须有 **save(operatorId)**、**delete(operatorId)**；**所有领域动作须带操作人参数**。  
     - **表现形式**：**领域类图 + 表格**。类图表达类名、属性（含类型）、关联；表格体现领域结构（对象、类型、属性、与其它对象关系）。  
     - **聚合与边界**：本领域聚合边界、聚合根及聚合内实体/值对象；一致性边界；跨聚合仅 ID 引用；可选 Repository/Factory/Gateway 方法清单或类图。

   - **领域规则**（如 3.x.2）  
     - 使用**表格**展示本领域的聚合内不变性与关键业务规则，列：聚合/对象、规则类型、规则描述、违反时表达。

   - **领域动作**（如 3.x.3）  
     - 使用**表格**列出本领域聚合根/实体的领域动作，列：聚合/实体、领域动作、职责、前置条件、后置条件/规则、领域事件。  
     - **每个领域动作须配有一张时序图**（Mermaid sequenceDiagram 或等价图），表达步骤顺序、Repository/Gateway 调用、事件发布时机。

   - **领域事件**（如 3.x.4）  
     - 使用**表格**列出本领域涉及的领域事件，列：事件名、触发时机、载荷要点、可订阅方/用途。

   在「领域模型设计」章首可保留 **3.1 业务层级划分**（表格列出层级与业务/子域说明），再按业务领域分节。详见 [domain-model-design.md](references/domain-model-design.md) 中「领域模型设计：领域类图与领域动作」及「**领域对象设计规范（专业版）**」小节。

4. **应用层设计（必选，当方案涉及 application 层时）**  
   与**领域模型设计**一致：**先根据业务层级/业务模块划分，再按具体模块编写**；模块与「领域模型设计」中的业务领域（如 3.2 用户、3.3 家庭、3.4 账本）一一对应，便于与 impl-application-module 落码对齐。  
   **文档目录结构**：  
   - **二级目录**：先 **4.1 业务模块划分**（表格列出本方案涉及的 application 模块，与 3.1 业务层级/领域对应），再按**业务模块**分节（如 4.2 用户（user）、4.3 家庭（family）、4.4 账本（book）、4.5 账户（account）；若认证单独成模块可为 4.2 认证（auth）等）。  
   - **三级目录**：每个业务模块下包含——**Service 方法清单**（本模块的 Service、方法签名、职责、入参/出参）、**方法时序逻辑**（本模块关键方法的执行步骤或时序图）。  
   - 示例：`4.2 用户（user）` → `4.2.1 Service 方法清单`、`4.2.2 方法时序逻辑`；`4.3 家庭（family）` → `4.3.1 Service 方法清单`、`4.3.2 方法时序逻辑`；跨模块或公共的 Service（如导入、统计、公众号绑定）可单独成节（如 4.x 导入（import）、4.x 统计（stats））。  
   - **方法时序逻辑**内容要求：**设计出的所有 Service 方法都须有时序图**描述实现逻辑；每个方法配一张 Mermaid sequenceDiagram（或等价图），表达步骤顺序（如 1. 参数校验 2. 解析操作人/鉴权 3. 加载聚合或 Repository 4. 调用领域动作或 Gateway 5. 持久化/发布事件 6. 组装返回）；跨 Service 调用、事务边界须标明。  
   详见 [implementation-layers.md](references/implementation-layers.md) 中「**应用层设计：Service 方法与时序逻辑**」小节。

5. **控制器/Adapter 层设计（必选，当方案涉及 adapter 层时）**  
   与**领域模型设计**一致：**先根据业务层级/业务模块划分，再按具体模块编写**；模块与「领域模型设计」及「应用层设计」中的业务模块对应，便于 **impl-adapter-module** 落码。  
   **接口约定（强制）**：  
   - **HTTP 方法仅允许两种**：**GET**（查询）、**POST**（增删改）；不得使用 PUT、DELETE、PATCH 等。  
   - **入参与返回值**：每个接口须用 **JSON** 方式描述入参和返回值（请求体/响应体的 JSON 示例或 schema），便于与 impl-client-module、impl-adapter-module 对齐。  
   **文档目录结构**：  
   - **二级目录**：先 **5.1 业务模块划分**（表格列出本方案涉及的 Controller/模块，与 4.x 应用层模块对应），再按**业务模块**分节（如 5.2 认证（auth）、5.3 家庭（family）等）。  
   - **三级目录**：每个业务模块下包含——**Controller 接口清单**（本模块的 GET/POST、路径、入参/出参 **JSON 描述**、职责）、**接口时序逻辑**（**每个接口**须配有时序图描述实现逻辑）。  
   - **接口时序逻辑**内容要求：**设计出的所有 API 接口都须有时序图**描述请求处理逻辑；每个接口配一张 Mermaid sequenceDiagram（Client→Controller→Service）；步骤含：接收请求/反序列化、鉴权/取 operatorId、参数校验、调用 Application Service、封装 Result 返回；异常与错误码映射可一并说明。  
   详见 [implementation-layers.md](references/implementation-layers.md) 中「**控制器设计：接口与时序逻辑**」小节。

6. **数据库设计（必选）**  
   须包含**数据库设计**模块，明确持久化方案，便于 **impl-infra-module** 落表与 Mapper。
   - **主键 id（强制）**：每张业务表的主键 **`id` 必须为 `BIGINT` 类型，且使用数据库自增**（如 MySQL：`BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY`）。**禁止**将主键 `id` 设计为 `VARCHAR/UUID` 等非整型自增方案（对外业务编号仍用 **`num`** 等独立字段 + 唯一索引）。领域层/Java 实体中与之对应的标识类型建议 **`Long`**。详见 [implementation-layers.md](references/implementation-layers.md)「数据库设计模块」。
   - **创建/更新时间（强制）**：须具备 **`create_time`（创建时间）**、**`update_time`（最后更新时间）** 两列；**禁止**使用 `created_at`、`updated_at` 等其它命名。时间类型**必须精确到毫秒**（如 MySQL：`DATETIME(3)`；PostgreSQL：`TIMESTAMP(3)`；DDL 与表结构描述须一致）。详见 [implementation-layers.md](references/implementation-layers.md)「数据库设计模块 → 时间字段约定」。
   - **表结构**：列出主要表名、字段（含类型、是否必填、索引）、主键与外键或逻辑关联；若有枚举/字典表一并列出。
   - **DDL 语句（强制）**：须**生成对应的 DDL 语句**（如 CREATE TABLE、CREATE INDEX 等），可直接执行建表；与表结构描述一致；**DDL 须体现上述 `id` 约定及 `create_time`/`update_time` 毫秒精度**。
   - **刷数/数据迁移**：若方案涉及**刷数**（历史数据迁移、初始化数据、枚举数据等），须**生成对应的 DML 语句**（如 INSERT、UPDATE）；注明执行顺序与前置条件。
   - **与领域对应**：标明表与领域聚合/实体的对应关系。
   - 可选：关键查询场景与索引建议、分表分库策略。详见 [implementation-layers.md](references/implementation-layers.md) 中「数据库设计模块」小节。

7. **模块变更清单**  
   **核心**：按层列出变更，且每条变更标明对应的**代码编写类** skill，便于实现时直接选用。
   - **facade**：新增/修改 DomainEntity、DomainEventDTO、DomainEventPublisher、CommonRequest、Result 等 → 使用 **impl-facade-module**。
   - **client**：新增/修改 Param、DTO、VO、Result、常量 → 使用 **impl-client-module**。
   - **domain**：新增/修改聚合、实体、Repository/Factory/Gateway 接口、值对象、领域事件常量；**须与本章「领域模型设计」中的领域类图、领域动作一致** → 使用 **impl-domain-module**。
   - **infra**：新增/修改 Entity、Mapper、Repository/Factory/Gateway 实现、common 下常量/异常/事件发布；**须与「数据库设计」中的表结构一致** → 使用 **impl-infra-module**。
   - **application**：新增/修改 CommandService、QueryService、StreamService 或 Agent 相关 config/hook/interceptor/service/tool → 使用 **impl-application-module**。
   - **adapter**：新增/修改 Controller、listener、或 config 下全局组件 → 使用 **impl-adapter-module**。

8. **代码分支命名（必选）**  
   技术方案中须**明确规定**本需求/修复对应的代码仓库分支名，便于实现时统一创建分支。  
   **命名规则**：  
   - **需求类（新功能、需求迭代）**：`feature-年月日-需求名称(英文)`。年月日为 8 位数字 `YYYYMMDD`，需求名称使用英文短横线连接（如多个词用 `-`），小写。示例：`feature-20250319-family-invite`、`feature-20250319-account-export`。  
   - **BUG 修复类**：`hotfix-年月日-bug名称(英文)`。年月日同上，bug 名称使用英文短横线连接，小写。示例：`hotfix-20250319-login-timeout`、`hotfix-20250319-amount-calculation`。  
   在方案中须写出**本条方案对应的具体分支名**（一条方案一个分支名），日期取方案文档日期或计划开发日期。

9. **实现顺序与依赖**  
   建议实现顺序（与技能使用顺序一致）：  
   **facade → client → domain → infra → application → adapter**。  
   若某需求只涉及部分层，则只列出涉及的层及顺序；跨层需求先写 domain/facade/client，再 infra，再 application，最后 adapter。

10. **接口与数据契约（可选）**  
   列出关键 API（路径、**仅 GET/POST**、入参与返回值 **JSON 描述**）及主要 DTO/VO 名称，便于与 **impl-client-module** 和 **impl-adapter-module** 对齐。

11. **其他（可选）**  
   外部依赖、配置项、非功能需求等，按需简短列出。

## 友好性自检

- [ ] 每个「模块变更」都能对应到**唯一**的代码编写类 skill（impl-facade-module、impl-client-module 等）。
- [ ] **架构设计**仅包含「代码结构」部分，且代码结构使用**四列表格**（**层**、**领域**、**包**、**职责**）体现模块与包结构，并根据业务情况填写。
- [ ] **领域模型设计**按**业务领域为二级目录**（如 3.2 用户、3.3 家庭、3.4 账本…），每个业务领域下包含**四个三级目录**：**领域模型**、**领域规则**、**领域动作**、**领域事件**；各节内容符合上条（类图+表格、规则表、动作表+每动作一时序图、事件表）；与模块变更清单中 domain 层描述一致。
- [ ] 所有聚合根与实体已具备**基本属性** id、num、create_no、update_no；均具备 **save**、**delete** 领域动作；**所有领域动作均包含操作人参数**（如 operatorId）。
- [ ] **数据库设计**已包含表结构、**DDL 语句**（可直接执行），若涉及刷数则含 **DML 语句**；与领域对应关系及模块变更清单中 infra 层描述一致；**各表主键 `id` 为 `BIGINT` 且自增**；**`create_time`/`update_time` 命名且时间类型毫秒精度**（见 §6 / reference 数据库模块）。
- [ ] **代码分支命名**已填写：需求类为 `feature-YYYYMMDD-英文名`，BUG 修复类为 `hotfix-YYYYMMDD-英文名`；方案中写出本条对应的具体分支名。
- [ ] 实现顺序与六层依赖一致，不出现「先写 adapter 再写 domain」等逆依赖。
- [ ] 领域/聚合命名在各层一致（如 conversation、message、auth），便于各 skill 的包结构约定对齐。
- [ ] 若涉及 Agent，在 application 与 adapter 的变更中明确是 Agent 子包结构（config/hook/interceptor/service/tool、单一 AgentService）。
- [ ] **设计出的所有方法都须有时序图**：领域动作（已有）、**应用层每个 Service 方法**、**控制器每个 API 接口**均配有时序图描述实现逻辑。
- [ ] **应用层设计**先**业务模块划分**（4.1），再**按业务模块为二级目录**，每模块下含 **Service 方法清单**与**方法时序逻辑**（**每个方法一张时序图**）；与领域模型设计中的业务领域对应。
- [ ] **控制器/Adapter 层设计**仅使用 **GET（查询）**与 **POST（增删改）**；每个接口的入参与返回值用 **JSON** 描述；每模块下 **每个接口一张时序图**；与应用层、领域模块对应。

## 使用完成后的最后一步：更新知识图谱

- **每次使用本技能完成交付后，最后一步须更新知识图谱**。根据本次产出（如新建/修改的技术方案文档），创建或更新 ontology 中的实体与关系（如 Document、Project、Task；`part_of`、`has_task` 等），或调用 **ontology** 技能、或向 `memory/ontology/graph.jsonl` 追加操作记录，使图谱与项目当前状态一致。详见 [ontology/SKILL.md](../ontology/SKILL.md)。

- 完整模板与「变更类型 → skill」映射表见 [implementation-layers.md](references/implementation-layers.md)。

## Reference

- **Pre-design Questions**: [design-questions.md](references/design-questions.md)
- **Domain Model Design Guide**: [domain-model-design.md](references/domain-model-design.md)
- **Implementation Layers (Application, Adapter, Database)**: [implementation-layers.md](references/implementation-layers.md)
