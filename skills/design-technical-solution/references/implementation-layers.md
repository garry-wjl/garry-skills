# Application, Adapter, Database and Module Changes

## Application Layer Design

### Service Method Checklist

For each service (CommandService, QueryService, StreamService), specify:
- **Method signature**: Name, input type (Param or primitive), return type (Result<VO>, void, Long)
- **Responsibility**: One-line business use case description
- **Input/Output**: Key fields or reference Param/VO names

### Method Sequence Logic

For each method, write execution steps (Mermaid sequenceDiagram or ordered list):
1. **Parameter validation**: Non-null, format, business rules
2. **Operation context**: Extract operatorId from context; permission check
3. **Create/Load aggregate through Factory**: use `Factory.create(...)` to create a new domain object from attributes, and `Factory.createByNum(...)` to load/build an existing domain object by business code; application must not directly `new` domain objects or call static `create` construction methods
4. **Invoke domain action or Gateway**: E.g., aggregate.record(...), aggregate.close(...), CategorySuggestGateway.suggest(...)
5. **Persist/Publish event**: Repository.save, DomainEventPublisher.publish; mark @Transactional if needed
6. **Assemble return**: Construct VO, Result return

**Example (FamilyCommandService.createFamily sequence)**:

| Step | Description | Dependency |
|------|-------------|-----------|
| 1 | Validate: name, ownerId non-null | — |
| 2 | Operator: operatorId from context; verify consistency with ownerId or admin role | — |
| 3 | Verify: ownerId user exists | UserRepository.findById(ownerId) |
| 4 | Create family aggregate through Factory: familyFactory.create(name, ownerId) | FamilyFactory |
| 5 | Persist & publish event | FamilyRepository.save; DomainEventPublisher.publish(FAMILY_CREATED) |
| 6 | Return: family id or FamilyVO | Result<String> or Result<FamilyVO> |

---

## Adapter Layer (Controller) Design

### Controller Interface Checklist

For each controller (CommandController, QueryController), specify:
- **HTTP method + path**: POST /api/xxx/command/create, GET /api/xxx/query/list
- **Input**: Request body type (Param), path/query params; auth required?
- **Output**: Result<VO>, Result<List<VO>>, etc.
- **Responsibility**: Which use case or Service method it calls

### Interface Sequence Logic

For each HTTP endpoint (Mermaid sequenceDiagram or ordered list):
1. **Receive request**: Deserialize request body/params; bind Param
2. **Authentication & operator**: Validate JWT; extract current userId as operatorId; return 401 if not logged in
3. **Basic validation**: Non-null, format checks (or delegate to Service)
4. **Call application layer**: Call CommandService/QueryService with Param and operatorId
5. **Assemble response**: Wrap Service return in Result.success(data); exception → Result.fail(error_code) or global exception handler

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
- Logical delete: `is_deleted INT (0 = not deleted, 1 = deleted)` with `@TableLogic`

### Table Structure Template

| Table | Description | Main Fields | Indexes |
|-------|------------|------------|---------|
| conversation | Conversation AR | **id(PK, BIGINT AI)**, num, create_no, update_no, title, status, **create_time(3)**, **update_time(3)**, is_deleted | num(UK), status, create_no |
| message | Message Entity | **id(PK, BIGINT AI)**, num, create_no, update_no, conversation_id(BIGINT FK), content, sender_id, **create_time(3)**, **update_time(3)**, is_deleted | conversation_id, num |

**Mapping**: `conversation` table ← Conversation AR; `message` table ← Message entity (logically belongs to Conversation)

### DDL and DML

- Generate **DDL statements** (CREATE TABLE, CREATE INDEX) that can be executed directly
- If data migration needed: generate **DML statements** (INSERT, UPDATE); note execution order and prerequisites
- Document data initialization, enum tables, audit/version tables if applicable

---

## Module Changes Checklist

| Layer | Change Items | Corresponding Skill |
|-------|--------------|------------------|
| facade | DomainEventConstant, DomainEntity updates, DomainEventDTO | impl-facade-module |
| client | New Param, DTO, VO, Result, API contracts, constants | impl-client-module |
| domain | Aggregates, entities, Repository/Factory/Gateway **interfaces**, ValueObjects, domain events | impl-domain-module |
| infra | Entities, Mappers, Repository/Factory/Gateway **implementations**, common/constant/event/util/client | impl-infra-module |
| application | CommandService, QueryService, StreamService, config/hook/interceptor/service/tool | impl-application-module |
| adapter | Controller, listener, config (BaseController, GlobalExceptionHandler, TokenFilter) | impl-adapter-module |

---

## Implementation Order (Default)

1. **facade** (if changes needed)
2. **client**
3. **domain**
4. **infra**
5. **application**
6. **adapter**

If plan only involves specific layers, follow only the stated layers in plan's "implementation order".

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
