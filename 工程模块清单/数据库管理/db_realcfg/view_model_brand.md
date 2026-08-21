---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# view_model_brand

- 数据库：`db_realcfg`
- 对象类型：视图

## 用途

机型-品牌联合视图：realcfg_model JOIN realcfg_brand。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE VIEW `view_model_brand` AS select `realcfg_model`.`brand_id` AS `brand_id`,`realcfg_model`.`modelid` AS `modelid`,`realcfg_model`.`name` AS `name`,`realcfg_model`.`alias_name` AS `alias_name`,`realcfg_model`.`type` AS `type`,`realcfg_model`.`os_name` AS `os_name`,`realcfg_model`.`dpi_width` AS `dpi_width`,`realcfg_model`.`dpi_height` AS `dpi_height`,`realcfg_model`.`screen_size` AS `screen_size`,`realcfg_model`.`pic_url` AS `pic_url`,`realcfg_model`.`pic_url_m` AS `pic_url_m`,`realcfg_model`.`pic_url_b` AS `pic_url_b`,`realcfg_model`.`cpu_freq` AS `cpu_freq`,`realcfg_model`.`cpu_num` AS `cpu_num`,`realcfg_model`.`cpu_model` AS `cpu_model`,`realcfg_model`.`cpu_brand` AS `cpu_brand`,`realcfg_model`.`cpu_processor` AS `cpu_processor`,`realcfg_model`.`gpu_vendor` AS `gpu_vendor`,`realcfg_model`.`gpu_renderer` AS `gpu_renderer`,`realcfg_model`.`gpu_version` AS `gpu_version`,`realcfg_model`.`ram` AS `ram`,`realcfg_model`.`rom` AS `rom`,`realcfg_model`.`nfc` AS `nfc`,`realcfg_model`.`bluetooth` AS `bluetooth`,`realcfg_model`.`bluetooth_version` AS `bluetooth_version`,`realcfg_model`.`on_shelve_time` AS `on_shelve_time`,`realcfg_model`.`off_shelve_time` AS `off_shelve_time`,`realcfg_model`.`fingermark` AS `fingermark`,`realcfg_model`.`otg` AS `otg`,`realcfg_model`.`status` AS `status`,`realcfg_model`.`createtime` AS `createtime`,`realcfg_model`.`updatetime` AS `updatetime`,`realcfg_brand`.`name` AS `brand_name`,`realcfg_brand`.`abbr` AS `brand_abbr`,`realcfg_brand`.`logo_url` AS `brand_logo_url`,`realcfg_brand`.`status` AS `brand_status`,`realcfg_brand`.`spelling` AS `brand_spelling`,`realcfg_model`.`logical_resolution` AS `logical_resolution`,`realcfg_model`.`weight` AS `weight` from (`realcfg_model` join `realcfg_brand` on((`realcfg_model`.`brand_id` = `realcfg_brand`.`id`)));
```

## 索引

- 视图，无索引

## 被哪些接口/mapper 方法使用

- `RealcfgModelDAOImpl`（pojo `RealcfgViewModelBrand`）← `ModelServiceImpl` ← 接口 [Model](../../平台配置（real-cfg）/07-开放接口文档/项目与平台配置/Model.md)
