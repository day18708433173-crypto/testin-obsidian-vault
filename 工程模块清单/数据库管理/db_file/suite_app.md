# suite_app -- 套件-应用绑定关系表

> 所属库：db_file
> 对应模型：`cn.testin.filecloud.model.SuiteApp`
> 对应 Mapper：`SuiteAppMapper.java` + `SuiteAppMapper.xml`
> 使用方：文件管理服务（fileupload 工程）、脚本服务（filesystem / filemanagement 工程）
> 分支：syy.release.z7.8.1.0

## 表结构

| 字段 | 类型 | 允许空 | 默认值 | 说明 |
|---|---|---|---|---|
| `suite_id` | int(11) | NOT NULL | -- | 套件 ID，复合主键（关联 [suite_info](suite_info.md).id） |
| `pkgid` | int(11) | NOT NULL | -- | 应用包 ID，复合主键（关联 [package_file](package_file.md).pkgid） |
| `package_name` | varchar(100) | YES | NULL | 应用包名（冗余） |
| `status` | int(11) | NOT NULL | -- | 数据状态 |
| `createtime` | bigint(20) | NOT NULL | -- | 创建时间（毫秒时间戳） |
| `updatetime` | bigint(20) | NOT NULL | -- | 更新时间（毫秒时间戳） |

## 索引

| 索引名 | 字段 | 类型 |
|---|---|---|
| PRIMARY | `suite_id`, `pkgid` | BTREE（复合主键） |

## 业务说明

`suite_app` 是多对多关联表，记录测试套件绑定了哪些应用版本包。一个套件可以绑定多个应用包，同一应用包的同一版本不会重复绑定到同一套件。

**关键设计：**
- 复合主键 `(suite_id, pkgid)` 确保同一套件不会重复绑定同一应用包
- `package_name` 冗余字段，方便查询时不必 JOIN package_file
- 通过 `delete` 方法按 suite_id（或 suite_id + pkgid）移除绑定

**操作场景：**
1. 套件中添加应用：INSERT 绑定记录
2. 套件中移除应用：DELETE 绑定记录
3. 查询套件下所有应用：`selectBySuiteId`
4. 按包名查找绑定关系：`getByPackageName`

## Mapper 操作

| 方法名 | SQL 类型 | 用途 |
|---|---|---|
| `selectBySuiteId` | SELECT | 按 suite_id + status 查套件下所有绑定应用 |
| `getByPackageName` | SELECT | 按 pkgid / packageName + status 查绑定关系 |
| `get` | SELECT | 按 suite_id + pkgid + status 精确查绑定 |
| `insert` | INSERT | 插入绑定关系 |
| `delete` | DELETE | 按 suite_id（可选 pkgid）删除绑定 |

## 关联关系

- **引用表：**
  - [suite_info](suite_info.md) -- `suite_id` 关联套件实体
  - [package_file](package_file.md) -- `pkgid` 关联应用包

## 涉及接口

- [SuiteController](../../脚本服务/07-开放接口文档/套件管理/SuiteController.md)（套件中增删应用、查询套件应用列表）

## 脚本服务侧使用

> 以下为脚本服务（filesystem / filemanagement 工程）侧视角，业务域：套件管理。

- 关联 Mapper（脚本服务侧）：`SuiteAppMapper`
- 相关接口（脚本服务侧）：[SuiteAppController](SuiteAppController.md)
- 脚本服务侧登记的关联关系（与上文一致）：
  - 引用：[suite_info](suite_info.md)（suite_id FK）
  - 引用：[package_file](package_file.md)（pkgid FK）
