# script_check_status_sync -- 脚本检查状态跨节点同步队列表

> 所属库：db_file
> 对应模型：`cn.testin.filecloud.model.ScriptCheckStatusSync`
> 对应 Mapper：`ScriptCheckStatusSyncMapper.java` + `ScriptCheckStatusSyncMapper.xml`
> 使用方：文件管理服务（fileupload 工程）、脚本服务（filesystem / filemanagement 工程）
> 分支：syy.release.z7.8.1.0

## 表结构

| 字段 | 类型 | 允许空 | 默认值 | 说明 |
|---|---|---|---|---|
| `id` | bigint(20) | NOT NULL | AUTO_INCREMENT | 主键 ID，自增 |
| `script_no` | int(11) | YES | NULL | 脚本编号 |
| `script_id` | int(11) | YES | NULL | 脚本唯一 ID（关联 [script_file](script_file.md).scriptid） |
| `check_status` | int(11) | YES | NULL | 检查结果状态 |
| `sync_count` | int(11) | YES | NULL | 同步次数（重试计数） |

## 索引

| 索引名 | 字段 | 类型 |
|---|---|---|
| PRIMARY | `id` | BTREE（主键） |
| `idx_script_no` | `script_no` | BTREE |

## 业务说明

`script_check_status_sync` 是一个**异步状态同步队列表**，解决分布式部署场景下脚本检查状态在多个节点间同步的问题。当脚本在一个节点上完成有效性检查后，检查结果需要同步到其他节点。

**工作流程：**
1. 节点 A 完成脚本检查 -> 将检查状态写入 [script_check](script_check.md) 表
2. 同时插入一条同步记录到此表，记录需要同步的 script_no
3. 后台同步任务通过 `selectByScriptNo` 获取待同步记录
4. 同步成功后通过 `delete` 移除记录（需 id + check_status 双重校验）
5. 若同步失败则通过 `updateById` 递增 sync_count 重试

**关键设计：**
- `selectByPageAndPageSize` 支持分页轮询待同步记录（id > lastId 方式游标遍历）
- `delete` 方法需要同时匹配 id 和 check_status，防止并发误删
- `sync_count` 记录重试次数，可用于监控和限制最大重试

**与 script_check 的区别：**
- [script_check](script_check.md) 是检查结果的实际存储
- 本表是同步队列，记录哪些脚本的检查状态需要推送到其他节点

## Mapper 操作

| 方法名 | SQL 类型 | 用途 |
|---|---|---|
| `selectByScriptNo` | SELECT | 按 script_no 查最新一条同步记录（ORDER BY id DESC LIMIT 1） |
| `selectByPageAndPageSize` | SELECT | 分页游标查询待同步记录（id > #{id} 增量方式） |
| `insert` | INSERT | 插入同步记录（useGeneratedKeys） |
| `updateById` | UPDATE | 按 id 更新 script_id + check_status + sync_count |
| `deleteById` | DELETE | 按 id 删除单条记录 |
| `delete` | DELETE | 按 id + check_status 双重校验后删除（防止并发冲突） |

## 关联关系

- **引用表：**
  - [script_file](script_file.md) -- `script_id` / `script_no` 关联源脚本
  - [script_check](script_check.md) -- `check_status` 来源于检查结果

## 涉及接口

- [ScriptCheckSyncController](../接口文档/sync/ScriptCheckSyncController.md)（同步任务调度/执行）

## 脚本服务侧使用

> 以下为脚本服务（filesystem / filemanagement 工程）侧视角，业务域：脚本校验。

- 关联 Mapper（脚本服务侧）：`ScriptCheckStatusSyncMapper`
- 相关接口（脚本服务侧）：[ScriptCheckStatusSyncController](ScriptCheckStatusSyncController.md)
- 脚本服务侧登记的关联关系（与上文一致）：
  - 引用：[script_check](script_check.md)（script_id FK）
  - 引用：[script_file](script_file.md)（script_id FK）
