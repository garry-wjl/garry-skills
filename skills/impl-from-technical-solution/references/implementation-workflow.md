# Technical Solution to Implementation Workflow

## Layer & Skill Correspondence

| Layer | Corresponding Skill | Typical Changes |
|-------|-------------------|-----------------|
| facade | impl-facade-module | DomainEntity, DomainEventDTO, DomainEventPublisher, CommonRequest, Result |
| client | impl-client-module | Param, DTO, VO, Result, API contracts, constants |
| domain | impl-domain-module | Aggregates, entities, Repository/Factory/Gateway **interfaces**, valueobject, domain events |
| infra | impl-infra-module | Entity, Mapper, Repository/Factory/Gateway **implementations**, common/constant/event/util/client |
| application | impl-application-module | CommandService, QueryService, StreamService, config/hook/interceptor/service/tool |
| adapter | impl-adapter-module | Controller, listener, config (BaseController, GlobalExceptionHandler, TokenFilter) |

---

## Default Implementation Order

Per dependency relationships, recommended:

1. **facade** (if plan changes needed)
2. **client**
3. **domain**
4. **infra**
5. **application**
6. **adapter**

If plan only involves specific layers, execute only the layers listed in plan's "implementation order".

---

## When to Call impl-third-party-sdk

- Plan "other" section or body mentions: new database, cache, HTTP client, doc generation, email, validation lib, etc. **third-party dependencies**.
- Current layer needs SDK not yet in project (e.g., infra needs MyBatis Plus, application needs Spring Security Crypto).
- **Call timing**: Before implementing the layer that needs the dependency (or as first step of that layer); configure dependency first, then code that layer.

---

## Extracting Technical Plan Key Points

From plan document, extract:

1. **Goals & Scope**: Understanding boundary; avoid unrelated implementation.
2. **Architecture & Layers**: Which layers, which domains (conversation/message/auth/agent, etc.); determines package/class names.
3. **Module Changes List**: Table or list; each row has "layer" + "changes" + (optional) "corresponding skill"; if no skill noted, use layer-skill table above to infer.
4. **Implementation Order**: Ordered list like "1. facade 2. client 3. domain..."; if absent, use default order above.
5. **Interface & Data Contract**: Paths, methods, input/output types (Param, VO, Result); used for client & adapter class naming & fields.
6. **Other**: Table structures (→ infra Entity/Mapper), config items, external dependencies (→ trigger impl-third-party-sdk).

---

## Single-Layer Execution Example (domain)

When implementation order reaches **domain** with changes "new Xxx aggregate, XxxRepository, XxxFactory":

1. Open **impl-domain-module** SKILL.md & references/; confirm common subdirectory, domain subdirectory, repository/factory/valueobject conventions.
2. Create/modify in domain: domain package `xxx/`, root entity `Xxx`, `repository/XxxRepository`, `factory/XxxFactory` (and valueobject if needed).
3. Entity extends DomainEntity, implements domainValidate/save/delete; Repository/Factory define interfaces only, no implementation.
4. Complete; move to next layer (infra) to implement interfaces & persistence.

Other layers same pattern: follow corresponding impl-*-module rules & Cases for that layer's changes; advance to next layer.
