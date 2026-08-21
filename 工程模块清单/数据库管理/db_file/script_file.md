# script_file -- 脚本实体主表

> 所属库：db_file
> 对应模型：`cn.testin.filecloud.model.ScriptFile`
> 对应 Mapper：`ScriptFileMapper.java` + `ScriptFileMapper.xml`
> 使用方：文件管理服务（fileupload 工程）、脚本服务（filesystem / filemanagement 工程）
> 分支：syy.release.z7.8.1.0

## 表结构

| 字段 | 类型 | 允许空 | 默认值 | 说明 |
|---|---|---|---|---|
| `scriptid` | int(11) | NOT NULL | AUTO_INCREMENT | 脚本 ID，主键，自增（每个版本一条） |
| `scriptno` | int(11) | YES | NULL | 脚本编号，相同脚本名共享一个编号，不同脚本名新建编号 |
| `projectid` | int(11) | YES | NULL | 项目 ID |
| `scriptName` | varchar(100) | YES | NULL | 脚本名称 |
| `scriptCreateTime` | bigint(20) | YES | NULL | 脚本创建时间（毫秒时间戳） |
| `scriptCreateUser` | int(11) | YES | NULL | 脚本创建人用户 ID |
| `scriptCreateDesc` | varchar(200) | YES | NULL | 脚本创建描述 |
| `ostype` | smallint(11) | YES | NULL | 脚本类型：1=Android, 2=iOS |
| `appid` | int(11) | YES | NULL | 应用 ID（关联 [common_app](common_app.md).appid） |
| `build` | int(11) | YES | 0 | Build 号（调用方指定） |
| `scriptUpdateTime` | bigint(20) | YES | NULL | 脚本更新时间（毫秒时间戳） |
| `scriptUpdateUserid` | int(11) | YES | NULL | 脚本更新人用户 ID |
| `scriptUpdateDesc` | varchar(200) | YES | NULL | 脚本更新描述 |
| `scriptDesignUserid` | int(11) | YES | NULL | 脚本设计人（默认创建人） |
| `scriptUpdateType` | int(11) | YES | NULL | 更新方式：1=itestin, 2=web, 3=历史记录 |
| `scriptmain` | longtext | YES | NULL | 脚本实体，数据库中以 JSON OBJECT 格式存储 |
| `fileid` | bigint(11) | YES | NULL | 文件 ID（关联 [common_file](common_file.md).fileid） |
| `isdelete` | smallint(6) | YES | 0 | 删除标志：0=未删除, 1=逻辑删除, 2=物理删除 |
| `adapterversionname` | varchar(50) | YES | NULL | 适配器版本名称 |
| `adapterversioncode` | varchar(50) | YES | NULL | 适配器版本码 |
| `remark` | varchar(100) | YES | NULL | 备注 |
| `scriptType` | int(11) | YES | 0 | 脚本类型：1=app, 3=web, 5=pc |
| `scripttags` | varchar(400) | YES | NULL | 脚本标签（逗号分隔） |
| `script_sig` | varchar(100) | YES | NULL | 客户端记录的脚本 MD5，回放时用于判断脚本是否变更 |
| `delete_tag` | varchar(255) | YES | '' | 删除标记堆栈（记录删除操作人信息） |
| `record_type` | int(11) | YES | -1 | 录制方式 |
| `channel_id` | varchar(300) | YES | NULL | 应用渠道号 |
| `step_file_id` | varchar(200) | YES | NULL | 步骤文件 ID |
| `declare_vars` | varchar(200) | YES | NULL | 声明变量 |
| `eid` | int(100) | YES | NULL | 企业 ID（多租户） |
| `pkg_id` | int(100) | YES | NULL | 应用所属包 ID（关联 [package_file](package_file.md).pkgid） |
| `appName` | varchar(255) | YES | NULL | 应用名称（冗余） |
| `ext` | varchar(2000) | YES | NULL | 扩展字段（JSON） |
| `history` | int(11) | NOT NULL | -- | 是否为历史记录（0=当前版本, 非0=历史版本） |
| `upload_file_user_name` | varchar(255) | YES | NULL | 上传人姓名 |
| `packageName` | varchar(255) | YES | NULL | 包名（冗余自 common_app） |
| `assoc_case_num` | int(100) | YES | NULL | 脚本关联用例数 |
| `script_uuid` | varchar(50) | YES | NULL | 脚本唯一标识（UUID） |

## 索引

| 索引名 | 字段 | 类型 |
|---|---|---|
| PRIMARY | `scriptid` | BTREE（主键） |
| `fk_script_fileid` | `fileid` | BTREE |
| `fk_script_appid` | `appid` | BTREE |
| `ind_script_file_scriptno` | `scriptno` | BTREE |
| `ind_scriptno` | `scriptno` | BTREE（冗余索引） |
| `fk_script_project` | `projectid` | BTREE |
| `idx_script_uuid` | `script_uuid` | BTREE |

## 业务说明

`script_file` 是整个自动化测试脚本系统的核心表，记录脚本的全部版本信息。每个脚本名对应一个 `scriptno`，同一脚本的每次修改产生一个带新 `scriptid` 的版本记录。

**核心设计模式 -- 版本化存储：**
- `scriptno` 标识"逻辑脚本"（同一命名下的所有版本共享）
- `scriptid` 标识"物理版本"（每次修改/上传产生新 ID）
- `history` 区分当前活跃版本(0)和历史版本(非0)
- `scriptmain` 以 JSON 存储在数据库，避免拆表复杂度
- `script_sig` 提供客户端侧 MD5 校验，回放时快速判断脚本是否变更

**更新方式 scriptUpdateType：**
- 1 = itestin 客户端直接上传
- 2 = Web 平台在线编辑
- 3 = 历史版本恢复

**删除策略：**
- 逻辑删除通过 isdelete 标记，同时在 delete_tag 追加删除人信息
- 支持按项目ID+scriptno+创建人+适配器版本等组合条件删除

**生命周期：**
1. 脚本录制/创建 -> 生成新 scriptid，scriptno 可能复用或新建
2. 脚本编辑 -> 复制旧记录生成新 scriptid（copyScriptFile），更新 scriptmain
3. 版本发布 -> history 标记旧版本，新版本设为当前
4. 脚本删除 -> isdelete 标记逻辑删除，保留 delete_tag 记录

## Mapper 操作

| 方法名 | SQL 类型 | 用途 |
|---|---|---|
| `selectByPrimaryKey` | SELECT | 按 scriptid 查脚本（含 BLOB scriptmain） |
| `selectAllByPrimaryKey` | SELECT | 按 scriptid 查全部字段（SELECT *） |
| `selectByConditions` | SELECT | 多条件动态查询（LEFT JOIN common_file 获取上传人姓名） |
| `selectByCommonFileId` | SELECT | 按 fileid 反查脚本 |
| `selectLastScriptFileByScriptNo` | SELECT | 按 scriptNo + projectId + history=0 查最新版本（LEFT JOIN script_check） |
| `getLastValidScriptFileByScriptNo` | SELECT | 查最新有效脚本（INNER JOIN script_check WHERE check_status=1） |
| `selectExistsRecords` | SELECT | 查未删除的指定 scriptNo 列表记录 |
| `selectLastUpdateScriptFileRecords` | SELECT | 查每个 scriptNo 的最新一条记录（子查询 max(scriptid) group by scriptno） |
| `queryPageListByCondition` | SELECT | 分页列表查询（含创建人/更新人筛选、描述/标签模糊搜索、渠道过滤） |
| `queryProjectScriptListByCreateTimeRange` | SELECT | 按时间范围查项目脚本（INNER JOIN common_file） |
| `insert` | INSERT | 完整插入脚本 |
| `insertSelective` | INSERT | 选择性插入，返回自增 scriptid |
| `copyScriptFile` | INSERT...SELECT | 复制一条脚本记录生成新版本（INSERT INTO...SELECT） |
| `updateByPrimaryKeySelective` | UPDATE | 按主键选择性更新 |
| `deleteByPrimaryKey` | DELETE | 按主键物理删除 |
| `deleteByClientCondition` | UPDATE | 按客户条件逻辑删除（设 isdelete=1 并追加 delete_tag） |

## 关联关系

- **被以下表引用：**
  - [script_step](script_step.md) -- `scriptid` 复合主键关联脚本步骤
  - [script_check](script_check.md) -- `script_id` 关联脚本有效性检查结果
  - [script_relation](script_relation.md) -- `script_id` / `parent_script_id` 关联父子脚本
  - [suite_script](suite_script.md) -- `scriptid` 关联套件中的脚本
  - [script_at_last](script_at_last.md) -- `script_id` 冗余最新版本
- **引用表：**
  - [common_file](common_file.md) -- `fileid` 外键关联脚本文件
  - [common_app](common_app.md) -- `appid` 外键关联被测应用
  - [package_file](package_file.md) -- `pkg_id` 关联应用包

## 涉及接口

- [ScriptController](../../文件管理服务/07-开放接口文档/脚本管理/ScriptController.md)（脚本 CRUD、版本管理、查询）
- [ScriptCheckController](../接口文档/script/ScriptCheckController.md)（脚本有效性检查入口）
- [ScriptRelationController](../接口文档/script/ScriptRelationController.md)（脚本关系查询）

## 脚本服务侧使用

> 以下为脚本服务（filesystem / filemanagement 工程）侧视角，业务域：脚本核心。

- 关联 Mapper（脚本服务侧）：`ScriptFileMapper`
- 相关接口（脚本服务侧）：[ScriptFileController](ScriptFileController.md)
- 脚本服务侧登记的关联关系（比上文多出 script_tag / script_dir_child / script_dir）：
  - 被引用：[script_step](script_step.md)（scriptid FK）
  - 被引用：[script_relation](script_relation.md)（script_id FK）
  - 被引用：[script_tag](script_tag.md)（script_no FK）
  - 被引用：[script_check](script_check.md)（scriptId FK）
  - 被引用：[script_dir_child](script_dir_child.md)（script_no FK）
  - 被引用：[suite_script](suite_script.md)（scriptid FK）
  - 引用：[common_file](common_file.md)（fileid FK）
  - 引用：[script_dir](script_dir.md)（scriptDirId FK）
- 字段差异核实结论（2026-08-12 对实库验证）：
  - 应用包 ID 列实库为 `pkg_id`（int(100)），**文件管理服务侧 DDL 正确**；脚本服务侧实体记的 `pkgid` 为实体属性命名，与实库列名不符
  - `scriptDirId`：实库无 `script_dir_id` 列，脚本服务侧实体的 `scriptDirId` 为**非持久化字段**（目录关联实际走 script_dir_child.script_no，见上）
