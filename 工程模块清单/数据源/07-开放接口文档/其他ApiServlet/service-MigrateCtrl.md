# service-MigrateCtrl — 数据迁移工具（ApiServlet）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/service/source/MigrateCtrl.java`
> 类级路由：`/source/MigrateCtrl`（完整前缀 `/openapi/source/MigrateCtrl`）
> 基类：`GenericBaseService`
> 业务：历史数据迁移辅助功能 — 实例表填充全局信息、清理空行。

## 方法列表总表

| # | 路径 | 方法名 | 说明 |
|---|------|--------|------|
| 1 | `/fillGlobalInfo` | fillGlobalInfo | 实例表添加全局变量/全局变量值 |
| 2 | `/deleteTableNullRow` | deleteTableNullRow | 删除表里的空行（迁移后清理） |

涉及表：`datatable_values`。

---

## 方法详解

### 1. POST /fillGlobalInfo — 填充全局信息

**入参**：`DatatableDTO`

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| （DatatableDTO 字段） | — | 否 | 表格操作参数（无 validateParams 校验，全部非必填）：sourceConfigId、globalId、eid、projectid、datatableValuesList、colConfigList、tagConfigList、rowList、colList、deleteIdList、updateBy、sync、iosTag、androidTag 等 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | Boolean | 是否填充成功 |

**Service**：`DatatableValuesService.fillGlobalInfo(dto)`

将全局表的数据填充到关联的实例表中。历史数据迁移时，实例表可能缺少全局变量引用，此接口批量补充。

### 2. POST /deleteTableNullRow — 删除空行

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| — | — | — | 无业务参数 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | Boolean | 是否删除成功 |

**Service**：`DatatableValuesService.deleteTableNullRow()`（无参，全量扫描）

迁移数据后，部分表的数据值全部为空（整行无有效数据），此接口清理这些空行。

## 使用场景

这两个接口属于运维/迁移工具性质，通常在版本升级后由管理员调用执行数据修复。
