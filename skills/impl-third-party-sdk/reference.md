# 可选 SDK 清单与引入方式

以下为可选的第三方 SDK、推荐版本及允许引入的模块。新增时：先在根 pom 的 `dependencyManagement` 中声明版本，再在对应模块的 `dependencies` 中引用（不写 version）。

---

## 一、可选 SDK 清单

| 名称 | 用途 | groupId | artifactId | 推荐版本 | 可引入模块 | 说明 |
|------|------|---------|------------|----------|------------|------|
| Lombok | 简化 getter/setter/构造等 | org.projectlombok | lombok | 1.18.30 | 所有（或由根统一引入） | scope=provided |
| Fastjson2 | JSON 序列化/反序列化 | com.alibaba.fastjson2 | fastjson2 | 2.0.47 | 所有（或由根统一引入） | |
| Hutool | 工具类库 | cn.hutool | hutool-all | 5.8.25 | 所有（或由根统一引入） | |
| MyBatis Plus | 持久化（Spring Boot 3） | com.baomidou | mybatis-plus-spring-boot3-starter | 3.5.15 | infra | 与 MySQL 搭配 |
| MySQL Driver | 数据库驱动 | com.mysql | mysql-connector-j | 由 Spring Boot BOM 管理 | infra | 不写 version |
| Redisson | 分布式锁/缓存 | org.redisson | redisson-spring-boot-starter | 3.24.3 | infra | |
| Knife4j | API 文档（OpenAPI 3） | com.github.xiaoymin | knife4j-openapi3-jakarta-spring-boot-starter | 4.4.0 | adapter、client、infra | |
| Jackson | JSON（与 Spring AI 兼容时统一） | com.fasterxml.jackson.core | jackson-databind / jackson-core / jackson-annotations | 2.18.4 | 按需 | 通常由 Spring Boot 带，需统一版本时可显式声明 |
| Flexmark Docx | Markdown 转 Word | com.vladsch.flexmark | flexmark-docx-converter | 0.64.8 | application | |
| JavaMail | 邮件发送 | com.sun.mail | javax.mail | 1.6.2 | infra | |
| Spring Security Crypto | 密码加密（如 BCrypt） | org.springframework.security | spring-security-crypto | 由 Spring Boot BOM 管理 | application | |
| Spring Boot Validation | 参数校验（@Valid） | org.springframework.boot | spring-boot-starter-validation | 由 Parent 管理 | client | |
| Spring Boot Web | HTTP | org.springframework.boot | spring-boot-starter-web | 由 Parent 管理 | adapter | |
| Spring Boot WebFlux | 响应式/SSE | org.springframework.boot | spring-boot-starter-webflux | 由 Parent 管理 | adapter | |
| Spring AI Alibaba Agent | 阿里 Agent 框架 | com.alibaba.cloud.ai | spring-ai-alibaba-agent-framework | 1.1.2.0 | infra | 按需 |
| Spring AI Alibaba DashScope | 通义模型 | com.alibaba.cloud.ai | spring-ai-alibaba-starter-dashscope | 1.1.2.0 | infra | 按需 |
| Spring AI OpenAI 兼容 | 兼容 OpenAI 的模型（如 Deepseek） | org.springframework.ai | spring-ai-starter-model-openai | 由 spring-ai-bom 管理 | infra | 按需 |
| Spring AI MCP Client | MCP 客户端 | org.springframework.ai | spring-ai-starter-mcp-client | 由 BOM 管理 | infra | 按需 |

---

## 二、根 pom 中需准备的版本变量与 dependencyManagement

在根 `pom.xml` 的 `<properties>` 中定义（若尚未存在）：

```xml
<lombok.version>1.18.30</lombok.version>
<fastjson2.version>2.0.47</fastjson2.version>
<hutool.version>5.8.25</hutool.version>
<mybatis-plus.version>3.5.15</mybatis-plus.version>
<redisson.version>3.24.3</redisson.version>
<knife4j.version>4.4.0</knife4j.version>
<jackson.version>2.18.4</jackson.version>
<flexmark-docx.version>0.64.8</flexmark-docx.version>
<mail.version>1.6.2</mail.version>
<spring-ai-alibaba.version>1.1.2.0</spring-ai-alibaba.version>
```

在 `<dependencyManagement><dependencies>` 中声明（子模块引用时不写 version）：

```xml
<!-- 通用 -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>${lombok.version}</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>com.alibaba.fastjson2</groupId>
    <artifactId>fastjson2</artifactId>
    <version>${fastjson2.version}</version>
</dependency>
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>${hutool.version}</version>
</dependency>

<!-- 持久化与缓存 -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-spring-boot3-starter</artifactId>
    <version>${mybatis-plus.version}</version>
</dependency>
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson-spring-boot-starter</artifactId>
    <version>${redisson.version}</version>
</dependency>

<!-- API 文档 -->
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-openapi3-jakarta-spring-boot-starter</artifactId>
    <version>${knife4j.version}</version>
</dependency>

<!-- 其他按需 -->
<dependency>
    <groupId>com.vladsch.flexmark</groupId>
    <artifactId>flexmark-docx-converter</artifactId>
    <version>${flexmark-docx.version}</version>
</dependency>
<dependency>
    <groupId>com.sun.mail</groupId>
    <artifactId>javax.mail</artifactId>
    <version>${mail.version}</version>
</dependency>
<dependency>
    <groupId>com.alibaba.cloud.ai</groupId>
    <artifactId>spring-ai-alibaba-agent-framework</artifactId>
    <version>${spring-ai-alibaba.version}</version>
</dependency>
<dependency>
    <groupId>com.alibaba.cloud.ai</groupId>
    <artifactId>spring-ai-alibaba-starter-dashscope</artifactId>
    <version>${spring-ai-alibaba.version}</version>
</dependency>
```

---

## 三、模块中引入方式示例

**规则**：在允许使用该 SDK 的模块的 `<dependencies>` 中只写 groupId、artifactId，不写 version。

- **infra 引入 MyBatis Plus + MySQL + Redisson**：
```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-spring-boot3-starter</artifactId>
</dependency>
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
</dependency>
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson-spring-boot-starter</artifactId>
</dependency>
```

- **adapter 引入 Knife4j**：
```xml
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-openapi3-jakarta-spring-boot-starter</artifactId>
</dependency>
```

- **client 引入 Validation**（版本由 Spring Boot Parent 管理）：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

- **application 引入 Flexmark**：
```xml
<dependency>
    <groupId>com.vladsch.flexmark</groupId>
    <artifactId>flexmark-docx-converter</artifactId>
</dependency>
```

---

## 四、按需求快速映射

| 用户需求 | 推荐 SDK | 引入模块 |
|----------|----------|----------|
| 数据库 / ORM / 持久化 | MyBatis Plus + mysql-connector-j | infra |
| 缓存 / 分布式锁 | Redisson | infra |
| API 文档 / Swagger | Knife4j | adapter（或 client/infra） |
| 参数校验 / @Valid | spring-boot-starter-validation | client |
| 密码加密 / BCrypt | spring-security-crypto | application |
| Markdown 转 Word | flexmark-docx-converter | application |
| 邮件 | javax.mail | infra |
| 大模型 / Agent | Spring AI 系列 | infra |
| 通用工具 / JSON | lombok、fastjson2、hutool | 根 pom 或各模块 |
