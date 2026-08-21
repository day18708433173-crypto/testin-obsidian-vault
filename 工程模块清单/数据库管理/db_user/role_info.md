---
module:
  - testin-core
  - real-cfg
type: SQL表
---

# role_info (db_user)

- 数据库：`db_user`
- 对象类型：表
- 用途：角色定义，存储系统角色名称、类型、所属企业等。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `role_info`  (
                              `id` int(11) NOT NULL AUTO_INCREMENT,
                              `name` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '角色名称',
                              `descr` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL,
                              `status` tinyint(11) NOT NULL DEFAULT 1 COMMENT '删除标识，0为已删除，1为正常',
                              `enterprise_id` int(11) NULL DEFAULT NULL COMMENT '企业ID',
                              PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

## 索引

- `PRIMARY KEY (`id`) USING BTREE`

## 平台基础功能服务（testin-core）视角

> 注意：该服务文档原记的字段表（`role_name`/`eid`/`role_type`/`createtime`/`updatetime`）经 2026-08-12 对实库验证不存在，真实字段以上方 DDL 为准，此处仅保留其接口关联信息。

- 关联接口：[service-Role](../../平台基础功能服务/07-开放接口文档/用户与认证/service-Role.md)

## 平台配置（real-cfg）视角

用户角色表（db_user 库）：平台配置 仅跨库只读 JOIN，不做写操作。

被哪些接口/mapper 方法使用：

- `RoleFunctionDAOImpl` / `ViewRoleFunctionDAOImpl` 中 `db_user.role_info` LEFT JOIN（接口 [ServiceCfg](../../平台配置（real-cfg）/07-开放接口文档/业务规则与界面/ServiceCfg.md) 系列）

## 字段定义核实结论（2026-08-12 对实库验证）

已对实库执行 `SHOW CREATE TABLE db_user.role_info` 验证：**以平台配置（real-cfg）所附 DDL 为准**（`name` varchar(32)、`descr` text、`status` tinyint(11) 删除标识、`enterprise_id` int(11)）。平台基础功能服务文档原记的 `role_name`、`eid`、`role_type`、`createtime`、`updatetime` 等字段在实库中均不存在，属文档记录有误。
