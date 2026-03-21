# 各模块可选第三方依赖清单

供各模块 Skill 在编写或校验 pom 时选择使用。版本以父 POM 的 `dependencyManagement` 为准，子模块引用时不写版本。

---

## 一、父 POM 统一管理的依赖与版本

| groupId | artifactId | 版本变量 / 版本 | 说明 |
|---------|------------|-----------------|------|
| org.projectlombok | lombok | 1.18.30 | 通用，provided |
| com.alibaba.fastjson2 | fastjson2 | 2.0.47 | JSON |
| cn.hutool | hutool-all | 5.8.25 | 工具库 |
| com.fasterxml.jackson.core | jackson-databind | 2.18.4 | Jackson（与 Spring AI 兼容时统一版本） |
| com.fasterxml.jackson.core | jackson-core | 2.18.4 | |
| com.fasterxml.jackson.core | jackson-annotations | 2.18.4 | |
| com.github.xiaoymin | knife4j-openapi3-jakarta-spring-boot-starter | 4.4.0 | API 文档 |
| com.baomidou | mybatis-plus-spring-boot3-starter | 3.5.15 | 持久化（仅 infra） |
| org.redisson | redisson-spring-boot-starter | 3.24.3 | 分布式锁/缓存（仅 infra） |
| com.vladsch.flexmark | flexmark-docx-converter | 0.64.8 | MD 转 Word（按需） |
| com.sun.mail | javax.mail | 1.6.2 | 邮件（按需） |
| com.alibaba.cloud.ai | spring-ai-alibaba-agent-framework | 1.1.2.0 | Spring AI 阿里（按需） |
| com.alibaba.cloud.ai | spring-ai-alibaba-starter-dashscope | 1.1.2.0 | DashScope（按需） |
| org.springframework.ai | spring-ai-bom | 1.1.2 | BOM import |

说明：Spring Boot、Spring Web、MySQL 驱动等由 Spring Boot Parent 或 BOM 管理，此处不重复列。

---

## 二、按模块选择依赖

### 1. facade

**仅允许使用「通用 SDK」**，不引入 Spring、持久化、文档等。

| groupId | artifactId | 说明 |
|---------|------------|------|
| org.projectlombok | lombok | 推荐 |
| com.alibaba.fastjson2 | fastjson2 | 推荐 |
| cn.hutool | hutool-all | 推荐 |

---

### 2. domain

**仅允许：facade + 通用 SDK**。不引入 Spring、MyBatis、Knife4j、Redis 等。

| groupId | artifactId | 说明 |
|---------|------------|------|
| com.garry | facade | 必须（本工程） |
| org.projectlombok | lombok | 推荐 |
| com.alibaba.fastjson2 | fastjson2 | 推荐 |
| cn.hutool | hutool-all | 推荐 |

---

### 3. client

**允许：通用 SDK + 校验 + API 文档注解**。不引入 Spring Web、domain、infra、facade 业务。

| groupId | artifactId | 说明 |
|---------|------------|------|
| org.projectlombok | lombok | 推荐 |
| com.alibaba.fastjson2 | fastjson2 | 推荐 |
| cn.hutool | hutool-all | 推荐 |
| org.springframework.boot | spring-boot-starter-validation | 推荐（@Valid 等） |
| com.github.xiaoymin | knife4j-openapi3-jakarta-spring-boot-starter | 可选（Swagger/OpenAPI 注解） |

---

### 4. infra

**允许：domain + 通用 SDK + 持久化/缓存/文档/邮件/Spring AI 等基础设施**。

| groupId | artifactId | 说明 |
|---------|------------|------|
| com.garry | domain | 必须（本工程） |
| org.projectlombok | lombok | 推荐 |
| com.alibaba.fastjson2 | fastjson2 | 推荐 |
| cn.hutool | hutool-all | 推荐 |
| com.baomidou | mybatis-plus-spring-boot3-starter | 推荐（持久化） |
| com.mysql | mysql-connector-j | 推荐（由 Spring Boot BOM 管理版本） |
| org.redisson | redisson-spring-boot-starter | 可选（分布式锁/缓存） |
| com.github.xiaoymin | knife4j-openapi3-jakarta-spring-boot-starter | 可选（API 文档） |
| com.sun.mail | javax.mail | 按需（邮件） |
| com.alibaba.cloud.ai | spring-ai-alibaba-agent-framework | 按需 |
| org.springframework.ai | spring-ai-starter-model-openai | 按需 |
| org.springframework.ai | spring-ai-starter-mcp-client | 按需 |

---

### 5. application

**允许：domain + infra + client + 通用 SDK + 安全/文档转换等应用层所需**。

| groupId | artifactId | 说明 |
|---------|------------|------|
| com.garry | domain | 必须 |
| com.garry | infra | 必须 |
| com.garry | client | 必须 |
| org.projectlombok | lombok | 推荐 |
| com.alibaba.fastjson2 | fastjson2 | 推荐 |
| cn.hutool | hutool-all | 推荐 |
| org.springframework.security | spring-security-crypto | 按需（如 BCrypt） |
| com.vladsch.flexmark | flexmark-docx-converter | 按需（MD 转 Word） |

---

### 6. adapter

**允许：application + client + Web + 通用 SDK + API 文档**。不直接依赖 domain/infra。

| groupId | artifactId | 说明 |
|---------|------------|------|
| com.garry | application | 必须 |
| com.garry | client | 必须 |
| org.springframework.boot | spring-boot-starter-web | 推荐 |
| org.springframework.boot | spring-boot-starter-webflux | 按需（如 SSE/WebClient） |
| com.github.xiaoymin | knife4j-openapi3-jakarta-spring-boot-starter | 推荐（API 文档） |
| org.projectlombok | lombok | 推荐 |
| com.alibaba.fastjson2 | fastjson2 | 推荐 |
| cn.hutool | hutool-all | 推荐 |

---

## 三、Maven 片段示例（父 POM dependencyManagement）

子模块引用时不再写 `<version>`，由父 POM 统一管理。

```xml
<!-- 父 pom.xml properties -->
<lombok.version>1.18.30</lombok.version>
<fastjson2.version>2.0.47</fastjson2.version>
<hutool.version>5.8.25</hutool.version>
<mybatis-plus.version>3.5.15</mybatis-plus.version>
<redisson.version>3.24.3</redisson.version>
<knife4j.version>4.4.0</knife4j.version>
<jackson.version>2.18.4</jackson.version>
<flexmark-docx.version>0.64.8</flexmark-docx.version>
<mail.version>1.6.2</mail.version>

<!-- dependencyManagement 中声明 -->
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
<!-- 其余依赖以项目根 pom 的 dependencyManagement 为准 -->
```

---

## 四、使用说明（供各模块 Skill 引用）

- 实现某模块时，仅从**该模块对应小节**中选择依赖，避免把 infra 专属依赖加入 facade/domain、或把 adapter 专属依赖加入 client。
- 新增第三方包时，优先在父 POM 的 `dependencyManagement` 中声明版本，再在对应模块的 `dependencies` 中引用。
- 若项目裁剪了 Spring AI、邮件等，则对应模块不选即可。
