# 数据库设计经验

数据库设计用于明确持久化模型、表结构、字段、索引、DDL 与必要 DML，确保基础设施层可以按表落 Entity、Mapper 和 RepositoryImpl。

## 强规约

1. **必须生成 DDL**：数据库设计必须包含可执行 DDL。
2. **涉及刷数必须生成 DML**：初始化、历史迁移、枚举数据等必须提供 DML。
3. **主键必须是 BIGINT 自增 id**：禁止使用 VARCHAR/UUID 作为业务表物理主键。
4. **业务编码使用 num**：对外业务编号使用 `num` 字段，并加唯一索引。
5. **时间字段必须是 create_time / update_time**：禁止 `created_at` / `updated_at`。
6. **时间精度必须到毫秒**：如 MySQL `DATETIME(3)`。
7. **审计字段必须与领域一致**：`create_no`、`update_no` 与领域对象基本属性对应。
8. **数据库设计必须映射领域模型**：每张表要说明对应的聚合根、实体或值对象持久化结构。
9. **不使用外键约束**：跨表关系通过业务字段和索引表达。
10. **软删除只属于持久化设计**：如需软删除，`is_deleted` 等软删除字段只能出现在数据库表和 infra Entity 中，不得映射为聚合根或领域实体属性。

## 必选内容

### 1. 表结构说明

| 表名 | 说明 | 对应领域对象 | 主要字段 | 索引 |
|------|------|--------------|----------|------|
| order | 订单表 | Order 聚合根 | id,num,user_num,status,create_time | uk_num,idx_user_num |

### 2. 字段设计

| 字段 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| id | BIGINT | 是 | 自增 | 物理主键 |
| num | VARCHAR(64) | 是 | 无 | 业务编码 |
| create_no | VARCHAR(64) | 是 | 无 | 创建人 |
| update_no | VARCHAR(64) | 是 | 无 | 更新人 |
| create_time | DATETIME(3) | 是 | CURRENT_TIMESTAMP(3) | 创建时间 |
| update_time | DATETIME(3) | 是 | CURRENT_TIMESTAMP(3) | 更新时间 |

### 3. DDL

```sql
CREATE TABLE `order` (
  `id` BIGINT NOT NULL AUTO_INCREMENT COMMENT '主键',
  `num` VARCHAR(64) NOT NULL COMMENT '业务编码',
  `create_no` VARCHAR(64) NOT NULL COMMENT '创建人',
  `update_no` VARCHAR(64) NOT NULL COMMENT '更新人',
  `create_time` DATETIME(3) NOT NULL DEFAULT CURRENT_TIMESTAMP(3) COMMENT '创建时间',
  `update_time` DATETIME(3) NOT NULL DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3) COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_order_num` (`num`)
) COMMENT='订单表';
```

### 4. DML（如需要）

```sql
INSERT INTO xxx_enum (...) VALUES (...);
```

### 5. 索引设计

说明：

- 唯一索引
- 查询索引
- 组合索引
- 是否支持分页/排序

### 6. Mapper 操作规约

- **查询必须使用 `LambdaQueryWrapper<Entity>`**：通过实体字段的 lambda 引用构造条件（如 `.eq(Entity::getNum, num)`），避免使用字符串列名的 `QueryWrapper`。
- **更新必须使用 `LambdaUpdateWrapper<Entity>`**：通过实体字段的 lambda 引用构造更新条件和赋值（如 `.eq(Entity::getNum, num).set(Entity::getStatus, status)`）。
- **禁止**在 Mapper/RepositoryImpl/QueryService 中使用字符串拼接 SQL 条件、`Map` 参数或 `UpdateWrapper`/`QueryWrapper` 的字符串列名重载。

## 常见设计经验

### 经验 1：先有领域模型，再有表

表结构应服务领域模型，不要直接把页面字段堆成表。

### 经验 2：num 是业务标识

应用层、接口层通常使用 num 作为业务标识，不暴露物理主键 id。

### 经验 3：索引来自查询场景

索引不能凭空设计，应对应 `QueryService` 的查询条件、排序和分页需求。

### 经验 4：刷数要说明执行顺序

涉及历史迁移时，DML 要说明前置条件、执行顺序和回滚策略。

## 自检清单

- [ ] 是否有表结构说明？
- [ ] 是否说明表与领域对象的映射关系？
- [ ] 是否生成可执行 DDL？
- [ ] 涉及刷数时是否生成 DML？
- [ ] 主键 id 是否 BIGINT 自增？
- [ ] 是否有 num 业务编码和唯一索引？
- [ ] 是否使用 create_time / update_time 且毫秒精度？
- [ ] 是否包含 create_no / update_no？
- [ ] 如有软删除字段，是否确认其只存在于数据库表和 infra Entity 中，而不进入聚合根或领域实体？
- [ ] 索引是否对应查询场景？
- [ ] 查询/更新是否统一使用 `LambdaQueryWrapper<Entity>` 和 `LambdaUpdateWrapper<Entity>`？
