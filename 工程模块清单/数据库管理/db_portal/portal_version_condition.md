# portal_version_condition (db_portal)

- 用途：门户版本条件，定义真机任务的应用版本筛选条件。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| appid | int(11) | App ID |
| app_name | varchar(100) | 应用名称 |
| package_name | varchar(128) | 包名 |
| projectid | int(11) | 项目 ID |
| condition_value | text | 条件值 |

- 关联接口：[portal-Task](../../平台基础功能服务/07-开放接口文档/任务管理/portal-Task.md)
