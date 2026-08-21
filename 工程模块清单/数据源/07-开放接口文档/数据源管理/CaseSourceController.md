# CaseSourceController — 用例数据源控制器

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/controller/testCase/CaseSourceController.java`
> 类级路由：`/datasource/test_case`（完整前缀 `/openapi/v3/datasource/test_case`）
> Service 实现：`CaseSourceService`、`CaseSourceTagConfigService`、`CaseSourceRelationService`
> 业务：用例数据源完整管理 — CRUD + 树 + 行列编辑 + 标签 + 用例关联绑定/解绑/同步 + 批量操作（27 端点）。

## 接口列表总表

### 基础 CRUD + 树（7 端点）

| # | 方法 | 路径 | 方法名 | 说明 |
|---|------|------|--------|------|
| 1 | GET | `/v3/datasource/test_case/tree` | getDataSourceTree | 查询目录树 |
| 2 | POST | `/v3/datasource/test_case/sources` | getList | 分页查询数据源列表 |
| 3 | POST | `/v3/datasource/test_case/source/add` | addSource | 添加数据源 |
| 4 | PUT | `/v3/datasource/test_case/source/edit` | editSource | 编辑数据源 |
| 5 | DELETE | `/v3/datasource/test_case/source` | deleteSource | 删除数据源（需 project_id/source_id/user_id） |
| 6 | POST | `/v3/datasource/test_case/source/move` | moveSource | 移动数据源 |
| 7 | POST | `/v3/datasource/test_case/source/base_info` | getDataSourceByIds | 根据 IDs 批量查询数据源基础信息 |

### 列与值编辑（2 端点）

| # | 方法 | 路径 | 方法名 | 说明 |
|---|------|------|--------|------|
| 8 | POST | `/v3/datasource/test_case/column/edit` | editColumn | 编辑列定义 |
| 9 | POST | `/v3/datasource/test_case/value/edit` | editValue | 编辑单元格值 |

### 表格数据（2 端点）

| # | 方法 | 路径 | 方法名 | 说明 |
|---|------|------|--------|------|
| 10 | GET | `/v3/datasource/test_case/table/data` | getTableData | 获取表格数据 |
| 11 | POST | `/v3/datasource/test_case/table/data/format` | getFormatTableData | 获取格式化表格数据（无需登录） |

### 行列操作（4 端点）

| # | 方法 | 路径 | 方法名 | 说明 |
|---|------|------|--------|------|
| 12 | POST | `/v3/datasource/test_case/table/rows/delete` | removeRows | 删除行 |
| 13 | POST | `/v3/datasource/test_case/table/columns/delete` | removeCols | 删除列 |
| 14 | POST | `/v3/datasource/test_case/table/rows/add` | addRows | 添加行 |
| 15 | POST | `/v3/datasource/test_case/table/columns/add` | addCols | 添加列 |

### 批量操作（1 端点）

| # | 方法 | 路径 | 方法名 | 说明 |
|---|------|------|--------|------|
| 16 | POST | `/v3/datasource/test_case/source/batch` | batchOperate | 批量操作 |

### 标签（1 端点）

| # | 方法 | 路径 | 方法名 | 说明 |
|---|------|------|--------|------|
| 17 | PUT | `/v3/datasource/test_case/tag/edit` | editTag | 编辑标签 |

### 用例关联（8 端点）

| # | 方法 | 路径 | 方法名 | 说明 |
|---|------|------|--------|------|
| 18 | POST | `/v3/datasource/test_case/source/add_case_relation` | addCaseRelation | 添加用例关联（绑定数据源） |
| 19 | POST | `/v3/datasource/test_case/source/remove_case_relation` | removeCaseRelation | 移除用例关联 |
| 20 | GET | `/v3/datasource/test_case/source` | getCaseSourceRelation | 按 case_id 查关联的数据源 |
| 21 | POST | `/v3/datasource/test_case/source` | getCaseSourceList | 批量查询用例数据源列表 |
| 22 | GET | `/v3/datasource/test_case/source/get_case_by_source` | getCaseBySource | 按 source_id 反查关联的用例 ID 列表 |
| 23 | GET | `/v3/datasource/test_case/source/count` | getCaseSourceCountMap | 统计各数据源下用例数量 |
| 24 | POST | `/v3/datasource/test_case/source/sync_case` | syncCase | 同步用例关联 |
| 25 | PUT | `/v3/datasource/test_case/source/edit_case_row_id` | editCaseRowId | 编辑用例关联的行 ID（回放调试用） |

### 其他（2 端点）

| # | 方法 | 路径 | 方法名 | 说明 |
|---|------|------|--------|------|
| 26 | GET | `/v3/datasource/test_case/table/get_row` | getTableRow | 获取表格行数据 |
| 27 | PUT | `/v3/datasource/test_case/source/unbind_case_row_id` | unbindCaseRowId | 解绑用例行 ID |

统一响应包装：`ResponseResult<T>`。GET 带 `@UnderlineToCamel`。

涉及表：`case_source`、`case_column_info`、`case_value_info`、`case_table_config`、`case_source_relation`、`case_source_tag_config`。

---

## 1. GET /v3/datasource/test_case/tree — 查询目录树

### 入口

`CaseSourceController.getDataSourceTree(@UnderlineToCamel @Valid DataSourceTreeRequestDTO request)`

### 请求参数（Query，@UnderlineToCamel）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 否 | 企业 ID |
| project_id | Integer | 是 | 项目 ID（@NotNull，下划线转驼峰 projectId） |
| uid | Integer | 否 | 用户 ID |
| user_name | String | 否 | 用户名 |
| user_id | Integer | 否 | 用户 ID |
| lazy_tree | Integer | 否 | 是否懒加载 0 懒加载 1 加载子目录 |
| parent_id | Long | 否 | 父数据源 id |

### 返回参数

`ResponseResult<ResponseListDTO<CaseSourceTreeResponseDTO>>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.list | List\<CaseSourceTreeResponseDTO\> | 树形数据源列表 |
| data.list[].id | Long | 主键 |
| data.list[].projectId | Integer | 项目 id |
| data.list[].parentId | Long | 父节点 id |
| data.list[].name | String | 目录/实例表名称 |
| data.list[].type | Short | 类型 0 目录 1 实例表 2 数据源 |
| data.list[].createUser | String | 创建人 |
| data.list[].updateUser | String | 更新人 |
| data.list[].createTime | Long | 创建时间 |
| data.list[].updateTime | Long | 更新时间 |
| data.list[].caseSourceOrder | Integer | 数据源排序 |
| data.list[].childrenList | List\<CaseSourceTreeResponseDTO\> | 子数据源信息 |

---

## 2. POST /v3/datasource/test_case/sources — 分页查询数据源列表

### 入口

`CaseSourceController.getList(@Valid @RequestBody CaseSourceQueryDTO)`

### 请求参数（JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| page | Integer | 否 | 页码 |
| pageSize | Integer | 否 | 每页条数 |
| eid | Integer | 否 | 企业 ID |
| projectId | Integer | 是 | 项目 ID（@NotNull） |
| parentId | Long | 否 | 父级 id |
| name | String | 否 | 实例表名称 |
| varName | String | 否 | 变量名 |
| varValue | String | 否 | 变量值 |
| fuzzyByValue | Integer | 否 | 模糊查询（默认精准匹配；仅 1/2 合法） |
| varInfo | Object | 否 | 变量名=变量值查询条件 |
| varInfo.varName | String | 否 | 变量名 |
| varInfo.varValue | String | 否 | 变量值 |
| caseSensitive | Integer | 否 | 是否区分大小写 |
| isPage | Integer | 否 | 是否分页 1 分页 0 不分页（默认 0） |
| caseSourceIdList | List\<Long\> | 否 | 用例数据源 id 列表 |

### 返回参数

`ResponseResult<Object>`，当 `isPage=0`（默认）返回 `ResultListDTO<CaseSourceResponseDTO>`，当 `isPage=1` 返回 `PageInfoList<CaseSourceResponseDTO>`。

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.list | List\<CaseSourceResponseDTO\> | 数据源列表 |
| data.list[].id | Long | 主键 |
| data.list[].projectId | Integer | 项目 id |
| data.list[].parentId | Long | 父节点 id |
| data.list[].name | String | 目录/实例表名称 |
| data.list[].type | Short | 类型 0 目录 1 实例表 2 数据源 |
| data.list[].createUserId | Integer | 创建人 id |
| data.list[].updateUserId | Integer | 更新人 id |
| data.list[].createTime | String | 创建时间 |
| data.list[].updateTime | String | 更新时间 |
| data.list[].caseSourceOrder | Integer | 排序值 |
| data.page | Integer | 页码（isPage=1 时返回） |
| data.pageSize | Integer | 每页条数（isPage=1 时返回） |
| data.totalRow | Long | 总记录数（isPage=1 时返回） |
| data.totalPage | Integer | 总页数（isPage=1 时返回） |

---

## 3. POST /v3/datasource/test_case/source/add — 添加数据源

### 入口

`CaseSourceController.addSource(@Valid @RequestBody CaseSourceAddDTO)`

### 请求参数（JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 否 | 企业 ID |
| projectId | Integer | 是 | 项目 ID（@NotNull） |
| parentId | Long | 是 | 父节点 id（@NotNull） |
| name | String | 是 | 名称（@NotNull） |
| type | Short | 是 | 节点类型 0 目录 1 实例表 2 数据源（@NotNull） |
| userName | String | 是 | 用户名（@NotNull） |
| userId | Integer | 是 | 用户 id（@NotNull） |

### 返回参数

`ResponseResult<CaseSourceResponseDTO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.id | Long | 新增记录主键 |
| data.projectId | Integer | 项目 id |
| data.parentId | Long | 父节点 id |
| data.name | String | 名称 |
| data.type | Short | 类型 0 目录 1 实例表 2 数据源 |
| data.createUserId | Integer | 创建人 id |
| data.updateUserId | Integer | 更新人 id |
| data.createTime | String | 创建时间 |
| data.updateTime | String | 更新时间 |
| data.caseSourceOrder | Integer | 排序值 |

---

## 4. PUT /v3/datasource/test_case/source/edit — 编辑数据源

### 入口

`CaseSourceController.editSource(@Valid @RequestBody CaseSourceOperateDTO)`

### 请求参数（JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 否 | 企业 ID |
| projectId | Integer | 是 | 项目 ID（@NotNull） |
| caseSourceId | Long | 是 | 用例数据源 id（@NotNull） |
| name | String | 是 | 名称（代码 `StringUtils.isEmpty` 校验非空） |
| targetParentDirId | Long | 否 | 移动目标目录 id（编辑时不用） |
| parentDirId | Long | 否 | 原数据源目录 id（同名校验用） |
| targetOrder | Integer | 否 | 移动目标顺序（编辑时不用） |
| type | Short | 否 | 用例数据源类型 |
| userName | String | 是 | 用户名（@NotNull） |
| userId | Integer | 是 | 用户 id（@NotNull） |

### 返回参数

`ResponseResult<ResultDTO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.result | Boolean | 是否编辑成功 |

---

## 5. DELETE /v3/datasource/test_case/source — 删除数据源

### 入口

`CaseSourceController.deleteSource(project_id, source_id, user_id)`

### 请求参数（Query）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| project_id | Integer | 是 | 项目 ID（无默认值） |
| source_id | Long | 是 | 用例数据源 id（无默认值） |
| user_id | Integer | 是 | 用户 id（无默认值） |

### 返回参数

`ResponseResult<ResultDTO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.result | Boolean | 是否删除成功（顶层目录返回 false） |

---

## 6. POST /v3/datasource/test_case/source/move — 移动数据源

### 入口

`CaseSourceController.moveSource(@Valid @RequestBody CaseSourceOperateDTO)`

### 请求参数（JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 否 | 企业 ID（默认 1） |
| projectId | Integer | 是 | 项目 ID（@NotNull） |
| caseSourceId | Long | 是 | 用例数据源 id（@NotNull） |
| name | String | 否 | 名称（移动时不用） |
| targetParentDirId | Long | 是 | 目标目录 id（代码 null 校验抛异常） |
| parentDirId | Long | 否 | 原数据源目录 id |
| targetOrder | Integer | 否 | 目标顺序（null 则移动到末尾） |
| type | Short | 否 | 用例数据源类型 |
| userName | String | 是 | 用户名（@NotNull） |
| userId | Integer | 是 | 用户 id（@NotNull） |

### 返回参数

`ResponseResult<ResultDTO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.result | Boolean | 是否移动成功 |

---

## 7. POST /v3/datasource/test_case/source/base_info — 根据 IDs 批量查询数据源基础信息

### 入口

`CaseSourceController.getDataSourceByIds(@RequestBody CaseDataSourceRequestDTO)`

### 请求参数（JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| caseIds | List\<Integer\> | 否 | 用例 id 列表 |
| dataSourceIds | List\<Integer\> | 否 | 数据源 id 列表 |

### 返回参数

`ResponseResult<ResultListDTO<CaseSource>>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.list | List\<CaseSource\> | 数据源基础信息列表 |
| data.list[].id | Long | 主键 |
| data.list[].eid | Integer | 企业 id |
| data.list[].projectId | Integer | 项目 id |
| data.list[].parentId | Long | 父节点 id |
| data.list[].name | String | 名称 |
| data.list[].caseSourceOrder | Integer | 当前目录下顺序 |
| data.list[].type | Short | 类型 0 目录 1 实例表 2 数据源 |
| data.list[].status | Short | 状态 1 正常 0 逻辑删除 |
| data.list[].createUserId | Integer | 创建人 id |
| data.list[].updateUserId | Integer | 更新人 id |
| data.list[].createTime | Date | 创建时间 |
| data.list[].updateTime | Date | 更新时间 |
| data.list[].caseTableRowId | Integer | 用例数据源某一行 |
| data.list[].caseId | Long | 关联的用例 id |

---

## 8. POST /v3/datasource/test_case/column/edit — 编辑列定义

### 入口

`CaseSourceController.editColumn(@Valid @RequestBody CaseColumnInfoDTO)`

### 请求参数（JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 否 | 企业 ID |
| projectId | Integer | 是 | 项目 ID（@NotNull） |
| caseSourceId | Long | 是 | 用例数据源 id（@NotNull） |
| varName | String | 是 | 变量名（@NotNull） |
| type | Short | 否 | 列类型（1 字符串） |
| colIndex | Integer | 是 | 列号（@NotNull） |
| desc | String | 否 | 列备注 |
| userName | String | 是 | 用户名（@NotNull） |
| userId | Integer | 是 | 用户 id（@NotNull） |
| showInReport | Integer | 否 | 是否在报告中展示 |

### 返回参数

`ResponseResult<ResultDTO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.result | Boolean | 是否编辑成功 |

---

## 9. POST /v3/datasource/test_case/value/edit — 编辑单元格值

### 入口

`CaseSourceController.editValue(@Valid @RequestBody CaseValueInfoDTO)`

### 请求参数（JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 否 | 企业 ID |
| projectId | Integer | 是 | 项目 ID（@NotNull） |
| caseSourceId | Long | 是 | 用例数据源 id（@NotNull） |
| values | List\<CaseValueInfoEditDTO\> | 否 | 编辑的多个单元格 |
| values[].colIndex | Integer | 是 | 列号（@NotNull） |
| values[].rowIndex | Integer | 是 | 行号（@NotNull） |
| values[].varValue | String | 否 | 单元格值 |
| values[].caseSourceId | Long | 是 | 用例数据源 id（@NotNull） |
| userName | String | 是 | 用户名（@NotNull） |
| userId | Integer | 是 | 用户 id（@NotNull） |

### 返回参数

`ResponseResult<ResultDTO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.result | Boolean | 是否编辑成功 |

---

## 10. GET /v3/datasource/test_case/table/data — 获取表格数据

### 入口

`CaseSourceController.getTableData(eid, project_id, table_id)`

### 请求参数（Query）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 否 | 企业 ID（默认 1） |
| project_id | Integer | 是 | 项目 ID（无默认值） |
| table_id | Long | 是 | 用例数据源表格 id（无默认值） |

### 返回参数

`ResponseResult<CaseTableDataResponseDTO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.colList | List\<CaseColumnInfoResponseDTO\> | 列信息列表 |
| data.colList[].id | Integer | 列 id |
| data.colList[].projectId | Integer | 项目 id |
| data.colList[].tableId | Long | 表格 id |
| data.colList[].varName | String | 列名 |
| data.colList[].type | Short | 列类型 |
| data.colList[].colIndex | Integer | 列号 |
| data.colList[].desc | String | 列备注 |
| data.colList[].showInReport | Integer | 是否在报告展示 |
| data.rowList | List\<CaseValueInfoResponseDTO\> | 行信息列表 |
| data.rowList[].rowIndex | Integer | 行号 |
| data.rowList[].valueList | List\<CaseTableCellDTO\> | 每行的单元格值信息 |
| data.rowList[].valueList[].id | Long | 单元格 id |
| data.rowList[].valueList[].projectId | Integer | 项目 id |
| data.rowList[].valueList[].tableId | Long | 表格 id |
| data.rowList[].valueList[].varValue | String | 单元格值 |
| data.rowList[].valueList[].rowIndex | Integer | 行号 |
| data.rowList[].valueList[].colIndex | Integer | 列号 |
| data.tagList | List\<CaseSourceTagFrontendResponseDTO\> | 标签列表 |
| data.tagList[].rowIndex | Integer | 行索引 |
| data.tagList[].tagInfoList | List\<CaseSourceTagConfigDTO\> | 标签信息列表 |
| data.tagList[].tagInfoList[].tagId | Integer | 标签 id |
| data.tagList[].tagInfoList[].tagName | String | 标签名称 |

---

## 11. POST /v3/datasource/test_case/table/data/format — 获取格式化表格数据

### 入口

`CaseSourceController.getFormatTableData(@RequestBody FormatTableDataDTO)`（无需登录）

### 请求参数（JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 否 | 企业 ID（默认 1） |
| projectId | Integer | 是 | 项目 ID（代码 null 校验抛异常） |
| tableId | Long | 否 | 表格 id（null 时按 caseId 反查） |
| caseId | Integer | 否 | 用例 id |
| rowIndexList | List\<Integer\> | 否 | 行索引列表（null 返回所有行） |
| ignoreEmptyValue | Boolean | 否 | 是否忽略空值（默认忽略） |
| tagList | List\<Integer\> | 否 | 包含标签列表 |
| noHasTagList | List\<Integer\> | 否 | 不包含标签列表 |

### 返回参数

`ResponseResult<TestCaseTableDataResultListDTO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.dataList | List\<CaseSourceDataResponseDTO\> | 数据集合 |
| data.dataList[].rowIndex | Integer | 行索引 |
| data.dataList[].type | Short | 数值类型 |
| data.dataList[].name | String | 列名称 |
| data.dataList[].value | String | 列值 |
| data.dataList[].desc | String | 列描述 |
| data.dataList[].showInReport | Integer | 是否在报告展示 |
| data.tagList | List\<CaseSourceTagExecResponseDTO\> | 标签集合 |
| data.tagList[].rowIndex | Integer | 行索引 |
| data.tagList[].tagInfoList | List\<TagInfo\> | 标签信息列表 |
| data.tagList[].tagNameList | List\<String\> | 标签名称列表 |
| data.tagList[].tagIdList | List\<Integer\> | 标签 id 列表 |
| data.tableName | String | 用例数据源名称 |
| data.tableId | Long | 用例数据源 id |

---

## 12. POST /v3/datasource/test_case/table/rows/delete — 删除行

### 入口

`CaseSourceController.removeRows(@Valid @RequestBody CaseRemoveRowOrColDTO)`

### 请求参数（JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 否 | 企业 ID |
| projectId | Integer | 是 | 项目 ID（@NotNull） |
| tableId | Long | 是 | 表格 id（@NotNull） |
| deleteRowList | List\<Integer\> | 否 | 需要删除的行号集合 |
| deleteColList | List\<Integer\> | 否 | 需要删除的列号集合（删除行时不用） |
| userName | String | 是 | 用户名（@NotNull） |
| userId | Integer | 是 | 用户 id（@NotNull） |

### 返回参数

`ResponseResult<ResultDTO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.result | Boolean | 是否删除成功 |

---

## 13. POST /v3/datasource/test_case/table/columns/delete — 删除列

### 入口

`CaseSourceController.removeCols(@Valid @RequestBody CaseRemoveRowOrColDTO)`

### 请求参数（JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 否 | 企业 ID |
| projectId | Integer | 是 | 项目 ID（@NotNull） |
| tableId | Long | 是 | 表格 id（@NotNull） |
| deleteRowList | List\<Integer\> | 否 | 需要删除的行号集合（删除列时不用） |
| deleteColList | List\<Integer\> | 否 | 需要删除的列号集合 |
| userName | String | 是 | 用户名（@NotNull） |
| userId | Integer | 是 | 用户 id（@NotNull） |

### 返回参数

`ResponseResult<ResultDTO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.result | Boolean | 是否删除成功 |

---

## 14. POST /v3/datasource/test_case/table/rows/add — 添加行

### 入口

`CaseSourceController.addRows(@Valid @RequestBody CaseAddColRowDTO)`

### 请求参数（JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 否 | 企业 ID |
| projectId | Integer | 是 | 项目 ID（@NotNull） |
| tableId | Long | 是 | 表格 id（@NotNull） |
| startOrder | Integer | 是 | 插入位置（0 插到最前，否则插入到该序号之后）（@NotNull） |
| count | Integer | 是 | 插入行数量（@NotNull） |
| userName | String | 是 | 用户名（@NotNull） |
| userId | Integer | 是 | 用户 id（@NotNull） |

### 返回参数

`ResponseResult<ResultDTO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.result | Boolean | 是否添加成功 |

---

## 15. POST /v3/datasource/test_case/table/columns/add — 添加列

### 入口

`CaseSourceController.addCols(@Valid @RequestBody CaseAddColRowDTO)`

### 请求参数（JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 否 | 企业 ID |
| projectId | Integer | 是 | 项目 ID（@NotNull） |
| tableId | Long | 是 | 表格 id（@NotNull） |
| startOrder | Integer | 是 | 插入位置（0 插到最前，否则插入到该序号之后）（@NotNull） |
| count | Integer | 是 | 插入列数量（@NotNull） |
| userName | String | 是 | 用户名（@NotNull） |
| userId | Integer | 是 | 用户 id（@NotNull） |

### 返回参数

`ResponseResult<ResultDTO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.result | Boolean | 是否添加成功 |

---

## 16. POST /v3/datasource/test_case/source/batch — 批量操作

### 入口

`CaseSourceController.batchOperate(@Valid @RequestBody CaseSourceBatchOperateDTO)`

### 请求参数（JSON Body，CaseSourceBatchOperateDTO extends CaseSourceQueryDTO）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| page | Integer | 否 | 页码 |
| pageSize | Integer | 否 | 每页条数 |
| eid | Integer | 否 | 企业 ID |
| projectId | Integer | 是 | 项目 ID（@NotNull） |
| parentId | Long | 否 | 父级 id |
| name | String | 否 | 实例表名称 |
| varName | String | 否 | 变量名 |
| varValue | String | 否 | 变量值 |
| fuzzyByValue | Integer | 否 | 模糊查询 |
| varInfo | Object | 否 | 变量名=变量值查询条件 |
| caseSensitive | Integer | 否 | 是否区分大小写 |
| isPage | Integer | 否 | 是否分页 |
| caseSourceIdList | List\<Long\> | 否 | 用例数据源 id 列表 |
| selectAllFlag | Integer | 否 | 是否全选 1 全选 0 不全选（默认 1） |
| dataSourceIds | List\<Long\> | 否 | 筛选出的 id 列表（不全选时需传） |
| batchType | Integer | 是 | 批量操作类型 1 批量替换 2 批量删除（@NotNull） |
| replaceParam | Object | 否 | 批量替换参数（批量替换时必填） |
| replaceParam.newVarValue | String | 是 | 替换后的新变量值（@NotNull） |
| replaceParam.caseSensitive | Integer | 否 | 是否区分大小写（默认 0） |
| replaceParam.varName | String | 否 | 变量名 |
| replaceParam.varValue | String | 否 | 需要替换的变量值 |
| userName | String | 是 | 用户名（@NotNull） |
| userId | Integer | 是 | 用户 id（@NotNull） |

### 返回参数

`ResponseResult<BatchOperateResponseDTO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.successCount | Integer | 操作成功数量 |
| data.failCount | Integer | 操作失败数量 |

---

## 17. PUT /v3/datasource/test_case/tag/edit — 编辑标签

### 入口

`CaseSourceController.editTag(@RequestBody CaseSourceTagUpdateDTO)`

### 请求参数（JSON Body，无 @Valid 校验）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| userId | Integer | 否 | 用户 id |
| projectId | Integer | 否 | 项目 id |
| eid | Integer | 否 | 企业 id |
| sourceId | Long | 否 | 用例数据源 id |
| tagIdList | List\<Integer\> | 否 | 标签 id 列表 |
| rowIndex | Integer | 否 | 行索引 |

### 返回参数

`ResponseResult<ResultDTO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.result | Boolean | 是否编辑成功 |

---

## 18. POST /v3/datasource/test_case/source/add_case_relation — 添加用例关联

### 入口

`CaseSourceController.addCaseRelation(@RequestBody CaseSourceRelationRequestDTO)`

### 请求参数（JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| caseSourceId | Long | 是 | 用例数据源 id（@NotNull） |
| caseId | Long | 是 | 用例 id（@NotNull） |
| caseTableRowId | Integer | 否 | 用例数据源某一行 |
| projectId | Integer | 是 | 项目 id（@NotNull） |
| eid | Integer | 是 | 企业 id（@NotNull） |
| userId | Integer | 否 | 用户 id |
| userName | String | 否 | 用户名 |

### 返回参数

`ResponseResult<ResultDTO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.result | Boolean | 是否绑定成功 |

---

## 19. POST /v3/datasource/test_case/source/remove_case_relation — 移除用例关联

### 入口

`CaseSourceController.removeCaseRelation(@RequestBody CaseSourceRelationRequestDTO)`

### 请求参数（JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| caseSourceId | Long | 是 | 用例数据源 id（@NotNull） |
| caseId | Long | 是 | 用例 id（@NotNull） |
| caseTableRowId | Integer | 否 | 用例数据源某一行 |
| projectId | Integer | 是 | 项目 id（@NotNull） |
| eid | Integer | 是 | 企业 id（@NotNull） |
| userId | Integer | 否 | 用户 id |
| userName | String | 否 | 用户名 |

### 返回参数

`ResponseResult<ResultDTO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.result | Boolean | 是否解绑成功 |

---

## 20. GET /v3/datasource/test_case/source — 按 case_id 查关联的数据源

### 入口

`CaseSourceController.getCaseSourceRelation(case_id)`

### 请求参数（Query）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| case_id | Integer | 是 | 用例 id（无默认值） |

### 返回参数

`ResponseResult<Object>`，data 为 `CaseSource` 实体。

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.id | Long | 主键 |
| data.eid | Integer | 企业 id |
| data.projectId | Integer | 项目 id |
| data.parentId | Long | 父节点 id |
| data.name | String | 名称 |
| data.caseSourceOrder | Integer | 当前目录下顺序 |
| data.type | Short | 类型 0 目录 1 实例表 2 数据源 |
| data.status | Short | 状态 1 正常 0 逻辑删除 |
| data.createUserId | Integer | 创建人 id |
| data.updateUserId | Integer | 更新人 id |
| data.createTime | Date | 创建时间 |
| data.updateTime | Date | 更新时间 |
| data.caseTableRowId | Integer | 用例数据源某一行 |
| data.caseId | Long | 关联的用例 id |

---

## 21. POST /v3/datasource/test_case/source — 批量查询用例数据源列表

### 入口

`CaseSourceController.getCaseSourceList(@RequestBody CaseDataSourceRequestDTO)`

### 请求参数（JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| caseIds | List\<Integer\> | 否 | 用例 id 列表 |
| dataSourceIds | List\<Integer\> | 否 | 数据源 id 列表 |

### 返回参数

`ResponseResult<ResultListDTO<CaseSource>>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.list | List\<CaseSource\> | 用例数据源列表（字段同第 20 节 data） |

---

## 22. GET /v3/datasource/test_case/source/get_case_by_source — 按 source_id 反查用例 ID 列表

### 入口

`CaseSourceController.getCaseBySource(source_id)`

### 请求参数（Query）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| source_id | Long | 是 | 数据源 id（无默认值） |

### 返回参数

`ResponseResult<List<Long>>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data[] | Long | 使用该数据源的用例 id 列表 |

---

## 23. GET /v3/datasource/test_case/source/count — 统计各数据源节点下用例数量

### 入口

`CaseSourceController.getCaseSourceCountMap(eid, project_id, parent_id)`

### 请求参数（Query）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 是 | 企业 id（无默认值） |
| project_id | Integer | 是 | 项目 id（无默认值） |
| parent_id | Long | 否 | 父数据源 id（默认 0） |

### 返回参数

`ResponseResult<Map<Long, Long>>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data | Map\<Long,Long\> | key=数据源节点 id，value=用例数量 |

---

## 24. POST /v3/datasource/test_case/source/sync_case — 同步用例关联

### 入口

`CaseSourceController.syncCase(@RequestBody CaseSourceRelationRequestDTO)`

### 请求参数（JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| caseSourceId | Long | 是 | 用例数据源 id（@NotNull + 代码 null 校验） |
| caseId | Long | 是 | 用例 id（@NotNull） |
| caseTableRowId | Integer | 否 | 用例数据源某一行 |
| projectId | Integer | 是 | 项目 id（@NotNull） |
| eid | Integer | 是 | 企业 id（@NotNull） |
| userId | Integer | 否 | 用户 id |
| userName | String | 否 | 用户名 |

### 返回参数

`ResponseResult<ResultDTO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.result | Boolean | 是否同步成功 |

---

## 25. PUT /v3/datasource/test_case/source/edit_case_row_id — 编辑用例关联的行 ID

### 入口

`CaseSourceController.editCaseRowId(@RequestBody CaseSourceRelationRequestDTO)`

### 请求参数（JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| caseSourceId | Long | 是 | 用例数据源 id（@NotNull） |
| caseId | Long | 是 | 用例 id（@NotNull） |
| caseTableRowId | Integer | 否 | 用例数据源某一行 |
| projectId | Integer | 是 | 项目 id（@NotNull） |
| eid | Integer | 是 | 企业 id（@NotNull） |
| userId | Integer | 否 | 用户 id |
| userName | String | 否 | 用户名 |

### 返回参数

`ResponseResult<ResultDTO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.result | Integer | 影响行数 |

---

## 26. GET /v3/datasource/test_case/table/get_row — 获取表格行数据

### 入口

`CaseSourceController.getTableRow(@UnderlineToCamel CaseSourceRelationRequestDTO)`

### 请求参数（Query，@UnderlineToCamel）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| case_source_id | Long | 是 | 用例数据源 id（代码 null 校验） |
| case_id | Long | 否 | 用例 id（此接口不用） |
| case_table_row_id | Integer | 是 | 行 id（代码 null 校验） |
| project_id | Integer | 否 | 项目 id（查询过滤用） |
| eid | Integer | 否 | 企业 id |
| user_id | Integer | 否 | 用户 id |
| user_name | String | 否 | 用户名 |

### 返回参数

`ResponseResult<ResultListDTO<CaseRowInfoResponseDTO>>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.list | List\<CaseRowInfoResponseDTO\> | 行数据列表 |
| data.list[].varName | String | 变量名 |
| data.list[].varValue | String | 变量值 |
| data.list[].type | short | 变量类型 |

---

## 27. PUT /v3/datasource/test_case/source/unbind_case_row_id — 解绑用例行 ID

### 入口

`CaseSourceController.unbindCaseRowId(@RequestBody CaseSourceRelationRequestDTO)`

### 请求参数（JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| caseSourceId | Long | 是 | 用例数据源 id（@NotNull） |
| caseId | Long | 是 | 用例 id（@NotNull） |
| caseTableRowId | Integer | 否 | 用例数据源某一行 |
| projectId | Integer | 是 | 项目 id（@NotNull） |
| eid | Integer | 是 | 企业 id（@NotNull） |
| userId | Integer | 否 | 用户 id |
| userName | String | 否 | 用户名 |

### 返回参数

`ResponseResult<ResultDTO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.result | Integer | 影响行数 |

---

## 核心业务：用例关联

CaseSourceController 区别于 NormalSourceController 的核心功能是用例关联管理：

### 绑定流程

```
POST /source/add_case_relation
→ case_source_relation 写入 (case_id, case_source_id, case_table_row_id)
→ 用例执行时通过 case_id 找到关联的实例表和数据行
```

### 回放调试

```
PUT /source/edit_case_row_id
→ 更新 case_table_row_id 指定回放时使用的数据行
```

### 反查

```
GET /source/get_case_by_source?source_id=xxx
→ 返回使用该数据源的用例 ID 列表
GET /source/count?eid=&project_id=&parent_id=0
→ 返回 Map<Long,Long> 各数据源节点下的用例数量
```

## 实现意图总结

- 与 NormalSourceController 共通的 CRUD/树/行列操作
- 额外提供用例 ↔ 数据源的双向绑定管理
- `syncCase` 支持批量同步用例关联
- `case_table_row_id` 机制支持脚本回放时指定具体数据行
