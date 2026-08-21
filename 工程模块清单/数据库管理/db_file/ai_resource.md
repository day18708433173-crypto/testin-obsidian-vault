# ai_resource

## 基本信息

| 属性 | 值 |
|------|-----|
| 数据库 | db_file |
| 业务域 | AI 资源标注 |
| 关联 Mapper | AiResourcesMapper |

## 字段定义

| 字段名 | 类型 | 键 | 说明 |
|--------|------|-----|------|
| id | Integer | PK | 主键ID（自增） |
| eid | Integer | | 企业ID |
| projectid | Integer | | 项目ID |
| uid | Integer | | 用户ID |
| package_name | String | | 应用包名 |
| app_name | String | | 应用名称 |
| app_version | String | | 应用版本 |
| icon_url | String | | 图标URL |
| big_url | String | | 大图（标注原图）URL |
| bounds | String | | 标注区域坐标 |
| small_url | String | | 小图（裁剪图）URL |
| rotation | String | | 旋转角度 |
| resolution | String | | 分辨率 |
| mark_name | String | | 标注名称 |
| status | Integer | | 状态（1=有效，0=删除） |
| createtime | Long | | 创建时间 |
| updatetime | Long | | 更新时间 |
| source_type | Integer | | 来源类型 |

## 关联关系

- 通过 [suite_resource](suite_resource.md)（resource_id → ai_resource.id）与套件关联，查询时 `INNER JOIN suite_resource` 按套件过滤
- 通过 eid、projectid 关联项目维度

## 相关接口

- [ResourcesMarkController](../../脚本服务/07-开放接口文档/AI资源标注/ResourcesMarkController.md)
- [Resources](../../脚本服务/07-开放接口文档/AI资源标注/Resources.md)

## 备注

- 所有列表查询强制 `status = 1`（逻辑删除）。
- `mark_name` 支持精确与模糊（LIKE %x%）两种匹配，由 `fuzzyByMarkName` 参数控制。
- 列表按 `updatetime DESC` 排序，支持 `page/pageSize` 分页。
