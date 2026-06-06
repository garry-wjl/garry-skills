# Config Classes and Controller Implementation

## Case 1: BaseController (Base class)

Path: `adapter/src/main/java/<BASE_PACKAGE_PATH>/adapter/config/BaseController.java`

Gets current user ID from request attribute (written by auth filter); doesn't import domain. If current user object etc. needed, define DTO in client, coordinate with filter on request attribute name, extend here (TODO).

```java
package BASE_PACKAGE.adapter.config;

import jakarta.annotation.Resource;
import jakarta.servlet.http.HttpServletRequest;

public class BaseController {
    @Resource
    protected HttpServletRequest request;

    private static final String ATTR_USER_ID = "userId";

    protected String getCurrentUserId() {
        return (String) request.getAttribute(ATTR_USER_ID);
    }

    protected boolean isLogin() {
        return getCurrentUserId() != null;
    }
}
```

---

## Case 2: GlobalExceptionHandler

Path: `adapter/src/main/java/<BASE_PACKAGE_PATH>/adapter/config/GlobalExceptionHandler.java`

Depends on client Result; catches all exceptions, returns `Result.fail(msg)` for compilation. Other exception types (business, param validation, DB, etc.) supplemented by project per need via @ExceptionHandler.

```java
package BASE_PACKAGE.adapter.config;

import BASE_PACKAGE.client.common.Result;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(Exception.class)
    public ResponseEntity<Result<?>> handleException(Exception e) {
        String msg = e.getMessage() != null ? e.getMessage() : "系统异常";
        return ResponseEntity.status(HttpStatus.OK).body(Result.fail(msg));
    }
}
```

---

## Case 3: TokenValidationFilter

Path: `adapter/src/main/java/<BASE_PACKAGE_PATH>/adapter/config/TokenValidationFilter.java`

Extends OncePerRequestFilter; defaults to pass-through for compilation. Project fills TODO: parse token from Header/params, validate, write current user ID to request attribute, whitelist paths, return 401 if unauthenticated.

```java
package BASE_PACKAGE.adapter.config;

import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

import java.io.IOException;

@Component
@Order(1)
public class TokenValidationFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {
        // TODO: Whitelist paths pass through
        // TODO: Parse token, validate & write request attribute, fail → 401 & return
        filterChain.doFilter(request, response);
    }
}
```

---

## Case 4: Command Controller (POST, write, calls application, returns Result)

Path: `adapter/conversation/controller/ConversationCommandController.java`

```java
package BASE_PACKAGE.adapter.conversation.controller;

import BASE_PACKAGE.adapter.config.BaseController;
import BASE_PACKAGE.application.conversation.ConversationCommandService;
import BASE_PACKAGE.client.common.Result;
import BASE_PACKAGE.client.conversation.CreateSessionParam;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.tags.Tag;
import jakarta.annotation.Resource;
import jakarta.validation.Valid;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@Tag(name = "会话管理")
@RestController
@RequestMapping("/conversation/command")
public class ConversationCommandController extends BaseController {
    @Resource
    private ConversationCommandService conversationCommandService;

    @Operation(summary = "创建会话")
    @PostMapping("/create")
    public Result<String> createSession(@RequestBody @Valid CreateSessionParam param) {
        String code = conversationCommandService.createConversation(param.getSessionTitle(), getCurrentUserId());
        return Result.ok(code);
    }
}
```

Key: Input = client Param + @Valid; only call application; return Result.ok(data); write operations use POST.

---

## Case 5: Query Controller (GET, read, calls application, returns Result)

Path: `adapter/conversation/controller/ConversationQueryController.java`

```java
package BASE_PACKAGE.adapter.conversation.controller;

import BASE_PACKAGE.adapter.config.BaseController;
import BASE_PACKAGE.application.conversation.ConversationQueryService;
import BASE_PACKAGE.client.common.Result;
import BASE_PACKAGE.client.conversation.ConversationVo;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.tags.Tag;
import jakarta.annotation.Resource;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@Tag(name = "会话查询")
@RestController
@RequestMapping("/conversation/query")
public class ConversationQueryController extends BaseController {
    @Resource
    private ConversationQueryService conversationQueryService;

    @Operation(summary = "获取会话列表")
    @GetMapping("/list")
    public Result<List<ConversationVo>> getSessionList() {
        List<ConversationVo> list = conversationQueryService.findByUserId(getCurrentUserId());
        return Result.ok(list);
    }
}
```

Key: Read operations use GET; call application QueryService only; return Result.ok(data).
