# 横切-SQL管理与数据库连接元数据

> 分支无关。数据源服务的「连接管理」实为 SQL 表达式定义 + 数据库连接元数据的登记，本服务不建立真实数据库连接、不执行 SQL。

## 定位澄清

`source_config` 树中 type=4（SQL 语句管理目录）、type=5（具体 SQL）、type=20（环境数据源）三类节点与 `source_sql` 表共同构成「SQL 取值来源」的定义层；但 **datasource-manage 不持有 JDBC 连接池、不执行 SQL、不生成随机值**——`valueType=SQL/RANDOM` 只是单元格的取值来源标记，真正执行由消费侧（执行/提测服务）完成。

## source_sql：SQL 表达式登记

`Sql`（`@TableName source_sql`）字段：

| 字段 | 含义 |
|---|---|
| `sourceConfigId` | 所属数据源/节点 |
| `name` | SQL 名称 |
| `content` | SQL 内容 |
| `dbName` | 数据库实例名 |
| `envId` / `dbAlias` / `dbConfigId` | 连接元数据（环境 id / 数据库别名 / 数据配置 id） |

`SqlCtrl`（ApiServlet `/source/SqlCtrl`）提供 `save` / `selectPage` / `selectAll` / `delete`，底层 `SqlServiceImpl` 为 MyBatis-Plus 薄封装，无执行逻辑。

## source_config 的 SQL 与连接字段

`SourceConfig` 实体带：

- `sqlId`：type=5 具体 SQL 节点关联 `source_sql.id`。
- `envId` / `dbAlias` / `dbConfigId`：环境数据源（type=20）的连接元数据，仅作登记。

`SourceTypeEnum` 实际只定义到 `SQL_SOURCE(4, "SQL语句管理")`；`SourceConfig.type` 注释中的「5 具体的 sql」「20 环境数据源」未进入该枚举，因此按 type 分发的导入策略不覆盖它们。

## 与列配置的衔接

`datatable_col_config.sqlCol` 记录「该列关联 SQL 查询结果的列名」（`configCol` 入参，注释标注暂未启用），是列 → SQL 结果列的映射预留字段。

## 关联

- [数据读取实现](../03-实现逻辑/数据读取实现.md)（SQL 表达式取值小节）
- [功能总览](../01-产品功能/功能总览.md)（SQL 管理一节）
