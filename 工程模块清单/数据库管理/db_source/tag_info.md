# tag_info (db_source)

- 用途：标签定义。用于标记数据行，支持检索和筛选。
- 主要字段：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | int(11) | 主键 |
| eid | int(11) | 企业 ID |
| project_id | int(11) | 项目组 ID |
| name | varchar(255) | 标签名称 |
| status | varchar(255) | 状态 |
| createtime | bigint(20) | 创建时间 |
| updatetime | bigint(20) | 更新时间 |

- 索引：`eid_projectid`、`updatetime`

- 关联接口：
  - [TagInfoController](../../数据源/07-开放接口文档/数据源管理/TagInfoController.md)
  - [service-TagInfoCtrl](../../数据源/07-开放接口文档/其他ApiServlet/service-TagInfoCtrl.md)
