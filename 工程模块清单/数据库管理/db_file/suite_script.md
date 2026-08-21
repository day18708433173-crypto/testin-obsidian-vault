# suite_script -- 套件-脚本绑定关系表

> 所属库：db_file
> 对应模型：`cn.testin.filecloud.model.SuiteScript`
> 对应 Mapper：`SuiteScriptMapper.java` + `SuiteScriptMapper.xml`
> 使用方：文件管理服务（fileupload 工程）、脚本服务（filesystem / filemanagement 工程）
> 分支：syy.release.z7.8.1.0

## 表结构

| 字段 | 类型 | 允许空 | 默认值 | 说明 |
|---|---|---|---|---|
| `suite_id` | int(11) | NOT NULL | -- | 套件 ID，复合主键（关联 [suite_info](suite_info.md).id） |
| `scriptid` | int(11) | NOT NULL | -- | 脚本 ID，复合主键（关联 [script_file](script_file.md).scriptid） |
| `status` | int(11) | NOT NULL | -- | 数据状态 |
| `createtime` | bigint(20) | NOT NULL | -- | 创建时间（毫秒时间戳） |
| `updatetime` | bigint(20) | NOT NULL | -- | 更新时间（毫秒时间戳） |

## 索引

| 索引名 | 字段 | 类型 |
|---|---|---|
| PRIMARY | `suite_id`, `scriptid` | BTREE（复合主键） |
| `scriptid` | `scriptid` | BTREE |

## 业务说明

`suite_script` 是多对多关联表，记录测试套件包含哪些脚本。一个套件可以包含多个脚本，同一脚本不会重复出现在同一套件中。

**关键设计：**
- 复合主键 `(suite_id, scriptid)` 防止重复绑定
- `scriptid` 独立索引支持反向查询：哪些套件包含了指定脚本
- 该表通过 MyBatis Generator 生成完整的 CRUD（含 Example 条件查询）
- `checkSuiteByProjectId` 是跨表校验方法：验证 suite_id 是否属于指定 projectId（通过查 suite_info）

**操作场景：**
1. 套件中添加脚本：INSERT 绑定记录
2. 套件中移除脚本：DELETE 绑定记录
3. 查询套件下所有脚本：`selectByExample` 按 suite_id 过滤
4. 反向查询脚本所属套件：按 scriptid 过滤

## Mapper 操作

| 方法名 | SQL 类型 | 用途 |
|---|---|---|
| `selectByPrimaryKey` | SELECT | 按 (suite_id, scriptid) 复合主键查询 |
| `selectByExample` | SELECT | 按 Example 条件动态查询 |
| `countByExample` | SELECT | 按条件统计 |
| `checkSuiteByProjectId` | SELECT | 跨表校验（查 suite_info 验证 suite_id 是否属于指定 project） |
| `insert` | INSERT | 完整插入绑定关系 |
| `insertSelective` | INSERT | 选择性插入 |
| `updateByPrimaryKey` | UPDATE | 按复合主键全量更新 |
| `updateByPrimaryKeySelective` | UPDATE | 按复合主键选择性更新 |
| `updateByExample` | UPDATE | 按条件全量更新 |
| `updateByExampleSelective` | UPDATE | 按条件选择性更新 |
| `deleteByPrimaryKey` | DELETE | 按复合主键删除 |
| `deleteByExample` | DELETE | 按条件删除 |

## 关联关系

- **引用表：**
  - [suite_info](suite_info.md) -- `suite_id` 关联套件实体
  - [script_file](script_file.md) -- `scriptid` 关联脚本

## 涉及接口

- [SuiteController](../../脚本服务/07-开放接口文档/套件管理/SuiteController.md)（套件中增删脚本、查询套件脚本列表）

## 脚本服务侧使用

> 以下为脚本服务（filesystem / filemanagement 工程）侧视角，业务域：套件管理。

- 关联 Mapper（脚本服务侧）：`SuiteScriptMapper`
- 相关接口（脚本服务侧）：[SuiteScriptController](SuiteScriptController.md)
- 脚本服务侧登记的关联关系（与上文一致）：
  - 引用：[suite_info](suite_info.md)（suiteId FK）
  - 引用：[script_file](script_file.md)（scriptid FK）
