# NormalSourceController — 普通数据源控制器

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/controller/normal/NormalSourceController.java`
> 类级路由：`/datasource/normal`（完整前缀 `/openapi/v3/datasource/normal`）
> Service 实现：`NormalSourceService`
> 业务：普通数据源（通用表数据）完整 CRUD + 树查询 + 行列操作 + 批量操作。

## 接口列表总表

| # | 方法 | 路径 | 方法名 | 说明 |
|---|------|------|--------|------|
| 1 | GET | `/v3/datasource/normal/tree` | getDataSourceTree | 查询目录树 |
| 2 | POST | `/v3/datasource/normal/sources` | getList | 分页查询数据源列表 |
| 3 | POST | `/v3/datasource/normal/source/add` | addSource | 添加数据源 |
| 4 | PUT | `/v3/datasource/normal/source/edit` | editSource | 编辑数据源 |
| 5 | DELETE | `/v3/datasource/normal/source` | deleteSource | 删除数据源 |
| 6 | POST | `/v3/datasource/normal/source/move` | moveSource | 移动数据源 |
| 7 | POST | `/v3/datasource/normal/column/edit` | editColumn | 编辑列定义 |
| 8 | POST | `/v3/datasource/normal/value/edit` | editValue | 编辑单元格值 |
| 9 | GET | `/v3/datasource/normal/table/data` | getTableData | 获取表格数据 |
| 10 | GET | `/v3/datasource/normal/table/data/format` | getFormatTableData | 获取格式化表格数据（无需登录） |
| 11 | POST | `/v3/datasource/normal/table/rows/delete` | removeRows | 删除行 |
| 12 | POST | `/v3/datasource/normal/table/columns/delete` | removeCols | 删除列 |
| 13 | POST | `/v3/datasource/normal/table/rows/add` | addRows | 添加行 |
| 14 | POST | `/v3/datasource/normal/table/columns/add` | addCols | 添加列 |
| 15 | POST | `/v3/datasource/normal/source/batch` | batchOperate | 批量操作 |

统一响应包装：`ResponseResult<T>`。GET 带 `@UnderlineToCamel`。

涉及表：`normal_source`、`normal_column_info`、`normal_value_info`、`normal_table_config`。

---

## 1. GET /v3/datasource/normal/tree — 查询目录树

### 入口

`NormalSourceController.getDataSourceTree(@UnderlineToCamel @Valid DataSourceTreeRequestDTO request)`

### 请求参数（Query，@UnderlineToCamel）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| project_id | Integer | 是 | 项目 ID（@NotNull，下划线转驼峰 projectId） |
| eid | Integer | 否 | 企业 ID |
| uid | Integer | 否 | 用户 ID |
| user_id | Integer | 否 | 用户 ID |
| user_name | String | 否 | 用户名 |
| lazy_tree | Integer | 否 | 是否懒加载 0 懒加载 1 加载子目录 |
| parent_id | Long | 否 | 父数据源 id |

### 返回参数

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.list | List\<NormalSourceTreeResponseDTO\> | 树形数据源列表 |
| data.list[].id | Long | 主键 |
| data.list[].projectId | Integer | 项目 id |
| data.list[].parentId | Long | 父节点 id |
| data.list[].name | String | 目录/实例表名称 |
| data.list[].type | Short | 类型 0 目录 1 实例表 |
| data.list[].createUser | String | 创建人 |
| data.list[].updateUser | String | 更新人 |
| data.list[].createTime | Long | 创建时间 |
| data.list[].updateTime | Long | 更新时间 |
| data.list[].childrenList | List\<NormalSourceTreeResponseDTO\> | 子数据源信息 |

### 调用链

```
NormalSourceController.getDataSourceTree
└─ NormalSourceService.getDataSourceTree
```

---

## 2. POST /v3/datasource/normal/sources — 分页查询数据源列表

### 入口

`NormalSourceController.getList(@Valid @RequestBody NormalSourceQueryDTO)`

### 请求参数（JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| page | Integer | 否 | 页码（默认 1） |
| pageSize | Integer | 否 | 每页条数（默认 10） |
| eid | Integer | 否 | 企业 ID（默认 1） |
| projectId | Integer | 是 | 项目 ID（@NotNull） |
| parentId | Long | 否 | 父级 id |
| name | String | 否 | 实例表名称 |
| varName | String | 否 | 变量名 |
| varValue | String | 否 | 变量值 |
| fuzzyByValue | Integer | 否 | 模糊查询变量值 0 不模糊 1 模糊 |
| varInfo | Object | 否 | 变量名=变量值查询条件 |
| varInfo.varName | String | 否 | 变量名 |
| varInfo.varValue | String | 否 | 变量值 |
| caseSensitive | Integer | 否 | 是否区分大小写（默认 0） |
| tagName | String | 否 | 标签名 |
| isPage | Integer | 否 | 是否分页 1 分页 0 不分页（默认 0） |

### 返回参数

`ResponseResult<Object>`，当 `isPage=0`（默认）返回 `ResultListDTO<NormalSourceResponseDTO>`，当 `isPage=1` 返回 `PageInfoList<NormalSourceResponseDTO>`。

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.list | List\<NormalSourceResponseDTO\> | 数据源列表 |
| data.list[].id | Long | 主键 |
| data.list[].projectId | Integer | 项目 id |
| data.list[].parentId | Long | 父节点 id |
| data.list[].name | String | 目录/实例表名称 |
| data.list[].type | Short | 类型 0 目录 1 实例表 |
| data.list[].createUser | String | 创建人 |
| data.list[].updateUser | String | 更新人 |
| data.list[].createTime | Long | 创建时间 |
| data.list[].updateTime | Long | 更新时间 |
| data.page | Integer | 页码（isPage=1 时返回） |
| data.pageSize | Integer | 每页条数（isPage=1 时返回） |
| data.totalRow | Long | 总记录数（isPage=1 时返回） |
| data.totalPage | Integer | 总页数（isPage=1 时返回） |

---

## 3. POST /v3/datasource/normal/source/add — 添加数据源

### 入口

`NormalSourceController.addSource(@Valid @RequestBody NormalSourceAddDTO)`

### 请求参数（JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 否 | 企业 ID（默认 1） |
| projectId | Integer | 是 | 项目 ID（@NotNull） |
| parentId | Long | 是 | 父节点 id（@NotNull） |
| name | String | 是 | 名称（@NotNull） |
| type | Short | 是 | 节点类型 0 目录 1 实例表（@NotNull） |
| userName | String | 是 | 用户名（@NotNull，openapi 自动传递） |

### 返回参数

`ResponseResult<NormalSourceResponseDTO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.id | Long | 新增记录主键 |
| data.projectId | Integer | 项目 id |
| data.parentId | Long | 父节点 id |
| data.name | String | 名称 |
| data.type | Short | 类型 0 目录 1 实例表 |
| data.createUser | String | 创建人 |
| data.updateUser | String | 更新人 |
| data.createTime | Long | 创建时间 |
| data.updateTime | Long | 更新时间 |

---

## 4. PUT /v3/datasource/normal/source/edit — 编辑数据源

### 入口

`NormalSourceController.editSource(@Valid @RequestBody NormalSourceOperateDTO)`

### 请求参数（JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 否 | 企业 ID |
| projectId | Integer | 是 | 项目 ID（@NotNull） |
| normalSourceId | Long | 是 | 存储数据源 id（@NotNull） |
| name | String | 是 | 存储数据源名称（代码 `StringUtils.isEmpty` 校验非空） |
| targetDirId | Long | 否 | 移动目标目录 id（编辑时不用） |
| userName | String | 是 | 用户名（@NotNull） |

### 返回参数

`ResponseResult<ResultDTO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.result | Boolean | 是否编辑成功 |

---

## 5. DELETE /v3/datasource/normal/source — 删除数据源

### 入口

`NormalSourceController.deleteSource(eid, project_id, source_id, user_name)`

### 请求参数（Query）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 否 | 企业 ID（默认 1） |
| project_id | Integer | 是 | 项目 ID（无默认值） |
| source_id | Long | 是 | 存储数据源 id（无默认值） |
| user_name | String | 是 | 用户名（无默认值） |

### 返回参数

`ResponseResult<ResultDTO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.result | Boolean | 是否删除成功（顶层目录返回 false） |

---

## 6. POST /v3/datasource/normal/source/move — 移动数据源

### 入口

`NormalSourceController.moveSource(@Valid @RequestBody NormalSourceOperateDTO)`

### 请求参数（JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 否 | 企业 ID（默认 1） |
| projectId | Integer | 是 | 项目 ID（@NotNull） |
| normalSourceId | Long | 是 | 存储数据源 id（@NotNull） |
| name | String | 否 | 存储数据源名称（移动时不用） |
| targetDirId | Long | 是 | 目标目录 id（代码 null 校验抛异常，实际必填） |
| userName | String | 是 | 用户名（@NotNull） |

### 返回参数

`ResponseResult<ResultDTO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.result | Boolean | 是否移动成功 |

---

## 7. POST /v3/datasource/normal/column/edit — 编辑列定义

### 入口

`NormalSourceController.editColumn(@Valid @RequestBody NormalColumnInfoDTO)`

### 请求参数（JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 否 | 企业 ID |
| projectId | Integer | 是 | 项目 ID（@NotNull） |
| normalSourceId | Long | 是 | 存储数据源 id（@NotNull） |
| varName | String | 是 | 变量名（@NotNull） |
| type | Short | 否 | 列类型（代码仅支持字符串 STRING，其它返回 false） |
| colIndex | Integer | 是 | 列号（@NotNull） |
| desc | String | 否 | 列备注 |
| userName | String | 是 | 用户名（@NotNull） |

### 返回参数

`ResponseResult<ResultDTO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.result | Boolean | 是否编辑成功 |

---

## 8. POST /v3/datasource/normal/value/edit — 编辑单元格值

### 入口

`NormalSourceController.editValue(@Valid @RequestBody NormalValueInfoDTO)`

### 请求参数（JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 否 | 企业 ID（默认 1） |
| projectId | Integer | 是 | 项目 ID（@NotNull） |
| normalSourceId | Long | 是 | 存储数据源 id（@NotNull） |
| values | List\<NormalValueInfoEditDTO\> | 否 | 编辑的多个单元格（空列表直接返回 true） |
| values[].colIndex | Integer | 是 | 列号（@NotNull） |
| values[].rowIndex | Integer | 是 | 行号（@NotNull） |
| values[].varValue | String | 否 | 单元格值 |
| values[].normalSourceId | Long | 是 | 存储数据源 id（@NotNull） |
| userName | String | 是 | 用户名（@NotNull） |

### 返回参数

`ResponseResult<ResultDTO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.result | Boolean | 是否编辑成功 |

---

## 9. GET /v3/datasource/normal/table/data — 获取表格数据

### 入口

`NormalSourceController.getTableData(eid, project_id, table_id)`

### 请求参数（Query）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 否 | 企业 ID（默认 1） |
| project_id | Integer | 是 | 项目 ID（无默认值） |
| table_id | Long | 是 | 存储数据源表格 id（无默认值） |

### 返回参数

`ResponseResult<Object>`，data 为 `NormalTableDataResponseDTO`。

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.colList | List\<NormalColumnInfoResponseDTO\> | 列信息列表 |
| data.colList[].id | Integer | 列 id |
| data.colList[].projectId | Integer | 项目 id |
| data.colList[].tableId | Long | 表格 id |
| data.colList[].varName | String | 列名 |
| data.colList[].type | Short | 列类型 |
| data.colList[].colIndex | Integer | 列号 |
| data.colList[].desc | String | 列备注 |
| data.rowList | List\<NormalValueInfoResponseDTO\> | 行信息列表 |
| data.rowList[].rowIndex | Integer | 行号 |
| data.rowList[].valueList | List\<NormalTableCellDTO\> | 每行的单元格值信息 |
| data.rowList[].valueList[].id | Long | 单元格 id |
| data.rowList[].valueList[].projectId | Integer | 项目 id |
| data.rowList[].valueList[].tableId | Long | 表格 id |
| data.rowList[].valueList[].varValue | String | 单元格值 |
| data.rowList[].valueList[].rowIndex | Integer | 行号 |
| data.rowList[].valueList[].colIndex | Integer | 列号 |

---

## 10. GET /v3/datasource/normal/table/data/format — 获取格式化表格数据

### 入口

`NormalSourceController.getFormatTableData(eid, project_id, table_id)`（无需登录）

### 请求参数（Query）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 否 | 企业 ID（默认 1） |
| project_id | Integer | 是 | 项目 ID（无默认值） |
| table_id | Long | 是 | 存储数据源表格 id（无默认值） |

### 返回参数

`ResponseResult<Object>`，data 为 `ResultListDTO<Map<String,String>>`（数组格式，每行以列名为键、单元格值为值）。

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.list | List\<Map\<String,String\>\> | 格式化后的数据行（键=列名，值=单元格值） |

---

## 11. POST /v3/datasource/normal/table/rows/delete — 删除行

### 入口

`NormalSourceController.removeRows(@Valid @RequestBody NormalRemoveRowOrColDTO)`

### 请求参数（JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 否 | 企业 ID（默认 1） |
| projectId | Integer | 是 | 项目 ID（@NotNull） |
| tableId | Long | 是 | 表格 id（@NotNull） |
| deleteRowList | List\<Integer\> | 否 | 需要删除的行号集合（空返回 true） |
| deleteColList | List\<Integer\> | 否 | 需要删除的列号集合（删除行时不用） |
| userName | String | 是 | 用户名（@NotNull） |

### 返回参数

`ResponseResult<ResultDTO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.result | Boolean | 是否删除成功 |

---

## 12. POST /v3/datasource/normal/table/columns/delete — 删除列

### 入口

`NormalSourceController.removeCols(@Valid @RequestBody NormalRemoveRowOrColDTO)`

### 请求参数（JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 否 | 企业 ID（默认 1） |
| projectId | Integer | 是 | 项目 ID（@NotNull） |
| tableId | Long | 是 | 表格 id（@NotNull） |
| deleteRowList | List\<Integer\> | 否 | 需要删除的行号集合（删除列时不用） |
| deleteColList | List\<Integer\> | 否 | 需要删除的列号集合（空返回 true） |
| userName | String | 是 | 用户名（@NotNull） |

### 返回参数

`ResponseResult<ResultDTO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.result | Boolean | 是否删除成功 |

---

## 13. POST /v3/datasource/normal/table/rows/add — 添加行

### 入口

`NormalSourceController.addRows(@Valid @RequestBody NormalAddColRowDTO)`

### 请求参数（JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 否 | 企业 ID（默认 1） |
| projectId | Integer | 是 | 项目 ID（@NotNull） |
| tableId | Long | 是 | 表格 id（@NotNull） |
| startOrder | Integer | 是 | 插入位置（0 插到最前，否则插入到该序号之后）（@NotNull） |
| count | Integer | 是 | 插入行数量（@NotNull，<=0 返回 false） |
| userName | String | 是 | 用户名（@NotNull） |

### 返回参数

`ResponseResult<ResultDTO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.result | Boolean | 是否添加成功 |

---

## 14. POST /v3/datasource/normal/table/columns/add — 添加列

### 入口

`NormalSourceController.addCols(@Valid @RequestBody NormalAddColRowDTO)`

### 请求参数（JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 否 | 企业 ID（默认 1） |
| projectId | Integer | 是 | 项目 ID（@NotNull） |
| tableId | Long | 是 | 表格 id（@NotNull） |
| startOrder | Integer | 是 | 插入位置（0 插到最前，否则插入到该序号之后）（@NotNull） |
| count | Integer | 是 | 插入列数量（@NotNull，<=0 返回 false） |
| userName | String | 是 | 用户名（@NotNull） |

### 返回参数

`ResponseResult<ResultDTO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.result | Boolean | 是否添加成功 |

---

## 15. POST /v3/datasource/normal/source/batch — 批量操作

### 入口

`NormalSourceController.batchOperate(@RequestBody @Valid NormalSourceBatchOperateDTO)`

### 请求参数（JSON Body，NormalSourceBatchOperateDTO extends NormalSourceQueryDTO）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| page | Integer | 否 | 页码 |
| pageSize | Integer | 否 | 每页条数 |
| eid | Integer | 否 | 企业 ID（默认 1） |
| projectId | Integer | 是 | 项目 ID（@NotNull） |
| parentId | Long | 否 | 父级 id |
| name | String | 否 | 实例表名称 |
| varName | String | 否 | 变量名 |
| varValue | String | 否 | 变量值 |
| fuzzyByValue | Integer | 否 | 模糊查询变量值 |
| varInfo | Object | 否 | 变量名=变量值查询条件 |
| caseSensitive | Integer | 否 | 是否区分大小写 |
| tagName | String | 否 | 标签名 |
| isPage | Integer | 否 | 是否分页 |
| selectAllFlag | Integer | 否 | 是否全选 1 全选 0 不全选（默认 1） |
| dataSourceIds | List\<Long\> | 否 | 筛选出的 id 列表（不全选时需传） |
| batchType | Integer | 是 | 批量操作类型 1 批量替换 2 批量删除（@NotNull） |
| replaceParam | Object | 否 | 批量替换参数（批量替换时必填，代码校验非空） |
| replaceParam.newVarValue | String | 是 | 替换后的新变量值（@NotNull） |
| replaceParam.caseSensitive | Integer | 否 | 是否区分大小写（默认 0） |
| replaceParam.varName | String | 否 | 变量名 |
| replaceParam.varValue | String | 否 | 需要替换的变量值 |
| userName | String | 是 | 用户名（@NotNull） |

### 返回参数

`ResponseResult<BatchOperateResponseDTO>`。批量替换返回替换的单元格数量，批量删除返回删除的实例表数量。

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.successCount | Integer | 操作成功数量 |
| data.failCount | Integer | 操作失败数量 |
