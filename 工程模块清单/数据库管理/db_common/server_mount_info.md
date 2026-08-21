# server_mount_info (db_common)

- 用途：服务器挂载点信息，记录每个节点的磁盘挂载路径。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| node_id | int(11) | 节点 ID |
| mount_path | varchar(255) | 挂载路径 |
| total_size | bigint(20) | 总容量 |
| create_time | datetime | 创建时间 |

- 关联接口：[DiskMonitorController](../../平台基础功能服务/07-开放接口文档/基础设施与统计/DiskMonitorController.md)
