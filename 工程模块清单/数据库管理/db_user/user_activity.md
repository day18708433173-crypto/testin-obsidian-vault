# user_activity (db_user)

- 用途：用户活动记录，用于统计用户活跃度。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| user_id | int(11) | 用户 ID |
| user_name | varchar(255) | 用户名 |
| eid | int(11) | 企业 ID |
| activity_date | date | 活跃日期 |
| login_count | int(11) | 登录次数 |
| create_time | datetime | 创建时间 |

- 关联接口：[UserActivityController](../../平台基础功能服务/07-开放接口文档/用户与认证/UserActivityController.md)
