# script_at_last -- 脚本最新版本非规范化表

> 所属库：db_file
> 对应模型：`cn.testin.filecloud.model.ScriptAtLast`
> 对应 Mapper：`ScriptAtLastMapper.java` + `ScriptAtLastMapper.xml`
> 使用方：文件管理服务（fileupload 工程）、脚本服务（filesystem / filemanagement 工程）
> 分支：syy.release.z7.8.1.0

## 表结构

| 字段 | 类型 | 允许空 | 默认值 | 说明 |
|---|---|---|---|---|
| `id` | int(11) | NOT NULL | AUTO_INCREMENT | 主键 ID，自增 |
| `script_id` | int(11) | NOT NULL | -- | 最新版本的脚本 ID（关联 [script_file](script_file.md).scriptid） |
| `script_no` | int(11) | NOT NULL | -- | 脚本编号（关联 [script_file](script_file.md).scriptno） |
| `project_id` | int(11) | NOT NULL | -- | 项目 ID |

## 索引

| 索引名 | 字段 | 类型 |
|---|---|---|
| PRIMARY | `id` | BTREE（主键） |
| `uniqueness` | `script_no`, `project_id` | BTREE（唯一索引） |
| `projectid` | `project_id` | BTREE |
| `idx_script_id_script_no` | `script_id`, `script_no` | BTREE |

## 业务说明

`script_at_last` 是一个典型的**非规范化冗余表**，用于加速"查询每个脚本的最新版本"这种高频查询。由于 `script_file` 表对同一脚本名保留所有历史版本，直接对 `MAX(scriptid) GROUP BY scriptno` 查询性能差，因此通过此表预先计算并缓存最新版本映射。

**数据刷新机制：**
- 由存储过程 `script_to_last()` 全量刷新
- 流程：TRUNCATE TABLE -> 游标遍历 script_file 按 scriptno 分组取 max(scriptid) -> 批量 INSERT
- 脏读容忍：允许短暂不一致，通过定期执行存储过程保证最终一致

**唯一约束含义：**
- `(script_no, project_id)` 的唯一索引确保同一脚本编号在同一项目中只有一条"最新版本"记录

## Mapper 操作

| 方法名 | SQL 类型 | 用途 |
|---|---|---|
| `selectByPrimaryKey` | SELECT | 按 id 查询 |
| `selectByExample` | SELECT | 按 Example 条件动态查询 |
| `countByExample` | SELECT | 按条件统计 |
| `insert` | INSERT | 完整插入 |
| `insertSelective` | INSERT | 选择性插入 |
| `updateByPrimaryKey` | UPDATE | 按主键全量更新 |
| `updateByPrimaryKeySelective` | UPDATE | 按主键选择性更新 |
| `updateByExample` | UPDATE | 按条件全量更新 |
| `updateByExampleSelective` | UPDATE | 按条件选择性更新 |
| `deleteByPrimaryKey` | DELETE | 按主键物理删除 |
| `deleteByExample` | DELETE | 按条件物理删除 |

## 关联关系

- **引用表：**
  - [script_file](script_file.md) -- `script_id` 和 `script_no` 关联源表

## 涉及接口

- [ScriptAtLastController](../接口文档/script/ScriptAtLastController.md)（获取最新版本脚本列表）
- 存储过程 `script_to_last()` 负责全量刷新此表

## 脚本服务侧使用

> 以下为脚本服务（filesystem / filemanagement 工程）侧视角，业务域：脚本校验。

- 关联 Mapper（脚本服务侧）：`ScriptAtLastMapper`
- 相关接口（脚本服务侧）：[ScriptAtLastController](ScriptAtLastController.md)
- 脚本服务侧登记的关联关系（与上文一致）：
  - 引用：[script_file](script_file.md)（script_id FK）
