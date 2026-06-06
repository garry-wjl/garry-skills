# Optional SDK Checklist and Introduction

New SDK: declare version in root pom `dependencyManagement`, then reference in module `dependencies` (no version tag).

---

## SDK Checklist

| Name | Purpose | groupId | artifactId | Recommended | Allowed Modules | Note |
|------|---------|---------|------------|----------|----------|------|
| Lombok | Simplify getter/setter/constructor | org.projectlombok | lombok | 1.18.30 | All (or unified in root) | scope=provided |
| Fastjson2 | JSON serialization | com.alibaba.fastjson2 | fastjson2 | 2.0.47 | All (or unified in root) | |
| Hutool | Utility library | cn.hutool | hutool-all | 5.8.25 | All (or unified in root) | |
| MyBatis Plus | Persistence (Spring Boot 3) | com.baomidou | mybatis-plus-spring-boot3-starter | 3.5.15 | infra | Pair with MySQL |
| MySQL Driver | DB driver | com.mysql | mysql-connector-j | Managed by Boot BOM | infra | No version tag |
| Redisson | Distributed lock/cache | org.redisson | redisson-spring-boot-starter | 3.24.3 | infra | |
| Knife4j | API doc (OpenAPI 3) | com.github.xiaoymin | knife4j-openapi3-jakarta-spring-boot-starter | 4.4.0 | adapter, client, infra | |
| Jackson | JSON (align with Spring AI) | com.fasterxml.jackson.core | jackson-databind / databind / annotations | 2.18.4 | Per need | Usually brought by Boot; version unification optional |
| Flexmark Docx | Markdown → Word | com.vladsch.flexmark | flexmark-docx-converter | 0.64.8 | application | |
| JavaMail | Email | com.sun.mail | javax.mail | 1.6.2 | infra | |
| Spring Security Crypto | Password encryption (BCrypt) | org.springframework.security | spring-security-crypto | Boot BOM managed | application | |
| Spring Boot Validation | Param validation (@Valid) | org.springframework.boot | spring-boot-starter-validation | Parent managed | client | |
| Spring Boot Web | HTTP | org.springframework.boot | spring-boot-starter-web | Parent managed | adapter | |
| Spring Boot WebFlux | Reactive/SSE | org.springframework.boot | spring-boot-starter-webflux | Parent managed | adapter | |
| Spring AI Alibaba Agent | Alibaba Agent framework | com.alibaba.cloud.ai | spring-ai-alibaba-agent-framework | 1.1.2.0 | infra | Per need |
| Spring AI Alibaba DashScope | Tongyi models | com.alibaba.cloud.ai | spring-ai-alibaba-starter-dashscope | 1.1.2.0 | infra | Per need |
| Spring AI OpenAI Compatible | OpenAI-compatible models (Deepseek) | org.springframework.ai | spring-ai-starter-model-openai | BOM managed | infra | Per need |
| Spring AI MCP Client | MCP client | org.springframework.ai | spring-ai-starter-mcp-client | BOM managed | infra | Per need |
