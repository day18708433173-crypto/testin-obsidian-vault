---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# realcfg_biz_config

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

业务配置表：按 key 保存业务级开关/参数。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `realcfg_biz_config`  (
                                       `biz_code` int(11) NOT NULL COMMENT '业务编码',
                                       `name` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '业务名称',
                                       `test_type` int(11) NOT NULL COMMENT '测试类型，0=兼容测试，1=功能测试，2=安装测试，3=卸载测试',
                                       `config` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL COMMENT '详细配置',
                                       `descr` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '描述',
                                       `status` int(11) NOT NULL COMMENT '数据状态,off = 0, on = 1',
                                       `show_sequence` int(11) NOT NULL DEFAULT 0,
                                       `createtime` bigint(20) NOT NULL COMMENT '创建时间',
                                       `updatetime` bigint(20) NOT NULL COMMENT '更新时间',
                                       PRIMARY KEY (`biz_code`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

## 索引

- `PRIMARY KEY (`biz_code`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `RealcfgBizConfigDAOImpl`（JDBC）← `BizConfigServiceImpl` ← 接口 [BizConfig](../../平台配置（real-cfg）/07-开放接口文档/业务规则与界面/BizConfig.md)
