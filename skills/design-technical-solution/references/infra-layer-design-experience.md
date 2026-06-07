# 基础设施层设计经验

基础设施层负责实现领域层定义的 Repository / Factory / Gateway 接口，并承载Entity、Mapper、外部客户端、事件发布实现等基础设施能力。

## 强规约

1. **无变更也要说明**：如果本次不涉及基础设施层，技术方案中必须写明「本次无基础设施层变更」。
2. **RepositoryImpl 不作为应用层查询入口**：RepositoryImpl 仅服务领域对象和 Factory 的持久化协作。
3. **RepositoryImpl 只注入 Mapper**：不得注入 Gateway、领域事件发布器、应用层服务、其他 Repository 等。
4. **FactoryImpl 只注入本聚合 Repository**：不得注入 Gateway、领域事件发布器。
5. **Entity 与数据库表一致**：Entity 字段必须与数据库设计中的表结构一致。
6. **Entity 不暴露给上层**：应用层、领域层、适配层不应直接使用 基础设施 Entity。
7. **Gateway 接口定义在领域层，实现在基础设施层**：领域层定义 Gateway 接口，基础设施层提供 GatewayImpl；第三方服务、编号生成、外部客户端 调用等通过 GatewayImpl 实现。
8. **事务边界不在 RepositoryImpl 中设计**：事务在应用层方法上声明。
9. **Spring Bean 注入必须使用 `@Resource`**：基础设施层允许注入的 Bean（如 Mapper、Repository、外部客户端等）必须使用 `@Resource`；禁止使用 `@Autowired`、构造器注入、Setter 注入或通过 `ApplicationContext` 手动获取 Bean。若某类有更严格注入限制（如 RepositoryImpl 仅允许注入 Mapper），同时遵守更严格限制。

## 必选内容

### 1. Entity / Mapper 设计

| Entity | 表 | Mapper | 字段说明 | 新增/修改 |
|--------|----|--------|----------|-----------|
| OrderEntity | order | OrderMapper | 与 order 表一致 | 新增 |

### 2. RepositoryImpl 设计

| 仓储实现类 | 实现接口 | 注入 Mapper | 职责 | 转换方法 |
|----------------|----------|-------------|------|----------|
| OrderRepositoryImpl | OrderRepository | OrderMapper | save/findByNum/deleteByNum | toEntity/to领域|

### 3. FactoryImpl 设计

| 工厂实现类 | 实现接口 | 注入 Repository | 方法 | 职责 |
|-------------|----------|----------------|------|------|
| OrderFactoryImpl | OrderFactory | OrderRepository | create/createByNum | 创建/加载 Order |

### 4. GatewayImpl 设计

Gateway 接口必须已在领域层定义，基础设施层只设计对应实现类。

| 网关实现类 | 实现的领域 Gateway 接口 | 外部依赖 | 方法 | 失败策略 |
|-------------|--------------------------|----------|------|----------|
| PaymentGatewayImpl | PaymentGateway | PaymentClient | pay | 超时/重试/降级 |

### 5. common 能力

包括：

- common/event：领域事件发布实现
- common/client：第三方 API 客户端
- common/constant：基础设施常量
- common/exception：基础设施异常
- common/util：工具类

## 常见设计经验

### 经验 1：RepositoryImpl 做持久化，不做业务判断

复杂业务判断应在领域对象中完成，RepositoryImpl 只负责查库、转对象、保存。

### 经验 2：`QueryService` 读库不等于 Repository 查询

应用层查询可以通过 `QueryService` 注入 Mapper 完成，不要把 RepositoryImpl 当通用查询服务。

### 经验 3：外部 客户端层放 common/client

多个 Gateway 复用的第三方客户端应放在基础设施层 common/client，避免散落在各业务包里。

### 经验 4：转换逻辑保持简单

Entity ↔ 领域对象转换应是字段映射和简单常量转换，不应引入复杂业务规则。

## 自检清单

- [ ] 是否说明本次是否有基础设施层变更？
- [ ] Entity 是否与数据库设计一致？
- [ ] RepositoryImpl 是否只注入 Mapper？
- [ ] FactoryImpl 是否只注入 Repository？
- [ ] Gateway 接口是否定义在领域层，GatewayImpl 是否实现在基础设施层并承接外部能力？
- [ ] 是否没有把Entity 暴露给应用层、领域层、适配层？
- [ ] 是否没有把 RepositoryImpl 设计为应用层查询入口？
- [ ] Spring Bean 注入是否统一使用 `@Resource`，且未使用 `@Autowired`、构造器注入、Setter 注入或 `ApplicationContext` 手动获取？
