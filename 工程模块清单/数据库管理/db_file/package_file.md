# package_file -- 应用安装包版本记录表

> 所属库：db_file
> 对应模型：`cn.testin.filecloud.model.PackageFile`
> 对应 Mapper：`PackageFileMapper.java` + `PackageFileMapper.xml`
> 使用方：文件管理服务（fileupload 工程）、脚本服务（filesystem / filemanagement 工程）
> 分支：syy.release.z7.8.1.0

## 表结构

| 字段 | 类型 | 允许空 | 默认值 | 说明 |
|---|---|---|---|---|
| `pkgid` | int(11) | NOT NULL | AUTO_INCREMENT | 包 ID，主键，自增 |
| `appid` | int(11) | YES | NULL | 应用 ID（关联 [common_app](common_app.md).appid） |
| `packageName` | varchar(100) | YES | NULL | 包名称（冗余自 common_app） |
| `iconFileUrl` | varchar(500) | YES | NULL | 应用图标 URL |
| `version_name` | varchar(50) | YES | NULL | APP 版本名称（调用方指定，如 "2.1.0"） |
| `version_code` | varchar(50) | YES | NULL | 版本码（调用方指定，如 "210"） |
| `build` | int(11) | YES | NULL | Build 号（调用方指定） |
| `pkgDesc` | varchar(200) | YES | NULL | 包描述 / 版本更新说明 |
| `startPath` | varchar(300) | YES | NULL | 启动路径（Activity/ViewController） |
| `uploadtime` | bigint(20) | YES | NULL | 应用上传时间（毫秒时间戳） |
| `uploadUser` | int(11) | YES | NULL | 上传人用户 ID |
| `fileid` | bigint(20) | NOT NULL | -- | 文件 ID（关联 [common_file](common_file.md).fileid），不允许为空 |
| `isdelete` | smallint(6) | YES | 0 | 删除标志：0=未删除, 1=逻辑删除, 2=物理删除 |
| `ostype` | smallint(6) | YES | NULL | 操作系统类别：1=Android, 2=iOS |
| `project_id` | int(11) | YES | NULL | 关联项目组 ID |
| `channel_id` | varchar(300) | YES | NULL | 应用渠道号（如华为、小米等渠道标识） |
| `eid` | int(100) | YES | NULL | 企业 ID（多租户） |
| `version_remark` | varchar(200) | YES | NULL | 版本备注（补充说明） |
| `appName` | varchar(255) | YES | NULL | 应用名称（冗余自 common_app） |

## 索引

| 索引名 | 字段 | 类型 |
|---|---|---|
| PRIMARY | `pkgid` | BTREE（主键） |
| `fk_pkg_fileid` | `fileid` | BTREE |
| `fk_pkg_appid` | `appid` | BTREE |

## 业务说明

`package_file` 记录应用的每个上传版本（APK/IPA 包）。一个应用可以有多个历史版本，每次上传新版本包在此表新增一条记录。

**关键设计：**
- 与 common_app 是 1:N 关系：一个应用对应多个版本包
- fileid 是必填项，指向 common_file 中实际的 APK/IPA 文件
- `fk_pkg_fileid` 和 `fk_pkg_appid` 两个外键索引支持快速关联查询
- `lastByAppid` 可查某应用在某项目下的最新包
- `deleteByClientCondition` 支持按项目ID+包名+版本+系统类型组合条件逻辑删除

**版本唯一性：** 同 appid + project_id 下可有多条记录，由 version_name + version_code + build 三元组标识不同版本

## Mapper 操作

| 方法名 | SQL 类型 | 用途 |
|---|---|---|
| `selectByPrimaryKey` | SELECT | 按 pkgid 查询包 |
| `selectByExample` | SELECT | 按 Example 条件动态查询 |
| `lastByAppid` | SELECT | 查询某应用某项目下的最新包（pkgid DESC LIMIT 1） |
| `selectByCommonFileId` | SELECT | 按 fileid 查询所属包 |
| `selectByCommonFileIdProjectId` | SELECT | 按 fileid + project_id + isdelete 精确查包 |
| `selectByCommonFileIdProjectIdUploadUid` | SELECT | 按 fileid + project_id + uploadUser 精确查包 |
| `countByExample` | SELECT | 按条件统计数量 |
| `insert` | INSERT | 完整插入包记录 |
| `insertSelective` | INSERT | 选择性插入，返回自增 pkgid |
| `updateByPrimaryKey` | UPDATE | 按主键全量更新 |
| `updateByPrimaryKeySelective` | UPDATE | 按主键选择性更新 |
| `updateByExample` | UPDATE | 按条件全量更新 |
| `updateByExampleSelective` | UPDATE | 按条件选择性更新 |
| `updateStatus` | UPDATE | 仅更新 isdelete 状态 |
| `deleteByClientCondition` | UPDATE | 按项目+包名+版本+系统组合条件逻辑删除 |
| `deleteByPrimaryKey` | DELETE | 按主键物理删除 |
| `deleteByExample` | DELETE | 按条件物理删除 |

## 关联关系

- **被以下表引用：**
  - [suite_app](suite_app.md) -- `pkgid` 关联套件中的应用包
- **引用表：**
  - [common_app](common_app.md) -- `appid` 外键关联（1:N）
  - [common_file](common_file.md) -- `fileid` 外键关联（1:1 包文件）

## 涉及接口

- [PackageController](../../文件管理服务/07-开放接口文档/文件上传/PackageController.md)（上传新版本、查询版本列表、删除版本）
- [ClientPackageController](../接口文档/package/ClientPackageController.md)（客户端拉取包信息）

## 脚本服务侧使用

> 以下为脚本服务（filesystem / filemanagement 工程）侧视角，业务域：App 管理。

- 关联 Mapper（脚本服务侧）：`PackageFileMapper`
- 相关接口（脚本服务侧）：[PackageFileController](PackageFileController.md)
- 脚本服务侧登记的关联关系：
  - 引用：[common_app](common_app.md)（appid FK）
  - 引用：[common_file](common_file.md)（fileid FK）
  - 被引用：[suite_app](suite_app.md)（pkgid FK）
  - 被引用：[script_file](script_file.md)（pkgid FK，脚本服务侧独有登记；文件管理服务侧的对应关系见 [script_file](script_file.md) 的 `pkg_id` 列）
