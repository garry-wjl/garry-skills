# Domain Interfaces (Repository, Factory, Gateway, ValueObject)

## Case 1: Repository Interface

Path: `domain/conversation/repository/ConversationRepository.java`

Repository **MUST and ONLY** have three methods (names & signatures fixed): **save**, **findByNum**, **deleteByNum**. No fourth method, no `build*By`, `findById`, `delete` aliases. Repository is for domain object / Factory / infra implementation collaboration only; application services must not inject or call Repository.

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

## Case 2: Factory Interface

Path: `domain/conversation/factory/ConversationFactory.java`

Factory **MUST contain exactly two methods only**: **create**, **createByNum**. Both methods build domain objects. `create(...)` only accepts business fields that users may fill when creating the domain object; it MUST NOT include operator, creator/updater, status, audit fields, generated business number, default values, or workflow/internal transition fields. **createByNum(String num)** in infra implementation **MUST only delegate** to repository's **findByNum(num)** to load/build the existing domain object by business code. No `createById(...)`, `rebuild(...)`, or other Factory methods are allowed.

```java
package BASE_PACKAGE.domain.conversation.factory;

import BASE_PACKAGE.domain.conversation.Conversation;

public interface ConversationFactory {
    /** Build a new domain object from required attributes (e.g., conversation title) */
    Conversation create(String title);
    /** Load/build an existing domain object by business code num */
    Conversation createByNum(String num);
}
```

---

## Case 3: Gateway Interface

Path: `domain/conversation/gateway/ConversationGateway.java`

**At least** includes **business ID generation** method; other methods per business logic. **Gateway definition**: In domain operations, any capability requiring **third-party interface, tools, etc.** exposed via gateway, implemented by infra.

```java
package BASE_PACKAGE.domain.conversation.gateway;

public interface ConversationGateway {
    /** Generate business ID (mandatory method) */
    String generateConversationId();
}
```

Example (e.g., report's KnowledgeBaseClient):

```java
package BASE_PACKAGE.domain.report.gateway;

import BASE_PACKAGE.domain.report.ReportPlan;
import java.util.Optional;

/** External/tool-based capability via gateway */
public interface KnowledgeBaseClient {
    Optional<ReportPlan> queryPlan(String userInput);
}
```

---

## Case 4: ValueObject

**Value objects use enum or constant**, per business need.

**Enum example** — Path: `domain/conversation/valueobject/ConversationStatus.java`

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

**Constant class example** (when enum unsuitable) — Path: `domain/conversation/valueobject/ConversationType.java`

```java
package BASE_PACKAGE.domain.conversation.valueobject;

public class ConversationType {
    public static final String CHAT = "CHAT";
    public static final String REPORT = "REPORT";
    private ConversationType() {}
}
```
