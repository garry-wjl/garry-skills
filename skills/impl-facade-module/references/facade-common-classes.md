# Facade Common Classes (Request, Result, Exception)

These three classes (if missing) created automatically in facade module for adapter/client/application reference.

---

## 1. CommonRequest.java

Path: `facade/src/main/java/<BASE_PACKAGE_PATH>/facade/request/CommonRequest.java`

```java
package BASE_PACKAGE.facade.request;

import lombok.Data;

@Data
public class CommonRequest {
    private String operatorId;
}
```

---

## 2. Result.java

Path: `facade/src/main/java/<BASE_PACKAGE_PATH>/facade/common/Result.java`

```java
package BASE_PACKAGE.facade.common;

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

---

## 3. BusinessException.java

Path: `facade/src/main/java/<BASE_PACKAGE_PATH>/facade/exception/BusinessException.java`

```java
package BASE_PACKAGE.facade.exception;

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
