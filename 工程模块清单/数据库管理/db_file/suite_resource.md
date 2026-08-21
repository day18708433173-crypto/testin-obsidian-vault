# suite_resource

## 基本信息

| 属性 | 值 |
|------|-----|
| 数据库 | db_file |
| 业务域 | 套件管理 / AI 资源标注 |
| 关联 Mapper | SuiteAiResourcesMapper |

## 字段定义

| 字段名 | 类型 | 键 | 说明 |
|--------|------|-----|------|
| suite_id | Integer | PK | 套件ID（联合主键） |
| resource_id | Integer | PK | AI资源ID（联合主键） |
| status | Integer | | 状态（1=有效，0=删除） |
| createtime | Long | | 创建时间 |
| updatetime | Long | | 更新时间 |

## 关联关系

- 引用：[suite_info](suite_info.md)（suite_id FK）
- 引用：[ai_resource](ai_resource.md)（resource_id FK）
- 是套件与 AI 标注资源的多对多关联表，[ai_resource](ai_resource.md) 的套件维度查询均通过本表 INNER JOIN 实现

## 相关接口

- [ResourcesMarkController](../../脚本服务/07-开放接口文档/AI资源标注/ResourcesMarkController.md)
- [SuiteController](../../脚本服务/07-开放接口文档/套件管理/SuiteController.md)

## 备注

- 删除为逻辑删除（`status = 0`），无物理删除语句。
