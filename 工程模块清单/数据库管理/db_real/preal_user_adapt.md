---
branch: syy.release.z7.8.1.0
module: real-test
type: SQL
database: MySQL (db_real)
table: preal_user_adapt
---

# preal_user_adapt

用户适配任务信息表（按 taskid 哈希分 100 张表 preal_user_adapt_00 ~ preal_user_adapt_99），记录每个测试任务的基本信息：企业、项目、用户、应用、执行状态、取消状态等。

## DDL (以 preal_user_adapt_00 为例)

```sql
CREATE TABLE `preal_user_adapt_00` (
  `vhost` int(8) DEFAULT NULL COMMENT '模块节点',
  `taskid` varchar(32) NOT NULL COMMENT '任务id',
  `eid` int(11) DEFAULT NULL COMMENT '企业id',
  `userid` int(11) NOT NULL COMMENT '用户信息',
  `projectid` int(11) NOT NULL COMMENT '项目组id',
  `test_type` int(2) NOT NULL COMMENT '测试类型',
  `appid` int(11) DEFAULT NULL COMMENT '应用id',
  `pkgid` int(11) DEFAULT NULL COMMENT '应用包id',
  `app_name` varchar(100) DEFAULT NULL COMMENT '应用名称',
  `package_name` varchar(128) DEFAULT NULL,
  `package_url` varchar(300) DEFAULT NULL,
  `app_version` varchar(64) DEFAULT NULL COMMENT '应用包版本',
  `exec_status` int(2) DEFAULT NULL COMMENT '执行状态',
  `cancelled` int(2) DEFAULT NULL COMMENT '取消状态：-1取消失败；0未取消；1取消中；2已取消',
  `check_app` int(2) DEFAULT NULL COMMENT '应用检测标识',
  `task_total` int(4) DEFAULT NULL COMMENT '任务总数',
  `biz_code` int(8) DEFAULT NULL COMMENT '业务编码',
  `descr` text COMMENT '任务描述',
  `content` text COMMENT 'JSON扩展字段(开始时间/完成时间/excel/initPwd等)',
  `charge` int(2) DEFAULT NULL COMMENT '付费标识：0免费；1收费',
  `status` int(2) DEFAULT NULL COMMENT '数据状态：0无效；1有效',
  `createtime` bigint(20) DEFAULT NULL COMMENT '创建时间',
  `updatetime` bigint(20) DEFAULT NULL COMMENT '更新时间',
  PRIMARY KEY (`taskid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 ROW_FORMAT=Compact;
```

## app处理服务 中的使用

### DAO/Service
- DAO: `PrealUserAdaptDAOImpl` -> `IPrealUserAdaptDAO`
- Service: `Task.add/cancel/detail/overview/userAdapt`, `Excel.reportExcel`, 几乎所有任务操作都涉及此表

### 关键SQL

| 操作 | SQL | 说明 |
| --- | --- | --- |
| 新增 | INSERT INTO preal_user_adapt_NN VALUES (...) | 创建任务时写入 |
| 查询 | SELECT * FROM preal_user_adapt_NN WHERE taskid=? | 获取用户适配信息 |
| 更新 | UPDATE preal_user_adapt_NN SET cancelled=?, exec_status=? WHERE taskid=? | 取消/状态变更 |
| content更新 | UPDATE preal_user_adapt_NN SET content=? WHERE taskid=? | Excel/initPwd等扩展字段更新 |

### 分表规则
taskId 哈希取模 100 决定分表后缀 (00~99)。
