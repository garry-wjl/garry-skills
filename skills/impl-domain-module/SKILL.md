---
name: impl-domain-module
description: Implements the domain module in a six-layer Java project. Each business domain package may contain only four subpackages — factory, gateway, repository, valueobject — plus aggregate-root Java files at the package root; no query, dto, or other folders. Repository: save, findByNum, deleteByNum only. Non-command reads via gateway interfaces. Domain depends only on facade. Use when adding or modifying domain code.
---

# Implement Domain Module

**分类**：代码编写类

Guides implementing the **domain** module：领域层，承载领域模型与领域逻辑，定义仓储/工厂/网关接口及值对象。

**参考现有代码**：若仓库中已存在 domain 或该领域代码，实现前须先阅读现有包结构、实体与 Repository/Factory/Gateway 接口命名、值对象定义，在现有基础上扩展或修改，保持风格一致。

## When to Use

- 用户要求「实现 domain 模块」、「添加 XX 领域对象」、「定义 XX 仓储/工厂/网关接口」。
- 在六层结构中新增或修改 domain 层代码。

## 职责与依赖

- **职责**：定义领域实体（继承 facade 的 `DomainEntity`）、值对象（valueobject）、**仓储接口**（Repository）、**工厂接口**（Factory）、**网关接口**（Gateway）；实体内封装领域规则与行为（如 save、updateTitle），通过注入的 Repository、Gateway、DomainEventPublisher 与外部协作。Repository 是领域层内部持久化协作接口，仅供领域对象、领域工厂及 infra 实现使用，不对 application 层开放调用。
- **依赖**：**仅依赖 facade**。不依赖 application、adapter、client、infra；infra 实现 domain 定义的 Repository、Factory、Gateway 接口。

## 包结构与约定

### 规则 1：必须存在 common 子包与 DomainEventConstant（跨领域常量）

- **domain 下必须存在 `common` 子包**，用于存放**跨领域**的常量等，不与具体业务领域绑定。
- **common 包下须创建 DomainEventConstant 类**，用于存储领域事件类型常量；供实体发布 `DomainEventDTO` 时使用。
- **常量约定**：所有事件常量必须是 **`public static final String`** 类型，命名**严格遵循业务规则**（如 `USER_REGISTERED`、`FAMILY_CREATED`、`BOOK_DELETED` 等，与业务语义一致）；不设子目录，按需增加其他跨领域常量类。

### 规则 1.1：领域模型必须具有业务编码 num

- **每一个领域模型**（聚合根/根实体）**都必须存在名为 `num` 的业务编码属性**，用于唯一标识该领域对象在业务侧的编号；由对应领域的 Gateway 接口提供生成方法（如 `generateNum()`），在 save 的赋值阶段若 num 为空则调用网关生成。

### 规则 2：领域模型使用 Lombok @Getter / @Setter

- **领域模型**（根实体、包根事件载荷类、值对象等）**必须使用 Lombok 的 `@Getter` 与 `@Setter` 注解**，避免手写 getter/setter；根实体若需对部分字段只读，可仅在该类上使用 `@Getter`，或对具体字段单独使用 `@Getter`/`@Setter`。若 domain 模块未从父 POM 或 facade 传递获得 Lombok，须在 domain 的 pom 中显式添加 Lombok 依赖并配置注解处理器。

### 规则 2.1：值对象中的贫血模型必须使用完整 Lombok 注解

- **贫血模型值对象**（如 FamilyMember、BookMember、BookTransaction 等仅含业务属性、不继承 DomainEntity、放在各领域 **valueobject/** 下的类）**必须**使用以下 Lombok 注解组合，便于构造、序列化与 infra 映射：
  - **@Getter**
  - **@Setter**
  - **@Builder**
  - **@AllArgsConstructor**
  - **@NoArgsConstructor**
- 枚举或常量类（如 ClassifyStatus、FamilyMemberRole）不适用本条，仍按原有方式定义。

### 规则 3：其余子包按业务领域划分

- **除 common 外**，其余子包按**业务领域**（聚合/限界上下文）与 infra 对应，**一个领域一个子包**（如 conversation、message、task、user、report）。

#### 规则 3.0：业务领域子包下**仅允许四个目录**（硬约束）

- 每个业务领域包（如 `domain/user`、`domain/book`）下**只能**存在以下**四个子目录（子包）**：**`factory/`**、**`gateway/`**、**`repository/`**、**`valueobject/`**。
- **绝对禁止**在该业务领域包下创建**第五个**子目录或任何其他目录名，包括但不限于 **`query/`**、**`dto/`**、**`service/`**、**`entity/`**、**`port/`**、**`spi/`**、**`model/`**、**`support/`** 等。
- 聚合根 `.java` 文件、可选的**事件载荷**等仅含属性的类，放在**该领域包根**（与 `User.java` 同级），**不**单独建子文件夹。
- **跨领域**内容仍仅放在 **`domain/common`**（本规则不约束 common）。

- 每个领域子包下**逻辑职责**划分如下：
  - **根实体**：聚合根（如 `Conversation`、`Message`），放在领域包根下；使用 **@Getter / @Setter**；继承 facade 的 `DomainEntity` 后，**必须完整实现其全部抽象方法**（`domainValidate()`、`save(operatorId)`、`delete(operatorId)`），不得只实现部分或留空；**每次完成领域操作后都必须发送领域事件**（save、delete 及任何会改变状态或持久化的领域方法，在六步最后一步发送对应类型事件）；事件通过注入的 `DomainEventPublisher` 发送，事件类型使用 common 中的 **DomainEventConstant**。**每个聚合根**须将**仓储、网关、领域事件发布器**定义为该对象的**属性**；实体按需持有其业务行为所需的领域协作依赖；**聚合根与领域实体不得包含 is_deleted/deleted/isDeleted 等软删除标记**。聚合根必须**提供传入必填字段的构造方法**：构造方法传入构建该领域对象所必须的数据字段（含仓储、网关、领域事件发布器），**不包含由状态机控制的状态字段**——状态字段应通过**领域方法**进行初始化与流转。
  - **repository/**：仓储**接口**为**固定三方法契约**，**仅允许**以下签名（领域根类型 `R` 随聚合替换）：`void save(R aggregate)`、`R findByNum(String num)`、`void deleteByNum(String num)`。**禁止**在此接口上声明任何其他方法（包括但不限于 `build*By`、`findById`、`findByEmail`、列表查询、`new*ForCreate`、统计等）；**不得**扩张 Repository 接口。参数与返回值为 domain 实体，**不**出现 infra 类型。
  - **factory/**：工厂**接口**，**只能包含两个方法**：**create(必填字段…)**、**createByNum(String num)**；两个方法都用于构建领域对象；`create(...)` 根据属性构建新的领域对象，`createByNum` **必须**等价于委托 `repository.findByNum(num)` 按业务编码加载并构建既有领域对象；**禁止**定义 `createById(...)`、`rebuild(...)` 或其他任何工厂方法；返回 domain 实体；实现类在 infra。
  - **gateway/**：网关**接口**，至少包含**生成业务编码**的方法。**范围**：除上述三方法仓储外的**所有出站/读模型协作**均放在 **gateway/**（由 infra 实现），例如：第三方 HTTP、编号规则、`FamilyRoleGateway` 式权限、以及**按主键/邮箱/列表/分页等持久化读接口**（命名可为 `XxxReadGateway` / `XxxReadPort` 等，但**源文件必须位于 `gateway/` 包内**）。**禁止**为读能力单建 `query/` 等目录。
  - **valueobject/**：值对象分为两类——（1）**贫血模型值对象**（如 FamilyMember、BookMember、BookTransaction）：仅含业务属性，不继承 DomainEntity，**必须**使用 `@Getter`、`@Setter`、`@Builder`、`@AllArgsConstructor`、`@NoArgsConstructor`；（2）**枚举或常量**（如 ClassifyStatus、FamilyMemberRole）：按业务所需确定。供实体与 repository 使用，无业务逻辑，仅表达领域概念。
  - **事件载荷（原 dto）**：若需**仅含属性的**事件载荷类，放在**领域包根**下（如 `ConversationEventDTO.java` 与 `Conversation.java` 同级），**不得**使用 `dto` 子目录；或使用 `DomainEventDTO.data(this)`。

### 领域方法实现顺序（所有领域方法须严格按此顺序执行）

所有领域方法的实现逻辑顺序为：

1. **初始化领域对象**：调用 `initialize(operatorId)`。
2. **领域规则校验**：例如校验当前状态是否允许本操作（如状态必须为 ACTIVE 等）。
3. **赋值**：对传入参数赋值、状态自动流转等。**save 方法在赋值阶段必须**：（1）对领域模型中**值对象类型**的属性进行初始化（若为 null 则初始化为空集合或默认值）；（2）若 **num 为空**（null 或空白），则通过**网关接口**的生成方法（如 `xxxGateway.generateNum()`）生成业务编码并赋值。
4. **领域完整性校验**：调用 `domainValidate()` 方法。
5. **持久化对象**：调用仓储的 **save** 方法（见下方「持久化约定」）。
6. **发布领域事件**：使用 `DomainEventPublisher.send(DomainEventDTO.builder().type(常量).id(事件ID).data(当前对象或仅含属性的DTO).sender(operatorId).build())` 发送；**data** 可为当前实体 **this** 或**领域包根**下仅含属性的载荷类（由 `toEventDTO()` 等转换），**不得**放在子目录中。

**领域事件约定**：**每次完成领域操作后都必须发送领域事件**。即：所有会改变状态或持久化的领域方法（包括 `save`、`delete` 以及业务方法如 `close`、`join`、`updateTitle` 等），在六步顺序的最后一步都必须调用 `DomainEventPublisher.send(...)` 发送与本次操作对应的事件类型（如新建发 CREATED、删除发 DELETED、更新发 UPDATED 或业务语义常量）；不得省略或仅在部分操作后发送。**禁止**使用 `boolean wasNew`（或类似“是否新建”）判断来决定是否发事件——**每次执行 save 都必须发送领域事件**，不做“仅新建时发送”的分支。

**持久化约定**：除删除外，所有需要持久化的领域操作**仅调用仓储的 `save`**（不区分新增与更新）。聚合根 **`delete(operatorId)`** 在持久化删除步骤须调用仓储 **`deleteByNum(getNum())`**（与仓储三方法契约一致）。

### 注释约定（实现时必须遵守）

- **方法注释**：对 public 方法及对外暴露的包级方法编写 **Javadoc**，至少包含方法用途说明、`@param`（参数）、必要时 `@return`、`@throws` 或业务约束说明。
- **代码行注释**：在实现过程中对**非自解释的逻辑**添加行内或块注释，包括但不限于：六步顺序的每一步（如 `// 1. 初始化对象`）、业务规则含义、状态流转、为何仅调 save 不区分新增/更新等，使阅读者能快速理解意图与顺序。

### 各目录职能摘要

**common 子包（跨领域）**

| 目录/位置 | 职能 |
|-----------|------|
| **common** | 跨领域常量（如领域事件类型常量 DomainEventConstant）；直接放在 domain/common 下，无子目录。 |

**按业务领域划分的子包下各目录**

| 目录 | 职能 |
|------|------|
| **根实体** | 聚合根，放在领域包根下；继承 DomainEntity 并**完整实现其全部抽象方法**（domainValidate、save、delete）；每次领域操作完成后发送领域事件；将仓储、网关、领域事件发布器定义为聚合根属性；不得包含 is_deleted/deleted/isDeleted 等软删除标记；提供传入必填字段（含仓储、网关、领域事件发布器，不含状态）的构造方法；状态通过领域方法初始化与流转；封装领域行为。 |
| **repository/** | **仅**三方法：`save(R)`、`R findByNum(String)`、`void deleteByNum(String)`；**禁止**第四方法及任意其他查询/写入签名。 |
| **factory/** | 工厂**接口**；只能包含两个方法：`create(必填字段…)`、`createByNum(String num)`；两个方法都用于构建领域对象；`createByNum` 按业务编码委托 `repository.findByNum(num)`；禁止定义其他工厂方法；实现类在 infra。 |
| **gateway/** | 生成业务编码；第三方/权限/**非仓储三方法的读能力**（按 id、邮箱、列表等）均在此定义接口；实现类在 infra。 |
| **valueobject/** | 贫血模型值对象须用 **@Getter / @Setter / @Builder / @AllArgsConstructor / @NoArgsConstructor**；枚举或常量按业务确定；供实体与 repository 使用，不依赖基础设施。 |
| **（包根）** | 聚合根 Java 文件；可选事件载荷 POJO（与根实体同级，**无子目录**）。 |

## 规则

1. **common 子包必须存在**，且包含 **DomainEventConstant** 类；常量均为 `public static final String`，命名严格遵循业务规则；不在 common 中放领域专属实体或接口。
2. **领域工厂接口**只能包含两个方法：**create**、**createByNum**；两个方法都用于构建领域对象；`create(...)` 根据属性构建新的领域对象；**createByNum(String num)** 的实现**只能**转调对应仓储的 **`findByNum(num)`**，根据业务编码加载并构建既有领域对象，不得在工厂内绕过仓储做持久化；**禁止**定义 `createById(...)`、`rebuild(...)` 或其他任何工厂方法。
3. **领域网关接口**至少包含：生成业务编码方法；凡领域操作中依赖**第三方接口、工具方法**等实现的能力，均通过网关定义，由 infra 实现。
4. **领域仓储接口（硬约束）**：**只能**包含且必须包含 **`void save(R)`、`R findByNum(String num)`、`void deleteByNum(String num)`** 三个方法，方法名、数量与语义须一致；**不得**增加别名（如 `buildBookBy`/`findById`/`delete`）或任何额外方法。Repository 仅供领域对象、Factory 与 infra 实现内部使用，application 层不得注入或调用。存量接口若不一致，新代码以本 skill 为准并逐步收敛。
5. **值对象**：贫血模型值对象（仅含属性、不继承 DomainEntity）**必须**使用 `@Getter`、`@Setter`、`@Builder`、`@AllArgsConstructor`、`@NoArgsConstructor`；枚举或常量根据业务所需确定。
6. **领域子包**与 infra 领域一一对应，一领域一子包；每子包下**仅允许四个子目录**：**factory、gateway、repository、valueobject**（规则 3.0）；根实体等在包根；**仅定义接口与领域模型**，实现全部在 infra。
7. 新增聚合时，若 facade 中尚未有 `DomainEntity`、`DomainEventDTO`、`DomainEventPublisher`，先使用 **impl-facade-module** 技能创建。
8. 领域逻辑放在实体或领域服务中，不在 application 中写 if/else 领域规则；application 只做「通过 Factory 取实体 → 调实体方法」，禁止直接调用 Repository。
9. domain 中不出现 MyBatis、Spring、Redis 等基础设施注解；仅定义接口与领域逻辑，实现全部在 infra。
10. **根实体必须完整实现 DomainEntity 抽象方法**：每一个继承 `DomainEntity` 的领域对象**必须完整实现其全部抽象方法**（`domainValidate()`、`save(operatorId)`、`delete(operatorId)`），不得只实现部分或留空；实现逻辑按技术方案与六步顺序编写。
11. **根实体依赖与构造**：每个聚合根将**仓储、网关、领域事件发布器**定义为该对象的**属性**；聚合根与领域实体不得包含 `is_deleted`、`deleted`、`isDeleted` 等软删除标记；提供**传入必填字段的构造方法**，入参为构建该领域对象必须的数据字段（包括仓储、网关、领域事件发布器），**不包括由状态机控制的状态字段**；状态字段通过**领域方法**进行初始化与流转。
12. **领域方法实现顺序**：所有领域方法按顺序执行——**1. 初始化对象**（`this.initialize(operatorId)`）→ **2. 领域规则校验**（如 `Assert.isTrue(状态条件, "错误信息")`）→ **3. 赋值/修改状态** → **4. 领域完整性校验**（`this.validate()`）→ **5. 持久化对象**（`repository.save(this)`）→ **6. 发布领域事件**（`domainEventPublisher.send(DomainEventDTO.builder().type(常量).id(事件ID).data(this或DTO).sender(operatorId).build())`）。示例见 reference 中 close() 方法。
13. **每次领域操作完成后必须发送领域事件**：所有会改变状态或持久化的领域方法（含 `save`、`delete` 及业务方法如 `close`、`join` 等），在六步最后一步**都必须**发送与本次操作对应的领域事件，不得省略。**禁止 wasNew 式判断**：不得出现 `boolean wasNew` 或“仅新建时发事件”的逻辑，**每次执行 save 都发领域事件**。
14. **持久化约定**：除 delete 外，所有需要持久化的领域操作**仅调用仓储的 save 方法**（不区分新增与更新）。
15. **save 的赋值阶段**：对值对象类型属性做初始化；若 `num` 为空则通过网关生成业务编码并赋值。
16. **领域模型必有 num**：每个领域模型（聚合根）都必须具备 **num** 业务编码属性；对应 Gateway 提供生成方法，save 时若 num 为空则调用网关生成。
17. **领域模型注解**：根实体与包根下事件载荷类使用 Lombok **@Getter** 与 **@Setter**（若仅部分字段可写，可对字段或类单独使用）；**贫血模型值对象**须额外使用 **@Builder**、**@AllArgsConstructor**、**@NoArgsConstructor**。
18. **注释要求**：实现时须写**方法注释**（Javadoc：用途、@param、@return/@throws 等）与**代码行注释**（六步顺序、业务规则、非自解释逻辑的简要说明）。
19. **目录结构**：业务领域包下**不得**出现 `query/`、`dto/` 等除 rule 3.0 所列四目录外的子包；存量代码须迁移（读接口 → `gateway/`，DTO → 包根类）。

## 使用完成后的最后一步：更新知识图谱

- **每次使用本技能完成交付后，最后一步须更新知识图谱**。根据本次产出（如新增/修改的 domain 聚合、实体、仓储），创建或更新 ontology 中的实体与关系（如 Project、Task、Document；`part_of`、`has_task` 等），或调用 **ontology** 技能、或向 `memory/ontology/graph.jsonl` 追加操作记录，使图谱与项目当前状态一致。详见 [ontology/SKILL.md](../ontology/SKILL.md)。

## Reference

- 与本 Skill 打包的 Case（包结构、common 常量、Repository/Factory/Gateway 接口、值对象、领域实体写法）：[references/domain-interfaces.md](references/domain-interfaces.md) 和 [references/domain-package-structure.md](references/domain-package-structure.md)。
- **本模块可选第三方依赖**（pom 依赖选择）：[module-dependencies.md](../module-dependencies.md) 中「domain」小节。
