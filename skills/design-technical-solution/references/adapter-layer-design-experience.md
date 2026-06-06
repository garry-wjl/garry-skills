# Adapter 层设计经验

Adapter 层是外部触发入口层，负责承接 HTTP 请求、定时任务触发、事件/MQ 消息监听，并委托 application 层完成用例。

## 强规约

1. **Adapter 不写业务逻辑**：只做协议适配、参数转换、鉴权上下文解析、调用 application。
2. **Adapter 入口不只有 Controller**：还包括 Scheduler/Job、MQ Listener、Event Listener、Config。
3. **HTTP 方法只允许 GET/POST**：GET 用于查询，POST 用于新增、修改、删除、触发动作。
4. **每个入口必须有时序图**：Controller、定时任务、事件监听入口都要有时序图。
5. **Controller 区分 Command/Query**：命令类入口走 CommandController，查询类入口走 QueryController；无法判断必须确认。
6. **定时任务必须说明幂等和并发控制**。
7. **事件监听必须说明消费幂等、重试、DLQ 和告警**。
8. **Adapter 只依赖 application 和 client**：不直接依赖 domain/infra 实现细节。

## 必选内容

### 1. Controller 接口清单

| Controller | 方法 | HTTP | 路径 | 入参 JSON | 出参 JSON | 调用 Service | 职责 |
|------------|------|------|------|-----------|-----------|--------------|------|
| OrderCommandController | create | POST | /order/command/create | {...} | {...} | OrderCommandService.createOrder | 创建订单 |
| OrderQueryController | detail | GET | /order/query/detail | query | {...} | OrderQueryService.detail | 订单详情 |

### 2. 定时任务清单

| Job | cron/触发 | 入参 | 调用 Service | 幂等策略 | 并发控制 | 失败处理 |
|-----|-----------|------|--------------|----------|----------|----------|
| OrderTimeoutJob | 0 */5 * * * ? | 无 | OrderCommandService.closeTimeout | orderNum 幂等 | 分布式锁 | 重试+告警 |

### 3. 事件监听清单

| Listener | 来源 | 消息/事件 | 调用 Service | 幂等策略 | 重试/DLQ | 告警 |
|----------|------|-----------|--------------|----------|----------|------|
| PaymentPaidListener | MQ topic/tag | PaymentPaidEvent | OrderCommandService.paySuccess | eventId | 3次重试+DLQ | 是 |

### 4. 入口时序图

HTTP：

```text
Client → Controller → Application Service → Result
```

定时任务：

```text
Scheduler → Job → Application Service → 完成/告警
```

事件监听：

```text
MQ/Event → Listener → Application Service → ACK/NACK/DLQ
```

## 常见设计经验

### 经验 1：Adapter 只适配协议

HTTP、MQ、定时任务都是外部触发协议，Adapter 不处理业务规则。

### 经验 2：命令和查询入口分开

按 CommandController / QueryController 区分入口，有利于后续权限、日志和审计。

### 经验 3：异步入口要重视幂等

MQ 和定时任务天然可能重复触发，必须设计幂等策略。

### 经验 4：异常处理要和入口类型匹配

HTTP 返回错误码；MQ 走重试/DLQ；定时任务走告警/补偿。

## 自检清单

- [ ] 是否覆盖 Controller / Scheduler / Listener / Config？
- [ ] HTTP 是否仅使用 GET/POST？
- [ ] 是否区分 CommandController / QueryController？
- [ ] 每个入口是否有时序图？
- [ ] 定时任务是否说明 cron、幂等、并发、失败处理？
- [ ] 事件监听是否说明来源、载荷、幂等、重试/DLQ、告警？
- [ ] Adapter 是否没有写业务逻辑？
- [ ] Adapter 是否只依赖 application 和 client？
