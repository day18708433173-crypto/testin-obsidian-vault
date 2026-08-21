# script_check -- 脚本有效性检查结果表

> 所属库：db_file
> 对应模型：`cn.testin.filecloud.model.ScriptCheck`
> 对应 Mapper：`ScriptCheckMapper.java` + `ScriptCheckMapper.xml`
> 使用方：文件管理服务（fileupload 工程）、脚本服务（filesystem / filemanagement 工程）
> 分支：syy.release.z7.8.1.0

## 表结构

| 字段 | 类型 | 允许空 | 默认值 | 说明 |
|---|---|---|---|---|
| `id` | int(11) | NOT NULL | AUTO_INCREMENT | 主键 ID，自增 |
| `script_id` | int(11) | NOT NULL | -- | 脚本 ID（关联 [script_file](script_file.md).scriptid） |
| `script_no` | int(11) | NOT NULL | -- | 脚本编号（关联 [script_file](script_file.md).scriptno） |
| `script_create_desc` | varchar(1024) | YES | NULL | 脚本描述（冗余自 script_file） |
| `check_status` | tinyint(4) | YES | NULL | 检查状态：1=有效, 0=无效 |
| `check_content` | text | YES | NULL | 检测内容详情（校验问题 JSON） |
| `check_time` | bigint(20) | YES | NULL | 检测时间（毫秒时间戳） |

## 索引

| 索引名 | 字段 | 类型 |
|---|---|---|
| PRIMARY | `id` | BTREE（主键） |
| `index_scriptid` | `script_id` | BTREE（唯一索引） |
| `index_scriptNo` | `script_no` | BTREE |

## 业务说明

`script_check` 记录每个脚本版本的有效性校验结果。脚本上传或编辑后，系统会对脚本内容进行有效性检查（语法验证、步骤完整性、控件有效性等），结果写入此表。

**关键设计：**
- `script_id` 唯一索引：每个脚本版本只保留一条最新检查结果
- `check_status` 为 1 表示脚本可正常执行，0 表示存在校验问题
- `check_content` 以 JSON 存储详细的检查问题列表（如具体哪些步骤有问题）
- 查询最新有效脚本时通过 `INNER JOIN script_check WHERE check_status=1` 过滤

**check_status 对业务的影响：**
- 执行器仅拉取 check_status=1 的脚本进行回放
- 状态同步表 [script_check_status_sync](script_check_status_sync.md) 依赖此结果进行跨节点同步

## Mapper 操作

| 方法名 | SQL 类型 | 用途 |
|---|---|---|
| `selectByPrimaryKey` | SELECT | 按 id 查询检查记录 |
| `selectByScriptId` | SELECT | 按 script_id 查询检查记录 |
| `listByConditions` | SELECT | 按 scriptId / scriptNo 组合条件查询 |
| `checkStatusByScriptNo` | SELECT | 按 scriptNo 查询所有关联记录的 DISTINCT check_status |
| `insert` | INSERT | 选择性插入检查结果，返回自增 id |
| `deleteByScriptid` | DELETE | 按 script_id 删除检查记录（更新脚本时清除旧结果） |

## 关联关系

- **引用表：**
  - [script_file](script_file.md) -- `script_id` / `script_no` 关联被检查的脚本
- **被以下表引用：**
  - [script_check_status_sync](script_check_status_sync.md) -- 依赖此表的检查结果进行跨节点同步

## 涉及接口

- [ScriptCheckController](../接口文档/script/ScriptCheckController.md)（脚本检查接口，写入/更新检查结果）

## 脚本服务侧使用

> 以下为脚本服务（filesystem / filemanagement 工程）侧视角，业务域：脚本校验。

- 关联 Mapper（脚本服务侧）：`ScriptCheckMapper`
- 相关接口（脚本服务侧）：[ScriptCheckController](ScriptCheckController.md)
- 脚本服务侧登记的关联关系（与上文一致）：
  - 引用：[script_file](script_file.md)（scriptId FK）
  - 被引用：[script_check_status_sync](script_check_status_sync.md)（通过 script_id）
