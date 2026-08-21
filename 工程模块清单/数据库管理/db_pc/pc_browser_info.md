---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# pc_browser_info

## 用途

PC浏览器信息表，记录上位机安装的浏览器类型和版本。关联到 view_pc_info 视图。

## 所属数据库

db_pc

## DDL

```sql
CREATE TABLE `pc_browser_info`  (
                                    `ucomid` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                    `type` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                    `version` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                    `createtime` bigint(20) NOT NULL,
                                    PRIMARY KEY (`ucomid`, `type`, `version`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

> DDL 来源：pocinit/src/mysql/db_pc.sql（命中）

## 设备控制中心 中的使用

PC浏览器信息表，记录上位机安装的浏览器类型和版本。关联到 view_pc_info 视图。
