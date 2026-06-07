# Case 1: 命令服务（CommandService）

编排写用例，通过 domain Factory 创建/加载实体并调用实体方法。

**要点**：**所有入参必须使用 XXXParamDTO 类封装**（定义在 client 层），不允许使用零散的原始类型参数。

## 代码示例

**路径**：`application/src/main/java/<BASE_PACKAGE_PATH>/application/conversation/ConversationCommandService.java`

```java
package BASE_PACKAGE.application.conversation;

import BASE_PACKAGE.client.conversation.dto.ConversationCreateParamDTO;
import BASE_PACKAGE.client.conversation.dto.ConversationDeleteParamDTO;
import BASE_PACKAGE.client.conversation.dto.ConversationUpdateParamDTO;
import BASE_PACKAGE.client.user.dto.UserExistsParamDTO;
import BASE_PACKAGE.application.user.UserQueryService;
import BASE_PACKAGE.domain.conversation.Conversation;
import BASE_PACKAGE.domain.conversation.factory.ConversationFactory;
import jakarta.annotation.Resource;
import org.redisson.api.RLock;
import org.redisson.api.RedissonClient;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.concurrent.TimeUnit;

/**
 * 对话命令服务：编排写用例，通过 domain Factory 创建/加载实体并调用实体方法。
 * 所有方法入参必须使用 client 层定义的 XXXParamDTO 类。
 * 写操作必须使用 Redis 分布式锁，应对多实例并发。
 */
@Service
public class ConversationCommandService {

    @Resource
    private ConversationFactory conversationFactory;

    @Resource
    private UserQueryService userQueryService;

    @Resource
    private RedissonClient redissonClient;

    @Transactional(rollbackFor = Exception.class)
    public String createConversation(ConversationCreateParamDTO param) {
        // 1. 参数校验已在 ParamDTO 或调用方完成
        // 2. 解析操作人
        String operatorId = param.getOperatorId();
        // 3. 获取 Redis 分布式锁，基于业务唯一标识
        String lockKey = "lock:conversation:create:" + param.getTitle();
        RLock lock = redissonClient.getLock(lockKey);
        try {
            if (!lock.tryLock(5, 10, TimeUnit.SECONDS)) {
                throw new RuntimeException("操作过于频繁，请稍后重试");
            }
            // 4. 如需校验数据，调用对应领域 QueryService
            UserExistsParamDTO existsParam = new UserExistsParamDTO();
            existsParam.setUserNum(param.getUserNum());
            if (!userQueryService.existsByNum(existsParam)) {
                throw new IllegalArgumentException("用户不存在");
            }
            // 5. 通过 Factory 创建领域对象
            Conversation conversation = conversationFactory.create(param.getTitle());
            // 6. 调用领域动作（内部完成规则校验、持久化、事件发布）
            conversation.save(operatorId);
            // 7. 返回业务标识
            return conversation.getNum();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            throw new RuntimeException("获取锁被中断", e);
        } finally {
            if (lock.isHeldByCurrentThread()) {
                lock.unlock();
            }
        }
    }

    @Transactional(rollbackFor = Exception.class)
    public void updateTitle(ConversationUpdateParamDTO param) {
        String lockKey = "lock:conversation:update:" + param.getConversationNum();
        RLock lock = redissonClient.getLock(lockKey);
        try {
            if (!lock.tryLock(3, 10, TimeUnit.SECONDS)) {
                throw new RuntimeException("操作过于频繁，请稍后重试");
            }
            Conversation conversation = conversationFactory.createByNum(param.getConversationNum());
            if (conversation != null) {
                conversation.updateTitle(param.getTitle(), param.getOperatorId());
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            throw new RuntimeException("获取锁被中断", e);
        } finally {
            if (lock.isHeldByCurrentThread()) {
                lock.unlock();
            }
        }
    }

    @Transactional(rollbackFor = Exception.class)
    public void deleteConversation(ConversationDeleteParamDTO param) {
        String lockKey = "lock:conversation:delete:" + param.getConversationNum();
        RLock lock = redissonClient.getLock(lockKey);
        try {
            if (!lock.tryLock(3, 10, TimeUnit.SECONDS)) {
                throw new RuntimeException("操作过于频繁，请稍后重试");
            }
            Conversation conversation = conversationFactory.createByNum(param.getConversationNum());
            if (conversation != null) {
                conversation.delete(param.getOperatorId());
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            throw new RuntimeException("获取锁被中断", e);
        } finally {
            if (lock.isHeldByCurrentThread()) {
                lock.unlock();
            }
        }
    }
}
```

## 对应的 client 层 ParamDTO 示例

**路径**：`client/src/main/java/<BASE_PACKAGE_PATH>/client/conversation/dto/ConversationCreateParamDTO.java`

```java
package BASE_PACKAGE.client.conversation.dto;

import lombok.Data;

/**
 * 创建对话参数
 */
@Data
public class ConversationCreateParamDTO {
    private String title;
    private String userNum;
    private String operatorId;
}
```

```java
package BASE_PACKAGE.client.conversation.dto;

import lombok.Data;

/**
 * 更新对话标题参数
 */
@Data
public class ConversationUpdateParamDTO {
    private String conversationNum;
    private String title;
    private String operatorId;
}
```

```java
package BASE_PACKAGE.client.conversation.dto;

import lombok.Data;

/**
 * 删除对话参数
 */
@Data
public class ConversationDeleteParamDTO {
    private String conversationNum;
    private String operatorId;
}
```

## 关键点

1. **仅依赖 domain 的 Factory**：不直接创建实体，而是通过 Factory 创建或加载；新建使用 `create(...)`，按业务编码加载使用 `createByNum(...)`；禁止直接 `new` 领域对象或直接调用领域对象静态 `create` 方法
2. **禁止依赖 Repository 与 Gateway**：CommandService 不注入、不调用 Repository 或 Gateway；如需查询/校验数据，调用对应领域的 QueryService（如 `UserQueryService.existsByNum`）
3. **调用实体行为**：通过 `conversation.save()`、`conversation.updateTitle()` 等方法执行业务逻辑。Gateway、Repository、DomainEventPublisher 由领域对象内部持有并调用
4. **事务边界**：在 application 方法上使用 `@Transactional(rollbackFor = Exception.class)`
5. **Redis 分布式锁**：写操作方法必须使用 Redis 分布式锁；锁 key 基于业务唯一标识（如 `num`），控制并发重复请求；锁有效期设置合理超时防止死锁；事务内嵌在锁区域内
6. **不含领域规则**：领域规则应在 domain 层实现，application 仅做编排
7. **操作人信息传入**：不从上下文获取，由 adapter 组装到 ParamDTO 中传入
8. **所有入参使用 ParamDTO 类**：禁止使用零散的 String、Long 等原始类型作为方法参数
9. **ParamDTO 定义在 client 层**：命名规则为 `XXXParamDTO`，按语义对应（Create/Update/Delete/Query）

## 与 domain 的交互流程

```
ConversationCommandService
    ↓
Redis Distributed Lock (acquire lock by business key)
    ↓
Domain Factory (create/load entity)
    ↓
Domain Entity (execute behavior, internally invokes Gateway/Repository/DomainEventPublisher)
    ↓
Release Lock
```