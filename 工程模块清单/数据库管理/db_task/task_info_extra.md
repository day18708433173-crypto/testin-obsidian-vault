---
branch: syy.release.z7.8.1.0
module: real-scheduling
type: SQL表
database: db_task
---

# task_info_extra

任务扩展信息表。存储任务的全局参数、依赖脚本、账号信息、串行/同步运行数、环境配置等扩展数据。

## DDL

```sql
CREATE TABLE `task_info_extra` (
    `taskid` varchar(32) NOT NULL,
    `vhost` int(11) NOT NULL,
    `params` text NULL COMMENT '全局参数',
    `depend_scripts` mediumtext NULL COMMENT '依赖脚本列表',
    `serial_run` int(11) NOT NULL COMMENT '串行执行',
    `sync_run_num` int(11) NOT NULL COMMENT '同步运行数量',
    `accounts` text NULL COMMENT '账号信息',
    `createtime` bigint(20) NOT NULL,
    `updatetime` bigint(20) NOT NULL,
    `expiretime` bigint(20) NOT NULL,
    `env_config` text NULL COMMENT '环境配置参数',
    `script_param_data` text COMMENT '东北证券定制化脚本参数数据',
    `device_offline_config` int(11) DEFAULT NULL COMMENT '设备离线配置 1等待上线 2跳过',
    `app_desc` varchar(200) NULL COMMENT 'app版本备注',
    `recover_script_infos` text NULL COMMENT '脚本恢复信息（代码中动态扩展）',
    PRIMARY KEY (`taskid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

## 字段说明

| 字段 | 说明 |
|------|------|
| taskid | 主键，任务 ID |
| params | 全局参数（JSON） |
| depend_scripts | 依赖脚本（前后置脚本） |
| serial_run | 是否串行执行 |
| sync_run_num | 同步并发运行数 |
| accounts | 账号列表 |
| env_config | 环境配置参数 |
| recover_script_infos | 脚本恢复映射（HashMap<Integer, Object>），补测时合并 |
| device_offline_config | 设备离线处理策略：1=等待上线，2=跳过相关设备 |

## 任务调度服务 中的使用

- **ITaskInfoExtraDAO**（`cn.testin.dao.interfaces.task.ITaskInfoExtraDAO`）：
  - `get(taskid)`: 查询扩展信息
  - `insert(extra)`: 任务初始化时写入
  - `update(extra)`: 更新（如补测合并 recoverScriptInfos）
  - `delete(taskid)`: 按任务 ID 删除
- **核心流程**：
  - `Task.init` -> get 检查是否存在 -> 不存在则 INSERT / 存在且非补测则 DELETE+INSERT / 补测则合并 recoverScriptInfos
- **POJO**：`cn.testin.pojo.task.DbTaskInfoExtra`
