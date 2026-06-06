# Client Layer Cases

Client defines data carriers (Param, VO, DTO) and validation; has NO domain/infra dependencies; uses naming *Param/*VO/*DTO per usage.

## Case 1: Unified Response Result

Path: `client/src/main/java/<BASE_PACKAGE_PATH>/client/common/Result.java`

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

## Case 2: Input Param (validation annotations)

Path: `client/src/main/java/<BASE_PACKAGE_PATH>/client/conversation/CreateSessionParam.java`

```java
package BASE_PACKAGE.client.conversation;

import jakarta.validation.constraints.NotBlank;
import lombok.Data;

@Data
public class CreateSessionParam {
    @NotBlank(message = "会话标题不能为空")
    private String sessionTitle;
}
```

---

## Case 3: View VO (return structure)

Path: `client/src/main/java/<BASE_PACKAGE_PATH>/client/conversation/ConversationVo.java`

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

Key: Client only carries data & validation annotations; no domain/infra dependencies; naming *Param/*VO/*DTO matches purpose.
