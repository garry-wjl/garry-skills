# CLAUDE.md - garry-skills 项目指南

## 项目概述

**garry-skills** 是一个 Claude AI 技能库，旨在帮助开发者高效地完成 Java 企业级应用的**产品设计**和**代码实现**。该项目包含两大类技能模块：设计类和代码编写类。

- **Repository**: https://github.com/garry-wjl/garry-skills
- **License**: Apache 2.0
- **Language**: 中文文档 + 代码框架

## 核心设计原则

### 1. 代码一致性约定
在设计类与代码编写类技能中，**必须**遵循以下约定：
- 如果代码仓库中已存在相关代码，须**先参考现有代码**（包结构、命名、接口与实现风格）
- 再设计方案或落码，确保与仓库整体一致
- 不允许引入不一致的代码风格或架构模式

### 2. 技能分层结构
技能分为三层：

**第一层：需求设计**
- `design-prd`: 编写产品需求文档（PRD）
- 输出：结构化的业务需求

**第二层：技术方案**
- `design-technical-solution`: 编写技术方案文档
- 输入：PRD
- 输出：模块变更清单、实现顺序、接口与数据契约

**第三层：代码实现**
- 7 个 `impl-*-module` 技能，按 DDD 架构分层
- 1 个 `impl-from-technical-solution` 技能，用于按顺序编排实现
- 1 个 `impl-third-party-sdk` 技能，用于 SDK 集成

### 3. DDD 架构层级

项目遵循 **Domain-Driven Design (DDD)** 架构，按以下层级组织代码：

| 层级 | 技能 | 职责 |
|------|------|------|
| **Facade** | `impl-facade-module` | 通用组件（DomainEntity、DTO、Result、事件等） |
| **Client** | `impl-client-module` | 客户端契约（Param、DTO、VO、API） |
| **Domain** | `impl-domain-module` | 业务逻辑（聚合、实体、仓储/工厂/网关接口、值对象） |
| **Infrastructure** | `impl-infra-module` | 持久化实现（Entity、Mapper、仓储实现） |
| **Application** | `impl-application-module` | 应用服务（Command/Query/Stream Service） |
| **Adapter** | `impl-adapter-module` | 外部接口（Controller、Listener、Config） |

## 工作流程

### 推荐的完整工作流

1. **需求阶段**
   - 使用 `design-prd` 技能编写产品需求文档
   
2. **设计阶段**
   - 使用 `design-technical-solution` 技能基于 PRD 生成技术方案
   
3. **实现阶段**
   - 使用 `create-java-project-structure` 创建项目结构框架
   - 使用 `impl-third-party-sdk` 集成必要的第三方依赖
   - 按技术方案的顺序，使用对应的 `impl-*-module` 技能实现各个模块
   - 或使用 `impl-from-technical-solution` 一站式编排所有模块

4. **UI 设计**
   - 如需前端，使用 `design-enterprise-ui-antd` 基于 Ant Design 进行 UI 设计

### 快速参考

**设计类技能** → 产出文档
**代码编写类技能** → 产出代码

## 模块依赖管理

- 各模块可选依赖清单详见 `skills/module-dependencies.md`
- 在配置模块依赖时，参考该文档以避免版本冲突或不必要的依赖

## 文件组织

```
garry-skills/
├── README.md                 # 项目总览
├── CLAUDE.md                 # 本文件
├── LICENSE                   # Apache 2.0 许可证
└── skills/
    ├── README.md             # 技能索引
    ├── module-dependencies.md # 模块依赖清单
    ├── design-prd/
    ├── design-technical-solution/
    ├── design-enterprise-ui-antd/
    ├── impl-facade-module/
    ├── impl-client-module/
    ├── impl-domain-module/
    ├── impl-infra-module/
    ├── impl-application-module/
    ├── impl-adapter-module/
    ├── impl-from-technical-solution/
    ├── impl-third-party-sdk/
    ├── create-java-project-structure/
    └── ontology/             # 本体论模块
```

每个技能目录包含：
- `SKILL.md` - 技能说明文档
- `reference.md` - 参考资料或示例

## 与 Claude 协作的最佳实践

### 1. 使用技能时的说明
调用技能时，在提示中包含足够的上下文：
- 当前项目的架构状态
- 已有的代码示例
- 具体的需求或目标

### 2. 参考现有代码
- 在设计或编码前，浏览 `skills/` 下相关技能目录的文档
- 查看 `reference.md` 了解示例和约定
- 确保新代码与现有风格一致

### 3. 按顺序实现
- 严格遵循技术方案中的实现顺序
- 不跳步或并行实现，以避免依赖问题
- 按照 DDD 分层顺序：Facade → Client → Domain → Infrastructure → Application → Adapter

### 4. 验证与反馈
- 实现完成后，验证代码是否符合 DDD 架构
- 检查命名和代码风格是否与现有代码一致
- 收集反馈并更新技能文档

## 约定与命名规范

### Java 命名
- **包名**: 使用小写、点分隔符，如 `com.example.domain.model`
- **类名**: PascalCase，如 `UserService`、`UserRepository`
- **方法名**: camelCase，如 `getUserById`、`createUser`
- **常量**: UPPER_CASE，如 `DEFAULT_TIMEOUT`

### DDD 相关
- **聚合根**: 以 `Root` 或业务名称命名，如 `UserRoot`、`OrderRoot`
- **仓储接口**: `*Repository`，如 `UserRepository`
- **工厂接口**: `*Factory`，如 `OrderFactory`
- **网关接口**: `*Gateway`，如 `PaymentGateway`
- **值对象**: 以 `VO` 或具体类型命名，如 `MoneyVO`、`AddressVO`
- **DTO**: `*DTO` 或 `*Dto`，如 `CreateUserDTO`
- **事件**: `*Event` 或 `*DomainEvent`，如 `UserCreatedEvent`

## 常见问题 (FAQ)

**Q: 如果项目中已有类似功能的代码，是否应该复用？**
A: 是的，必须复用。在设计前先审查现有代码，理解其结构和约定，然后按同样的模式实现新功能。

**Q: 可以并行实现多个模块吗？**
A: 不建议。虽然模块相对独立，但模块间存在依赖关系（如 Domain 依赖 Facade，Application 依赖 Domain）。按推荐顺序实现可以减少返工。

**Q: 如何处理第三方依赖冲突？**
A: 参考 `skills/module-dependencies.md` 了解推荐的版本。如有冲突，优先在该文档中更新，然后通知团队。

**Q: 可以修改 DDD 架构层级吗？**
A: 不建议。该架构经过验证，修改会影响一致性。如有特殊需求，先与团队讨论。

## 维护与更新

本文档由项目维护者更新。若有问题或建议，请提交 Issue 或 Pull Request。

---

**最后更新**: 2026-06-06
**维护者**: garry-wjl
