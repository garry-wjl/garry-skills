# Facade, Infrastructure, Application, Adapter, Database and Module Changes

## Application Layer Design

### Service Method Checklist

For each service (CommandService, QueryService, StreamService), specify:
- **Method signature**: Name, input type (ParamDTO class), return type (client DTO, void, String/Long business identifier)
- **Responsibility**: One-line business use case description
- **Input/Output**: Key fields or reference ParamDTO/DTO names
- **Repository constraint**: Application services MUST NOT inject or call domain Repository interfaces. Query requirements are provided by the corresponding domain QueryService.

### Method Sequence Logic

For each method, write execution steps (Mermaid sequenceDiagram or ordered list):
1. **Parameter validation**: Non-null, format, business rules
2. **Operation context**: Extract operatorId from context; permission check
3. **Query through QueryService when needed**: If the use case needs reads or validation data, call the corresponding domain's QueryService; do NOT inject/call Repository from application
4. **Build aggregate through Factory**: use only `Factory.create(...)` to build a new domain object from attributes, and `Factory.createByNum(...)` to build an existing domain object by business code; Factory must not contain `createById(...)`, `rebuild(...)`, or other methods; application must not directly `new` domain objects or call static `create` construction methods
5. **Invoke domain action or Gateway**: E.g., aggregate.record(...), aggregate.close(...), CategorySuggestGateway.suggest(...)
6. **Persist/Publish event**: performed inside domain actions through domain object dependencies; mark @Transactional on application method if needed
7. **Assemble return**: Construct client DTO or business identifier return

**Example (FamilyCommandService.createFamily sequence)**:

| Step | Description | Dependency |
|------|-------------|-----------|
| 1 | Validate: name, ownerId non-null | — |
| 2 | Operator: operatorId from context; verify consistency with ownerId or admin role | — |
| 3 | Verify: owner exists through UserQueryService | UserQueryService.existsByNum(ownerNum) |
| 4 | Create family aggregate through Factory: familyFactory.create(name) | FamilyFactory；create 仅接收用户创建家庭时填写的字段 |
| 5 | Invoke domain action: family.save(operatorId) | Family aggregate handles persistence/event internally |
| 6 | Return: family num or client DTO | String or client DTO |

---

## Adapter Layer (External Entry) Design

Adapter is the external trigger entry layer. It includes HTTP controllers, scheduled jobs, event/MQ listeners, and adapter-level configuration.

### Controller Interface Checklist

For each controller (CommandController, QueryController), specify:
- **HTTP method + path**: POST /api/xxx/command/create, GET /api/xxx/query/list
- **Input**: Request body type (Param), path/query params; auth required?
- **Output**: Result<VO>, Result<List<VO>>, etc.
- **Responsibility**: Which use case or Service method it calls

### Scheduled Job Checklist

For each scheduled job, specify:
- **Job name**: Clear business name
- **Trigger**: cron/fixed delay/manual trigger
- **Idempotency**: idempotent key or duplicate handling
- **Concurrency control**: single-node/multi-node lock strategy
- **Application call**: Which CommandService/QueryService method it calls
- **Failure handling**: retry, alert, compensation

### Event/MQ Listener Checklist

For each listener, specify:
- **Source**: MQ topic/tag, event type, subscription group
- **Payload**: message schema or event DTO
- **Idempotency**: message key, consumed log, duplicate handling
- **Retry/DLQ**: retry count, dead-letter queue, manual compensation
- **Application call**: Which CommandService/QueryService method it calls
- **Failure handling**: exception strategy and alerting

### Entry Sequence Logic

For each adapter entry (Mermaid sequenceDiagram or ordered list):
1. **Receive trigger**: HTTP request, scheduler tick, MQ/event message
2. **Authentication/context**: Validate JWT or build system operator/context when applicable
3. **Basic validation**: Non-null, format checks (or delegate to Service)
4. **Call application layer**: Call CommandService/QueryService with ParamDTO and operator/context
5. **Acknowledge/respond**: Return Result for HTTP, complete job, ACK/NACK message; exception → Result.fail/retry/DLQ/alert

**HTTP Method Convention**: Only **GET** (read) and **POST** (create/update/delete); NO PUT, DELETE, PATCH

**Example (POST /api/family/command/create sequence)**:

| Step | Description | Note |
|------|-------------|------|
| 1 | Receive CreateFamilyParam from request body; bind name, ownerId | @RequestBody |
| 2 | Auth: Validate JWT; extract userId; if ownerId != userId, check admin role | 401 if not logged in |
| 3 | Param validation: name, ownerId non-null | Return 400 if invalid |
| 4 | Call FamilyCommandService.createFamily(param, operatorId) | Pass current user as operatorId |
| 5 | Return Result.success(familyId) or Result.success(FamilyVO) | GlobalExceptionHandler maps exceptions to Result.fail |

---

## Database Design

### Mandatory Conventions

**Primary Key `id`**:
- Type: **BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY** (MySQL)
- Java mapping: **Long**
- **Never use string UUID as physical primary key** (except non-business tables under special sk conditions)

**Time Fields (`create_time` / `update_time`)**:
- **Column naming**: MUST be `create_time` and `update_time` (NOT `created_at`, `updated_at`)
- **Precision**: **Millisecond (3 decimal places)**
  - MySQL: `DATETIME(3)` or `TIMESTAMP(3)`
  - PostgreSQL: `TIMESTAMP(3) WITH TIME ZONE` or `TIMESTAMP(3)`
- **MySQL Example**:
  ```sql
  create_time DATETIME(3) NOT NULL DEFAULT CURRENT_TIMESTAMP(3),
  update_time DATETIME(3) NOT NULL DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3)
  ```

**Audit Fields**:
- `create_no`: Creator ID (VARCHAR)
- `update_no`: Last updater ID (VARCHAR)
- Unique key on `num` field: `UNIQUE KEY uk_xxx_num (num)`
- 软删除字段：如需软删除，可在数据库表与 infra Entity 中使用 `is_deleted INT (0 = 未删除, 1 = 已删除)` 和 `@TableLogic`；该字段不得出现在聚合根或领域实体中。

### Table Structure Template

| Table | Description | Main Fields | Indexes |
|-------|------------|------------|---------|
| conversation | Conversation 聚合根对应表 | **id(PK, BIGINT AI)**, num, create_no, update_no, title, status, **create_time(3)**, **update_time(3)**, is_deleted（仅 DB/Entity） | num(UK), status, create_no |
| message | Message 实体对应表 | **id(PK, BIGINT AI)**, num, create_no, update_no, conversation_id(BIGINT FK), content, sender_id, **create_time(3)**, **update_time(3)**, is_deleted（仅 DB/Entity） | conversation_id, num |

**Mapping**: `conversation` table ← Conversation AR; `message` table ← Message entity (logically belongs to Conversation)

### DDL and DML

- Generate **DDL statements** (CREATE TABLE, CREATE INDEX) that can be executed directly
- If data migration needed: generate **DML statements** (INSERT, UPDATE); note execution order and prerequisites
- Document data initialization, enum tables, audit/version tables if applicable

---

## 模块变更清单

| 层 | 变更项 | 对应 Skill |
|-------|--------------|------------------|
| facade | DomainEntity、DomainEventDTO、DomainEventPublisher、Result、CommonRequest 等通用契约 | impl-facade-module |
| client | 新增或修改 ParamDTO、DTO、VO、Result、API 契约、常量 | impl-client-module |
| domain | 聚合、实体、Repository/Factory/Gateway 接口、值对象、领域事件 | impl-domain-module |
| infra | Entity、Mapper、Repository/Factory/Gateway 实现、common/constant/event/util/client | impl-infra-module |
| application | CommandService、QueryService、StreamService、config/hook/interceptor/service/tool | impl-application-module |
| adapter | Controller、Scheduler/Job、Listener、config（BaseController、GlobalExceptionHandler、TokenFilter 等） | impl-adapter-module |

---

## 实现顺序与模块自检门禁

默认实现顺序：

1. **facade**（如有变化）
2. **client**
3. **domain**
4. **infra**
5. **application**
6. **adapter**

如果方案只涉及部分层，只列出涉及层的顺序。

每完成一个模块设计后，必须先完成该模块自检；自检通过后，才能进入下一模块。若自检发现遗漏、冲突或依赖不一致，必须先修正当前模块。

---

## Code Branch Naming

- **Feature**: `feature-YYYYMMDD-requirement-name-en`, e.g., `feature-20250319-family-invite`
- **Bugfix**: `hotfix-YYYYMMDD-bug-name-en`, e.g., `hotfix-20250319-login-timeout`

Plan **MUST specify** the branch name for this design (one plan = one branch name).

---

## API Contract Example (JSON)

**POST /xxx/command/create**
Request:
```json
{ "name": "string", "ownerId": "string" }
```
Response:
```json
{ "code": 200, "msg": "success", "data": "familyId123" }
```

**GET /xxx/query/list**
Response:
```json
{ "code": 200, "msg": "success", "rows": [ { "id": "...", "name": "..." } ] }
```
