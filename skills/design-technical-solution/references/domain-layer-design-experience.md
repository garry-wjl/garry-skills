# 领域层设计经验

领域层承载核心业务模型、业务规则、领域动作、领域事件，以及 Repository / Factory / Gateway 接口定义。

详细领域模型规范见 [domain-model-design.md](domain-model-design.md)。

## 强规约

1. **必须按业务领域分节**：领域层设计必须先进行业务层级划分，再按业务领域分节。
2. **每个领域固定五类内容**：领域模型、领域工厂、领域规则、领域动作、领域事件。
3. **聚合根和实体必须具备基本属性**：`id`、`num`、`create_no`、`update_no`；值对象无需。
4. **聚合根和可持久化实体必须具备 save/delete**：且所有领域动作必须包含 `operatorId`。
5. **必须设计领域工厂**：每个聚合根必须有对应 `*Factory`，且 Factory **只能包含** `create(...)` 与 `createByNum(...)` 两个方法。
6. **两个 Factory 方法都用于构建领域对象**：`create(...)` 根据属性构建新的领域对象；`createByNum(...)` 根据业务编码 num 通过 Repository 获取数据并构建既有领域对象。
7. **领域对象构建必须通过 Factory**：application 不得直接 `new` 领域对象，也不得直接调用领域对象静态 `create`。
8. **Repository 只定义三方法**：`save(R)`、`findByNum(String num)`、`deleteByNum(String num)`。
9. **Repository 不给 application 使用**：Repository 是领域对象/Factory 与 infra 的持久化协作接口。
10. **跨聚合只引用 ID**：聚合之间不得直接持有对象引用。
11. **领域层不出现基础设施实现**：不得出现 Mapper、Entity、MyBatis、Redis、HTTP client 等实现细节。

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
| Order | 聚合根 | id,num,create_no,update_no,status | 包含 OrderItem | 订单聚合 |
| OrderItem | 实体 | id,num,sku,quantity | 属于 Order | 订单明细 |
| Money | 值对象 | amount,currency | 被 Order 使用 | 金额 |

### 3. 领域工厂

| Factory | 方法 | 入参 | 返回值 | 职责 | 依赖 |
|---------|------|------|--------|------|------|
| OrderFactory | create | userNum, items | Order | 根据属性构建新的订单领域对象 | OrderGateway |
| OrderFactory | createByNum | orderNum | Order | 根据业务编码通过 Repository 获取数据并构建订单领域对象 | OrderRepository |

> 注意：Factory 只允许上述两个方法，不允许设计 `createById(...)`、`rebuild(...)` 或其他工厂方法。

### 4. 领域规则

| 聚合/对象 | 规则类型 | 规则描述 | 违反时表达 |
|-----------|----------|----------|------------|
| Order | 状态规则 | 已取消订单不能支付 | 抛业务异常 |

### 5. 领域动作

| 聚合/实体 | 领域动作 | 职责 | 前置条件 | 后置条件/规则 | 领域事件 |
|-----------|----------|------|----------|---------------|----------|
| Order | pay(operatorId) | 支付订单 | 状态为待支付 | 状态变为已支付 | ORDER_PAID |

每个领域动作必须配时序图。

### 6. 领域事件

| 事件名 | 触发时机 | 载荷要点 | 可订阅方/用途 |
|--------|----------|----------|----------------|
| ORDER_PAID | 订单支付成功 | orderNum,userNum,amount | 通知、积分 |

## 常见设计经验

### 经验 1：从业务动作反推领域模型

不要只从数据库表推领域对象。先识别业务动作，再判断需要哪些聚合、实体和值对象承载规则。

### 经验 2：Factory 是构建入口，不是业务服务

Factory 负责创建/加载领域对象，不承载复杂业务编排。业务编排仍在 application，业务规则在领域对象。

### 经验 3：Repository 保持窄接口

Repository 不做列表、分页、统计等查询；这些查询由 QueryService + Mapper 承载。

### 经验 4：领域事件描述业务事实

事件命名应表达已经发生的业务事实，例如 ORDER_PAID，而不是 DO_PAY_ORDER。

## 自检清单

- [ ] 是否按业务领域分节？
- [ ] 每个领域是否包含领域模型、领域工厂、领域规则、领域动作、领域事件？
- [ ] 每个聚合根是否有 Factory，且 Factory 是否只包含 create(...) 与 createByNum(...) 两个方法，并且两个方法都用于构建领域对象？
- [ ] Repository 是否只有 save/findByNum/deleteByNum？
- [ ] 领域动作是否包含 operatorId？
- [ ] 每个领域动作是否有时序图？
- [ ] 是否没有把基础设施实现细节写入领域层？
