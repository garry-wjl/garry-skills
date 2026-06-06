# Module Order, Dependencies and Directory Structure

## Module Order and Dependencies

Build order (Maven reactor): **facade → domain → infra → client → application → adapter**.

Dependency rules (no reverse or cross-layer):

- **adapter** → application
- **application** → client, domain, infra
- **domain** → facade
- **infra** → domain
- **client** → (none)
- **facade** → (none)

---

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

---

## Optional package-info.java (for empty modules to compile)

If you want `mvn compile` to succeed without adding business code, add one `package-info.java` per module under the module's base package. Example for `facade` (package BASE_PACKAGE.facade):

```java
/**
 * Facade - interface definitions.
 */
package BASE_PACKAGE.facade;
```

Do the same for `BASE_PACKAGE.domain`, `BASE_PACKAGE.infra`, `BASE_PACKAGE.client`, `BASE_PACKAGE.application`, `BASE_PACKAGE.adapter`. No other .java files.
