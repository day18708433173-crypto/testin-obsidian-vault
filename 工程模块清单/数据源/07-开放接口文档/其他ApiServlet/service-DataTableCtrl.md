# service-DataTableCtrl — 数据表操作（ApiServlet）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/service/source/DataTableCtrl.java`
> 类级路由：`/source/DataTableCtrl`（完整前缀 `/openapi/source/DataTableCtrl`）
> 基类：`GenericBaseService`
> 业务：在线表格数据操作 — 行/列增删、列配置、标签配置、单元格编辑、数据查询、批量替换、脚本参数获取。

## 方法列表总表

| # | 路径 | 方法名 | 说明 |
|---|------|--------|------|
| 1 | `/insertRowOrCol` | insertRowOrCol | 插入行或列 |
| 2 | `/deleteRowOrCol` | deleteRowOrCol | 删除行或列 |
| 3 | `/configCol` | configCol | 配置列属性 |
| 4 | `/configTag` | configTag | 配置行标签 |
| 5 | `/deleteTag` | deleteTag | 删除行标签 |
| 6 | `/updateData` | updateData | 更新单元格数据（增/改/删） |
| 7 | — | batchReplaceVarValue | 批量替换变量值（需 sid 校验） |
| 8 | `/selectData` | selectData | 查询表格数据（全局表/实例表） |
| 9 | `/selectGlobalData` | selectGlobalData | 查询全局表值 |
| 10 | `/getScriptParamData` | getScriptParamData | 获取脚本参数数据（提测业务消费） |
| 11 | `/selectValueByExpression` | selectValueByExpression | 按表达式列表获取值 |
| 12 | `/copyData` | copyData | 复制数据并应用标签 |

涉及表：`datatable_config`、`datatable_col_config`、`datatable_values`、`datatable_tag_config`。

---

## 关键方法详解

### 1. POST /insertRowOrCol — 插入行/列

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| sourceConfigId | Long | 是 | 表 ID |
| type | String | 是 | 1=行 2=列 |
| insertCount | String | 是 | 插入行数/列数 |
| startNumber | String | 是 | 从第几行/列开始插入（数据库实际行列号） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | DatatableConfigVO | 更新后的行列配置 |
| data.data.indexList | List\<Integer\> | 新增的行/列号列表 |
| data.data.colIndex | Integer | 列号 |
| data.data.id | Integer | 配置 id |
| data.data.eid | Integer | 企业 id |
| data.data.projectid | Integer | 项目 id |
| data.data.sourceConfigId | Long | 数据表 id |
| data.data.rowOrder | String | 行的顺序 |
| data.data.colOrder | String | 列的顺序 |
| data.data.rowMaxIndex | Integer | 表中行的最大值 |
| data.data.colMaxIndex | Integer | 表中列的最大值 |
| data.data.createtime / updatetime | Long | 创建时间 / 更新时间 |

**返回**：`DatatableConfigVO`（更新后的行列配置）

### 2. POST /deleteRowOrCol — 删除行/列

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| sourceConfigId | Long | 是 | 表 ID |
| type | String | 是 | 1=行 2=列 |
| deleteIndexList | List\<Integer\> | 是 | 要删除的行/列号列表 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | Boolean | 是否删除成功 |

### 3. POST /configCol — 配置列属性

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectid | Integer | 是 | 项目 id |
| sourceConfigId | Long | 是 | 表 id |
| name | String | 是 | 变量名 |
| type | Integer | 是 | 变量类型 |
| colIndex | Integer | 是 | 列号 |
| scope | String | 是 | 作用域 GLOBAL/LOCAL |
| id | Integer | 否 | 有则修改，无则新增 |
| descr | String | 否 | 变量描述 |
| sqlCol | String | 否 | sql 列名（暂未启用） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | Boolean | 是否保存成功 |

### 4. POST /configTag — 配置行标签

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| sourceConfigId | Long | 否 | 表 id |
| rowIndex | Integer | 否 | 行号 |
| tagId | Integer | 否 | 标签 id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | Boolean | 是否保存成功 |

### 5. POST /deleteTag — 删除行标签

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| id | Long | 否 | 标签配置 id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | Boolean | 是否删除成功 |

### 6. POST /updateData — 更新单元格数据

支持批量更新单元格值（增/改/删）。入口参数 `DatatableDTO` 包含更新项列表。

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| sourceConfigId | Long | 是 | 表 id（validateParams 判空） |
| datatableValuesList | List\<DatatableValues\> | 否 | 单元格数据列表；每项字段：id（新增传空/修改删除传返回值）、rowIndex、colIndex、paramValue（删除时置空）、valueType、sourceConfigId、paramType、expression 等 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | Boolean | 是否操作成功 |

### 7. batchReplaceVarValue — 批量替换变量值

无独立路径映射（内部方法）。先通过 `OnlineApi.getUserOnline(sid)` 校验用户合法性（调用 [user-manager](../../../平台基础功能服务/00-首页.md)），然后批量替换变量值。返回成功/失败计数。

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| condition | JSONObject | 否 | 查询条件（SelectDTO）；字段：name、scriptNo、typeList、eid、projectid、colName、paramValue、fuzzyByValue、tagName、caseSensitive、sourceId、parentId、conditions（QueryConditionDTO 列表：colName/varValue/operation）、selectAllFlag、dataSourceIds |
| replaceParam | JSONObject | 否 | 替换参数（ReplaceVarValueDTO）；字段：conditions（同左）、newVarValue、caseSensitive、updateTime |

> 顶层 `sid` 必填（`OnlineApi.getUserOnline(sid)` 校验，无 sid 抛「参数 sid 无效」）。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.successCount | Integer | 成功数量 |
| data.failCount | Integer | 失败数量 |

### 8. POST /selectData — 查询表格数据

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| sourceConfigId | Long | 是 | 表 id |
| globalId | Long | 否 | 全局表 id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | DataTableVO | 表格数据 |
| data.data.colConfigList | List\<ColConfig\> | 列信息列表（name/type/descr/scope/colIndex/showInReport/sqlCol 等） |
| data.data.rowInfoVOList | List\<RowInfoVO\> | 行信息列表（rowNumber、valueList、tagInfoList、tagConfigList） |
| data.data.value | String | 具体值 |
| data.data.code | String | 0 正常，1 没查到值，2 表达式不对 |
| data.data.valueList | List\<String\> | 值列表 |
| data.data.global | String | 实例表能否添加全局变量：0 不能，1 能 |

**返回**：`DataTableVO`（含列定义 + 行数据 + 标签信息）

### 9. POST /selectGlobalData — 查询全局表值

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| sourceConfigId | Long | 是 | 表 id |
| colName | String | 是 | 列名 |
| index | String | 是 | 行号 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | DataTableVO | 表格数据 |
| data.data.value | String | 具体值 |
| data.data.colConfigList | List\<ColConfig\> | 列信息列表 |
| data.data.valueList | List\<String\> | 值列表 |
| data.data.code | String | 0 正常，1 没查到值，2 表达式不对 |

### 10. POST /getScriptParamData — 获取脚本参数数据

核心消费接口。app处理服务 执行脚本时通过此方法获取数据源数据：
- 入参包含 `sourceConfigId`、标签 include/exclude 列表
- 返回 `ScriptParamData`（变量列表 + 值 + 标签筛选后的数据行）

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| （SourceConfigDTO 字段） | — | 否 | 脚本/数据源条件：id、eid、projectid、lazy、parentId、parentIdList、targetId、scriptNoList、sourceConfigId、scriptNo、url、updateBy、name、sourceId、type、treeType、treeTypeList、caseSensitive、colName、colType、paramValue、fuzzyByValue、tagName、importType、scriptDesc、sourceIdList、scriptNoRelationMap、childrenList、tableData、status、conditions、scriptNoRowIds、tagList、noHasTagList、needAllTag、needGlobal |
| tagList | JSONArray | 否 | 包含标签 id 列表（元素为 Integer） |
| noHasTagList | JSONArray | 否 | 排除标签 id 列表（元素为 Integer） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.list | List\<ScriptParamData\> | 脚本参数数据 |
| data.list[].id | Integer | 自增 id |
| data.list[].paramName | String | 参数名称 |
| data.list[].paramValue | List\<ScriptParamValueInfo\> | 参数值列表（每项：colId、rowId、value、skip） |
| data.list[].paramDesc | String | 参数描述 |
| data.list[].appId | Integer | 应用 id |
| data.list[].projectId | Integer | 项目组 id |
| data.list[].sourceId | String | 数据源 id |
| data.list[].createBy | Integer | 创建者 |
| data.list[].createTime | Long | 创建时间 |
| data.list[].isGlobal | int | 是否全局参数：0 否 1 是 |
| data.list[].type | String | 参数类型：normal 普通 / component 控件 |
| data.list[].scriptid | Integer | 脚本 id |
| data.list[].scriptNo | Integer | 脚本编号 |
| data.list[].varParamType | Integer | 变量类型：1 字符 2 数字 |
| data.list[].hide | Integer | 是否隐藏：0 隐藏 1 显示 |

### 11. POST /selectValueByExpression — 按表达式取值

入参传入表达式列表，返回按顺序对应的值列表。用于需要按指定列和条件取值的场景。

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| globalId | Long | 是 | 全局表 id |
| expressionList | List\<String\> | 否 | 表达式列表 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | DataTableVO | 表格数据 |
| data.data.valueList | List\<String\> | 按表达式顺序返回的值列表 |
| data.data.code | String | 0 正常，1 没查到值，2 表达式不对 |
| data.data.value | String | 具体值 |
| data.data.colConfigList | List\<ColConfig\> | 列信息列表 |

### 12. POST /copyData — 复制数据并应用标签

从源表复制数据到目标表并应用指定标签。

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| （DatatableDTO 字段） | — | 否 | 表格操作参数：sourceConfigId、globalId、tagConfigList、colConfigList、datatableValuesList、rowList、colList、updateBy、sync、iosTag、androidTag 等（无 validateParams 校验） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | Boolean | 是否复制成功 |
