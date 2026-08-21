# service-ColConfigCtrl — 列属性管理（ApiServlet）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/service/source/ColConfigCtrl.java`
> 类级路由：`/source/ColConfigCtrl`（完整前缀 `/openapi/source/ColConfigCtrl`）
> 基类：`GenericBaseService`
> 业务：数据表列属性的批量更新、添加列、脚本变量同步。

## 方法列表总表

| # | 路径 | 方法名 | 说明 |
|---|------|--------|------|
| 1 | `/updateBatch` | updateBatch | 批量修改列属性（**已废弃 @Deprecated**） |
| 2 | `/addCol` | addCol | 添加列并设第一行默认值（**已废弃 @Deprecated**） |
| 3 | `/syncScript` | syncScript | 同步脚本中的变量到数据表 |

涉及表：`datatable_col_config`、`datatable_values`。

---

## 关键方法详解

### 1. POST /updateBatch — 批量修改列属性（@Deprecated）

**入参**：`DatatableDTO`，含 `colConfigList`（列配置列表，每项含 `id`、`sqlCol` 等）

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| colConfigList | List\<ColConfig\> | 是 | 列配置列表；每项字段：id、eid、projectId、sourceConfigId、name（变量名）、type、quoteType、descr、scope、colIndex、showInReport、sqlCol、updateBy、defaultValue（缺省 list 为空时抛「列表不能为空」） |
| eid | Integer | 否 | 企业 id（循环写入每条 colConfig） |
| projectid | Integer | 否 | 项目 id（循环写入每条 colConfig） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | Boolean | 是否保存成功 |

循环设置每条记录的 `eid`、`projectId` 后调用 `colConfigService.saveOrUpdateBatch`。

### 2. POST /addCol — 添加列（@Deprecated）

**入参**：`DatatableDTO`，需 `sourceConfigId`、`updateBy`、`columnInfoList`（name + defaultValue）

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| sourceConfigId | Long | 是 | 表 id（validateParams 判空） |
| updateBy | String | 是 | 更新人（validateParams 判空） |
| columnInfoList | List\<ColConfig\> | 否 | 列信息；每项字段：name（变量名）、defaultValue（默认值） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | Boolean | 是否添加成功 |

在一个表中添加多列，并为每列第一行设默认值。

### 3. POST /syncScript — 同步脚本变量

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| sourceConfigId | Long | 是 | 表 id（validateParams 判空） |
| updateBy | String | 是 | 更新人（validateParams 判空） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | Boolean | 是否同步成功 |

**核心功能**：数据表已建立后，脚本又新增了变量。点击数据表右上角的"拉取"按钮，将新增的脚本变量同步到数据源表中（只同步变量名，不同步默认值）。

**同步策略**：
- 如果空列存在 → 填充到空列位置
- 如果没有空列 → 向后追加新列

**异常处理**：捕获 `unique_name` 冲突 → "重复的列名，请刷新页面后重试"

**实现意图总结**：
当脚本参数发生变化（新增变量），需要保持数据源表与脚本变量的同步。该接口避免用户手动逐列添加变量名。
