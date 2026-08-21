---
branch: syy.release.z7.8.1.0
module: real-scheduling
type: SQL表
database: db_task
---

# task_info

子任务表（对应设备）。每个子任务代表某个任务（taskid）分配到某台终端设备（deviceid）上的调度单元。

## DDL

```sql
CREATE TABLE `task_info` (
    `vhost` int(11) NOT NULL,
    `req_id` varchar(32) NOT NULL COMMENT '请求id，防止重复处理',
    `subtaskid` varchar(32) NOT NULL COMMENT '子任务（任务针对某台终端）',
    `taskid` varchar(32) NOT NULL COMMENT '任务（用户提测的任务）',
    `user_id` int(11) NOT NULL COMMENT '用户id',
    `test_type` int(2) NULL DEFAULT 0 COMMENT 'nativeApp测试类型: 1-功能测试、2-安装测试、3-卸载测试',
    `appid` int(11) NULL DEFAULT 0,
    `app_url` varchar(300) NULL DEFAULT NULL COMMENT '应用包下载地址',
    `syspf_id` int(11) NULL DEFAULT NULL,
    `package_name` varchar(128) NULL DEFAULT NULL COMMENT '应用包名',
    `start_path` text NULL COMMENT '启动路径',
    `app_md5` varchar(64) NULL DEFAULT NULL COMMENT '应用包MD5值',
    `deviceid` varchar(128) NOT NULL DEFAULT '' COMMENT '设备ID',
    `account_num` int(11) NULL DEFAULT NULL COMMENT '账号数量',
    `level` int(4) NOT NULL DEFAULT 0 COMMENT '测试任务优先级别',
    `exec_num` int(4) NOT NULL DEFAULT 0 COMMENT '任务执行次数',
    `exec_status` int(1) NOT NULL DEFAULT 0 COMMENT '任务执行状态: -2待调度 -1临时 0待执行 1执行中 2预完成 3完成 4取消',
    `error_code` int(11) NULL DEFAULT NULL,
    `task_type` int(1) NULL DEFAULT 1 COMMENT '任务类型【1：正常；2：补测】',
    `account_id` varchar(64) NULL DEFAULT NULL COMMENT '测试账号',
    `account_pwd` varchar(64) NULL DEFAULT NULL COMMENT '测试密码',
    `account_extension` varchar(1024) NULL DEFAULT NULL COMMENT '账号扩展信息',
    `certificate` text NULL COMMENT '证书信息',
    `standard` varchar(2048) NULL DEFAULT NULL COMMENT '测试指标',
    `depend_scripts` mediumtext NULL COMMENT '依赖脚本列表',
    `status` int(4) NOT NULL DEFAULT 1 COMMENT '数据有效性',
    `createtime` bigint(20) NOT NULL COMMENT '创建时间',
    `updatetime` bigint(20) NOT NULL COMMENT '修改时间',
    `publishtime` bigint(20) NOT NULL COMMENT '任务匹配生效时间',
    `matchtime` bigint(20) NULL DEFAULT NULL COMMENT '子任务匹配下发时间',
    `expiretime` bigint(20) NOT NULL COMMENT '任务过期时间',
    `exec_standard` varchar(32) NULL DEFAULT NULL COMMENT '执行策略 fast/normal/simple/monkey/script/data',
    `allow_deviceids` text NULL COMMENT '可允许执行的设备id列表',
    `unique_key` varchar(512) NULL DEFAULT NULL COMMENT '任务全局唯一标识',
    `capture_rules` text NULL COMMENT '网络报文抓包规则',
    `round` smallint(6) NULL DEFAULT NULL COMMENT '任务执行轮数',
    `round_period` bigint(20) NULL DEFAULT NULL,
    `round_max` int(11) NULL DEFAULT 1,
    `network_simulation` smallint(1) NULL DEFAULT NULL COMMENT '模拟网络 0否1是',
    `network_config` varchar(512) NULL DEFAULT NULL COMMENT '模拟网络参数配置',
    `param` text NULL COMMENT '全局参数',
    `eid` int(11) NOT NULL DEFAULT 0,
    `projectid` int(11) NULL DEFAULT NULL,
    `browser_type` varchar(128) NULL DEFAULT NULL COMMENT '浏览器类型(WEB)',
    `browser_version` varchar(128) NULL DEFAULT NULL COMMENT '浏览器版本(WEB)',
    `ucomid` varchar(128) NULL DEFAULT NULL COMMENT '上位机id',
    `biz_code` int(11) NULL DEFAULT NULL COMMENT '业务编码',
    `source` varchar(128) NULL DEFAULT NULL COMMENT '设备/浏览器云信息',
    `os_name` varchar(128) NULL DEFAULT NULL COMMENT '系统类型(WEB)',
    `suite_id` int(11) NULL DEFAULT 0 COMMENT '跨端应用ID',
    `sid` varchar(255) NULL DEFAULT NULL COMMENT '浏览器执行sid',
    `cross_taskid` varchar(32) NULL DEFAULT NULL COMMENT '多端任务id',
    `resource_type` varchar(32) NULL DEFAULT NULL COMMENT 'app/web/pc',
    `task_descr` varchar(255) NULL DEFAULT NULL COMMENT '任务描述',
    `sync_match_queue` int(11) NOT NULL DEFAULT 0 COMMENT '同步匹配队列:0未同步1已同步',
    `match_type` int(11) NOT NULL DEFAULT 1 COMMENT '匹配方式',
    `match_resource` text NULL COMMENT '任务匹配设备信息',
    `data_row` int(11) NULL DEFAULT NULL COMMENT '数据驱动使用的数据行',
    PRIMARY KEY (`subtaskid`),
    INDEX `req_id`(`req_id`),
    INDEX `exec_status`(`exec_status`),
    INDEX `expiretime`(`expiretime`),
    INDEX `exec_num`(`exec_num`),
    INDEX `taskid`(`taskid`),
    INDEX `deviceid`(`deviceid`),
    INDEX `exec_standard`(`exec_standard`),
    INDEX `browser_type`(`browser_type`),
    INDEX `browser_version`(`browser_version`),
    INDEX `ucomid`(`ucomid`),
    INDEX `source`(`source`),
    INDEX `os_name`(`os_name`),
    INDEX `syncMatchQueue_execStatus`(`sync_match_queue`, `exec_status`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='子任务表，对应设备';
```

## 字段说明

| 字段 | 说明 |
|------|------|
| subtaskid | 主键，全网唯一子任务 ID |
| taskid | 关联的用户提测任务 ID |
| deviceid | 指定的目标设备 ID |
| exec_status | -2待调度 / -1临时 / 0待执行 / 1执行中 / 2预完成 / 3完成 / 4取消 |
| exec_standard | 执行策略：fast(快速)/normal(普通)/simple(兼容)/script(兼容脚本)/data(数据驱动) |
| level | 优先级，数值越大越优先 |
| match_type | 匹配方式：1=按设备ID匹配 |
| resource_type | app / web / pc |

## 任务调度服务 中的使用

- **ITaskInfoDAO**（`cn.testin.dao.interfaces.task.ITaskInfoDAO`）：所有增删改查
  - `batchInsert`: 初始化时批量写入子任务
  - `get(subtaskid)`: 按子任务 ID 查询
  - `list(conditionMap, sortFields, offset, limit)`: 条件分页查询
  - `listByPending(vhost, max)`: 查询待处理任务（取消/过期/超次）
  - `update(taskInfo)`: 更新执行状态/匹配时间等
  - `deleteByTaskId(taskid)`: 按任务 ID 删除
  - `deleteByReqId(reqId)`: 按请求 ID 删除（初始化失败回滚）
- **核心流程**：
  - `Task.init` -> batchInsert（入库临时状态-1，然后更新为0待执行）
  - `Task.match` -> 查询待执行子任务，匹配后更新 deviceid/exec_status
  - `Task.recover` -> 执行中子任务重置为待执行
  - `Task.cancel` -> 未开始子任务设为取消状态(4)
  - `Task.list/levelTaskList` -> 条件分页查询
- **Mapper XML**：`task_info` 的 CRUD 通过 DAO JDBC 实现（非 MyBatis XML）
- **POJO**：`cn.testin.pojo.task.DbTaskInfo`
