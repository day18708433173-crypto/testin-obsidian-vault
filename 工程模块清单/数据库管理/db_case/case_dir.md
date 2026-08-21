# case_dir (db_case)

- 用途：用例目录树，支持多级目录结构。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| eid | int(10) | 企业 ID |
| project_id | int(10) | 项目 ID |
| case_dir_name | varchar(255) | 目录名称 |
| parent_id | int(11) | 父级目录 ID |
| case_dir_order | int(10) | 目录排序 |
| status | int(1) | 状态（0=无效 1=有效） |

- 关联接口：[CaseDirController](../../平台基础功能服务/07-开放接口文档/测试用例/CaseDirController.md)
