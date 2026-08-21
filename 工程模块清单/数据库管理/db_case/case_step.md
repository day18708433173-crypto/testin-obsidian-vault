# case_step (db_case)

- 用途：用例步骤，每个步骤可关联一个脚本，定义执行顺序和运行端。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| case_id | int(11) | 用例 ID |
| script_no | int(10) | 脚本编号 |
| step_desc | varchar(256) | 步骤描述 |
| script_type | int(1) | 脚本类型 |
| step_expect | varchar(256) | 预期结果 |
| step_order | int(10) | 步骤顺序 |
| parallel_flag | int(11) | 并行执行标记 |
| run_as_os_type | smallint(2) | 运行端类型 |
| from_group | varchar(32) | 归属组别 |
| script_status | int(1) | 脚本状态 |

- 关联接口：[CaseStepController](../../平台基础功能服务/07-开放接口文档/测试用例/CaseStepController.md)
