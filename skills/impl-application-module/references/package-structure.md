# 包结构与目录示例

## 目录规范

按业务领域/能力划分子包，与 domain 聚合或用例对应。

### 规则 2a：domain 层已存在该领域

领域子包下仅两个 Service：
- `*CommandService`（写操作）
- `*QueryService`（读操作）

### 规则 2b：domain 层不存在该领域

在子包内分层结构：
- `config/` - 应用层配置类
- `hook/` - 各阶段钩子
- `interceptor/` - 拦截器
- `service/` - 仅一个 Service，作为核心编排入口
- `tool/` - 可执行工具

## 示例目录树

```
application/
├── conversation/                    ← 规则 2a：仅两个 Service
│   ├── ConversationCommandService.java
│   └── ConversationQueryService.java
├── message/                         ← 规则 2a：仅两个 Service
│   ├── MessageCommandService.java
│   └── MessageQueryService.java
├── auth/                            ← 规则 2a：仅两个 Service
│   ├── AuthCommandService.java
│   └── AuthQueryService.java
├── report/                          ← 规则 2b：分层结构
│   ├── config/
│   │   └── ReportAgentConfig.java
│   ├── hook/
│   │   ├── ReportPlanAgentHook.java
│   │   └── ReportValidateAgentHook.java
│   ├── interceptor/
│   │   └── ReportModelInterceptor.java
│   ├── service/
│   │   └── ReportAgentService.java    ← 仅此一个 Service
│   └── tool/
│       └── ReportTools.java
└── common/                          ← 可选：跨领域通用服务
    └── CommonApplicationService.java
```

## 设计要点

- **规则 2a 简洁性**：仅两个 Service，减少包层级，提高可读性
- **规则 2b 结构化**：通过 config/hook/interceptor/service/tool 分离关注点
- **common 可选**：仅在存在跨多领域通用服务时创建
- **包名一致性**：与 domain 层聚合名称对应，便于追踪业务逻辑
