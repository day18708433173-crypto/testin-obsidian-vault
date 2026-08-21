# script_relation -- 脚本父子调用关系表

> 所属库：db_file
> 对应模型：`cn.testin.filecloud.model.ScriptRelation`
> 对应 Mapper：`ScriptRelationMapper.java` + `ScriptRelationMapper.xml`
> 使用方：文件管理服务（fileupload 工程）、脚本服务（filesystem / filemanagement 工程）
> 分支：syy.release.z7.8.1.0

## 表结构

| 字段 | 类型 | 允许空 | 默认值 | 说明 |
|---|---|---|---|---|
| `id` | int(11) | NOT NULL | AUTO_INCREMENT | 主键 ID，自增 |
| `script_id` | int(11) | YES | NULL | 脚本 ID（关联 [script_file](script_file.md).scriptid） |
| `script_no` | int(11) | YES | NULL | 脚本编号（关联 [script_file](script_file.md).scriptno） |
| `parent_script_id` | int(11) | YES | NULL | 父脚本 ID（关联 [script_file](script_file.md).scriptid） |
| `parent_script_no` | int(11) | YES | NULL | 父脚本编号（关联 [script_file](script_file.md).scriptno） |
| `type` | int(1) | NOT NULL | 0 | 调用类型：0=被注释掉的调用脚本, 1=被调用脚本的注释 |

## 索引

| 索引名 | 字段 | 类型 |
|---|---|---|
| PRIMARY | `id` | BTREE（主键） |
| `idx_script_no` | `script_no` | BTREE |
| `idx_parent_script_no_parent_script_id` | `parent_script_no`, `parent_script_id` | BTREE |

## 业务说明

`script_relation` 记录了脚本之间的父子调用关系，构成**脚本调用图**。当某个脚本的步骤中通过 `child_script_id` 引用了另一个脚本时，在此表登记一条父子关系。

**关键设计：**
- 方向性：`(parent_script_id, parent_script_no)` -> `(script_id, script_no)` 表示 parent 调用 child
- `type` 区分调用关系的活跃状态：
  - 0 = 调用关系存在但被注释掉（代码中注释了调用语句）
  - 1 = 调用语句被注释（注释中引用了脚本）
- 支持递归查询：MySQL 函数 `getParentScripts(scriptNo)` 通过此表递归查找所有上级脚本

**用途：**
1. 脚本回放时自动拉取依赖的子脚本
2. 脚本删除/更新时检查影响范围（谁调用了这个脚本）
3. 脚本拓扑图可视化
4. 通过 `listByConditions` 传入 projectId 时 INNER JOIN script_file 过滤项目范围

## Mapper 操作

| 方法名 | SQL 类型 | 用途 |
|---|---|---|
| `selectByPrimaryKey` | SELECT | 按 id 查询关系 |
| `selectByScriptId` | SELECT | 按 script_id 查询该脚本的所有调用关系 |
| `listByConditions` | SELECT | 多条件查询（scriptId, scriptNo, parentScriptId, parentScriptNo, type, projectId） |
| `insert` | INSERT | 选择性插入调用关系，返回自增 id |
| `deleteByScriptid` | DELETE | 按 script_id 删除该脚本的所有调用关系 |

## 关联关系

- **引用表：**
  - [script_file](script_file.md) -- script_id 和 parent_script_id 都关联脚本表
- **相关函数：**
  - MySQL FUNCTION `getParentScripts(scriptNo)` 依赖此表递归查询

## 涉及接口

- [ScriptRelationController](../接口文档/script/ScriptRelationController.md)（查询脚本调用图、添加/移除调用关系）

## 脚本服务侧使用

> 以下为脚本服务（filesystem / filemanagement 工程）侧视角，业务域：脚本关系。

- 关联 Mapper（脚本服务侧）：`ScriptRelationMapper`
- 相关接口（脚本服务侧）：[ScriptRelationController](ScriptRelationController.md)
- 脚本服务侧登记的关联关系（与上文一致）：
  - 引用：[script_file](script_file.md)（script_id FK）
  - 引用：[script_file](script_file.md)（parent_script_id FK）
