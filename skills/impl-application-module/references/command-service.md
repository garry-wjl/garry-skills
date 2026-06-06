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
import BASE_PACKAGE.domain.conversation.Conversation;
import BASE_PACKAGE.domain.conversation.factory.ConversationFactory;
import jakarta.annotation.Resource;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

/**
 * 对话命令服务：编排写用例，通过 domain Factory 创建/加载实体并调用实体方法。
 * 所有方法入参必须使用 client 层定义的 XXXParamDTO 类。
 */
@Service
public class ConversationCommandService {

    @Resource
    private ConversationFactory conversationFactory;

    @Transactional(rollbackFor = Exception.class)
    public String createConversation(ConversationCreateParamDTO param) {
        Conversation conversation = conversationFactory.create(param.getTitle());
        conversation.save(param.getOperatorId());
        return conversation.getNum();
    }

    @Transactional(rollbackFor = Exception.class)
    public void updateTitle(ConversationUpdateParamDTO param) {
        Conversation conversation = conversationFactory.createByNum(param.getConversationNum());
        if (conversation != null) {
            conversation.updateTitle(param.getTitle(), param.getOperatorId());
        }
    }

    @Transactional(rollbackFor = Exception.class)
    public void deleteConversation(ConversationDeleteParamDTO param) {
        Conversation conversation = conversationFactory.createByNum(param.getConversationNum());
        if (conversation != null) {
            conversation.delete(param.getOperatorId());
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
2. **调用实体行为**：通过 `conversation.save()`、`conversation.updateTitle()` 等方法执行业务逻辑
3. **事务边界**：在 application 方法上使用 `@Transactional(rollbackFor = Exception.class)`
4. **不含领域规则**：领域规则应在 domain 层实现，application 仅做编排
5. **操作人信息传入**：不从上下文获取，由 adapter 组装到 ParamDTO 中传入
6. **所有入参使用 ParamDTO 类**：禁止使用零散的 String、Long 等原始类型作为方法参数
7. **ParamDTO 定义在 client 层**：命名规则为 `XXXParamDTO`，按语义对应（Create/Update/Delete/Query）

## 与 domain 的交互流程

```
ConversationCommandService
    ↓
Domain Factory (create/load entity)
    ↓
Domain Entity (execute behavior)
    ↓
Infra Repository (persist data)
```