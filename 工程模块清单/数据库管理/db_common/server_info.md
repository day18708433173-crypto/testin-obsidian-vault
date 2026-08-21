# server_info (db_common)

- 用途：服务器节点信息，记录各执行节点的 IP、状态等。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键，节点 ID |
| node_ip | varchar(255) | 节点 IP |
| node_name | varchar(255) | 节点名称 |
| status | int(11) | 状态（0=离线 1=在线） |
| create_time | datetime | 创建时间 |
| update_time | datetime | 更新时间 |

- 关联接口：[DiskMonitorController](../../平台基础功能服务/07-开放接口文档/基础设施与统计/DiskMonitorController.md)
