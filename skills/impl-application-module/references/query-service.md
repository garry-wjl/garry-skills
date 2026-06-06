# Case 2: 查询服务（QueryService）

编排读用例，注入 infra Mapper 做只读查询并转 client DTO 返回。

**要点**：**所有入参必须使用 XXXParamDTO 类封装**（定义在 client 层），不允许使用零散的原始类型参数。

## 代码示例

**路径**：`application/src/main/java/<BASE_PACKAGE_PATH>/application/conversation/ConversationQueryService.java`

```java
package BASE_PACKAGE.application.conversation;

import BASE_PACKAGE.client.conversation.dto.ConversationQueryParamDTO;
import BASE_PACKAGE.client.conversation.dto.ConversationListDto;
import BASE_PACKAGE.infra.conversation.entity.ConversationEntity;
import BASE_PACKAGE.infra.conversation.mapper.ConversationMapper;
import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import jakarta.annotation.Resource;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.stream.Collectors;

/**
 * 对话查询服务：只读查询，注入 infra Mapper，转 client DTO。
 * 所有方法入参必须使用 client 层定义的 XXXParamDTO 类。
 */
@Service
public class ConversationQueryService {

    @Resource
    private ConversationMapper conversationMapper;

    public List<ConversationListDto> findByUserId(ConversationQueryParamDTO param) {
        long offset = (long) (param.getPageNum() - 1) * param.getPageSize();
        List<ConversationEntity> list = conversationMapper.selectList(
                new LambdaQueryWrapper<ConversationEntity>()
                        .eq(ConversationEntity::getUserId, param.getUserId())
                        .eq(ConversationEntity::getDeleted, 0)
                        .orderByDesc(ConversationEntity::getCreateTime)
                        .last("LIMIT " + offset + "," + param.getPageSize()));
        return list.stream().map(this::toDto).collect(Collectors.toList());
    }

    private ConversationListDto toDto(ConversationEntity entity) {
        ConversationListDto dto = new ConversationListDto();
        dto.setId(entity.getCode());
        dto.setSessionTitle(entity.getTitle());
        dto.setUserId(entity.getUserId());
        dto.setCreateTime(entity.getCreateTime());
        dto.setUpdateTime(entity.getUpdateTime());
        return dto;
    }
}
```

## 对应的 client 层 ParamDTO 示例

**路径**：`client/src/main/java/<BASE_PACKAGE_PATH>/client/conversation/dto/ConversationQueryParamDTO.java`

```java
package BASE_PACKAGE.client.conversation.dto;

import lombok.Data;

/**
 * 查询对话列表参数
 */
@Data
public class ConversationQueryParamDTO {
    private String userId;
    private int pageNum;
    private int pageSize;
}
```

## 关键点

1. **仅做只读查询**：使用 infra 的 Mapper 查询数据，不执行写操作
2. **注入 infra 依赖**：可注入 `Mapper` 和 `Entity`，直接查询数据库
3. **转换为 client DTO**：将数据库实体转换为客户端数据传输对象返回，**不返回 VO**
4. **不含业务逻辑**：仅做数据查询和转换，复杂业务逻辑应在 domain 实现
5. **支持分页/排序**：通过 QueryWrapper 灵活处理查询条件
6. **所有入参使用 ParamDTO 类**：禁止使用零散的 String、Long、int 等原始类型作为方法参数
7. **DTO vs VO**：
   - **DTO**：由 client 层定义，用于跨层传输数据（application ↔ adapter），不包含展示逻辑
   - **VO**：由 adapter 层从 DTO 转换而来，用于最终返回给客户端，包含展示相关的字段和逻辑（adapter → client）

## 替代方案：通过 domain Repository 查询

如果希望读操作也经过 domain 层，可改为注入 domain 的 Repository 接口：

```java
@Service
public class ConversationQueryService {

    @Resource
    private ConversationRepository conversationRepository;  // domain Repository 接口

    public List<ConversationListDto> findByUserId(ConversationQueryParamDTO param) {
        List<Conversation> conversations = conversationRepository.findByUserId(param.getUserId(), param.getPageNum(), param.getPageSize());
        return conversations.stream().map(this::toDto).collect(Collectors.toList());
    }

    private ConversationListDto toDto(Conversation conversation) {
        ConversationListDto dto = new ConversationListDto();
        dto.setId(conversation.getNum());
        dto.setSessionTitle(conversation.getTitle());
        // ... 其他字段
        return dto;
    }
}
```

## 数据流向

```
ConversationQueryService (application)
    ↓
Infra Mapper (query database)
    ↓
Infra Entity (data rows)
    ↓
Client DTO (return to adapter)
    ↓
Adapter (convert DTO to VO)
    ↓
Client VO (return to frontend)
```

## 重要约束

- ❌ **不返回 VO**：VO 由 adapter 层负责生成
- ❌ **不返回 entity**：application 层不能暴露 infra entity
- ❌ **不返回领域对象**：application 层不能暴露 domain 对象
- ❌ **不在 application 层定义 DTO**：DTO 必须定义在 client 层
- ❌ **不使用零散原始类型作入参**：所有入参必须使用 client 层定义的 XXXParamDTO 类
- ✅ **仅返回 client 层定义的 DTO**：application 层唯一允许的返回类型