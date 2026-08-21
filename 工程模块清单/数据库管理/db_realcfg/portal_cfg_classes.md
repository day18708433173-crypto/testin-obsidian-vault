---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# portal_cfg_classes

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

门户菜单表：门户导航菜单树（title/pid/url/order_index/css）。

## DDL

> pocinit DDL 来源未命中，以下字段根据 pojo + DAO SQL **推断**。

| 字段 | 类型（推断） | 说明 |
|---|---|---|
| id | bigint | 主键（序列 seq_portal_cfg_classes） |
| title | varchar | 菜单标题 |
| pid | bigint | 父菜单 ID |
| url | varchar | 菜单链接 |
| order_index | bigint | 排序序号 |
| css | varchar | 样式 |
| updatetime | bigint | 更新时间（DAO update 时写入） |

## 索引

- 推断：主键 id；常用查询列为 pid、order_index

## 被哪些接口/mapper 方法使用

- `PortalCfgClassesDAOImpl`（JDBC）← `PortalCfgClassesServiceImpl` ← 接口 [PortalCfg](../../平台配置（real-cfg）/07-开放接口文档/业务规则与界面/PortalCfg.md).getMenu
