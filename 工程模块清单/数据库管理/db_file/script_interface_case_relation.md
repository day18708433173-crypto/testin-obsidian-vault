# script_interface_case_relation

## 基本信息

| 属性 | 值 |
|------|-----|
| 数据库 | db_file |
| 业务域 | 用例转换 |
| 关联 Mapper | ScriptCaseRelationMapper |

## 字段定义

| 字段名 | 类型 | 键 | 说明 |
|--------|------|-----|------|
| id | Integer | PK | 主键ID |
| script_no | Integer | | 脚本编号（script_file.scriptno） |
| case_id | Integer | | 接口用例ID（外部 InterfaceCase 服务） |
| status | Integer | | 状态（1=有效，0=解除关联） |
| create_time | Date | | 创建时间 |
| update_time | Date | | 更新时间 |

## 关联关系

- 引用：[script_file](script_file.md)（script_no → scriptno）
- case_id 指向 [InterfaceCase](../../外部服务/InterfaceCase.md) 服务的接口用例，非本库外键

## 相关接口

- [TestCaseController](../../脚本服务/07-开放接口文档/用例转换/TestCaseController.md)

## 备注

- 解除关联为逻辑删除（`status = 0`），支持按 script_no 批量解除。
- 注意时间字段为 `Date`（TIMESTAMP），与库内多数表的 `Long` 毫秒时间戳不同。
