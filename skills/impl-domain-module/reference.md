# Reference: Domain 层 Case

以下 Case 与 Skill 打包，供实现 domain 时参考。将 `BASE_PACKAGE` 替换为项目包名。domain 仅依赖 facade；实体继承 DomainEntity，实现 domainValidate/save/delete，通过 DomainEventPublisher 发事件。

**约定**：领域模型（根实体、包根载荷类、valueobject 等）使用 Lombok **@Getter** 与 **@Setter**；实现时须写**方法注释**（Javadoc）与**代码行注释**（六步顺序、业务规则等），见下方示例。

---

## 包结构约定

- **必须存在 common 子包**，且创建 **DomainEventConstant** 类；常量均为 `public static final String`，命名严格遵循业务规则；直接放在 `domain/common` 下，无子目录。
- **其余按业务领域划分子包**，一领域一子包；**每个业务领域包下仅允许四个子目录**：`factory/`、`gateway/`、`repository/`、`valueobject/`（**禁止** `query/`、`dto/` 等）；聚合根与可选事件载荷 `.java` 放在**领域包根**。
  - **repository/** — 仓储接口（**仅** save、findByNum、deleteByNum）
  - **factory/** — 工厂接口（至少：create(必填字段)、createByNum(num)）
  - **gateway/** — 网关接口：生成业务编码；第三方、权限、**非仓储读的查询**（如按邮箱、主键、列表）等均定义于此包
  - **valueobject/** — 值对象（贫血模型或枚举/常量）

示例目录树：
```
domain/
├── common/
│   └── DomainEventConstant.java
├── conversation/
│   ├── Conversation.java           ← 根实体
│   ├── ConversationEventDTO.java   ← 可选；事件载荷，与根实体同级，不建 dto 子目录
│   ├── repository/
│   ├── factory/
│   ├── gateway/
│   └── valueobject/
├── message/
│   ├── Message.java
│   ├── repository/
│   ├── factory/
│   ├── gateway/
│   └── valueobject/
├── report/
│   ├── Report.java
│   ├── repository/
│   ├── factory/
│   ├── gateway/
│   └── valueobject/
├── task/
│   ├── Task.java
│   ├── repository/
│   ├── factory/
│   ├── gateway/
│   └── valueobject/
└── user/
    ├── User.java
    ├── repository/
    ├── factory/
    ├── gateway/
    └── valueobject/
```

---

## Case 0: common 子包（DomainEventConstant）

**domain/common** 下须创建 **DomainEventConstant**，存储领域事件类型常量；供实体发布 `DomainEventDTO` 时使用。**约定**：所有常量均为 **`public static final String`**，命名**严格遵循业务规则**（与业务语义一致）。

路径：`domain/src/main/java/<BASE_PACKAGE_PATH>/domain/common/DomainEventConstant.java`

```java
package BASE_PACKAGE.domain.common;

/**
 * 领域事件类型常量，供实体发布 DomainEventDTO 时使用；命名严格遵循业务规则。
 */
public class DomainEventConstant {

    public static final String CONVERSATION_SAVE = "CONVERSATION_SAVE";
    public static final String CONVERSATION_CLOSE = "CONVERSATION_CLOSE";
    public static final String CONVERSATION_TITLE_UPDATE = "CONVERSATION_TITLE_UPDATE";
    public static final String CONVERSATION_DELETE = "CONVERSATION_DELETE";

    public static final String MESSAGE_SAVE = "MESSAGE_SAVE";
    public static final String MESSAGE_START = "MESSAGE_START";
    public static final String MESSAGE_COMPLETE = "MESSAGE_COMPLETE";
    public static final String MESSAGE_FAIL = "MESSAGE_FAIL";
    public static final String MESSAGE_CONTENT_UPDATE = "MESSAGE_CONTENT_UPDATE";
    public static final String MESSAGE_STATUS_UPDATE = "MESSAGE_STATUS_UPDATE";
    public static final String MESSAGE_DELETE = "MESSAGE_DELETE";
}
```

---

## Case 1: 仓储接口（domain 定义，infra 实现）

仓储接口**必须且只能**包含以下三个方法（名称与签名固定）：**`save`**、**`findByNum`**、**`deleteByNum`**。禁止第四方法及任意 `build*By`、`findById`、`delete` 等别名。

路径：`domain/src/main/java/<BASE_PACKAGE_PATH>/domain/conversation/repository/ConversationRepository.java`

```java
package BASE_PACKAGE.domain.conversation.repository;

import BASE_PACKAGE.domain.conversation.Conversation;

public interface ConversationRepository {
    void save(Conversation conversation);
    Conversation findByNum(String num);
    void deleteByNum(String num);
}
```

---

## Case 2: 工厂接口（domain 定义，infra 实现）

工厂接口**至少**包含两个方法：**create**、**createByNum**。**createByNum(String num)** 在 infra 实现中**必须**仅委托对应仓储的 **`findByNum(num)`**，不得在该方法内扩展其他仓储调用。其余方法结合技术方案按需创建。

路径：`domain/src/main/java/<BASE_PACKAGE_PATH>/domain/conversation/factory/ConversationFactory.java`

```java
package BASE_PACKAGE.domain.conversation.factory;

import BASE_PACKAGE.domain.conversation.Conversation;

public interface ConversationFactory {
    /** 根据必填字段（如对话标题）构建领域对象 */
    Conversation create(String title);
    /** 根据业务编码构建领域对象 */
    Conversation createByNum(String num);
}
```

---

## Case 3: 网关接口（domain 定义，infra 实现）

网关接口**至少**包含**生成业务编码**的方法；其他方法根据业务决定。**网关的定义**：在领域操作中，凡需通过**第三方接口、工具方法**等实现的能力，均通过网关暴露，由 infra 实现。

路径：`domain/src/main/java/<BASE_PACKAGE_PATH>/domain/conversation/gateway/ConversationGateway.java`

```java
package BASE_PACKAGE.domain.conversation.gateway;

public interface ConversationGateway {
    /** 生成业务编码（至少包含的方法） */
    String generateConversationId();
}
```

调用外部服务示例（如 report 的 KnowledgeBaseClient）：路径 `domain/report/gateway/KnowledgeBaseClient.java`

```java
package BASE_PACKAGE.domain.report.gateway;

import BASE_PACKAGE.domain.report.ReportPlan;
import java.util.Optional;

/** 通过第三方/工具实现的能力放在网关中定义 */
public interface KnowledgeBaseClient {
    Optional<ReportPlan> queryPlan(String userInput);
}
```

---

## Case 4: valueobject（值对象：枚举或常量）

领域中的**值对象均使用枚举或常量**，根据业务所需确定。

**枚举示例** — 路径：`domain/conversation/valueobject/ConversationStatus.java`

```java
package BASE_PACKAGE.domain.conversation.valueobject;

public enum ConversationStatus {
    ACTIVE("ACTIVE"),
    CLOSED("CLOSED"),
    DELETED("DELETED");

    private final String value;
    ConversationStatus(String value) { this.value = value; }
    public String getValue() { return value; }
}
```

**常量类示例**（当不宜用枚举时）— 路径：`domain/conversation/valueobject/ConversationType.java`

```java
package BASE_PACKAGE.domain.conversation.valueobject;

public class ConversationType {
    public static final String CHAT = "CHAT";
    public static final String REPORT = "REPORT";
    private ConversationType() {}
}
```

---

## Case 5a: 事件载荷类（可选，与根实体同级）

发布领域事件时，**data** 可使用当前实体 **this**（如示例 `close()` 中），也可使用**仅含属性的载荷类**。该类必须放在**领域包根**（与 `Conversation.java` 同级），**禁止**使用 `dto/` 子目录；仅包含可序列化属性（不含仓储、网关等依赖），实体中提供 `toEventDTO()` 等转换方法。

路径示例：`domain/conversation/ConversationEventDTO.java`（使用 @Getter/@Setter）

```java
package BASE_PACKAGE.domain.conversation;

import lombok.Getter;
import lombok.Setter;

/**
 * 会话领域事件载荷 DTO，仅含属性，供 DomainEventDTO.data 使用。
 */
@Getter
@Setter
public class ConversationEventDTO {
    private String id;
    private String num;
    private String title;
    private String status;
}
```

实体内提供转换方法，例如：`ConversationEventDTO toEventDTO() { ... }`，在发布事件时调用 `domainEventPublisher.send(DomainEventDTO.builder().type(DomainEventConstant.XXX).data(toEventDTO()).build())`。

---

## Case 5: 领域实体（继承 DomainEntity，领域方法六步顺序，持久化仅 save）

路径：`domain/src/main/java/<BASE_PACKAGE_PATH>/domain/conversation/Conversation.java`（要点）

**（0）领域模型注解与注释**  
根实体类上使用 Lombok **@Getter**、**@Setter**；所有 public 领域方法写 Javadoc，六步顺序每步用行注释标出（见下方 close 示例）。

**（1）继承与抽象方法**  
继承 facade 的 `DomainEntity` 后，**必须完整实现其全部抽象方法**：`domainValidate()`、`save(operatorId)`、`delete(operatorId)`，不得只实现部分或留空。**每次完成领域操作后都必须发送领域事件**（save、delete 及任何业务方法在六步最后一步发送对应类型事件）。

**（2）仓储、网关为属性**  
每个领域对象将**仓储、网关**（以及如需发事件则含 `DomainEventPublisher`）定义为该对象的**属性**，由 infra 的 Factory 在创建时通过构造方法传入。

**（3）传入必填字段的构造方法**  
提供**传入必填字段的构造方法**，入参为构建该领域对象必须的数据字段（包括仓储、网关、事件发布器），**不包括由状态机控制的状态字段**；状态通过**领域方法**进行初始化与流转。

**（4）领域方法实现顺序（所有领域方法须严格按此顺序）**

1. **初始化对象**：调用 `this.initialize(operatorId)`。
2. **领域规则校验**：例如当前状态是否允许本操作（如 `Assert.isTrue(ConversationStatus.ACTIVE.equals(this.status), "只能关闭状态为ACTIVE的对话")`）。
3. **赋值/修改状态**：对传入参数赋值、状态流转等（如 `this.status = ConversationStatus.CLOSED`）。
4. **领域完整性校验**：调用 `this.validate()`（内部会调 `domainValidate()`）。
5. **持久化对象**：调用仓储的 **save** 方法（`conversationRepository.save(this)`）。**约定**：除 **delete** 外，所有需要持久化的领域操作**仅调用仓储的 save 方法**。
6. **发布领域事件**：调用 `domainEventPublisher.send(...)`；**每次领域操作完成后都必须发送**，不得省略。**data** 可为当前实体 **this** 或仅含属性的 DTO。

**（5）领域方法实现示例（close 关闭对话）**

以下为严格按六步顺序实现并带**方法注释 + 代码行注释**的示例，可直接作为模板：

```java
/**
 * 关闭对话：将状态置为 CLOSED 并持久化、发布领域事件。
 *
 * @param operatorId 操作人ID，用于审计与事件 sender
 */
public void close(String operatorId) {
    // 1. 初始化领域对象（更新 updateNo、updateTime 等）
    this.initialize(operatorId);

    // 2. 领域规则校验：仅 ACTIVE 状态允许关闭
    Assert.isTrue(ConversationStatus.ACTIVE.equals(this.status), "只能关闭状态为ACTIVE的对话");

    // 3. 赋值/状态流转
    this.status = ConversationStatus.CLOSED;

    // 4. 领域完整性校验（含 domainValidate）
    this.validate();

    // 5. 持久化对象（约定：仅调 save，不区分新增/更新）
    conversationRepository.save(this);

    // 6. 发布领域事件
    domainEventPublisher.send(DomainEventDTO.builder()
            .type(DomainEventConstant.CONVERSATION_CLOSE)
            .id(IdUtil.getSnowflakeNextIdStr())
            .data(this)
            .sender(operatorId)
            .build());
}
```

说明：**data** 此处使用当前实体 **this**；若需仅含属性的快照可改为 `data(toEventDTO())`，载荷类定义在**领域包根**。事件 **id** 可使用网关或工具生成（如 `IdUtil.getSnowflakeNextIdStr()`），**sender** 传 `operatorId`。

**（6）根实体类注解与注释约定**

- 根实体类上使用 Lombok **@Getter**、**@Setter**（对 final 的仓储/网关字段可仅用 @Getter，或保持 @Setter 由构造注入）。
- 每个 public 领域方法应有 Javadoc（用途、@param、必要时 @return/@throws）；六步顺序的每一步用行注释标出（如 `// 1. 初始化对象`），便于核对与维护。

**（7）其他示例要点（伪代码，含注释）**

```java
/** 持久化当前实体；新建与更新统一走 save。完成后必须发送领域事件（新建发 CREATED，更新可不发或发 UPDATED）。 */
@Override
public void save(String operatorId) {
    // 1～4 步省略（同前）
    this.initialize(operatorId);
    this.validate();
    boolean wasNew = (getId() == null);
    conversationRepository.save(this);  // 5. 仅 save，不区分新增/更新
    if (wasNew) {  // 6. 每次领域操作完成后都必须发送领域事件
        domainEventPublisher.send(DomainEventDTO.builder().type(DomainEventConstant.CONVERSATION_SAVE).id(...).data(this).sender(operatorId).build());
    }
}

/** 删除当前对话并发布删除事件。 */
@Override
public void delete(String operatorId) {
    setUpdateNo(operatorId);
    validate();
    conversationRepository.deleteByNum(getNum());  // 仓储三方法契约：deleteByNum
    domainEventPublisher.send(DomainEventDTO.builder()
            .type(DomainEventConstant.CONVERSATION_DELETE).id(...).data(this).sender(operatorId).build());
}
```

- domain 内不出现 MyBatis、Spring 等基础设施注解；仅接口与领域逻辑。
