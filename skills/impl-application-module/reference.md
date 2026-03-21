# Reference: Application 层 Case

以下 Case 与 Skill 打包，供实现 application 时参考。将 `BASE_PACKAGE` 替换为项目包名。Application 依赖 client、domain、infra；写用例通过 domain Factory 与实体方法，读用例可注入 infra Mapper 或 domain Repository。

---

## 包结构约定

- **按业务领域/能力划分子包**，与 domain 聚合或用例对应；可选 **common** 子包存放跨领域通用应用服务。
- **非 Agent 开发**：领域子包下**只有两个 Service**——一个写操作（*CommandService）、一个读操作（*QueryService）。
- **Agent 开发**：在子包内再分子目录 **config/**、**hook/**、**interceptor/**、**service/**、**tool/**；其中 **service/** 下**只有一个** AgentService（如 ReactAgentService）。

示例目录树：
```
application/
├── conversation/                    ← 非 Agent：仅两个 Service
│   ├── ConversationCommandService.java
│   └── ConversationQueryService.java
├── message/                         ← 非 Agent：仅两个 Service
│   ├── MessageCommandService.java
│   └── MessageQueryService.java
├── auth/                            ← 非 Agent：仅两个 Service（或单一 AuthService 时读写在同包，仍建议拆成读/写两个）
│   ├── AuthCommandService.java
│   └── AuthQueryService.java
└── agent/                           ← Agent：子包内再分 config/hook/interceptor/service/tool，service 下仅一个 AgentService
    ├── config/
    │   └── ReactAgentConfig.java
    ├── hook/
    │   ├── ReportPlanAgentHook.java
    │   └── ...
    ├── interceptor/
    │   └── ReportSupervisorMainModelInterceptor.java
    ├── service/
    │   └── ReactAgentService.java    ← 仅此一个 AgentService
    └── tool/
        └── ExecutionTools.java
```

---

## Case 1: 命令服务（Factory 创建实体，调实体方法，事务在 application）

路径：`application/src/main/java/<BASE_PACKAGE_PATH>/application/conversation/ConversationCommandService.java`

```java
package BASE_PACKAGE.application.conversation;

import BASE_PACKAGE.domain.conversation.Conversation;
import BASE_PACKAGE.domain.conversation.factory.ConversationFactory;
import jakarta.annotation.Resource;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

/**
 * 对话命令服务：编排写用例，通过 domain Factory 创建/加载实体并调用实体方法。
 */
@Service
public class ConversationCommandService {

    @Resource
    private ConversationFactory conversationFactory;

    public String createConversation(String title, String operatorId) {
        Conversation conversation = conversationFactory.create(title);
        conversation.save(operatorId);
        return conversation.getNum();
    }

    @Transactional(rollbackFor = Exception.class)
    public void updateTitle(String conversationNum, String title, String operatorId) {
        Conversation conversation = conversationFactory.createByNum(conversationNum);
        if (conversation != null) {
            conversation.updateTitle(title, operatorId);
        }
    }

    @Transactional(rollbackFor = Exception.class)
    public void deleteConversation(String conversationNum, String operatorId) {
        Conversation conversation = conversationFactory.createByNum(conversationNum);
        if (conversation != null) {
            conversation.delete(operatorId);
        }
    }
}
```

要点：只依赖 domain 的 Factory；通过 Factory 创建或加载实体后调用实体行为；事务边界在 application 方法上；不写领域规则。

---

## Case 2: 查询服务（注入 infra Mapper 只读查询，转 client VO）

路径：`application/src/main/java/<BASE_PACKAGE_PATH>/application/conversation/ConversationQueryService.java`

```java
package BASE_PACKAGE.application.conversation;

import BASE_PACKAGE.client.conversation.ChatSessionVo;
import BASE_PACKAGE.infra.conversation.entity.ConversationEntity;
import BASE_PACKAGE.infra.conversation.mapper.ConversationMapper;
import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import jakarta.annotation.Resource;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.stream.Collectors;

/**
 * 对话查询服务：只读查询，注入 infra Mapper，转 client VO。
 */
@Service
public class ConversationQueryService {

    @Resource
    private ConversationMapper conversationMapper;

    public List<ChatSessionVo> findByUserId(String userId, int pageNum, int pageSize) {
        long offset = (long) (pageNum - 1) * pageSize;
        List<ConversationEntity> list = conversationMapper.selectList(
                new LambdaQueryWrapper<ConversationEntity>()
                        .eq(ConversationEntity::getUserId, userId)
                        .eq(ConversationEntity::getDeleted, 0)
                        .orderByDesc(ConversationEntity::getCreateTime)
                        .last("LIMIT " + offset + "," + pageSize));
        return list.stream().map(this::toVo).collect(Collectors.toList());
    }

    private ChatSessionVo toVo(ConversationEntity entity) {
        ChatSessionVo vo = new ChatSessionVo();
        vo.setId(entity.getCode());
        vo.setSessionTitle(entity.getTitle());
        vo.setUserId(entity.getUserId());
        vo.setCreateTime(entity.getCreateTime());
        vo.setUpdateTime(entity.getUpdateTime());
        return vo;
    }
}
```

要点：读用例可注入 infra 的 Mapper 与 Entity，仅做只读查询并转换为 client 的 VO；不在此做写操作。若希望读也经 domain，可改为注入 domain 的 Repository 接口（由 infra 实现）。
