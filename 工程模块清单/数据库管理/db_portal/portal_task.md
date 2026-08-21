# portal_task (db_portal)

- 用途：真机门户任务表（按 eid 分表 portal_task_00 ~ portal_task_99），存储真机测试任务的详细信息。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| eid | int(11) | 企业 ID |
| projectid | int(11) | 项目 ID |
| userid | int(11) | 用户 ID |
| taskid | varchar(64) | 任务 ID |
| suite_id | int(11) | 应用 ID |
| appid | int(11) | App ID |
| pkgid | int(11) | 包 ID |
| channel_id | varchar(300) | 应用渠道号 |
| app_name | varchar(100) | 应用名称 |
| package_url | varchar(300) | 包下载地址 |
| package_name | varchar(128) | 应用包名 |
| app_version | varchar(64) | 应用版本 |

- 关联接口：[portal-Task](../../平台基础功能服务/07-开放接口文档/任务管理/portal-Task.md)
