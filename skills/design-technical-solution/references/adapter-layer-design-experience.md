# 适配层设计经验

适配层是外部触发入口层，负责承接 HTTP 请求、定时任务触发、事件/MQ 消息监听，并委托应用层完成用例。

## 强规约

1. **适配层不写业务逻辑**：只做协议适配、参数转换、鉴权上下文解析，并调用应用层服务。
2. **适配层入口不只有控制器**：还包括定时任务、消息监听器、事件监听器和配置类。
3. **HTTP 方法只允许 GET/POST**：GET 用于查询，POST 用于新增、修改、删除、触发动作。
4. **每个入口必须有时序图**：控制器、定时任务、事件监听入口都要有时序图。
5. **控制器区分命令和查询**：命令类入口走 `CommandController`，查询类入口走 `QueryController`；无法判断时必须确认。
6. **定时任务必须说明幂等和并发控制**。
7. **事件监听必须说明消费幂等、重试、DLQ 和告警**。
8. **适配层只依赖应用层和客户端层**：不直接依赖领域层或基础设施层实现细节。
9. **Spring Bean 注入必须使用 `@Resource`**：适配层注入应用层服务、配置组件等 Spring Bean 时，必须使用 `@Resource`；禁止使用 `@Autowired`、构造器注入、Setter 注入或通过 `ApplicationContext` 手动获取 Bean。

## 必选内容

### 1. 控制器接口清单

| 控制器 | 方法 | HTTP | 路径 | 入参 JSON | 出参 JSON | 调用服务 | 职责 |
|------------|------|------|------|-----------|-----------|--------------|------|
| OrderCommandController | create | POST | /order/command/create | {...} | {...} | OrderCommandService.createOrder | 创建订单 |
| OrderQueryController | detail | GET | /order/query/detail | query | {...} | OrderQueryService.detail | 订单详情 |

### 2. 定时任务清单

| 任务 | cron/触发 | 入参 | 调用服务 | 幂等策略 | 并发控制 | 失败处理 |
|-----|-----------|------|--------------|----------|----------|----------|
| OrderTimeoutJob | 0 */5 * * * ? | 无 | OrderCommandService.closeTimeout | orderNum 幂等 | 分布式锁 | 重试+告警 |

### 3. 事件监听清单

| 监听器 | 来源 | 消息/事件 | 调用服务 | 幂等策略 | 重试/DLQ | 告警 |
|----------|------|-----------|--------------|----------|----------|------|
| PaymentPaidListener | MQ topic/tag | PaymentPaidEvent | OrderCommandService.paySuccess | eventId | 3次重试+DLQ | 是 |

### 4. 入口时序图

HTTP：

```text
客户端 → 控制器 → 应用层服务 → 统一返回结果
```

定时任务：

```text
调度器 → 定时任务 → 应用层服务 → 完成/告警
```

事件监听：

```text
MQ/事件 → 监听器 → 应用层服务 → ACK/NACK/DLQ
```

## 常见设计经验

### 经验 1：适配层只做协议适配

HTTP、MQ、定时任务都是外部触发方式，适配层不处理业务规则。

### 经验 2：命令和查询入口分开

按 `CommandController` / `QueryController` 区分入口，有利于后续权限、日志和审计。

### 经验 3：异步入口要重视幂等

MQ 和定时任务天然可能重复触发，必须设计幂等策略。

### 经验 4：异常处理要和入口类型匹配

HTTP 返回错误码；MQ 走重试/DLQ；定时任务走告警/补偿。

## 自检清单

- [ ] 是否覆盖控制器、定时任务、监听器和配置类？
- [ ] HTTP 是否仅使用 GET/POST？
- [ ] 是否区分命令类控制器和查询类控制器？
- [ ] 每个入口是否有时序图？
- [ ] 定时任务是否说明 cron、幂等、并发、失败处理？
- [ ] 事件监听是否说明来源、载荷、幂等、重试/DLQ、告警？
- [ ] 适配层是否没有写业务逻辑？
- [ ] 适配层是否只依赖应用层和客户端层？
- [ ] Spring Bean 注入是否统一使用 `@Resource`，且未使用 `@Autowired`、构造器注入、Setter 注入或 `ApplicationContext` 手动获取？
