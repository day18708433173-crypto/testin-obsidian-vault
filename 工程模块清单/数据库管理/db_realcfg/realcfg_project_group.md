---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# realcfg_project_group

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

项目组-设备组授权表：项目组可用设备组及过期时间。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `realcfg_project_group`  (
                                          `eid` int(11) NOT NULL,
                                          `projectid` int(11) NOT NULL COMMENT '项目组id',
                                          `type` int(11) NOT NULL DEFAULT 1,
                                          `devicegroupid` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '云',
                                          `status` int(255) NOT NULL COMMENT '状态off = 0, on = 1',
                                          `createtime` bigint(20) NOT NULL COMMENT '创建时间',
                                          `updatetime` bigint(20) NOT NULL COMMENT '更新时间',
                                          `expiretime` bigint(20) NOT NULL,
                                          PRIMARY KEY (`eid`, `projectid`, `type`) USING BTREE,
                                          UNIQUE INDEX `devicegroupid`(`devicegroupid`) USING BTREE,
                                          INDEX `projectid`(`projectid`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

## 索引

- `PRIMARY KEY (`eid`, `projectid`, `type`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `RealcfgProjectGroupDAOImpl`（JDBC）← `ProjectGroupServiceImpl` ← 接口 [ProjectGroup](../../平台配置（real-cfg）/07-开放接口文档/项目与平台配置/ProjectGroup.md)
- `RealCfgProjectGroupMapper`（MyBatis）：selectByEidAndProjectId ← `UcomIdService`
- 被视图 [view_project_group_source](view_project_group_source.md)、[view_filterproper_source](view_filterproper_source.md) 引用
