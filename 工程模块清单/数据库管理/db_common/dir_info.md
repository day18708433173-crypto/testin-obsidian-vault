# dir_info (db_common)

- 用途：目录信息表（心跳检测模块），记录各节点工作目录。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| dir_path | varchar(255) | 目录路径 |
| dir_type | int(11) | 目录类型 |
| node_id | int(11) | 节点 ID |
| status | int(11) | 状态 |

- 关联接口：[HeartBeatController](../../平台基础功能服务/07-开放接口文档/基础设施与统计/HeartBeatController.md)、[DirQuartzJobController](../../平台基础功能服务/07-开放接口文档/基础设施与统计/DirQuartzJobController.md)
