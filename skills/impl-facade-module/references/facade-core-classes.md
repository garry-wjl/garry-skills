# Facade Core Classes

Replace BASE_PACKAGE with project base package (e.g., com.garry, com.example). Lombok: Result, DomainEntity use @Getter, @Setter; DomainEventDTO uses @AllArgsConstructor, @NoArgsConstructor, @Builder, @Getter, @Setter.

---

## 1. DomainEntity.java

Path: `facade/src/main/java/<BASE_PACKAGE_PATH>/facade/domain/DomainEntity.java`

```java
package BASE_PACKAGE.facade.domain;

import cn.hutool.core.lang.Assert;
import lombok.Getter;
import lombok.Setter;

import java.time.LocalDateTime;

@Getter
@Setter
public abstract class DomainEntity {
    protected Long id;
    protected LocalDateTime createTime;
    protected String createId;
    protected LocalDateTime updateTime;
    protected String updateId;

    public void initialize(String operatorId) {
        LocalDateTime now = LocalDateTime.now();
        this.createTime = this.updateTime == null ? now : this.updateTime;
        this.createId = this.createId == null ? operatorId : this.createId;
        this.updateTime = now;
        this.updateId = operatorId;
    }

    public void validate() {
        Assert.notNull(this.createId, "实体创建人不能为空");
        Assert.notNull(this.updateId, "实体更新人不能为空");
        Assert.notNull(this.createTime, "实体创建时间不能为空");
        Assert.notNull(this.updateTime, "实体更新时间不能为空");
        domainValidate();
    }

    public abstract void domainValidate();
    public abstract void save(String operatorId);
    public abstract void delete(String operatorId);
}
```

---

## 2. DomainEventDTO.java

Path: `facade/src/main/java/<BASE_PACKAGE_PATH>/facade/domain/DomainEventDTO.java`

```java
package BASE_PACKAGE.facade.domain;

import lombok.*;

@AllArgsConstructor
@NoArgsConstructor
@Builder
@Getter
@Setter
public class DomainEventDTO {
    private String id;
    private String type;
    private Object data;
    private Long time;
    private String sender;
}
```

---

## 3. DomainEventPublisher.java

Path: `facade/src/main/java/<BASE_PACKAGE_PATH>/facade/domain/DomainEventPublisher.java`

```java
package BASE_PACKAGE.facade.domain;

/**
 * Domain event publisher interface, implemented by infra (e.g., Spring ApplicationEventPublisher-based).
 */
public interface DomainEventPublisher {
    void send(DomainEventDTO eventDTO);
}
```

---

## 4. Infra Implementation Example (Reference)

If project needs default infra implementation, add in `infra` module depending on Spring's `ApplicationEventPublisher`:

```java
package BASE_PACKAGE.infra.common.event;

import BASE_PACKAGE.facade.domain.DomainEventDTO;
import BASE_PACKAGE.facade.domain.DomainEventPublisher;
import jakarta.annotation.Resource;
import lombok.extern.slf4j.Slf4j;
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.stereotype.Component;

@Slf4j
@Component
public class CommonDomainEventPublisher implements DomainEventPublisher {
    @Resource
    private ApplicationEventPublisher publisher;

    @Override
    public void send(DomainEventDTO eventDTO) {
        log.info("eventDTO: {}", eventDTO);
        publisher.publishEvent(eventDTO);
    }
}
```

Note: Infra depends on facade (and Spring starters); this is reference only; focus facade skill on creating facade's three classes.
