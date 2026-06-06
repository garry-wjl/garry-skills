# Infra Package Structure Convention

Audit fields & project convention match (create_no, update_no, create_time, update_time, is_deleted); no foreign keys.

---

## Package Structure Rules

- **common subdirectory (必须)**: Stores cross-domain constants, events, exceptions, utils, and **external API calls**:
  - `infra/common/constant` — e.g., DeleteFlagConstant, LockKeyConstant
  - `infra/common/event` — e.g., CommonDomainEventPublisher
  - `infra/common/exception` — e.g., LockException
  - `infra/common/util` — e.g., JsonUtils
  - **`infra/common/client`** — **client placed in infra common subdirectory**. External HTTP/API calls completed here: **interface definition** in client; **client/param** for interface params; **client/dto** for API-specific request/response DTOs; implementation possible in client or elsewhere; domain gateway injects interface, calls within gateway; param/dto not exposed to domain, gateway does param/dto ↔ domain object conversion.

- **Domain-based subdirectories**: One domain one subdirectory; **no client subdirectory in domain-based subdirectories**; each domain subdirectory contains only: entity, mapper, repository, factory, gateway. External API calls centralized in common/client.

**Example directory tree:**
```
infra/
├── common/
│   ├── constant/
│   ├── event/
│   ├── exception/
│   ├── util/
│   └── client/          ← External HTTP/API: interface + param + dto
│       ├── param/       ← Interface params
│       ├── dto/         ← API-specific request/response DTOs
│       └── (interface & implementation classes)
├── conversation/        ← Domain subdirectory has NO client
│   ├── entity/
│   ├── factory/
│   ├── gateway/
│   ├── mapper/
│   └── repository/
├── message/
│   ├── entity/
│   ├── factory/
│   ├── gateway/
│   ├── mapper/
│   └── repository/
└── ...
```
