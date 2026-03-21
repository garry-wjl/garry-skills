---
name: create-java-project-structure
description: Scaffolds a Java multi-module Maven project with six layers (adapter, application, client, domain, facade, infra) and no business code. Use when the user asks to create a new Java project structure, scaffold backend skeleton, or create only directories and POMs for adapter/application/client/domain/facade/infra.
---

# Create Java Project Structure (Skeleton Only)

Scaffolds a **structure-only** Java multi-module project: six modules, directory tree, and minimal POMs. **No Java source files and no business subpackages** (e.g. no conversation, message, user modules). Business code is added later by the user.

## Workflow: Information First, Then Create

**必须严格按顺序执行**：先收集用户提供的创建信息，确认无误后再生成项目结构。

1. **Step 1：收集创建项目所需信息（先执行）**
   - 在生成任何文件或目录之前，向用户索要或确认以下三项信息。
   - 若用户已在一句话中给出部分信息，可只追问未给出的项；若一项都未给出，则一次性列出三项请用户填写。
   - **未收集齐三项前，禁止执行 Step 2，禁止创建或写入任何文件。**

2. **Step 2：生成项目结构（后执行）**
   - 仅在获得并确认 groupId、artifactId、rootDir 之后，再根据 [reference.md](reference.md) 创建根 POM、六模块 POM 及目录树。

**向用户索要信息时**，可这样表述（中英文皆可）：

- “创建项目结构前需要您提供以下信息：1) **groupId**（Maven 组 ID，如 `com.example`，将作为 Java 包前缀）；2) **artifactId**（项目名，如 `my-service`）；3) **rootDir**（项目根目录，如 `service-api` 或当前目录 `.`）。请逐项提供或一次性给出。”
- 用户回复后，用简短列表复述 groupId、artifactId、rootDir，并询问“以上信息无误则开始生成项目结构”；确认后再执行创建。

**创建前自检**（三项均满足后才可执行 Step 2）：
- [ ] 已获得或推断出 `groupId`
- [ ] 已获得或推断出 `artifactId`
- [ ] 已获得或推断出 `rootDir`（未提供时可用当前工作目录 `.` 并告知用户）

## When to Use

- User asks to "创建 Java 项目结构"、"搭建后端骨架"、"只生成项目结构不要业务代码"等.
- User specifies the six-layer layout (adapter, application, client, domain, facade, infra) and wants only directories and POMs.

## Parameters to Collect（创建前必填）

在 **Step 1** 中向用户收集或确认，并满足后再进入 Step 2：

| Parameter    | Example    | Note |
|-------------|------------|------|
| `groupId`   | `com.example` | Maven groupId；同时作为 Java 包前缀（base package）。 |
| `artifactId`| `my-service`  | 项目名称，即父 POM 的 artifactId。 |
| `rootDir`   | `service-api` 或 `.` | 项目根目录（相对或绝对路径），在其下创建六模块及 pom.xml。 |

Base package = groupId 按点分隔（如 `com.example`）；目录路径 = groupId 中的点改为斜杠（如 `com/example`）。

## Module Order and Dependencies

Build order (Maven reactor): **facade → domain → infra → client → application → adapter**.

Dependency rules (no reverse or cross-layer):

- **adapter** → application
- **application** → client, domain, infra
- **domain** → facade
- **infra** → domain
- **client** → (none)
- **facade** → (none)

## What to Generate

1. **Root POM** (`pom.xml` at rootDir)
   - `packaging`: `pom`
   - `modules`: facade, domain, infra, client, application, adapter (order above)
   - **通用三方包**：在根 pom 的 `dependencyManagement` 中声明版本，并在根 pom 的 `dependencies` 中**直接引入** **lombok**、**fastjson2**、**hutool-all**，子模块通过继承获得，**不在各模块中重复声明**。
   - 存储/缓存的版本在 `dependencyManagement` 中声明（mybatis-plus、redisson），供 infra 引用。
   - 具体片段见 [reference.md](reference.md) 第 1 节。

2. **Six module POMs**
   - Each: `parent` = root project, `artifactId` = module name, `packaging` = jar.
   - **通用包**：各模块均不再引入 lombok、fastjson2、hutool-all，由根 pom 继承。
   - **infra 模块**：除 **domain** 外，仅引入**存储与缓存** SDK：`mybatis-plus-spring-boot3-starter`、`mysql-connector-j`、`redisson-spring-boot-starter`。

3. **Directory tree only (no .java files)**
   - Each module: `src/main/java/<basePackagePath>/<module>/` and `src/main/resources/`.
   - Optional placeholder packages (no .java): e.g. `domain/common`, `infra/common/constant`, `infra/common/util`, `adapter/config`.
   - Use empty directories; optionally add `.gitkeep` under empty dirs so Git tracks them.

Do **not** create:

- Any `.java` file (no controllers, entities, services, repositories).
- Business subpackages (e.g. no `conversation`, `message`, `task`, `user`, `report`, `auth`, `agent`).

## Directory Layout (Template)

Replace `<basePackagePath>` with groupId path (e.g. `com/example`), `<artifactId>` with root project name:

```
<rootDir>/
├── pom.xml
├── facade/
│   ├── pom.xml
│   └── src/main/
│       ├── java/<basePackagePath>/facade/
│       └── resources/
├── domain/
│   ├── pom.xml
│   └── src/main/
│       ├── java/<basePackagePath>/domain/
│       └── resources/
├── infra/
│   ├── pom.xml
│   └── src/main/
│       ├── java/<basePackagePath>/infra/
│       ├── resources/
│       └── resources/db/migration/
├── client/
│   ├── pom.xml
│   └── src/main/
│       ├── java/<basePackagePath>/client/
│       └── resources/
├── application/
│   ├── pom.xml
│   └── src/main/
│       ├── java/<basePackagePath>/application/
│       └── resources/
└── adapter/
    ├── pom.xml
    └── src/main/
        ├── java/<basePackagePath>/adapter/
        └── resources/
```

Optional: under `infra` add `infra/common/constant`, `infra/common/util`; under `domain` add `domain/common`; under `adapter` add `adapter/config`. Still no .java files.

## Verification

After generation:

- Run `mvn clean compile -f <rootDir>/pom.xml`. It may fail if no sources (empty modules); then add a single empty `package-info.java` per module or skip with `maven.compiler.failOnMissingSources=false` in root POM for skeleton. Prefer adding minimal `package-info.java` in each module’s package so `mvn compile` succeeds.

## 使用完成后的最后一步：更新知识图谱

- **每次使用本技能完成交付后，最后一步须更新知识图谱**。根据本次产出（如新建的项目 groupId/artifactId、根目录），创建或更新 ontology 中的实体与关系（如 Project、Document；`part_of` 等），或调用 **ontology** 技能、或向 `memory/ontology/graph.jsonl` 追加操作记录，使图谱与项目当前状态一致。详见 [ontology/SKILL.md](../ontology/SKILL.md)。

## Reference

- Exact POM templates and optional package-info content: [reference.md](reference.md).
