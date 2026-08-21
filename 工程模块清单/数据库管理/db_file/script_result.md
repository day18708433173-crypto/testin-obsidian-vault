# script_result -- 测试结果文件表

> 所属库：db_file
> 对应模型：`cn.testin.filecloud.model.ScriptResult`
> 对应 Mapper：`ScriptResultMapper.java` + `ScriptResultMapper.xml`
> 使用方：文件管理服务（fileupload 工程）、脚本服务（filesystem / filemanagement 工程）
> 分支：syy.release.z7.8.1.0

## 表结构

| 字段 | 类型 | 允许空 | 默认值 | 说明 |
|---|---|---|---|---|
| `id` | bigint(20) | NOT NULL | AUTO_INCREMENT | 主键 ID，自增 |
| `resultfile_id` | bigint(20) | YES | 0 | 结果文件 ID（关联 [common_file](common_file.md).fileid） |
| `resultfile_url` | varchar(200) | YES | NULL | 结果文件 URL |
| `resultfile_name` | varchar(200) | YES | NULL | 结果文件名称 |
| `parentfile_id` | bigint(20) | YES | NULL | 父文件 ID（.test 文件的 fileid，关联 [common_file](common_file.md).fileid） |
| `parentfile_url` | varchar(200) | YES | NULL | 父文件 URL |
| `parentfile_name` | varchar(200) | YES | NULL | 父文件名称 |
| `resultfile_uploadtime` | bigint(20) | YES | NULL | 结果文件上传时间（毫秒时间戳） |
| `expectedfile_name` | varchar(200) | YES | '' | 预期文件名称（用于断言比对） |
| `script_id` | varchar(20) | YES | NULL | 脚本 ID（关联 [script_file](script_file.md).scriptid） |
| `is_clean` | int(1) | YES | 0 | 是否可清除：1=可清除, 0=未清除 |

## 索引

| 索引名 | 字段 | 类型 |
|---|---|---|
| PRIMARY | `id` | BTREE（主键） |
| `pk_parent_id` | `parentfile_id` | BTREE |
| `parentfile_url` | `parentfile_url` | BTREE |
| `index_clean` | `is_clean` | BTREE |

## 业务说明

`script_result` 记录测试脚本执行产生的结果文件和预期文件。每次回放执行产生的截图、日志、性能数据等结果文件在此登记。

**关键设计：**
- **结果文件 vs 父文件**：`parentfile_id/url/name` 对应 .test 格式的测试描述文件，`resultfile_id/url/name` 对应执行后产出的结果文件
- **预期文件**：`expectedfile_name` 指向一个基准/预期结果文件，回放时与实际结果比对
- **清理机制**：`is_clean` 标记结果文件是否可被清理任务回收
  - `is_clean=0` 表示结果仍在使用/不可清理
  - `is_clean=1` 表示已标记可清理，`listcleanabled` 查询分页获取这些记录
  - `listUncleaned` 支持按时间范围和 ID 范围查询未清理的结果

**生命周期：**
1. 测试执行完成 -> 结果写入 common_file + script_result
2. 结果文件使用期 -> is_clean=0
3. 超过保留期限 -> 标记 is_clean=1
4. 定时清理任务 -> 查 listcleanabled 并物理删除文件记录

## Mapper 操作

| 方法名 | SQL 类型 | 用途 |
|---|---|---|
| `selectByPrimaryKey` | SELECT | 按 id 查询结果记录 |
| `selectByExample` | SELECT | 按 Example 条件动态查询 |
| `selectByParentFileId` | SELECT | 按 parentfile_id 查关联的结果文件 |
| `listUncleaned` | SELECT | 查询未清理的结果（is_clean=0，支持时间范围和ID过滤） |
| `listcleanabled` | SELECT | 查询可清理的结果（is_clean=1，limit 0,200） |
| `countByExample` | SELECT | 按条件统计数量 |
| `insert` | INSERT | 完整插入结果记录 |
| `insertSelective` | INSERT | 选择性插入，返回自增 id |
| `updateByPrimaryKey` | UPDATE | 按主键全量更新 |
| `updateByPrimaryKeySelective` | UPDATE | 按主键选择性更新 |
| `updateByExample` | UPDATE | 按条件全量更新 |
| `updateByExampleSelective` | UPDATE | 按条件选择性更新 |
| `deleteByPrimaryKey` | DELETE | 按主键物理删除 |
| `deleteByExample` | DELETE | 按条件物理删除 |
| `deleteByParentFileId` | DELETE | 按 parentfile_id 批量删除 |

## 关联关系

- **引用表：**
  - [common_file](common_file.md) -- `resultfile_id` 和 `parentfile_id` 关联文件实体
  - [script_file](script_file.md) -- `script_id` 关联产生结果的脚本

## 涉及接口

- [ResultController](../接口文档/result/ResultController.md)（结果文件上传、查询、清理）

## 脚本服务侧使用

> 以下为脚本服务（filesystem / filemanagement 工程）侧视角，业务域：执行结果。

- 关联 Mapper（脚本服务侧）：`ScriptResultMapper`
- 相关接口（脚本服务侧）：[ScriptResultController](ScriptResultController.md)
- 脚本服务侧登记的关联关系：
  - 引用：[common_file](common_file.md)（resultfile_id FK）
  - 引用：[script_file](script_file.md)（通过 parentfile_id 关联）
- 字段/关联核实结论（2026-08-12 对实库验证）：
  - `parentfile_id` 实库注释为 ".test文件的id"（`resultfile_id` 注释为"对应common_file的file_id"），即**以文件管理服务侧为准**：关联 [common_file](common_file.md) 的 .test 测试描述文件；脚本服务侧"通过 parentfile_id 关联 script_file"的说法与实库注释不符
  - `expectedfile_name`、`script_id`、`is_clean` 三列在实库中真实存在，脚本服务侧实体登记不全
