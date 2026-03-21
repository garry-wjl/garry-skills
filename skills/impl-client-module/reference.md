# Reference: Client 层 Case

以下 Case 与 Skill 打包，供实现 client 时参考。将 `BASE_PACKAGE` 替换为项目包名。若项目将 Result/CommonRequest/BusinessException 放在 facade，则 adapter/application 依赖 facade 使用，client 仅保留业务 Param/VO/DTO。

---

## Case 1: 统一响应 Result（若在 client 中定义）

路径：`client/src/main/java/<BASE_PACKAGE_PATH>/client/common/Result.java`

```java
package BASE_PACKAGE.client.common;

import lombok.Getter;
import lombok.Setter;
import java.util.List;

@Getter
@Setter
public class Result<T> {
    private Integer code;
    private String msg;
    private T data;
    private List<T> rows;

    public static <T> Result<T> ok(T data) {
        Result<T> r = new Result<>();
        r.code = 200;
        r.msg = "操作成功";
        r.data = data;
        return r;
    }

    public static <T> Result<T> ok(List<T> rows) {
        Result<T> r = new Result<>();
        r.code = 200;
        r.msg = "查询成功";
        r.rows = rows;
        return r;
    }

    public static <T> Result<T> fail(String msg) {
        Result<T> r = new Result<>();
        r.code = 500;
        r.msg = msg;
        return r;
    }

    public static <T> Result<T> fail(Integer code, String msg) {
        Result<T> r = new Result<>();
        r.code = code;
        r.msg = msg;
        return r;
    }
}
```

---

## Case 2: 入参 Param（校验注解）

路径：`client/src/main/java/<BASE_PACKAGE_PATH>/client/conversation/CreateSessionParam.java`

```java
package BASE_PACKAGE.client.conversation;

import BASE_PACKAGE.client.common.CommonRequest;
import jakarta.validation.constraints.NotBlank;
import lombok.Data;

@Data
public class CreateSessionParam {
    @NotBlank(message = "会话标题不能为空")
    private String sessionTitle;
}
```

---

## Case 3: 视图 VO（返回结构）

路径：`client/src/main/java/<BASE_PACKAGE_PATH>/client/conversation/ConversationVo.java`

```java
package BASE_PACKAGE.client.conversation;

import io.swagger.v3.oas.annotations.media.Schema;
import lombok.Data;

@Data
public class ConversationVo {
    private String id;
    private String sessionTitle;
    private String groupName;
}
```

要点：client 仅做数据承载与校验注解；不依赖 domain/infra；命名 *Param / *VO / *DTO 与用途一致。
