---
name: impl-application-module
description: Implements the application module (application services, use cases, transaction boundaries) in a six-layer Java project. Application depends on client, domain, and infra; orchestrates domain objects and delegates persistence to infra. Use when adding or modifying application services, use cases, or transaction boundaries.
---

# Implement Application Module

**分类**：代码编写类

Guides implementing the **application** module：应用服务层，编排用例、划定事务边界。

**参考现有代码**：若仓库中已存在 application 或该领域 Service（CommandService、QueryService 等），实现前须先阅读现有包结构、Service 命名与编排方式，在现有基础上扩展或修改，保持风格一致。

## When to Use

- 用户要求「实现 application 模块」、「写 XX 服务」、「添加 XX 用例」。
- 在六层结构中新增或修改 application 层代码。

## 职责与依赖

- **职责**：编排用例（如创建会话、发送消息、登录）；**写操作**通过 domain 的 Factory 创建或加载领域对象，调用领域对象方法（save、updateTitle、delete 等）；**读操作**可注入 infra 的 Mapper 做只读查询并转 client VO，或通过 domain 的 Repository 接口；需要事务时在 application 方法上加 `@Transactional`；不包含领域规则，领域规则在 domain 中。
- **依赖**：**仅依赖 client、domain、infra**。注入 domain 的 Factory、Repository 等接口（由 infra 实现），以及 client 的 DTO/Param/VO；读场景可注入 infra 的 Mapper；不直接依赖 adapter 或 facade（除 facade 中通用类型若被 domain 使用则经 domain 间接依赖）。

## 包结构与约定

### 规则 1：按业务领域/能力划分子包

- **application 下按业务领域/能力划分子包**，与 domain 聚合或用例边界对应（如 conversation、message、auth、agent）。
- 若存在**跨多领域**的通用应用服务或组件，可增设 **common** 子包统一放置；否则不必强制 common。

### 规则 2：领域子包内结构——非 Agent 与 Agent 二选一

- **非 Agent 开发**：领域子包下**只有两个 Service**——一个**写操作** Service、一个**读操作** Service。
  - **\*CommandService**（写）：通过 domain Factory 创建或加载实体，调用 `entity.save(operatorId)`、`entity.updateTitle(...)` 等；事务边界在 application 方法上（`@Transactional(rollbackFor = Exception.class)`）。流式写（如 SSE）的编排也可放在此 Service 或单独与 adapter 约定。
  - **\*QueryService**（读）：注入 infra 的 Mapper 做只读查询并转 client VO，或通过 domain Repository 查询。
- **Agent 开发**：在子包内再分子目录，分离配置、钩子、拦截器、核心服务、工具：
  - **config/**：Agent 应用层配置类（如 ReactAgentConfig）。
  - **hook/**：Agent 各阶段钩子（如 ReportPlanAgentHook）。
  - **interceptor/**：拦截器（如模型调用前后拦截）。
  - **service/**：**仅放一个 AgentService**（如 ReactAgentService），作为该 Agent 核心编排入口。
  - **tool/**：Agent 可执行工具（如 ExecutionTools）。

### 各目录职能摘要

**非 Agent 领域子包下（仅两个 Service）**

| 类型 | 职能 |
|------|------|
| **\*CommandService** | 写用例编排；注入 domain Factory（及必要时 Repository）；通过 Factory 创建/加载实体后调用实体行为；方法上加 @Transactional。 |
| **\*QueryService** | 读用例编排；注入 infra Mapper 或 domain Repository；只读查询，转 client VO 返回。 |

**Agent 领域子包下（config / hook / interceptor / service / tool）**

| 目录 | 职能 |
|------|------|
| **config/** | Agent 应用层配置类（如 Agent 配置 Bean）。 |
| **hook/** | Agent 各阶段钩子。 |
| **interceptor/** | 模型调用前后等拦截器。 |
| **service/** | **仅一个 AgentService**，作为该 Agent 核心编排入口。 |
| **tool/** | Agent 可执行工具类。 |

**common 子包（可选）**

| 目录 | 职能 |
|------|------|
| **common** | 跨多领域的通用应用服务或组件；无此需求可不建。 |

## 规则

1. **写用例**：先通过 **domain 的 Factory** 创建或加载领域对象，再调用领域对象行为（save、updateTitle、delete 等）；不在 application 中写领域规则。
2. **读用例**：可注入 infra 的 Mapper 做只读查询并转 client VO，或通过 domain Repository；不在此做写操作。
3. 需要事务时在 application 方法上加 `@Transactional(rollbackFor = Exception.class)`，不在 domain 或 infra 单方法上随意加事务。
4. 不注入 adapter 或 HTTP 相关类；操作人 ID 等由 adapter 传入 application 方法参数。

## 使用完成后的最后一步：更新知识图谱

- **每次使用本技能完成交付后，最后一步须更新知识图谱**。根据本次产出（如新增/修改的 application 用例、服务），创建或更新 ontology 中的实体与关系（如 Project、Task、Document；`part_of`、`has_task` 等），或调用 **ontology** 技能、或向 `memory/ontology/graph.jsonl` 追加操作记录，使图谱与项目当前状态一致。详见 [ontology/SKILL.md](../ontology/SKILL.md)。

## Reference

- 与本 Skill 打包的 Case（CommandService 编排 domain Factory 与实体方法、事务边界；QueryService 读库转 VO）：[reference.md](reference.md)。
- **本模块可选第三方依赖**（pom 依赖选择）：[module-dependencies.md](../module-dependencies.md) 中「application」小节。
