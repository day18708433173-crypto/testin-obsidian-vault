---
branch: syy.release.z7.8.1.0
module: real-test
type: SQL
database: MySQL (db_real)
table: real_report_share
---

# real_report_share

报告分享表，存储测试报告分享的密钥（skey）和配置信息，支持报告分享、测试计划分享、Web PC 报告分享等类型。

## DDL

```sql
CREATE TABLE `real_report_share` (
  `skey` varchar(64) NOT NULL COMMENT '分享短地址(唯一密钥)',
  `taskid` varchar(128) NOT NULL COMMENT '任务id',
  `report_key` varchar(128) DEFAULT NULL COMMENT '查看报告key',
  `content` text COMMENT 'JSON扩展：{eid,projectid,userid,reportKeyExpiretime}',
  `status` int(11) NOT NULL COMMENT '状态：0=off, 1=on',
  `createtime` bigint(20) NOT NULL COMMENT '创建时间',
  `updatetime` bigint(20) NOT NULL COMMENT '更新时间',
  PRIMARY KEY (`skey`),
  UNIQUE KEY `taskid`(`taskid`),
  UNIQUE KEY `uk_report_key`(`report_key`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 ROW_FORMAT=Compact;
```

## app处理服务 中的使用

### DAO/Service
- DAO: `RealReportShareDAOImpl` -> `IRealReportShareDAO`
- Service: `Task.share/shareInfo`, `TestPlanController.shareExcel`

### 关键SQL

| 操作 | SQL | 说明 |
| --- | --- | --- |
| 新增 | INSERT INTO real_report_share(skey, taskid, content, status, createtime, updatetime) VALUES (...) | 首次分享 |
| 查询(skey) | SELECT * FROM real_report_share WHERE skey=? | 通过密钥获取 |
| 查询(taskid) | SELECT * FROM real_report_share WHERE taskid=? | 通过任务ID获取 |
| 更新状态 | UPDATE real_report_share SET status=? WHERE taskid=? | 启用/关闭分享 |
