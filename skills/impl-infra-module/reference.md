# Reference: Infra 层 Case

以下 Case 与 Skill 打包，供实现 infra 时参考。将 `BASE_PACKAGE` 替换为项目包名。审计字段与项目规范一致（如 create_no、update_no、create_time、update_time、is_deleted）；不使用外键。

**与本 Skill 强约束对齐**：Entity 用 **`@Data`** + **字段注释**；**FactoryImpl 只注入 Repository**；**RepositoryImpl 构造器/字段仅允许本聚合（含子表）的 Mapper**，**不得**注入 Gateway、`DomainEventPublisher`、`ApplicationEventPublisher` 或任何非 Mapper Bean；**存在性只按 `num` 查库判定**。

---

## 包结构约定

- **必须存在 common 子包**，存放跨领域常量、事件、异常、工具及**外部 API 调用**：
  - `infra/common/constant` — 如 DeleteFlagConstant、LockKeyConstant
  - `infra/common/event` — 如 CommonDomainEventPublisher
  - `infra/common/exception` — 如 LockException
  - `infra/common/util` — 如 JsonUtils
  - **`infra/common/client`** — **client 放在 infra 的 common 子包下面**。调用外部第三方 HTTP 接口或 API 时在此完成：**接口定义**放在 client 包下；**client/param** 放接口的**参数对象**；**client/dto** 放 **API 专用**的请求/响应 DTO；实现类可在 client 下，领域 gateway 注入该接口并调用。
- **其余按业务领域划分子包**，一领域一子包。**按业务领域划分的子包中没有 client 子包**；每领域子包下仅含 entity、mapper、repository、factory、gateway。外部 API 调用统一使用 common/client。

示例目录树：
```
infra/
├── common/
│   ├── constant/
│   ├── event/
│   ├── exception/
│   ├── util/
│   └── client/          ← 外部 HTTP/API 时：接口 + param + dto
│       ├── param/       ← 接口参数对象
│       ├── dto/         ← API 专用请求/响应 DTO
│       └── (接口与实现类)
├── conversation/        ← 领域子包中无 client
│   ├── entity/
│   ├── factory/
│   ├── gateway/
│   ├── mapper/
│   └── repository/
├── message/
│   ├── entity/
│   ├── factory/
│   ├── gateway/
│   ├── mapper/
│   └── repository/
└── ...
```

---

## Case 0: common 子包（跨领域常量、异常、事件、工具）

**common/constant** — 逻辑删除标记示例：
```java
package BASE_PACKAGE.infra.common.constant;

public class DeleteFlagConstant {
    public static final int NOT_DELETED = 0;
    public static final int DELETED = 1;
}
```

**common/exception** — 基础设施异常示例：
```java
package BASE_PACKAGE.infra.common.exception;

public class LockException extends RuntimeException {
    public LockException(String message) { super(message); }
    public LockException(String message, Throwable cause) { super(message, cause); }
}
```

**common/event** — 见 Case 4（CommonDomainEventPublisher）。**common/util** — 按需放 JSON、日期等工具类。

**common/client**（调用外部第三方 HTTP/API 时）— 在 common/client 下完成接口定义，param 放参数对象，dto 放 API 专用 DTO，见长文末 Case 0b。

---

## Case 0b: common/client（外部 API 接口、param、dto）

**client 位于 infra 的 common 子包下**（路径：`infra/common/client`）。需要调用外部第三方 HTTP 接口或 API 时，在此完成：**接口定义**、**client/param**（接口参数对象）、**client/dto**（API 专用请求/响应 DTO）。

接口定义（路径示例：`infra/common/client/SomeThirdPartyApiClient.java`）：
```java
package BASE_PACKAGE.infra.common.client;

import BASE_PACKAGE.infra.common.client.dto.SomeResponseDTO;
import BASE_PACKAGE.infra.common.client.param.SomeRequestParam;

/**
 * 某第三方 HTTP/API 客户端接口，实现类在同一包或 client 下。
 */
public interface SomeThirdPartyApiClient {

    SomeResponseDTO call(SomeRequestParam param);
}
```

**client/param** — 接口参数对象（路径示例：`infra/common/client/param/SomeRequestParam.java`）：
```java
package BASE_PACKAGE.infra.common.client.param;

import lombok.Data;

@Data
public class SomeRequestParam {
    /** 检索关键词 */
    private String keyword;
    /** 分页大小 */
    private Integer pageSize;
}
```

**client/dto** — API 专用请求/响应 DTO（路径示例：`infra/common/client/dto/SomeResponseDTO.java`）：
```java
package BASE_PACKAGE.infra.common.client.dto;

import lombok.Data;

@Data
public class SomeResponseDTO {
    /** 第三方业务码 */
    private Integer code;
    /** 提示信息 */
    private String message;
    /** 载荷 */
    private Object data;
}
```

领域 gateway 实现类需调用该 API 时，注入 `SomeThirdPartyApiClient`，在 gateway 内调用接口即可；不把 param/dto 暴露给 domain，由 gateway 做 param/dto 与领域对象的转换。

---

## Case 1: Entity（PO，@Data + 字段注释）

- 使用 **`lombok.Data`**，**不要**手写 getter/setter。
- **每个字段**上方或行尾注释：含义、单位、枚举或约束。

路径：`infra/src/main/java/<BASE_PACKAGE_PATH>/infra/conversation/entity/ConversationEntity.java`

```java
package BASE_PACKAGE.infra.conversation.entity;

import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableField;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableLogic;
import com.baomidou.mybatisplus.annotation.TableName;
import lombok.Data;

import java.time.LocalDateTime;

/**
 * 会话表 PO，与表 conversation 一致；无业务逻辑。
 */
@Data
@TableName("conversation")
public class ConversationEntity {

    /** 主键，自增或雪花由表结构决定 */
    @TableId(type = IdType.AUTO)
    private Long id;
    /** 业务编码 num，全局唯一 */
    @TableField("num")
    private String num;
    /** 业务侧展示编码 code（若表中有） */
    @TableField("code")
    private String code;
    /** 会话标题 */
    @TableField("title")
    private String title;
    /** 归属用户ID */
    @TableField("user_id")
    private String userId;
    /** 状态枚举值，如 ACTIVE/CLOSED */
    @TableField("status")
    private String status;
    /** 创建人编号 */
    @TableField("create_no")
    private String createNo;
    /** 最后更新人编号 */
    @TableField("update_no")
    private String updateNo;
    /** 创建时间 */
    @TableField("create_time")
    private LocalDateTime createTime;
    /** 更新时间 */
    @TableField("update_time")
    private LocalDateTime updateTime;
    /** 逻辑删除：0 未删除，1 已删除 */
    @TableLogic
    @TableField("is_deleted")
    private Integer isDeleted;
}
```

---

## Case 2: Factory 实现（仅注入 Repository）

- **只**注入本聚合 **ConversationRepository**。
- **不要**注入 `DomainEventPublisher`、**不要**注入各类 **Gateway**。
- `createByNum` 委托 `conversationRepository.buildConversationBy(num)`。
- 若领域对象构造仍需要 Gateway/Publisher：**由应用层**在工厂/仓储返回后设置，或调整 domain 构造；**不在 RepositoryImpl 注入或 toDomain 内引用**这两类 Bean（见 SKILL 规则 8）。

路径：`infra/src/main/java/<BASE_PACKAGE_PATH>/infra/conversation/factory/ConversationFactoryImpl.java`

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
        // 领域构造仅依赖 Repository 时；若仍缺 Gateway/Publisher，由 Repository 加载路径或应用层补全
        return new Conversation(title, conversationRepository);
    }

    @Override
    public Conversation createByNum(String num) {
        return conversationRepository.buildConversationBy(num);
    }
}
```

---

## Case 3: Repository 实现（只注入 Mapper；存在性按 num）

- **仅**注入 **`ConversationMapper`**（及本聚合子表 Mapper，若有）。**绝对禁止**注入 `DomainEventPublisher`、`ApplicationEventPublisher`、任意 **Gateway**、其他领域 Mapper、其他 Repository 或任何非 Mapper 依赖。
- **判断是否存在**：用业务编码 **`num`** 查询数据库（`selectOne`/`selectCount` 条件 `num = ?`），**有记录即存在，无即不存在**；不要用主键 id 作为「是否存在」的唯一依据（除非领域明确约定）。
- `buildConversationBy(num)`：按 `num` 查 Entity，转领域对象；查不到返回 `null`。
- `save/delete`：Entity↔领域转换仅在 repository 内完成。

示例骨架（仅注入 `ConversationMapper`；Lambda 查询需 `import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper`）：
```java
@Component
public class ConversationRepositoryImpl implements ConversationRepository {

    @Resource
    private ConversationMapper conversationMapper;

    /** 是否存在：仅用业务编码 num 查库，有行则存在 */
    private boolean existsByNum(String num) {
        if (num == null || num.isBlank()) {
            return false;
        }
        return conversationMapper.selectCount(
                new LambdaQueryWrapper<ConversationEntity>().eq(ConversationEntity::getNum, num)) > 0;
    }

    @Override
    public Conversation buildConversationBy(String num) {
        if (num == null || num.isBlank()) {
            return null;
        }
        ConversationEntity e = conversationMapper.selectOne(
                new LambdaQueryWrapper<ConversationEntity>().eq(ConversationEntity::getNum, num));
        // 未查到即不存在
        return e == null ? null : toDomain(e);
    }

    // save / delete / toEntity 略；toDomain 仅做字段映射；若领域根仍需 Gateway/Publisher，由 application 在拿到领域对象后装配，勿在本类注入
}
```

---

## Case 4: DomainEventPublisher 实现（委托 Spring 事件）

路径：`infra/src/main/java/<BASE_PACKAGE_PATH>/infra/common/event/CommonDomainEventPublisher.java`

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
