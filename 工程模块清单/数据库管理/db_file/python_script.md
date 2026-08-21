# python_script

## 基本信息

| 属性 | 值 |
|------|-----|
| 数据库 | db_file |
| 业务域 | 其他 |
| 关联 Mapper | PythonScriptMapper |

## 字段定义

| 字段名 | 类型 | 键 | 说明 |
|--------|------|-----|------|
| id | — | PK | 主键ID |
| eid | — | | 企业ID |
| project_id | — | | 项目ID |
| name | — | | 脚本名称 |
| descr | — | | 脚本描述 |
| exec_location | — | | 执行位置 |
| create_user_id | — | | 创建人ID |
| create_user_name | — | | 创建人用户名 |
| update_user_id | — | | 更新人ID |
| update_user_name | — | | 更新人用户名 |
| create_time | Long | | 创建时间 |
| update_time | Long | | 更新时间 |
| is_delete | — | | 是否删除 |
| script_content | — | | 脚本内容 |

## 关联关系

- 独立表，通过 eid、project_id 关联项目维度

## 相关接口

- [PythonScriptController](PythonScriptController.md)
