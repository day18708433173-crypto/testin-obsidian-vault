# common_file -- 文件存储中心表

> 所属库：db_file
> 对应模型：`cn.testin.filecloud.model.CommonFile`
> 对应 Mapper：`CommonFileMapper.java` + `CommonFileMapper.xml`
> 使用方：文件管理服务（fileupload 工程）、脚本服务（filesystem / filemanagement 工程）
> 分支：syy.release.z7.8.1.0

## 表结构

| 字段 | 类型 | 允许空 | 默认值 | 说明 |
|---|---|---|---|---|
| `fileid` | bigint(20) | NOT NULL | AUTO_INCREMENT | 文件 ID，主键，自增 |
| `type` | smallint(11) | YES | NULL | 文件类型：1=视频, 2=图片, 3=日志, 4=APP, 5=报告, 6=脚本, 7=证书 |
| `size` | bigint(11) | YES | NULL | 文件大小（字节） |
| `createUserId` | int(11) | YES | NULL | 创建用户 ID |
| `upload_user_name` | varchar(100) | YES | NULL | 上传人姓名 |
| `expireTime` | bigint(20) | YES | NULL | 文件过期时间（毫秒时间戳），用于过期清理 |
| `createTime` | bigint(20) | YES | NULL | 文件创建时间（毫秒时间戳） |
| `url` | varchar(300) | YES | NULL | 文件下载 URL / 存储路径 |
| `remark` | varchar(200) | YES | NULL | 文件备注信息 |
| `isdelete` | smallint(6) | YES | 0 | 删除标志：0=未删除, 1=逻辑删除, 2=物理删除 |
| `filemd5` | varchar(50) | YES | NULL | 文件 MD5 指纹，用于去重检测 |

## 索引

| 索引名 | 字段 | 类型 |
|---|---|---|
| PRIMARY | `fileid` | BTREE（主键） |
| `ind_file_md5` | `filemd5` | BTREE |

## 业务说明

`common_file` 是整个 文件管理服务 系统的核心文件存储表，充当文件注册中心。系统中所有上传的实体文件（APK、IPA、测试脚本、执行日志、报告、截图、证书等）都在此表登记一条元数据记录，实际文件内容存储于对象存储（OSS/MinIO 等），本表仅存储文件元信息和访问 URL。

**关键字段含义：**
- **type** 区分文件大类：视频(1)、图片(2)、日志(3)、APP(4)、报告(5)、脚本(6)、证书(7)
- **filemd5** 用于检测重复上传，同一文件通过 MD5 去重避免冗余存储
- **isdelete** 支持逻辑删除和物理删除两种模式，物理删除=2 表示可被清理任务回收
- **expireTime** 配合定时任务使用，超过过期时间且 type=-1 的记录会被标记删除
- **url** 存储 OSS/CDN 访问路径，前端通过此 URL 展示或下载文件

**生命周期：**
1. 文件上传时，先写 common_file 记录，返回 fileid
2. 业务表（如 package_file、script_file）通过 fileid 外键关联
3. 定时任务扫描 `type=-1` 且 `expireTime < now` 的记录进行过期清理
4. 业务删除文件时，通过 `isdelete=1` 逻辑删除；确认后可物理删除(`isdelete=2`)

## Mapper 操作

| 方法名 | SQL 类型 | 用途 |
|---|---|---|
| `selectByPrimaryKey` | SELECT | 按 fileid 查询单条文件记录 |
| `selectByExample` | SELECT | 按 Example 条件动态查询 |
| `selectByUrl` | SELECT | 按 url 精确查询文件 |
| `selectByMd5` | SELECT | 按 filemd5 查第一条匹配记录（去重查询） |
| `selectByMd5AndUid` | SELECT | 按 md5 + 可选 uid 查匹配记录（带用户过滤的去重） |
| `selectByIdList` | SELECT | 按 fileIdList 批量查询（IN 子句） |
| `selectExpiredFileList` | SELECT | 查询过期文件列表（type=-1, expireTime < now, isdelete=0），limit 50 |
| `countByExample` | SELECT | 按条件统计数量 |
| `insert` | INSERT | 完整插入一条文件记录 |
| `insertSelective` | INSERT | 选择性插入（只插非 null 字段），返回自增 fileid |
| `batchInsert` | INSERT | 批量插入文件记录 |
| `copyCommonFile` | INSERT...SELECT | 复制一条已有文件记录（除 fileid 外字段相同），返回新 fileid |
| `updateByPrimaryKey` | UPDATE | 按主键全量更新 |
| `updateByPrimaryKeySelective` | UPDATE | 按主键选择性更新 |
| `updateByExample` | UPDATE | 按条件全量更新 |
| `updateByExampleSelective` | UPDATE | 按条件选择性更新 |
| `updateToDeleted` | UPDATE | 批量逻辑删除（设置 isdelete=1） |
| `deleteByPrimaryKey` | DELETE | 按主键物理删除 |
| `deleteByExample` | DELETE | 按条件物理删除 |

## 关联关系

- **被以下表引用（通过 fileid 外键）：**
  - [package_file](package_file.md) -- `fileid` 关联应用包文件
  - [script_file](script_file.md) -- `fileid` 关联脚本文件
  - [script_result](script_result.md) -- `resultfile_id` 和 `parentfile_id` 关联结果文件和父文件
- **引用表：** 无直接引用其他表

## 涉及接口

- [FileController](../接口文档/file/FileController.md)（文件上传、下载、查询、删除）
- [CommonController](../接口文档/file/CommonController.md)（公共文件操作）

## 脚本服务侧使用

> 以下为脚本服务（filesystem / filemanagement 工程）侧视角，业务域：App 管理。

- 关联 Mapper（脚本服务侧）：`CommonFileMapper`
- 相关接口（脚本服务侧）：[CommonFileController](CommonFileController.md)
- 脚本服务侧登记的关联关系（与上文一致）：
  - 被引用：[script_file](script_file.md)（fileid FK）
  - 被引用：[package_file](package_file.md)（fileid FK）
  - 被引用：[script_result](script_result.md)（resultfile_id FK）
