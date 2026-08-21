# upload_service_auth -- 外部服务 API 鉴权密钥表

> 所属库：db_file
> 对应模型：`cn.testin.filecloud.model.UploadServiceAuth`
> 对应 Mapper：`UploadServiceAuthMapper.java` + `UploadServiceAuthMapper.xml`

## 表结构

| 字段 | 类型 | 允许空 | 默认值 | 说明 |
|---|---|---|---|---|
| `id` | int(11) | NOT NULL | AUTO_INCREMENT | 主键 ID，自增 |
| `service_id` | varchar(50) | NOT NULL | -- | 业务平台唯一标识（如 itestin、webportal 等） |
| `private_key` | varchar(100) | YES | NULL | 私钥 / API Key |
| `create_time` | datetime | YES | NULL | 创建时间 |
| `create_by` | int(11) | YES | NULL | 创建人用户 ID |
| `status` | smallint(1) | YES | NULL | 状态：0=正常, 1=失效 |

## 索引

| 索引名 | 字段 | 类型 |
|---|---|---|
| PRIMARY | `id` | BTREE（主键） |

## 业务说明

`upload_service_auth` 管理外部业务平台调用 文件管理服务 服务时的 API 鉴权凭证。每个接入的平台（服务）获得一对 service_id + private_key，用于 API 请求的身份认证。

**鉴权流程：**
1. 外部服务请求文件上传 -> 携带 service_id + private_key
2. 文件管理服务 服务通过 `get` 方法验证：`status=0 AND (service_id=? OR private_key=?)`
3. 只有 status=0（正常状态）的凭证才能通过认证

**安全设计：**
- `private_key` 应加密存储（业务代码层面处理）
- `status=1` 可快速吊销某个服务的访问权限，无需删除记录
- 插入时 create_time 默认使用 MySQL NOW()，status 默认 0

## Mapper 操作

| 方法名 | SQL 类型 | 用途 |
|---|---|---|
| `selectByPrimaryKey` | SELECT | 按 id 查询鉴权记录 |
| `get` | SELECT | 按 service_id / private_key 查询有效记录（status=0） |
| `insert` | INSERT | 插入鉴权密钥（create_time 用 now(), status 默认 0），返回自增 id |
| `update` | UPDATE | 选择性更新（service_id, private_key, status） |

## 关联关系

- **被引用情况：** 无直接外键引用，独立鉴权表

## 涉及接口

- [AuthController](../接口文档/auth/AuthController.md)（鉴权服务调用、密钥管理）
