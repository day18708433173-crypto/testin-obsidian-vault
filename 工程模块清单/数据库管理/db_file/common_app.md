# common_app -- 应用注册表

> 所属库：db_file
> 对应模型：`cn.testin.filecloud.model.CommonApp`
> 对应 Mapper：`CommonAppMapper.java` + `CommonAppMapper.xml`
> 使用方：文件管理服务（fileupload 工程）、脚本服务（filesystem / filemanagement 工程）
> 分支：syy.release.z7.8.1.0

## 表结构

| 字段 | 类型 | 允许空 | 默认值 | 说明 |
|---|---|---|---|---|
| `appid` | int(11) | NOT NULL | AUTO_INCREMENT | 应用 ID，主键，自增 |
| `appName` | varchar(100) | YES | NULL | 应用名称 |
| `packageName` | varchar(100) | YES | NULL | 上传应用的包名（Android Package / iOS Bundle ID） |
| `appDesc` | varchar(200) | YES | NULL | 应用介绍 / 描述 |
| `appcreateTime` | bigint(20) | YES | NULL | 应用创建时间（毫秒时间戳） |
| `isdelete` | smallint(6) | YES | 0 | 删除标志：0=未删除, 1=逻辑删除, 2=物理删除 |
| `signfileid` | bigint(20) | YES | NULL | Keystore 签名文件的 fileid（关联 [common_file](common_file.md)） |
| `key_pass` | varchar(100) | YES | NULL | 签名密码（Keystore key password） |
| `store_pass` | varchar(100) | YES | NULL | 签名库密码（Keystore store password） |
| `alias_name` | varchar(100) | YES | NULL | 应用签名别名 |
| `app_md5` | varchar(100) | YES | NULL | 上传 APK 后计算的指纹（MD5） |
| `app_key` | varchar(100) | YES | NULL | 应用密钥（预留） |
| `app_ext` | varchar(100) | YES | `{"appType":"1","appTypeName":"101001"}` | 扩展字段（JSON），记录 appType 等额外信息 |
| `eid` | int(100) | YES | NULL | 企业 ID（多租户标识） |
| `osType` | int(11) | YES | NULL | 操作系统类型标识 |

## 索引

| 索引名 | 字段 | 类型 |
|---|---|---|
| PRIMARY | `appid` | BTREE（主键） |
| `packageName` | `packageName`, `osType`, `appName` | BTREE（唯一索引） |

## 业务说明

`common_app` 是应用注册表，系统中每个被测应用在此登记一条记录。一个应用（packageName + osType + appName 唯一）可以有多个版本（对应 [package_file](package_file.md) 中的多条记录）。

**关键设计：**
- tri-key 唯一索引：packageName + osType + appName 确保同企微/同OS下不重复注册同名应用
- 签名信息直接冗余在应用表上，用于自动化测试时的重签名操作
- app_ext 存放 JSON 扩展信息，如 `appType`（应用类型，1=原生, 4=小程序等）
- osType 区分 Android(1) 和 iOS(2) 等不同平台的应用
- eid 用于多租户隔离

## Mapper 操作

| 方法名 | SQL 类型 | 用途 |
|---|---|---|
| `selectByPrimaryKey` | SELECT | 按 appid 查询应用 |
| `selectByExample` | SELECT | 按 Example 条件动态查询 |
| `queryOneAppByPackageName` | SELECT | 按 packageName + osType 精确查一条 |
| `queryOneAppByPackageNameAndAppName` | SELECT | 按 packageName + osType + appName 精确查一条 |
| `selectByMap` | SELECT | 按 packageName + appid 组合查询 |
| `countByExample` | SELECT | 按条件统计数量 |
| `insert` | INSERT | 完整插入应用记录 |
| `insertSelective` | INSERT | 选择性插入，返回自增 appid |
| `updateByPrimaryKey` | UPDATE | 按主键全量更新 |
| `updateByPrimaryKeySelective` | UPDATE | 按主键选择性更新 |
| `updateByExample` | UPDATE | 按条件全量更新 |
| `updateByExampleSelective` | UPDATE | 按条件选择性更新 |
| `deleteByPrimaryKey` | DELETE | 按主键物理删除 |
| `deleteByExample` | DELETE | 按条件物理删除 |

## 关联关系

- **被以下表引用（通过 appid 外键）：**
  - [package_file](package_file.md) -- `appid` 关联应用
  - [script_file](script_file.md) -- `appid` 关联脚本所属应用
- **引用表：**
  - [common_file](common_file.md) -- `signfileid` 关联签名证书文件

## 涉及接口

- [AppController](../../文件管理服务/07-开放接口文档/文件上传/AppController.md)（应用注册、查询、更新、删除）
- [PackageController](../../文件管理服务/07-开放接口文档/文件上传/PackageController.md)（上传应用包时关联应用）

## 脚本服务侧使用

> 以下为脚本服务（filesystem / filemanagement 工程）侧视角，业务域：App 管理。

- 关联 Mapper（脚本服务侧）：`CommonAppMapper`
- 相关接口（脚本服务侧）：[CommonAppController](CommonAppController.md)
- 脚本服务侧登记的关联关系（与上文一致）：
  - 被引用：[package_file](package_file.md)（appid FK）
  - 被引用：[script_file](script_file.md)（appid FK）
- 差异说明：脚本服务侧实体字段采用 `appCreateTime` / `aliasName` / `os_type` 命名风格，与上文 DDL 列名 `appcreateTime` / `alias_name` / `osType` 为同列不同命名；脚本服务侧实体未列出 `eid` 字段
