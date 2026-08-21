# script_action_log

## 基本信息

| 属性 | 值 |
|------|-----|
| 数据库 | db_file |
| 业务域 | 其他 |
| 关联 Mapper | 本仓库无直接 Mapper（见下方"写入链路"） |

## 字段定义

| 字段名 | 类型 | 键 | 说明 |
|--------|------|-----|------|
| id | — | PK | 主键ID |
| eid | — | | 企业ID |
| project_id | — | | 项目ID |
| module_id | — | | 模块ID |
| status | Byte | | 状态 |
| fail_count | — | | 失败数量 |
| success_count | — | | 成功数量 |
| total_count | — | | 总数量 |
| fail_cause | — | | 失败原因 |
| ext | — | | 扩展信息 |
| action_type | — | | 操作类型 |
| request_id | — | | 请求ID |
| file_url | — | | 文件URL/结果 |
| create_user_id | — | | 创建人ID |
| create_user_name | — | | 创建人用户名 |
| desc | — | | 描述 |
| create_time | Long | | 创建时间 |
| update_time | Long | | 更新时间 |

## 关联关系

- 独立表，通过 eid、project_id 关联项目维度

## 写入链路（重要）

脚本服务（本仓库）**不直接读写** 该表。写入链路为：

1. `ActionLogService`（cn.testin.file.service）在脚本导出/复制时构建 `ActionLog` 对象（模型：cn.testin.file.model.ActionLog）
2. 序列化为 JSON 后 `LPUSH` 到 Redis 队列 **`queue:action_log`**（`RedisConstants.REDIS_QUEUE_KEY_ACTION`）
3. 由 平台基础功能服务 侧消费者异步消费并落库到本表

对应模型字段与本表字段一一对应（result 字段对应导出文件下载 URL）。

## 相关接口

- [ScriptController](../../脚本服务/07-开放接口文档/脚本管理/ScriptController.md)（脚本复制/导出触发日志上报）
- [ExportScript](../../脚本服务/07-开放接口文档/脚本管理/ExportScript.md)
