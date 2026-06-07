---
name: impl-from-technical-solution
description: Implements code from a technical solution document by orchestrating impl-*-module skills (facade, client, domain, infra, application, adapter) and impl-third-party-sdk in dependency order. Use when the user provides a technical solution document and asks to implement it, write the code, or 按方案落地.
---

# 根据技术方案编写代码

**分类**：代码编写类

以**技术方案文档**为输入，按文档中的**实现顺序**与**模块变更清单**，依次调用对应的 **impl-*-module** 与 **impl-third-party-sdk** 技能，协作完成六层代码编写。方案文档结构应与**技术方案设计类**技能 **design-technical-solution** 产出一致（含模块变更清单、实现顺序、接口与数据契约等）；若方案不符合该结构，可先用 design-technical-solution 补全或重写。

## 参考现有代码

**若代码仓库中已存在相关层或领域代码**，每一层实现前须先阅读该层现有代码（包结构、类命名、已有接口与实现风格），在现有基础上扩展或修改，保持与仓库代码一致；不脱离现有代码另起一套命名或结构。调用各 impl-*-module 时，同样遵循「先看仓库再落码」的约定。

## When to Use

- 用户**已提供技术方案文档**，并要求「按方案实现」、「落地该方案」、「根据方案写代码」。
- 输入为一份技术方案（Markdown 或粘贴内容），需按层完成代码实现。

## 前置条件

- 技术方案文档应包含（或可推断）：
  - **模块变更清单**：各层（facade/client/domain/infra/application/adapter）的变更项。
  - **实现顺序**：建议的落码顺序（通常 facade → client → domain → infra → application → adapter）。
  - **接口与数据契约**（若有）：API 路径、入参/出参类型，便于 client 与 adapter 对齐。
- 若文档缺少上述结构或难以解析，先使用**技术方案设计类**技能 **design-technical-solution** 补全或重写方案，再执行本技能。

## 执行流程

1. **解析技术方案**
   - 从文档中提取：目标与范围、涉及层次与领域、**模块变更清单**、**实现顺序**、接口与数据契约、其他（表结构、外部依赖等）。
   - 若文档中有「对应 Skill」列，直接按该列确定每层使用的 skill；否则按「变更类型 → Skill」映射（见 [references/implementation-workflow.md](references/implementation-workflow.md)）确定。

2. **按实现顺序逐层实现**
   - 严格按文档中的**实现顺序**执行，不跳过、不逆序。
   - 对**每一层**：
     - 根据**模块变更清单**中该层的变更项，**调用对应的 impl-*-module 技能**完成代码（读取该技能的 SKILL.md 与 reference.md，按其中规则与 Case 实现）。
     - 若该层无变更（如「无」或空），则跳过。
   - **技能对应关系**：
     - facade 变更 → **impl-facade-module**
     - client 变更 → **impl-client-module**
     - domain 变更 → **impl-domain-module**
     - infra 变更 → **impl-infra-module**
     - application 变更 → **impl-application-module**
     - adapter 变更 → **impl-adapter-module**

3. **依赖与三方包**
   - 若方案中「其他」或「接口与数据契约」提到**新增第三方库、依赖、SDK**（如新数据库驱动、新 HTTP 客户端、新工具库），在**该层实现前或首次用到时**调用 **impl-third-party-sdk** 技能，在根 pom 与对应模块 pom 中完成依赖配置，再继续该层代码。

4. **收尾**
   - 完成所有列出的层后，可简要核对：清单中的变更是否均已实现、接口与数据契约是否与 client/adapter 一致。

## 协作约定

- **单层单技能**：每一层只由对应的一个 impl-*-module 负责，不在 application 里写 domain 逻辑、不在 adapter 里写 application 逻辑。
- **顺序不可逆**：先 facade/client/domain，再 infra，再 application，最后 adapter；避免未定义接口就写实现、未定义 Param 就写 Controller。
- **领域命名一致**：文档中的领域/聚合名（如 conversation、message）在各层包名、类名中保持一致，与各 impl-* skill 的包结构约定一致。

## 使用完成后的最后一步：更新知识图谱

- **每次使用本技能完成交付后，最后一步须更新知识图谱**。根据本次产出（如按技术方案完成的各层实现、对应方案文档），创建或更新 ontology 中的实体与关系（如 Project、Document、Task；`part_of`、`has_task` 等），或调用 **ontology** 技能、或向 `memory/ontology/graph.jsonl` 追加操作记录，使图谱与项目当前状态一致。详见 [ontology/SKILL.md](../ontology/SKILL.md)。



## Spring Bean 注入约束

- **必须使用 `@Resource` 注入**：在 Spring 中注入 Bean 时，必须使用 `@Resource` 注解进行字段注入；禁止使用 `@Autowired`、构造器注入、Setter 注入或通过 `ApplicationContext` 手动获取 Bean。若某模块规则进一步限制可注入类型（如 RepositoryImpl 仅允许注入 Mapper），则同时遵守该模块的更严格约束。

## 后端基础工具优先级

- **Hutool 优先**：在后端开发过程中，凡涉及字符串判断、集合判空、对象判空、日期处理、类型转换、JSON 辅助、加解密、随机值、ID 生成等基础工具操作，务必优先使用 **Hutool**（如 `StrUtil`、`CollUtil`、`ObjectUtil`、`DateUtil`、`Convert` 等）。只有在 Hutool 中找不到合适能力或无法满足业务/性能/安全要求时，才考虑 JDK 原生工具、Spring 工具类或其他第三方方案。

## Reference

- 变更类型与 impl-* skill 的映射、示例流程见 [references/implementation-workflow.md](references/implementation-workflow.md)。
