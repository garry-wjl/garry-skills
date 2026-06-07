# Infra Implementation Cases

Per strong constraints: Entity uses **@Data** + field comments; **FactoryImpl injects only Repository**; **RepositoryImpl constructor/fields allow ONLY aggregates (including sub-tables) Mapper**, absolutely **NO** DomainEventPublisher, ApplicationEventPublisher, any **Gateway**, other domain Mappers, other Repository or non-Mapper beans; existence by `num` lookup only.

---

## Case 1: Entity (PO, @Data + field comments)

- Use **@Data**, do NOT hand-write getter/setter.
- **Each field**: comment above or end-of-line with meaning, unit, enum or constraint.

Path: `infra/conversation/entity/ConversationEntity.java`

```java
package BASE_PACKAGE.infra.conversation.entity;

import com.baomidou.mybatisplus.annotation.*;
import lombok.Data;
import java.time.LocalDateTime;

@Data
@TableName("conversation")
public class ConversationEntity {
    @TableId(type = IdType.AUTO)
    private Long id;
    @TableField("num")
    private String num;
    @TableField("title")
    private String title;
    @TableField("user_id")
    private String userId;
    @TableField("status")
    private String status;
    @TableField("create_no")
    private String createNo;
    @TableField("update_no")
    private String updateNo;
    @TableField("create_time")
    private LocalDateTime createTime;
    @TableField("update_time")
    private LocalDateTime updateTime;
    @TableLogic
    @TableField("is_deleted")
    private Integer isDeleted;
}
```

---

## Case 2: Factory Implementation (inject only Repository)

- Inject **only** ConversationRepository (aggregate's own Mapper).
- **Never** inject DomainEventPublisher; **never** inject any **Gateway**.
- Implement exactly two methods only: `create(...)` builds a new domain object from user-visible business fields supplied during creation; it must not include operator, creator/updater, status, audit fields, generated business number, default values, or workflow/internal transition fields. `createByNum` delegates only to `conversationRepository.findByNum(num)` to load/build the existing domain object by business code. No `createById(...)`, `rebuild(...)`, or other Factory methods are allowed.
- If domain needs Gateway/Publisher still: **application layer after Factory returns, set it**, or adjust domain construction; application must not call Repository directly; **RepositoryImpl never injects or references these two bean types** (see SKILL rule 8).

Path: `infra/conversation/factory/ConversationFactoryImpl.java`

```java
package BASE_PACKAGE.infra.conversation.factory;

import BASE_PACKAGE.domain.conversation.Conversation;
import BASE_PACKAGE.domain.conversation.factory.ConversationFactory;
import BASE_PACKAGE.domain.conversation.repository.ConversationRepository;
import jakarta.annotation.Resource;
import org.springframework.stereotype.Component;

@Component
public class ConversationFactoryImpl implements ConversationFactory {
    @Resource
    private ConversationRepository conversationRepository;

    @Override
    public Conversation create(String title) {
        return new Conversation(title, conversationRepository);
    }

    @Override
    public Conversation createByNum(String num) {
        return conversationRepository.findByNum(num);
    }
}
```

---

## Case 3: Repository Implementation (inject only Mapper; existence by num)

- Inject **only** ConversationMapper (and aggregate's sub-table Mappers). **Absolutely NO** DomainEventPublisher, ApplicationEventPublisher, any **Gateway**, other domain Mapper, other Repository or non-Mapper beans.
- **Existence check**: Query DB via business ID `num` (selectOne/selectCount with `num = ?`); **has record = exists, no record = not exists**; don't use main key id alone (unless domain explicitly states).
- `findByNum(num)`: Query Entity by `num`, convert to domain; not found → return `null`. This method is for domain object / Factory collaboration only, not an application-layer query API.
- `save/delete`: Entity ↔ domain conversion only in repository.

Skeleton (only ConversationMapper injected; Lambda query needs `import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper`):

```java
@Component
public class ConversationRepositoryImpl implements ConversationRepository {
    @Resource
    private ConversationMapper conversationMapper;

    private boolean existsByNum(String num) {
        if (num == null || num.isBlank()) return false;
        return conversationMapper.selectCount(
            new LambdaQueryWrapper<ConversationEntity>().eq(ConversationEntity::getNum, num)) > 0;
    }

    @Override
    public Conversation findByNum(String num) {
        if (num == null || num.isBlank()) return null;
        ConversationEntity e = conversationMapper.selectOne(
            new LambdaQueryWrapper<ConversationEntity>().eq(ConversationEntity::getNum, num));
        return e == null ? null : toDomain(e);
    }

    // save/delete/toEntity skipped; toDomain field-mapping only; if domain still needs Gateway/Publisher, application equips after domain object retrieval, don't inject here
}
```

---

## Case 4: DomainEventPublisher Implementation

Path: `infra/common/event/CommonDomainEventPublisher.java`

```java
package BASE_PACKAGE.infra.common.event;

import BASE_PACKAGE.facade.domain.DomainEventDTO;
import BASE_PACKAGE.facade.domain.DomainEventPublisher;
import jakarta.annotation.Resource;
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.stereotype.Component;

@Component
public class CommonDomainEventPublisher implements DomainEventPublisher {
    @Resource
    private ApplicationEventPublisher publisher;

    @Override
    public void send(DomainEventDTO eventDTO) {
        publisher.publishEvent(eventDTO);
    }
}
```
