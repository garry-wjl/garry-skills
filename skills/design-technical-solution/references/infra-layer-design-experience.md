# 基础设施层设计经验

基础设施层负责实现 domain 定义的 Repository / Factory / Gateway 接口，并承载 Entity、Mapper、外部 client、事件发布实现等基础设施能力。

## 强规约

1. **无变更也要说明**：如果本次不涉及 infra 层，技术方案中必须写明「本次无基础设施层变更」。
2. **RepositoryImpl 不作为 application 查询入口**：RepositoryImpl 仅服务领域对象和 Factory 的持久化协作。
3. **RepositoryImpl 只注入 Mapper**：不得注入 Gateway、DomainEventPublisher、Application Service、其他 Repository 等。
4. **FactoryImpl 只注入本聚合 Repository**：不得注入 Gateway、DomainEventPublisher。
5. **Entity 与数据库表一致**：Entity 字段必须与数据库设计中的表结构一致。
6. **Entity 不暴露给上层**：application、domain、adapter 不应直接使用 infra Entity。
7. **GatewayImpl 承接外部能力**：第三方服务、编号生成、外部 client 调用等通过 GatewayImpl 实现。
8. **事务边界不在 RepositoryImpl 中设计**：事务在 application 方法上声明。

## 必选内容

### 1. Entity / Mapper 设计

| Entity | 表 | Mapper | 字段说明 | 新增/修改 |
|--------|----|--------|----------|-----------|
| OrderEntity | order | OrderMapper | 与 order 表一致 | 新增 |

### 2. RepositoryImpl 设计

| RepositoryImpl | 实现接口 | 注入 Mapper | 职责 | 转换方法 |
|----------------|----------|-------------|------|----------|
| OrderRepositoryImpl | OrderRepository | OrderMapper | save/findByNum/deleteByNum | toEntity/toDomain |

### 3. FactoryImpl 设计

| FactoryImpl | 实现接口 | 注入 Repository | 方法 | 职责 |
|-------------|----------|----------------|------|------|
| OrderFactoryImpl | OrderFactory | OrderRepository | create/createByNum | 创建/加载 Order |

### 4. GatewayImpl 设计

| GatewayImpl | 实现接口 | 外部依赖 | 方法 | 失败策略 |
|-------------|----------|----------|------|----------|
| PaymentGatewayImpl | PaymentGateway | PaymentClient | pay | 超时/重试/降级 |

### 5. common 能力

包括：

- common/event：领域事件发布实现
- common/client：第三方 API client
- common/constant：基础设施常量
- common/exception：基础设施异常
- common/util：工具类

## 常见设计经验

### 经验 1：RepositoryImpl 做持久化，不做业务判断

复杂业务判断应在领域对象中完成，RepositoryImpl 只负责查库、转对象、保存。

### 经验 2：QueryService 读库不等于 Repository 查询

应用层查询可以通过 QueryService 注入 Mapper 完成，不要把 RepositoryImpl 当通用查询服务。

### 经验 3：外部 client 放 common/client

多个 Gateway 复用的第三方 client 应放在 infra common/client，避免散落在各业务包里。

### 经验 4：转换逻辑保持简单

Entity ↔ Domain 转换应是字段映射和简单常量转换，不应引入复杂业务规则。

## 自检清单

- [ ] 是否说明本次是否有基础设施层变更？
- [ ] Entity 是否与数据库设计一致？
- [ ] RepositoryImpl 是否只注入 Mapper？
- [ ] FactoryImpl 是否只注入 Repository？
- [ ] GatewayImpl 是否承接外部能力？
- [ ] 是否没有把 Entity 暴露给 application/domain/adapter？
- [ ] 是否没有把 RepositoryImpl 设计为 application 查询入口？
