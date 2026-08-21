# suite_info -- 测试套件主表

> 所属库：db_file
> 对应模型：`cn.testin.filecloud.model.SuiteInfo`
> 对应 Mapper：`SuiteInfoMapper.java` + `SuiteInfoMapper.xml`
> 使用方：文件管理服务（fileupload 工程）、脚本服务（filesystem / filemanagement 工程）
> 分支：syy.release.z7.8.1.0

## 表结构

| 字段 | 类型 | 允许空 | 默认值 | 说明 |
|---|---|---|---|---|
| `id` | int(11) | NOT NULL | AUTO_INCREMENT | 主键 ID，自增 |
| `name` | varchar(100) | NOT NULL | -- | 套件名称 |
| `eid` | int(11) | NOT NULL | -- | 企业 ID（多租户） |
| `projectid` | int(11) | NOT NULL | -- | 项目组 ID |
| `userid` | int(11) | NOT NULL | -- | 创建人用户 ID |
| `user_name` | varchar(32) | YES | NULL | 创建人姓名 |
| `descr` | varchar(200) | YES | NULL | 备注 / 描述 |
| `icon_url` | varchar(255) | YES | NULL | 套件图标 URL |
| `status` | int(11) | NOT NULL | -- | 数据状态（1=有效, 2=无效等） |
| `createtime` | bigint(20) | NOT NULL | -- | 创建时间（毫秒时间戳） |
| `updatetime` | bigint(20) | NOT NULL | -- | 更新时间（毫秒时间戳） |

## 索引

| 索引名 | 字段 | 类型 |
|---|---|---|
| PRIMARY | `id` | BTREE（主键） |
| `projectid` | `projectid`, `name` | BTREE（唯一索引） |

## 业务说明

`suite_info` 是测试套件的实体表，一个套件组合了一组应用和脚本用于批量测试执行。它本身不直接存储应用和脚本，而是通过 [suite_app](suite_app.md) 和 [suite_script](suite_script.md) 两个关联表建立多对多关系。

**核心设计：**
- 同一项目内套件名称唯一（projectid + name 唯一索引）
- status 控制套件的有效状态，查询时通常带上 status 过滤条件
- 套件删除通常是软删除（修改 status），而非物理删除

**套件聚合查询：**
- `selectAppCount` 通过 suite_app 表统计每个套件下绑定的应用数量
- `suiteCondition` 获取简化的下拉选项列表（id + name）

## Mapper 操作

| 方法名 | SQL 类型 | 用途 |
|---|---|---|
| `selectByConditions` | SELECT | 分页查询套件列表（按 projectid + name + status 过滤） |
| `selectCountByConditions` | SELECT | 按条件统计套件数量 |
| `selectAppCount` | SELECT | 统计指定套件 ID 列表下各自绑定的应用数量（GROUP BY suite_id） |
| `getByName` | SELECT | 按名称 + status 查套件 |
| `suiteCondition` | SELECT | 获取指定项目下的套件下拉列表（id + name） |
| `get` | SELECT | 按 id + status 精确查单条套件 |
| `insert` | INSERT | 选择性插入套件，返回自增 id |
| `update` | UPDATE | 选择性更新套件（name, descr, icon_url, status, updatetime） |

## 关联关系

- **被以下表引用：**
  - [suite_app](suite_app.md) -- `suite_id` 关联套件下的应用
  - [suite_script](suite_script.md) -- `suite_id` 关联套件下的脚本
- **相关视图：**
  - `view_suite_app` 聚合 suite_info + suite_app

## 涉及接口

- [SuiteController](../../脚本服务/07-开放接口文档/套件管理/SuiteController.md)（套件 CRUD、绑定/解绑应用和脚本）

## 脚本服务侧使用

> 以下为脚本服务（filesystem / filemanagement 工程）侧视角，业务域：套件管理。

- 关联 Mapper（脚本服务侧）：`SuiteInfoMapper`
- 相关接口（脚本服务侧）：[SuiteInfoController](SuiteInfoController.md)
- 脚本服务侧登记的关联关系（比上文多出 suite_group_script）：
  - 被引用：[suite_script](suite_script.md)（suiteId FK）
  - 被引用：[suite_group_script](suite_group_script.md)（suiteId FK）
  - 被引用：[suite_app](suite_app.md)（suite_id FK）
- 字段差异核实结论（2026-08-12 对实库验证）：实库无 `app_count` 列，脚本服务侧实体的 `appCount` 确认为**查询聚合填充的非持久化字段**
