# Reference: 技术方案 → 实现技能映射与流程

## 层级与 Skill 对应表

| 层级 | 对应 Skill | 典型变更项 |
|------|------------|------------|
| facade | impl-facade-module | DomainEntity、DomainEventDTO、DomainEventPublisher、CommonRequest、Result |
| client | impl-client-module | Param、DTO、VO、Result、API 契约、常量 |
| domain | impl-domain-module | 聚合、实体、Repository/Factory/Gateway 接口、valueobject、领域事件常量 |
| infra | impl-infra-module | Entity、Mapper、Repository/Factory/Gateway 实现、common 下 constant/event/exception/util/client |
| application | impl-application-module | CommandService、QueryService、StreamService、Agent 的 config/hook/interceptor/service/tool |
| adapter | impl-adapter-module | Controller、listener、config（BaseController、GlobalExceptionHandler、TokenValidationFilter 等） |

## 实现顺序（默认）

按依赖关系，推荐顺序为：

1. **facade**（若方案中有变更）
2. **client**
3. **domain**
4. **infra**
5. **application**
6. **adapter**

若方案仅涉及部分层，只按方案中的「实现顺序」执行列出的层。

## 何时调用 impl-third-party-sdk

- 方案中「其他」或正文提到：新增数据库、缓存、HTTP 客户端、文档生成、邮件、校验库等**第三方依赖**。
- 当前层实现需要某 SDK 但项目尚未引入（如 infra 用 MyBatis Plus、application 用 Spring Security Crypto）。
- 调用时机：在**需要该依赖的那一层实现之前**（或该层实现的第一步）先完成依赖配置，再写该层代码。

## 解析技术方案时的提取要点

从文档中尽量提取：

- **1. 目标与范围**：用于理解边界，避免实现无关内容。
- **2. 架构与分层**：涉及哪些层、哪些领域（conversation/message/auth/agent 等），用于确定包名与类名。
- **3. 模块变更清单**：表格或列表，每行包含「层级」+「变更项」+（可选）「对应 Skill」；无「对应 Skill」时用本 reference 的层级与 Skill 对应表推断。
- **4. 实现顺序**：有序列表，如「1. facade 2. client 3. domain …」；若无，按默认顺序执行。
- **5. 接口与数据契约**：路径、方法、入参/出参类型（Param、VO、Result），用于 client 与 adapter 的类名与字段。
- **6. 其他**：表结构（给 infra Entity/Mapper）、配置项、外部依赖（触发 impl-third-party-sdk）。

## 单层执行示例（domain）

当实现顺序轮到 **domain** 且变更清单为「新增 Xxx 聚合、XxxRepository、XxxFactory」时：

1. 打开 **impl-domain-module** 的 SKILL.md 与 reference.md，确认 common 子包、领域子包、repository/factory/valueobject 等约定。
2. 在 domain 下新建或修改：领域包 `xxx/`、根实体 `Xxx`、`repository/XxxRepository`、`factory/XxxFactory`（及 valueobject 若需要）。
3. 实体继承 DomainEntity，实现 domainValidate/save/delete；Repository/Factory 仅定义接口，不写实现。
4. 完成后再进入下一层（infra）实现对应接口与持久化。

其他层同理：按对应 impl-*-module 的规则与 Case 完成该层变更，再进入下一层。
