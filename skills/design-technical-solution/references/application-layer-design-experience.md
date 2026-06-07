# 应用层设计经验

应用层负责编排业务用例、划定事务边界、协调领域对象与查询服务。应用层不承载领域规则。

## 强规约

1. **命令服务 / 查询服务分离**：写操作进入 `CommandService`，读操作进入 `QueryService`。
2. **应用层禁止调用 Repository**：任何应用层服务都不得注入或调用领域 Repository。
3. **命令服务通过工厂获取领域对象**：新建用 `create(...)`，按业务编码加载用 `createByNum(...)`。
4. **查询通过查询服务**：命令服务如需查询/校验数据，调用对应领域的 `QueryService`。
5. **`QueryService` 可注入 Mapper**：`QueryService` 做只读查询，可注入基础设施层 Mapper 并转客户端 DTO。
6. **入参必须是 ParamDTO 类**：应用层方法参数必须使用 `XXXParamDTO` 类封装。
7. **返回值只能是客户端 DTO 或业务标识**：不得返回 VO、领域对象、实体对象或应用层自定义对象。
8. **事务边界在应用层**：需要事务时在应用层方法上声明。
9. **每个服务方法必须有时序图**。
10. **Spring Bean 注入必须使用 `@Resource`**：应用层服务注入 Factory、QueryService、Mapper 等 Spring Bean 时，必须使用 `@Resource`；禁止使用 `@Autowired`、构造器注入、Setter 注入或通过 `ApplicationContext` 手动获取 Bean。

## 必选内容

### 1. 业务模块划分

| 业务模块 | 服务 | 类型 | 职责 |
|----------|---------|------|------|
| order | OrderCommandService | Command | 创建、支付、取消订单 |
| order | OrderQueryService | Query | 查询订单列表、详情 |

### 2. 服务方法清单

| 服务 | 方法 | 入参 | 返回 | 职责 | 事务 |
|---------|------|------|------|------|------|
| OrderCommandService | createOrder | OrderCreateParamDTO | String | 创建订单 | 是 |
| OrderQueryService | listOrders | OrderQueryParamDTO | List<OrderDTO> | 查询订单 | 否 |

### 3. 方法时序逻辑

命令服务示例：

```text
1. 参数校验
2. 获取 operatorId
3. 如需校验用户存在，调用 UserQueryService
4. 通过 OrderFactory.create(items, remark) 创建领域对象（create 只传用户创建订单时填写的字段）
5. 调用 order.save(operatorId)
6. 返回 orderNum
```

查询服务示例：

```text
1. 参数校验
2. 通过 Mapper 查询数据
3. 实体对象转换为客户端 DTO
4. 返回 DTO
```

## 常见设计经验

### 经验 1：命令服务只做编排

不要在命令服务中写业务规则判断，业务规则应由领域对象承载。

### 经验 2：查询服务是应用层查询出口

跨领域查询、存在性校验、列表详情查询都应通过查询服务暴露。

### 经验 3：事务包住完整用例

事务应覆盖一个完整用例，而不是散落在 Repository 或领域方法上。

### 经验 4：ParamDTO 能稳定接口

多个原始参数容易失控，统一 ParamDTO 有利于后续扩展。

## 自检清单

- [ ] 是否按业务模块划分命令服务和查询服务？
- [ ]应用层是否未注入或调用 Repository？
- [ ] 命令服务是否通过领域工厂获取领域对象？
- [ ] 查询/校验是否通过查询服务？
- [ ] `QueryService` 是否只做只读查询并返回客户端 DTO？
- [ ] 方法入参是否都是 XXXParamDTO？
- [ ] 返回值是否没有 VO、实体对象、领域对象？
- [ ] 每个服务方法是否有时序图？
- [ ] Spring Bean 注入是否统一使用 `@Resource`，且未使用 `@Autowired`、构造器注入、Setter 注入或 `ApplicationContext` 手动获取？
