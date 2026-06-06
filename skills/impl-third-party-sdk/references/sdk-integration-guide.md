# SDK Integration Guide

---

## Root POM Version Variables & dependencyManagement

Define in root `pom.xml` `<properties>` (if missing):

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

Declare in root `<dependencyManagement><dependencies>` (child modules reference without version):

```xml
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
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-openapi3-jakarta-spring-boot-starter</artifactId>
    <version>${knife4j.version}</version>
</dependency>
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

## Module Import Examples (no version tag)

**infra** + MyBatis Plus + MySQL + Redisson:
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

**adapter** + Knife4j:
```xml
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-openapi3-jakarta-spring-boot-starter</artifactId>
</dependency>
```

**client** + Validation (version by Boot Parent):
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

**application** + Flexmark:
```xml
<dependency>
    <groupId>com.vladsch.flexmark</groupId>
    <artifactId>flexmark-docx-converter</artifactId>
</dependency>
```

---

## Quick SDK-to-Requirement Mapping

| User Need | Recommended SDK | Module |
|-----------|----------|---------|
| Database / ORM / Persistence | MyBatis Plus + mysql-connector-j | infra |
| Cache / Distributed Lock | Redisson | infra |
| API Doc / Swagger | Knife4j | adapter (or client/infra) |
| Param Validation / @Valid | spring-boot-starter-validation | client |
| Password Encrypt / BCrypt | spring-security-crypto | application |
| Markdown → Word | flexmark-docx-converter | application |
| Email | javax.mail | infra |
| LLM / Agent | Spring AI series | infra |
| General Tools / JSON | lombok, fastjson2, hutool | root or modules |
