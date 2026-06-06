# Domain Package Structure Convention

## Package Structure Rules

- **common subdirectory (必须)**: Create **DomainEventConstant** class; all constants `public static final String`, strictly follow business rules; place directly in domain/common without subdirs.
- **Domain-based subdirectories**: Each business domain one subdirectory; **each domain subdirectory allows ONLY four subdirs**: factory/, gateway/, repository/, valueobject/ (NO query/, dto/, etc.); aggregate root and optional event payload .java in **domain subdirectory root**.
  - **repository/** — Repository interfaces (ONLY: save, findByNum, deleteByNum)
  - **factory/** — Factory interfaces (generally two core methods: create(required fields), createByNum(num))
  - **gateway/** — Gateway interfaces: generate business ID; third-party, permission, non-repository queries (e.g., by email, by ID, list)
  - **valueobject/** — Value objects (anemic or enum/constant)

**Example directory tree:**
```
domain/
├── common/
│   └── DomainEventConstant.java
├── conversation/
│   ├── Conversation.java           ← Root entity
│   ├── ConversationEventDTO.java   ← Optional; event payload, same level as root, no dto subdir
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
└── ...
```

---

## Case 0: DomainEventConstant

Path: `domain/src/main/java/<BASE_PACKAGE_PATH>/domain/common/DomainEventConstant.java`

```java
package BASE_PACKAGE.domain.common;

public class DomainEventConstant {
    public static final String CONVERSATION_SAVE = "CONVERSATION_SAVE";
    public static final String CONVERSATION_CLOSE = "CONVERSATION_CLOSE";
    public static final String CONVERSATION_TITLE_UPDATE = "CONVERSATION_TITLE_UPDATE";
    public static final String CONVERSATION_DELETE = "CONVERSATION_DELETE";
    
    public static final String MESSAGE_SAVE = "MESSAGE_SAVE";
    public static final String MESSAGE_START = "MESSAGE_START";
    public static final String MESSAGE_COMPLETE = "MESSAGE_COMPLETE";
    public static final String MESSAGE_FAIL = "MESSAGE_FAIL";
    public static final String MESSAGE_DELETE = "MESSAGE_DELETE";
}
```

All constants `public static final String`, naming strictly follows business rules.
