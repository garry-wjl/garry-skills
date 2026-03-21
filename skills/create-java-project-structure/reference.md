# Reference: POM Templates and Optional package-info

Replace placeholders:

- `GROUP_ID` → e.g. `com.example`
- `ARTIFACT_ID` → e.g. `my-service`
- `VERSION` → e.g. `1.0.0-SNAPSHOT`
- `JAVA_VERSION` → e.g. `21`
- `BASE_PACKAGE` → same as GROUP_ID, e.g. `com.example`

---

## 1. Root pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.5.10</version>
        <relativePath/>
    </parent>

    <groupId>GROUP_ID</groupId>
    <artifactId>ARTIFACT_ID</artifactId>
    <version>VERSION</version>
    <packaging>pom</packaging>
    <name>ARTIFACT_ID</name>
    <description>Multi-module project (adapter, application, client, domain, facade, infra)</description>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>JAVA_VERSION</maven.compiler.source>
        <maven.compiler.target>JAVA_VERSION</maven.compiler.target>
        <java.version>JAVA_VERSION</java.version>
        <!-- 通用三方包版本（供各模块引用） -->
        <lombok.version>1.18.30</lombok.version>
        <fastjson2.version>2.0.47</fastjson2.version>
        <hutool.version>5.8.25</hutool.version>
        <!-- 存储与缓存（infra 使用，版本在根 pom 统一管理） -->
        <mybatis-plus.version>3.5.15</mybatis-plus.version>
        <redisson.version>3.24.3</redisson.version>
    </properties>

    <modules>
        <module>facade</module>
        <module>domain</module>
        <module>infra</module>
        <module>client</module>
        <module>application</module>
        <module>adapter</module>
    </modules>

    <dependencyManagement>
        <dependencies>
            <dependency><groupId>GROUP_ID</groupId><artifactId>facade</artifactId><version>${project.version}</version></dependency>
            <dependency><groupId>GROUP_ID</groupId><artifactId>domain</artifactId><version>${project.version}</version></dependency>
            <dependency><groupId>GROUP_ID</groupId><artifactId>infra</artifactId><version>${project.version}</version></dependency>
            <dependency><groupId>GROUP_ID</groupId><artifactId>client</artifactId><version>${project.version}</version></dependency>
            <dependency><groupId>GROUP_ID</groupId><artifactId>application</artifactId><version>${project.version}</version></dependency>
            <dependency><groupId>GROUP_ID</groupId><artifactId>adapter</artifactId><version>${project.version}</version></dependency>
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
            <!-- 存储与缓存（仅 infra 模块引用，版本在此统一管理） -->
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
        </dependencies>
    </dependencyManagement>

    <!-- 通用包：由根目录统一引入，子模块继承，不在各模块重复声明 -->
    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>com.alibaba.fastjson2</groupId>
            <artifactId>fastjson2</artifactId>
        </dependency>
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
        </dependency>
    </dependencies>

    <build>
        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <configuration>
                        <source>${java.version}</source>
                        <target>${java.version}</target>
                        <encoding>${project.build.sourceEncoding}</encoding>
                    </configuration>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>
</project>
```

---

## 2. facade/pom.xml

通用包由根 pom 统一引入并继承，各模块不再重复声明。

```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>GROUP_ID</groupId>
        <artifactId>ARTIFACT_ID</artifactId>
        <version>VERSION</version>
    </parent>
    <artifactId>facade</artifactId>
    <packaging>jar</packaging>
    <name>facade</name>
    <description>Facade - interface definitions</description>
</project>
```

---

## 3. domain/pom.xml

```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>GROUP_ID</groupId>
        <artifactId>ARTIFACT_ID</artifactId>
        <version>VERSION</version>
    </parent>
    <artifactId>domain</artifactId>
    <packaging>jar</packaging>
    <name>domain</name>
    <description>Domain - business domain logic</description>
    <dependencies>
        <dependency><groupId>GROUP_ID</groupId><artifactId>facade</artifactId></dependency>
    </dependencies>
</project>
```

---

## 4. infra/pom.xml

infra 除 **domain** 外，仅引入**存储（数据库）**与**缓存**相关 SDK；通用包（lombok、fastjson2、hutool）由根 pom 继承，不在此声明。

```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>GROUP_ID</groupId>
        <artifactId>ARTIFACT_ID</artifactId>
        <version>VERSION</version>
    </parent>
    <artifactId>infra</artifactId>
    <packaging>jar</packaging>
    <name>infra</name>
    <description>Infrastructure - technical implementations (DB, cache)</description>
    <dependencies>
        <dependency><groupId>GROUP_ID</groupId><artifactId>domain</artifactId></dependency>
        <!-- MyBatis-Plus（Spring Boot 3 专用） -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-spring-boot3-starter</artifactId>
        </dependency>
        <!-- MySQL Driver -->
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
        </dependency>
        <!-- Redisson -->
        <dependency>
            <groupId>org.redisson</groupId>
            <artifactId>redisson-spring-boot-starter</artifactId>
        </dependency>
    </dependencies>
</project>
```

---

## 5. client/pom.xml

```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>GROUP_ID</groupId>
        <artifactId>ARTIFACT_ID</artifactId>
        <version>VERSION</version>
    </parent>
    <artifactId>client</artifactId>
    <packaging>jar</packaging>
    <name>client</name>
    <description>Client - DTOs and external API client abstractions</description>
</project>
```

---

## 6. application/pom.xml

```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>GROUP_ID</groupId>
        <artifactId>ARTIFACT_ID</artifactId>
        <version>VERSION</version>
    </parent>
    <artifactId>application</artifactId>
    <packaging>jar</packaging>
    <name>application</name>
    <description>Application - application services and use cases</description>
    <dependencies>
        <dependency><groupId>GROUP_ID</groupId><artifactId>domain</artifactId></dependency>
        <dependency><groupId>GROUP_ID</groupId><artifactId>infra</artifactId></dependency>
        <dependency><groupId>GROUP_ID</groupId><artifactId>client</artifactId></dependency>
    </dependencies>
</project>
```

---

## 7. adapter/pom.xml

```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>GROUP_ID</groupId>
        <artifactId>ARTIFACT_ID</artifactId>
        <version>VERSION</version>
    </parent>
    <artifactId>adapter</artifactId>
    <packaging>jar</packaging>
    <name>adapter</name>
    <description>Adapter - HTTP controllers, event consumers</description>
    <dependencies>
        <dependency><groupId>GROUP_ID</groupId><artifactId>application</artifactId></dependency>
        <dependency><groupId>GROUP_ID</groupId><artifactId>client</artifactId></dependency>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-web</artifactId></dependency>
    </dependencies>
</project>
```

---

## 8. Optional package-info.java (for empty modules to compile)

If you want `mvn compile` to succeed without adding business code, add one `package-info.java` per module under the module’s base package. Example for `facade` (package BASE_PACKAGE.facade):

```java
/**
 * Facade - interface definitions.
 */
package BASE_PACKAGE.facade;
```

Do the same for `BASE_PACKAGE.domain`, `BASE_PACKAGE.infra`, `BASE_PACKAGE.client`, `BASE_PACKAGE.application`, `BASE_PACKAGE.adapter`. No other .java files.
