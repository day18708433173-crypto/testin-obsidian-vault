# server_disk_usage_records (db_common)

- 用途：服务器磁盘使用记录，定期采集并存储磁盘使用量数据。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| mount_id | int(11) | 挂载点 ID |
| used_size | bigint(20) | 已用量 |
| free_size | bigint(20) | 可用量 |
| usage_percent | double | 使用率 |
| record_time | datetime | 记录时间 |
| create_time | datetime | 创建时间 |

- 关联接口：[DiskMonitorController](../../平台基础功能服务/07-开放接口文档/基础设施与统计/DiskMonitorController.md)
