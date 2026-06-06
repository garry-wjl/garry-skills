# 应用层设计经验

应用层负责编排业务用例、划定事务边界、协调领域对象与查询服务。应用层不承载领域规则。

## 强规约

1. **CommandService / QueryService 分离**：写操作进入 CommandService，读操作进入 QueryService。
2. **application 禁止调用 Repository**：任何 application Service 都不得注入或调用 domain Repository。
3. **CommandService 通过 Factory 获取领域对象**：新建用 `create(...)`，按业务编码加载用 `createByNum(...)`。
4. **查询通过 QueryService**：CommandService 如需查询/校验数据，调用对应领域 QueryService。
5. **QueryService 可注入 Mapper**：QueryService 做只读查询，可注入 infra Mapper 并转 client DTO。
6. **入参必须是 ParamDTO 类**：application 方法参数必须使用 `XXXParamDTO` 类封装。
7. **返回值只能是 client DTO 或业务标识**：不得返回 VO、领域对象、Entity 或 application 自定义对象。
8. **事务边界在 application**：需要事务时在 application 方法上声明。
9. **每个 Service 方法必须有时序图**。

## 必选内容

### 1. 业务模块划分

| 业务模块 | Service | 类型 | 职责 |
|----------|---------|------|------|
| order | OrderCommandService | Command | 创建、支付、取消订单 |
| order | OrderQueryService | Query | 查询订单列表、详情 |

### 2. Service 方法清单

| Service | 方法 | 入参 | 返回 | 职责 | 事务 |
|---------|------|------|------|------|------|
| OrderCommandService | createOrder | OrderCreateParamDTO | String | 创建订单 | 是 |
| OrderQueryService | listOrders | OrderQueryParamDTO | List<OrderDTO> | 查询订单 | 否 |

### 3. 方法时序逻辑

CommandService 示例：

```text
1. 参数校验
2. 获取 operatorId
3. 如需校验用户存在，调用 UserQueryService
4. 通过 OrderFactory.create(...) 创建领域对象
5. 调用 order.save(operatorId)
6. 返回 orderNum
```

QueryService 示例：

```text
1. 参数校验
2. 通过 Mapper 查询数据
3. Entity 转 client DTO
4. 返回 DTO
```

## 常见设计经验

### 经验 1：CommandService 只做编排

不要在 CommandService 中写业务规则判断，业务规则应由领域对象承载。

### 经验 2：QueryService 是应用层查询出口

跨领域查询、存在性校验、列表详情查询都应通过 QueryService 暴露。

### 经验 3：事务包住完整用例

事务应覆盖一个完整用例，而不是散落在 Repository 或 domain 方法上。

### 经验 4：ParamDTO 能稳定接口

多个原始参数容易失控，统一 ParamDTO 有利于后续扩展。

## 自检清单

- [ ] 是否按业务模块划分 CommandService / QueryService？
- [ ] application 是否未注入或调用 Repository？
- [ ] CommandService 是否通过 Factory 获取领域对象？
- [ ] 查询/校验是否通过 QueryService？
- [ ] QueryService 是否只做只读查询并返回 client DTO？
- [ ] 方法入参是否都是 XXXParamDTO？
- [ ] 返回值是否没有 VO、Entity、领域对象？
- [ ] 每个 Service 方法是否有时序图？
