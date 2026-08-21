# script_group

## 基本信息

| 属性 | 值 |
|------|-----|
| 数据库 | db_file |
| 业务域 | 脚本组 |
| 关联 Mapper | ScriptGroupMapper |

## 字段定义

| 字段名 | 类型 | 键 | 说明 |
|--------|------|-----|------|
| group_id | — | PK | 脚本组ID |
| group_name | — | | 脚本组名称 |
| appid | — | | 应用ID |
| projectid | — | | 项目ID |
| group_scriptid_array | — | | 组内脚本ID数组 |
| group_scriptno_array | — | | 组内脚本编号数组 |
| group_desc | — | | 脚本组描述 |
| userid | — | | 用户ID |
| creator | — | | 创建人 |
| status | Byte | | 状态 |
| createtime | Long | | 创建时间 |
| updatetime | Long | | 更新时间 |
| os_type | — | | 系统类型 |
| content | CLOB | | 内容（二进制） |
| script_type | — | | 脚本类型 |

## 关联关系

- 被引用：[suite_group_script](suite_group_script.md)（groupId FK）

## 相关接口

- `ScriptGroupController` / `ScriptGroup`（ApiServlet）：脚本组功能接口已下线，接口文档已移除（2026-08-12）
