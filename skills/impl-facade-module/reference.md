# Reference: Facade 基础类与接口模板

将 `BASE_PACKAGE` 替换为项目包名（如 `com.garry`、`com.example`）。若项目规范使用 `create_no`/`update_no`，将 `DomainEntity` 中的 `createId`/`updateId` 改为对应字段名。

**Lombok 约定**：Result、DomainEntity 使用 `@Getter`、`@Setter`；DomainEventDTO 使用 `@AllArgsConstructor`、`@NoArgsConstructor`、`@Builder`、`@Getter`、`@Setter`；CommonRequest 使用 `@Data`。

---

## 1. DomainEntity.java

路径：`facade/src/main/java/<BASE_PACKAGE_PATH>/facade/domain/DomainEntity.java`  
（BASE_PACKAGE_PATH = BASE_PACKAGE 的路径形式，如 `com/garry`）

```java
package BASE_PACKAGE.facade.domain;

import cn.hutool.core.lang.Assert;
import lombok.Getter;
import lombok.Setter;

import java.time.LocalDateTime;

/**
 * 领域实体基类
 * 包含通用审计字段与 validate/save/delete 抽象方法，子类实现 domainValidate、save、delete。
 */
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

路径：`facade/src/main/java/<BASE_PACKAGE_PATH>/facade/domain/DomainEventDTO.java`

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

路径：`facade/src/main/java/<BASE_PACKAGE_PATH>/facade/domain/DomainEventPublisher.java`

```java
package BASE_PACKAGE.facade.domain;

/**
 * 领域事件发布器接口，由 infra 层实现（如基于 Spring ApplicationEventPublisher）。
 */
public interface DomainEventPublisher {
    void send(DomainEventDTO eventDTO);
}
```

---

## 4. infra 层实现示例（参考）

若项目需要在 infra 中提供默认实现，可在 `infra` 模块中新增实现类，依赖 Spring 的 `ApplicationEventPublisher`：

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

注意：infra 模块需依赖 facade（以及 Spring 相关 starter），此处仅作参考；实现 facade 技能时以创建 facade 内三类为主。

---

## 5. Request / Result / Exception 模板（缺失时自动创建）

将 `BASE_PACKAGE` 替换为项目包名（如 `com.garry`、`com.example`）。以下三类放在 facade 模块中，供 adapter/client 等引用。

### 5.1 CommonRequest.java

路径：`facade/src/main/java/<BASE_PACKAGE_PATH>/facade/request/CommonRequest.java`

```java
package BASE_PACKAGE.facade.request;

import lombok.Data;

/**
 * 通用请求基类，所有请求参数类可继承此类。
 */
@Data
public class CommonRequest {
    /**
     * 操作人ID
     */
    private String operatorId;
}
```

### 5.2 Result.java

路径：`facade/src/main/java/<BASE_PACKAGE_PATH>/facade/common/Result.java`

```java
package BASE_PACKAGE.facade.common;

import lombok.Getter;
import lombok.Setter;

import java.util.List;

/**
 * 统一响应结果类（code、msg、data/rows）。
 */
@Getter
@Setter
public class Result<T> {
    private Integer code;
    private String msg;
    private T data;
    private List<T> rows;

    public static <T> Result<T> ok(T data) {
        Result<T> result = new Result<>();
        result.code = 200;
        result.msg = "操作成功";
        result.data = data;
        return result;
    }

    public static <T> Result<T> ok(List<T> rows) {
        Result<T> result = new Result<>();
        result.code = 200;
        result.msg = "查询成功";
        result.rows = rows;
        return result;
    }

    public static <T> Result<T> fail(String msg) {
        Result<T> result = new Result<>();
        result.code = 500;
        result.msg = msg;
        return result;
    }

    public static <T> Result<T> fail(Integer code, String msg) {
        Result<T> result = new Result<>();
        result.code = code;
        result.msg = msg;
        return result;
    }
}
```

### 5.3 BusinessException.java

路径：`facade/src/main/java/<BASE_PACKAGE_PATH>/facade/exception/BusinessException.java`

```java
package BASE_PACKAGE.facade.exception;

/**
 * 业务异常，供应用层/领域层抛出，由全局异常处理器统一拦截并返回 Result。
 */
public class BusinessException extends RuntimeException {

    private static final long serialVersionUID = 1L;

    private final Integer code;

    public BusinessException(String message) {
        super(message);
        this.code = 500;
    }

    public BusinessException(Integer code, String message) {
        super(message);
        this.code = code != null ? code : 500;
    }

    public BusinessException(String message, Throwable cause) {
        super(message, cause);
        this.code = 500;
    }

    public Integer getCode() {
        return code;
    }
}
```
