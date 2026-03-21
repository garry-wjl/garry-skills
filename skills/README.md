# Skills 分类索引

本目录下的技能分为两大类，便于按用途选用。

**约定**：设计类与代码编写类技能在**代码仓库中已存在相关代码**时，须先参考现有代码（包结构、命名、接口与实现风格），再设计方案或落码，保持与仓库一致。

## 一、设计类

| 技能名称 | 说明 |
|----------|------|
| **design-prd** | 编写 PRD（产品需求文档）：产出结构化需求文档，作为 **design-technical-solution** 的输入。 |
| **design-technical-solution** | 编写技术方案文档：以 PRD 为输入，产出含模块变更清单、实现顺序、接口与数据契约的文档，与代码编写类技能一一对应。 |
| **design-enterprise-ui-antd** | 企业级应用 UI 设计：以 Ant Design 为框架，创建独立 UI 工程；涉及 Agent 会话时使用 Ant Design X；色调与风格基于 Ant Design 配色方案供用户选择。 |

## 二、代码编写类

| 技能名称 | 说明 |
|----------|------|
| **impl-facade-module** | 实现 facade 模块（DomainEntity、DomainEventDTO、DomainEventPublisher、Result 等）。 |
| **impl-client-module** | 实现 client 模块（Param、DTO、VO、Result、API 契约）。 |
| **impl-domain-module** | 实现 domain 模块（聚合、实体、Repository/Factory/Gateway 接口、值对象）。 |
| **impl-infra-module** | 实现 infra 模块（Entity、Mapper、Repository/Factory/Gateway 实现、common 下常量/异常/事件等）。 |
| **impl-application-module** | 实现 application 模块（CommandService、QueryService、StreamService 或 Agent 相关子包）。 |
| **impl-adapter-module** | 实现 adapter 模块（Controller、listener、config）。 |
| **impl-from-technical-solution** | 根据技术方案文档按实现顺序编排上述 impl-*-module 与 impl-third-party-sdk，完成代码编写。 |
| **impl-third-party-sdk** | 选择并引入第三方 SDK（根 pom 与各模块 pom 依赖配置）。 |

## 其他

| 技能名称 | 说明 |
|----------|------|
| **create-java-project-structure** | 创建 Java 项目结构。 |

- 各模块可选依赖清单见 [module-dependencies.md](module-dependencies.md)。
