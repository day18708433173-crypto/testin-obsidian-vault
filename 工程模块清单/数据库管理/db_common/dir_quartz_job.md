# dir_quartz_job (db_common)

- 用途：目录与定时任务的关联表，管理各节点的定时清理任务。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| dir_id | int(11) | 目录 ID |
| job_id | int(11) | 定时任务 ID |

- 关联接口：[DirQuartzJobController](../../平台基础功能服务/07-开放接口文档/基础设施与统计/DirQuartzJobController.md)
