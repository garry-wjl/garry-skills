#领域层设计经验

领域层承载核心业务模型、业务动作、业务规则、领域工厂、领域网关、领域事件，以及 Repository / Factory / Gateway 接口定义。

详细领域模型规范见 [domain-model-design.md](domain-model-design.md)。

## 强规约

1. **必须按业务领域分节**：领域层设计必须先进行业务层级划分，再按业务领域分节。
2. **每个领域固定六类内容**：领域模型、领域动作、领域规则、领域工厂、领域网关、领域事件。
3. **聚合根和实体必须具备基本属性**：`id`、`num`、`create_no`、`update_no`；聚合根还必须持有 `Repository`、`Gateway`、`DomainEventPublisher` 三类领域协作依赖属性；值对象无需。
4. **软删除标记不得出现在领域模型中**：聚合根和领域实体不得包含 `is_deleted`、`deleted`、`isDeleted` 等软删除标记；软删除只属于数据库表和 infra Entity 的持久化实现细节。
5. **聚合根和可持久化实体必须具备 save/delete**：且所有领域动作必须包含 `operatorId`。
5. **必须设计领域动作**：领域动作要表达业务行为，而不是简单 CRUD；每个领域动作必须说明前置条件、后置结果、触发事件，并配时序图。
6. **必须设计领域规则**：聚合内不变性、状态流转、权限/资格、额度/数量等规则必须显式列出。
7. **必须设计领域工厂**：每个聚合根必须有对应 `*Factory`，且 Factory **只能包含** `create(...)` 与 `createByNum(...)` 两个方法。
8. **两个 Factory 方法都用于构建领域对象**：`create(...)` 根据创建该领域对象时用户可能填写的业务字段构建新的领域对象，参数不包含操作人、状态、审计字段、系统生成编号等内部流转字段；`createByNum(...)` 根据业务编码 num 通过 Repository 获取数据并构建既有领域对象。
9. **必须设计领域网关**：领域层需要的外部能力或跨边界协作能力必须定义 Gateway 接口。
10. **领域网关定义在领域层，实现在基础设施层**：领域层只定义 Gateway 接口，基础设施提供 GatewayImpl；领域层不依赖基础设施实现。
11. **Repository 只定义三方法**：`save(R)`、`findByNum(String num)`、`deleteByNum(String num)`。
12. **Repository 不给应用层使用**：Repository 是领域对象/Factory 与基础设施层的持久化协作接口。
13. **跨聚合只引用 ID**：聚合之间不得直接持有对象引用。
14. **领域层不出现基础设施实现**：不得出现 Mapper、Entity、MyBatis、Redis、HTTP 客户端等实现细节。

## 必选内容

### 1. 业务层级划分

| 层级 | 业务领域 | 子域/说明 |
|------|----------|-----------|
| 核心域 | order | 订单创建、支付、取消 |
| 支撑域 | user | 用户基础信息 |

### 2. 领域模型

输出领域类图 + 表格：

| 对象 | 类型 | 属性 | 关系 | 说明 |
|------|------|------|------|------|
| Order | 聚合根 | id,num,create_no,update_no,status,OrderRepository,OrderGateway,DomainEventPublisher | 包含 OrderItem | 订单聚合 |
| OrderItem | 实体 | id,num,sku,quantity | 属于 Order | 订单明细 |
| Money | 值对象 | amount,currency | 被 Order 使用 | 金额 |

### 3. 领域动作

| 聚合/实体 | 领域动作 | 职责 | 前置条件 | 后置条件/规则 | 领域事件 |
|-----------|----------|------|----------|---------------|----------|
| Order | pay(operatorId) | 支付订单 | 状态为待支付 | 状态变为已支付 | ORDER_PAID |

每个领域动作必须配时序图。

### 4. 领域规则

| 聚合/对象 | 规则类型 | 规则描述 | 违反时表达 |
|-----------|----------|----------|------------|
| Order | 状态规则 | 已取消订单不能支付 | 抛业务异常 |

### 5. 领域工厂

| 工厂 | 方法 | 入参 | 返回值 | 职责 | 依赖 |
|---------|------|------|--------|------|------|
| OrderFactory | create | items, remark | Order | 根据用户创建订单时填写的商品明细和备注构建新的订单领域对象 | OrderGateway |
| OrderFactory | createByNum | orderNum | Order | 根据业务编码通过 Repository 获取数据并构建订单领域对象 | OrderRepository |

> 注意：Factory 只允许上述两个方法，不允许设计 `createById(...)`、`rebuild(...)` 或其他工厂方法。

### 6. 领域网关

| 网关 | 方法 | 入参 | 返回值 | 职责 | 外部依赖 | 失败策略 |
|---------|------|------|--------|------|----------|----------|
| OrderGateway | generateNum | 无 | String | 生成订单业务编码 | 编号规则/云服务 | 失败抛业务异常 |
| PaymentGateway | pay | paymentParam | PaymentResult | 调用支付能力 | 支付服务 | 超时/重试/降级 |

> 注意：Gateway 接口定义在领域层，GatewayImpl 实现在基础设施层。

### 7. 领域事件

| 事件名 | 触发时机 | 载荷要点 | 可订阅方/用途 |
|--------|----------|----------|----------------|
| ORDER_PAID | 订单支付成功 | orderNum,userNum,amount | 通知、积分 |

## 常见设计经验

### 经验 1：从业务动作反推领域模型

不要只从数据库表推领域对象。先识别业务动作，再判断需要哪些聚合、实体和值对象承载规则。

### 经验 2：先动作，后规则

领域动作描述“业务能做什么”，领域规则描述“什么时候能做、做到什么程度、违反如何处理”。

### 经验 3：Factory 是构建入口，不是业务服务

Factory 负责构建领域对象，不承载复杂业务编排。业务编排在 application，业务规则在领域对象。

### 经验 4：Gateway 表达领域所需能力

领域网关不是第三方 SDK 的简单搬运，而是用领域语言表达领域需要的外部能力。

### 经验 5：Repository 保持窄接口

Repository 不做列表、分页、统计等查询；这些查询由 `QueryService` + Mapper 承载。

### 经验 6：领域事件描述业务事实

事件命名应表达已经发生的业务事实，例如 ORDER_PAID，而不是 DO_PAY_ORDER。

## 自检清单

- [ ] 是否按业务领域分节？
- [ ] 每个领域是否包含领域模型、领域动作、领域规则、领域工厂、领域网关、领域事件？
- [ ] 每个聚合根是否持有 Repository、Gateway、DomainEventPublisher 三类领域协作依赖属性？
- [ ] 聚合根和领域实体中是否没有 is_deleted/deleted/isDeleted 等软删除标记？
- [ ] 每个聚合根是否有 Factory，且 Factory 是否只包含 create(...) 与 createByNum(...) 两个方法，并且两个方法都用于构建领域对象？
- [ ] create(...) 的参数是否只包含用户创建领域对象时可能填写的业务字段，且不包含操作人、状态、审计字段、系统生成编号等内部流转字段？
- [ ] 是否定义了领域网关 Gateway 接口，并说明由基础设施实现？
- [ ] Repository 是否只有 save/findByNum/deleteByNum？
- [ ] 领域动作是否包含 operatorId？
- [ ] 每个领域动作是否有时序图？
- [ ] 是否没有把基础设施实现细节写入领域层？
