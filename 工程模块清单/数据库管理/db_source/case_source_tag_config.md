# case_source_tag_config (db_source)

- 用途：用例数据源行标签关联，记录某实例表某一行绑定的标签。
- 实体类：`cn.testin.pojo.testCase.CaseSourceTagConfig`
- 主要字段：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | int(11) | 主键（自增） |
| case_source_id | bigint(20) | 用例实例表 ID（关联 [case_source](case_source.md)） |
| row_index | int(11) | 行索引 |
| tag_id | int(11) | 标签 ID（关联 [tag_info](tag_info.md)） |
| create_time | datetime | 创建时间 |
| create_user_id | int(11) | 创建人 ID |

- 与 [datatable_tag_config](datatable_tag_config.md)（legacy 表族）职责相同；标签用于数据行的标记与筛选，支持批量操作。

- 关联接口：
  - [CaseSourceController](../../数据源/07-开放接口文档/数据源管理/CaseSourceController.md)
  - [TagInfoController](../../数据源/07-开放接口文档/数据源管理/TagInfoController.md)
