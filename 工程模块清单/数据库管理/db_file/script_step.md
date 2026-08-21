# script_step -- 脚本步骤明细表

> 所属库：db_file
> 对应模型：`cn.testin.filecloud.model.ScriptStep`
> 对应 Mapper：`ScriptStepMapper.java` + `ScriptStepMapper.xml`
> 使用方：文件管理服务（fileupload 工程）、脚本服务（filesystem / filemanagement 工程）
> 分支：syy.release.z7.8.1.0

## 表结构

| 字段 | 类型 | 允许空 | 默认值 | 说明 |
|---|---|---|---|---|
| `scriptid` | int(11) | NOT NULL | -- | 脚本 ID，复合主键（关联 [script_file](script_file.md).scriptid） |
| `stepid` | smallint(6) | NOT NULL | -- | 步骤 ID，复合主键（脚本内序号） |
| `steptype` | varchar(50) | YES | NULL | 步骤类型（如 click、input、swipe、assert 等） |
| `stepname` | varchar(1000) | YES | NULL | 步骤名称 / 描述文本 |
| `stepimage` | varchar(200) | YES | NULL | 步骤截图 URL |
| `stepdesc` | text | YES | NULL | 步骤描述（长文本） |
| `remark` | varchar(200) | YES | NULL | 步骤备注 |
| `child_script_id` | int(11) | YES | NULL | 调用的子脚本 ID |
| `child_script_no` | int(11) | YES | NULL | 调用的子脚本编号 |
| `child_script_uploaduser` | int(11) | YES | NULL | 子脚本上传人 |
| `ext` | text | YES | NULL | 步骤的全部属性（JSON 格式扩展） |
| `expr` | varchar(255) | YES | NULL | 表达式（条件判断逻辑） |
| `small_img` | varchar(255) | YES | NULL | 小图 URL |
| `thumb_img` | varchar(255) | YES | NULL | 缩略图 URL |

## 索引

| 索引名 | 字段 | 类型 |
|---|---|---|
| PRIMARY | `scriptid`, `stepid` | BTREE（复合主键） |

## 业务说明

`script_step` 记录自动化测试脚本的每一个执行步骤。一个脚本（scriptid）包含多个步骤，通过 stepid 排序。这是测试脚本的步骤级拆解存储，用于回放引擎按序执行。

**核心设计：**
- 复合主键 `(scriptid, stepid)` 确保同一脚本内步骤不重复
- `steptype` 标识操作类型：click(点击), input(输入), swipe(滑动), assert(断言), sleep(等待), 等
- `ext` 以 JSON 存储步骤的完整属性集（坐标、文本、控件指纹等）
- `child_script_id/no` 支持步骤级脚本嵌套调用（子脚本引用）
- `expr` 存储条件表达式用于条件分支执行
- `stepimage` + `small_img` + `thumb_img` 提供多分辨率截图用于控件匹配

**与 script_file 的关系：**
- 一个 script_file 对应 0~N 条 script_step
- 脚本删除/更新时，先删旧步骤（`deleteByScriptId`），再批量插入新步骤
- scriptmain（JSON）中包含步骤数据，script_step 是其结构化存储补充

## Mapper 操作

| 方法名 | SQL 类型 | 用途 |
|---|---|---|
| `selectByPrimaryKey` | SELECT | 按 (scriptid, stepid) 查单条步骤（含 BLOB stepdesc, ext） |
| `selectByScriptId` | SELECT | 查询某脚本的全部步骤（SELECT *） |
| `insert` | INSERT | 完整插入步骤 |
| `insertSelective` | INSERT | 选择性插入步骤 |
| `updateByPrimaryKey` | UPDATE | 按复合主键更新（不含 BLOB 字段） |
| `updateByPrimaryKeyWithBLOBs` | UPDATE | 按复合主键更新（含 BLOB 字段 stepdesc, ext） |
| `updateByPrimaryKeySelective` | UPDATE | 按复合主键选择性更新 |
| `deleteByPrimaryKey` | DELETE | 按 (scriptid, stepid) 删除单条步骤 |
| `deleteByScriptId` | DELETE | 删除某脚本的全部步骤（用于脚本更新时的步骤刷新） |

## 关联关系

- **引用表：**
  - [script_file](script_file.md) -- `scriptid` 关联所属脚本版本
  - [script_file](script_file.md) -- `child_script_id` / `child_script_no` 关联调用的子脚本

## 涉及接口

- [ScriptController](../../文件管理服务/07-开放接口文档/脚本管理/ScriptController.md)（保存脚本时写入步骤）
- [StepController](../接口文档/step/StepController.md)（步骤查询）

## 脚本服务侧使用

> 以下为脚本服务（filesystem / filemanagement 工程）侧视角，业务域：脚本步骤。

- 关联 Mapper（脚本服务侧）：`ScriptStepMapper`
- 相关接口（脚本服务侧）：[ScriptStepController](ScriptStepController.md)
- 脚本服务侧登记的关联关系（与上文一致）：
  - 引用：[script_file](script_file.md)（scriptid FK）
- 字段差异核实结论（2026-08-12 对实库验证）：实库无 `describe`、`uuid` 两列（步骤描述列为 `stepdesc`），脚本服务侧实体的这两个字段为**非持久化/映射命名**字段
