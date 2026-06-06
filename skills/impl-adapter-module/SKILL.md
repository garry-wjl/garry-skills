---
name: impl-adapter-module
description: Implements the adapter module (HTTP controllers, event consumers) in a six-layer Java project. Adapter only depends on application and client; delegates all logic to application layer. Use when adding REST APIs, MQ/event consumers, or adapter config in the adapter layer.
---

# Implement Adapter Module

**分类**：代码编写类

Guides implementing the **adapter** module：入站适配层，包含 HTTP 控制器与 MQ/本地事件消费者。

**参考现有代码**：若仓库中已存在 adapter 或该领域 Controller、listener、config，实现前须先阅读现有包结构、请求路径与入参出参约定，在现有基础上扩展或修改，保持与仓库一致。

## When to Use

- 用户要求「实现 adapter 模块」、「添加 XX 接口」、「写 XX 的 Controller」或「MQ/事件消费者」。
- 在六层结构中新增或修改 adapter 层代码。

## 职责与依赖

- **职责**：接收 HTTP 请求或事件，解析参数后调用 **application** 层，将返回结果封装为统一响应（如 client 的 `Result<T>`）；不写业务逻辑。
- **依赖**：**仅依赖 application 与 client**。不在 adapter 中直接引用 domain、infra、facade 的业务类型；当前用户 ID 等通过 request 属性（由认证过滤器/拦截器写入）或 BaseController 提供，避免 adapter 依赖 domain。

## 包结构与约定

### 规则 1：必须存在 config 子包，默认内置三类

- **adapter 下必须存在 `config` 子包**，用于存放**跨领域/全局**的控制器基类、异常处理、过滤器、Web/认证相关配置等，不与具体领域绑定。
- **config 默认内置以下三个类**（实现仅为可编译通过的框架代码 + TODO 标记，无需预填用户业务逻辑）：
  - **BaseController**：所有 Controller 的基类，提供 `getCurrentUserId()`、`isLogin()` 等（从 request 属性读取，与认证过滤器约定一致）。
  - **GlobalExceptionHandler**：全局异常处理，统一返回 `Result.fail(msg)`；兜底处理 `Exception`，其余异常类型由项目按需补充。
  - **TokenValidationFilter**：认证过滤器（如 Token 校验）；放行请求或校验后写入 request 属性，由项目在 TODO 处补充解析/校验逻辑与白名单。
- 其他 Web/HTTP 相关配置 Bean 可按需放在 config 下。

### 规则 2：其余子包按领域划分，启动类在 adapter 根目录

- **除 config 外**，其余子包按**领域**与 application/domain 对应，**一个领域一个子包**（如 conversation、message、auth、agent、resource）。
- 每个领域子包下可包含以下目录（按需）：
  - **controller/**：该领域的 HTTP 控制器。
    - 若该领域区分读/写（与 application 的 CommandService、QueryService 对应）：包内为 **\*CommandController**（写，POST）、**\*QueryController**（读，GET）。
    - 若该领域入口单一：可仅一个 **\*Controller**（如 AuthController、AgentController）。
  - **listener/**：**按需**存在。当该领域需要监听事件（如 MQ 消息、Spring 应用事件、领域事件）时，在该领域下增加 **listener/** 子包，存放事件监听类；收到事件后解析并调用 application 层方法，不写业务逻辑。
- **应用的启动类**（Spring Boot `@SpringBootApplication` 主类）放在 **adapter 的根目录**下（即 `adapter` 包根，与 config、各领域子包平级），例如 `adapter/<ApplicationName>Application.java`。

### 各目录职能摘要

**config 子包下（跨领域/全局）**

| 类型/类 | 职能 |
|---------|------|
| **BaseController** | 控制器基类；提供 getCurrentUserId() 等，从 request 属性读取（与认证拦截器约定一致）。 |
| **GlobalExceptionHandler** | 全局异常处理；统一封装为 Result.fail(msg)。 |
| **Filter / Interceptor** | 认证、鉴权、Token 校验等；将当前用户 ID 等写入 request 属性。 |
| **其他 config** | Web/HTTP 相关配置 Bean。 |

**按领域划分的子包下**

| 目录 | 职能 |
|------|------|
| **controller/** | 该领域的 HTTP 控制器；*CommandController（写、POST）、*QueryController（读、GET）；或单一 *Controller。入参为 client 的 Param/DTO + @Valid；只调 application 一层；返回 Result.ok(data) 或 Result.fail(msg)。 |
| **listener/** | 按需。该领域的事件监听者（MQ 消费者、@EventListener 等）；收到事件后解析并调 application 一层，不写业务逻辑。 |

## 规则

1. 每个接口方法：参数校验（Param + @Valid）→ 调 application 一层方法 → 返回 `Result.ok(data)` 或 `Result.fail(msg)`。
2. 不在 controller 或事件消费者中写 if/else 业务分支；复杂分支放在 application 或 domain。
3. 新 Controller 继承 **BaseController**（若项目有），通过 `getCurrentUserId()` 获取操作人并传入 application；否则通过 request 或 application 提供的上下文获取操作人。
4. 接口约定：**GET** 表示查询，**POST** 表示写操作（与项目 API 约定一致）。

## 使用完成后的最后一步：更新知识图谱

- **每次使用本技能完成交付后，最后一步须更新知识图谱**。根据本次产出（如新增/修改的 adapter 接口、控制器），创建或更新 ontology 中的实体与关系（如 Project、Task、Document；`part_of`、`has_task` 等），或调用 **ontology** 技能、或向 `memory/ontology/graph.jsonl` 追加操作记录，使图谱与项目当前状态一致。详见 [ontology/SKILL.md](../ontology/SKILL.md)。

## Reference

- 与本 Skill 打包的 Case（包结构、config 默认内置 BaseController/GlobalExceptionHandler/TokenValidationFilter、CommandController、QueryController）：[references/adapter-config-and-controllers.md](references/adapter-config-and-controllers.md) 和 [references/adapter-package-structure.md](references/adapter-package-structure.md)。
- **本模块可选第三方依赖**（pom 依赖选择）：[module-dependencies.md](../module-dependencies.md) 中「adapter」小节。
