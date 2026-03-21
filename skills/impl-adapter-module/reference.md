# Reference: Adapter 层 Case

以下 Case 与 Skill 打包，供实现 adapter 时参考。将 `BASE_PACKAGE` 替换为项目包名。Adapter 仅依赖 application 与 client；入参为 client 的 Param，返回 client 的 Result/VO。

---

## 包结构约定

- **必须存在 config 子包**，且**默认内置** BaseController、GlobalExceptionHandler、TokenValidationFilter 三个类（仅框架代码 + TODO，可编译通过）。
- **其余按领域划分子包**，与 application/domain 对应。每领域子包下包含 **controller/**；若该领域需要监听事件（MQ、Spring 事件、领域事件等），再增加 **listener/** 子包存放监听类。
- **应用启动类**（`@SpringBootApplication`）放在 **adapter 根目录**下，与 config、各领域子包平级。

示例目录树：
```
adapter/
├── <ApplicationName>Application.java ← 启动类，在 adapter 根目录
├── config/                           ← 跨领域/全局
│   ├── BaseController.java
│   ├── GlobalExceptionHandler.java
│   ├── TokenValidationFilter.java
│   └── ...
├── conversation/                     ← 按领域划分
│   └── controller/
│       ├── ConversationCommandController.java
│       └── ConversationQueryController.java
├── message/                          ← 该领域需监听事件时增加 listener/
│   ├── controller/
│   │   ├── MessageCommandController.java
│   │   └── MessageQueryController.java
│   └── listener/
│       └── MessageEventListener.java
├── auth/
│   └── controller/
│       └── AuthController.java        ← 入口单一，仅一个 Controller
├── agent/
│   └── controller/
│       └── AgentController.java
└── resource/
    └── controller/
        └── EmailCodeController.java
```

---

## Case 1: config 默认内置 — BaseController（基类控制器）

路径：`adapter/src/main/java/<BASE_PACKAGE_PATH>/adapter/config/BaseController.java`

仅依赖 jakarta.servlet，从 request 属性取当前用户 ID（由认证过滤器写入）；不引用 domain。若需当前用户对象等，可在 client 定义 DTO 并与过滤器约定 request 属性名后在此扩展 getter（TODO）。

```java
package BASE_PACKAGE.adapter.config;

import jakarta.annotation.Resource;
import jakarta.servlet.http.HttpServletRequest;

/**
 * 基类控制器：所有控制器的父类，提供当前用户等上下文。
 * 当前用户ID 由认证过滤器写入 request 属性，属性名需与 {@link TokenValidationFilter} 等约定一致。
 */
public class BaseController {

    @Resource
    protected HttpServletRequest request;

    /** 当前用户ID 在 request 中的属性名，需与认证过滤器约定一致 */
    private static final String ATTR_USER_ID = "userId";

    protected String getCurrentUserId() {
        return (String) request.getAttribute(ATTR_USER_ID);
    }

    protected boolean isLogin() {
        return getCurrentUserId() != null;
    }

    // TODO: 若需当前用户对象、用户编码等，可在 client 定义 DTO，由认证过滤器写入 request 后在此增加 getCurrentUser() 等方法。
}
```

---

## Case 2: config 默认内置 — GlobalExceptionHandler（全局异常处理）

路径：`adapter/src/main/java/<BASE_PACKAGE_PATH>/adapter/config/GlobalExceptionHandler.java`

依赖 client 的 `Result`；兜底处理所有异常并返回 `Result.fail(msg)`，保证可编译。其余异常类型（如业务异常、参数校验、数据库异常等）由项目按需补充 @ExceptionHandler。

```java
package BASE_PACKAGE.adapter.config;

import BASE_PACKAGE.client.common.Result;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

/**
 * 全局异常处理：统一返回 Result 格式，避免 500 直接透出。
 * TODO: 可补充 @ExceptionHandler(BusinessException.class)、MethodArgumentNotValidException、ConstraintViolationException 等。
 */
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

## Case 3: config 默认内置 — TokenValidationFilter（Token 校验过滤器）

路径：`adapter/src/main/java/<BASE_PACKAGE_PATH>/adapter/config/TokenValidationFilter.java`

继承 `OncePerRequestFilter`，默认直接放行，保证可编译。由项目在 TODO 处补充：从 Header/参数解析 token、校验 token、将当前用户 ID 写入 request 属性、白名单路径放行、未认证时返回 401 等。

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

/**
 * Token 校验过滤器：在请求进入 Controller 前校验 token 并将当前用户写入 request。
 * TODO: 从 request 解析 token（如 Header Authorization: Bearer xxx 或 query token）；校验 token 并解析用户；
 * TODO: 将当前用户 ID 等写入 request.setAttribute(ATTR_USER_ID, userId)，与 BaseController 约定一致；
 * TODO: 白名单路径（如 /auth/login、/doc.html、/swagger-ui、/v3/api-docs）不校验直接放行；
 * TODO: 未认证时返回 401 JSON，并 return，不调用 filterChain.doFilter。
 */
@Component
@Order(1)
public class TokenValidationFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {
        // TODO: 白名单路径直接放行
        // TODO: 解析 token，校验并写入 request 属性，失败则写 401 并 return
        filterChain.doFilter(request, response);
    }
}
```

---

## Case 4: 命令 Controller（写操作，POST，调用 application，返回 Result）

路径：`adapter/src/main/java/<BASE_PACKAGE_PATH>/adapter/conversation/controller/ConversationCommandController.java`

```java
package BASE_PACKAGE.adapter.conversation.controller;

import BASE_PACKAGE.adapter.config.BaseController;
import BASE_PACKAGE.application.conversation.ConversationCommandService;
import BASE_PACKAGE.client.common.Result;
import BASE_PACKAGE.client.conversation.CreateSessionParam;
import BASE_PACKAGE.client.conversation.UpdateSessionParam;
import BASE_PACKAGE.client.conversation.DeleteSessionParam;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.tags.Tag;
import jakarta.annotation.Resource;
import jakarta.validation.Valid;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@Tag(name = "会话管理", description = "会话命令接口")
@RestController
@RequestMapping("/conversation/command")
public class ConversationCommandController extends BaseController {

    @Resource
    private ConversationCommandService conversationCommandService;

    @Operation(summary = "创建会话")
    @PostMapping("/create")
    public Result<String> createSession(@RequestBody @Valid CreateSessionParam param) {
        String code = conversationCommandService.createConversation(
                param.getSessionTitle(),
                getCurrentUserId());
        return Result.ok(code);
    }

    @Operation(summary = "更新会话")
    @PostMapping("/update")
    public Result<Void> updateSession(@RequestBody @Valid UpdateSessionParam param) {
        conversationCommandService.updateTitle(
                param.getNum(), param.getTitle(), getCurrentUserId());
        return Result.ok(null);
    }

    @Operation(summary = "删除会话")
    @PostMapping("/delete")
    public Result<Void> deleteSession(@RequestBody @Valid DeleteSessionParam param) {
        conversationCommandService.deleteConversation(param.getNum(), getCurrentUserId());
        return Result.ok(null);
    }
}
```

要点：入参为 client 的 Param + `@Valid`；只调 application 一层；返回 `Result.ok(data)`；写操作用 POST。

---

## Case 5: 查询 Controller（读操作，GET，调用 application，返回 Result）

路径：`adapter/src/main/java/<BASE_PACKAGE_PATH>/adapter/conversation/controller/ConversationQueryController.java`

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

@Tag(name = "会话查询", description = "会话查询接口")
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

要点：读操作用 GET；只调 application 的 QueryService；返回 `Result.ok(data)`。
