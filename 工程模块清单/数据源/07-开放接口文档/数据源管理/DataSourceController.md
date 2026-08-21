# DataSourceController — 数据源主控制器

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/controller/DataSourceController.java`
> 类级路由：`/datasource`（完整前缀 `/openapi/v3/datasource`）
> Service 实现：`SourceConfigService`、`ColConfigService`、`DatatableValuesService`
> 业务：数据源综合查询（按脚本条件/树/配置/执行概要）、标签操作（标记造数失败/通用标签）、脚本参数获取、批量移动。

## 接口列表总表

| # | 方法 | 路径 | 方法名 | 说明 |
|---|------|------|--------|------|
| 1 | POST | `/v3/datasource/datasources/scripts` | getDataSourceByScriptCondition | 按脚本条件查询数据源（app处理服务 消费） |
| 2 | POST | `/v3/datasource/executive_summaries` | getExecutiveSummaries | 获取报告执行摘要数据 |
| 3 | GET | `/v3/datasource/datasources` | getDataSourceTree | 获取数据源树 |
| 4 | POST | `/v3/datasource/source_configs` | getDataSourceConfig | 按条件分页查询数据源配置 |
| 5 | POST | `/v3/datasource/tag/operate` | operateTag | 标记造数失败（国金证券定制） |
| 6 | POST | `/v3/datasource/common/tag/operate` | CommonOperateTag | 通用标签操作 |
| 7 | POST | `/v3/datasource/getScriptParamDataNew` | getScriptParamDataNew | 提测业务获取数据源数据 |
| 8 | POST | `/v3/datasource/source/move` | batchMove | 批量移动数据源 |

统一响应包装：`ResponseResult<T> { int code; String msg; T data }`。
GET 查询接口带 `@UnderlineToCamel`：下划线 query 参数自动转驼峰绑定 DTO。

涉及表：`source_config`、`datatable_col_config`、`datatable_values`、`datatable_tag_config`、`tag_info`。

---

## 1. POST /v3/datasource/datasources/scripts — 按脚本条件查询数据源

### 入口

`DataSourceController.getDataSourceByScriptCondition(@RequestBody @Valid DataSourceByScriptRequestDTO request)`

### 请求参数（DataSourceByScriptRequestDTO，JSON Body，@Valid）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectId | Integer | 是 | 项目 ID（@NotNull） |
| eid | Integer | 否 | 企业 ID |
| uid | Integer | 否 | 用户 ID |
| userName | String | 否 | 用户名 |
| scriptNos | List\<Integer\> | 否 | 脚本编号列表（与 condition.scriptType 至少一项非空，否则抛「参数错误」） |
| condition | ScriptConditionDTO | 否 | 脚本条件（缺省自动 new 空对象） |
| condition.scriptType | Integer | 否 | 脚本类型（scriptNos 为空时必填） |
| condition.scriptName | String | 否 | 脚本名称 |
| condition.dirId | Integer | 否 | 目录 id |
| condition.scriptTag | String | 否 | 脚本标签 |
| condition.updateUserIds | List\<Integer\> | 否 | 更新人 id 组合（or 关系） |
| condition.designUserIds | List\<Integer\> | 否 | 设计人 id 组合（or 关系） |
| condition.scriptNo | Integer | 否 | 脚本编号 |
| condition.suiteId | Integer | 否 | 应用 ID（脚本类型 1 生效） |
| condition.checkStatus | Integer | 否 | 检查状态 1 有效 0 无效 |
| condition.unassigned | Integer | 否 | 是否未分配目录 |

### 返回参数

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.list | List\<DataSourceResponseBaseDTO\> | 数据源基础信息列表 |
| data.list[].dataSourceName | String | 数据源名称 |
| data.list[].id | Integer | 数据源 id |

### 实现意图

核心消费接口。app处理服务 执行脚本时通过此接口批量获取脚本关联的数据源变量信息。校验：scriptNos 为空且 condition.scriptType 也为空时抛参数错误。

### 调用链

```
DataSourceController.getDataSourceByScriptCondition
├─ 参数校验：scriptNos 非空 或 scriptType 非空
└─ SourceConfigService.getDataSourceByScriptCondition
```

---

## 2. POST /v3/datasource/executive_summaries — 报告执行摘要数据

### 入口

`DataSourceController.getExecutiveSummaries(@RequestBody @Valid ExecSummaryDataRequestDTO request)`

### 请求参数（ExecSummaryDataRequestDTO，JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| scriptNos | List\<Integer\> | 是 | 脚本编号列表（@NotEmpty） |
| sourceId | Long | 是 | 数据源 id（@NotNull） |
| tagList | List\<Integer\> | 否 | 提测选择的包含 tag 的 id |

### 返回参数

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.list | List\<ExecSummaryDataShowInReportDTO\> | 执行摘要数据列表（仅 show_in_report=1 的列） |
| data.list[].scriptNo | Integer | 脚本编号 |
| data.list[].dataSourceId | Long | 数据源 id |
| data.list[].sourceConfigId | Long | 全局表/实例表 id |
| data.list[].colConfigs | List\<ColConfigShowInReportDTO\> | 列配置项（每项：name、desc） |
| data.list[].tags | Map\<Integer,String\> | 标签信息（key=row_index，value=标签，多标签逗号分隔） |

### 实现意图

根据数据表 ID 列表查询标记为"报告可见"的列和对应值，用于测试报告中的执行摘要展示。

### 调用链

```
DataSourceController.getExecutiveSummaries
└─ ColConfigService.getExecutiveSummaries
```

---

## 3. GET /v3/datasource/datasources — 获取数据源树

### 入口

`DataSourceController.getDataSourceTree(@UnderlineToCamel @Valid DataSourceTreeRequestDTO request)`

### 请求参数（Query，@UnderlineToCamel）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| project_id | Integer | 是 | 项目 ID（@NotNull，下划线自动转驼峰 projectId） |
| eid | Integer | 否 | 企业 ID |
| uid | Integer | 否 | 用户 ID |
| user_id | Integer | 否 | 用户 ID |
| user_name | String | 否 | 用户名 |
| lazy_tree | Integer | 否 | 是否懒加载：0 懒加载 1 加载子目录 |
| parent_id | Long | 否 | 父数据源 id |

### 返回参数

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.list | List\<DataSourceTreeResponseDTO\> | 树形数据源列表 |
| data.list[].createTime | Long | 创建时间 |
| data.list[].updateTime | Long | 更新时间 |
| data.list[].id | Long | 数据源 id |
| data.list[].dataSourceName | String | 数据源名称 |
| data.list[].parentId | Long | 父节点 id |
| data.list[].type | Integer | 数据源类型（SourceTypeEnum） |
| data.list[].updateName | String | 更新用户 |
| data.list[].scriptNo | Integer | 关联的脚本 id |
| data.list[].children | List\<DataSourceTreeResponseDTO\> | 子数据源信息 |

### 实现意图

按条件查询数据源树结构，返回层级化的数据源目录/表/SQL 节点。

### 调用链

```
DataSourceController.getDataSourceTree
└─ SourceConfigService.getDataSourceTree
```

---

## 4. POST /v3/datasource/source_configs — 分页查询数据源配置

### 入口

`DataSourceController.getDataSourceConfig(@RequestBody @Valid DataSourceRequestDTO request)`

### 请求参数（DataSourceRequestDTO，JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectId | Integer | 是 | 项目 ID（@NotNull） |
| eid | Integer | 否 | 企业 ID |
| uid | Integer | 否 | 用户 ID |
| userName | String | 否 | 用户名 |
| page | Integer | 否 | 页码（缺省 DEFAULT_PAGE） |
| pageSize | Integer | 否 | 每页条数（缺省 DEFAULT_PAGE_SIZE） |
| scriptNos | List\<Integer\> | 否 | 脚本编号列表 |
| types | List\<Integer\> | 否 | 数据源类型集合 |
| sourceIds | List\<Integer\> | 否 | 数据源 id 集合 |
| tagIds | List\<Integer\> | 否 | tagId 集合 |
| tagName | String | 否 | tag 名称 |
| parentSourceId | Integer | 否 | 父数据源 id |
| caseSensitive | Integer | 否 | 是否区分大小写 |
| varName | String | 否 | 变量名 |
| varValue | String | 否 | 变量值 |
| operation | Integer | 否 | varName/varValue 匹配操作（0 无关系 1 同变量匹配） |
| name | String | 否 | 数据源名称 |

### 返回参数

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.totalRow | Long | 总记录数 |
| data.totalPage | Integer | 总页数 |
| data.pageSize | Integer | 每页条数 |
| data.page | Integer | 页码 |
| data.list | List\<DataSourceTreeResponseDTO\> | 数据源列表（字段见「获取数据源树」data.list 项） |

### 调用链

```
DataSourceController.getDataSourceConfig
└─ SourceConfigService.getDataSourceConfig
```

---

## 5. POST /v3/datasource/tag/operate — 标记造数失败（国金证券定制）

### 入口

`DataSourceController.operateTag(@RequestBody TagOperateDTO request)`

### 请求参数（TagOperateDTO，JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| name | String | 否 | 变量名（空字符串 = 清除所有造数失败标签） |
| values | List\<String\> | 否 | 变量值列表（标记这些值所在的行） |
| extNameList | Set\<String\> | 否 | 变量名扩展（多个变量代表同一含义） |
| projectIdList | List\<Integer\> | 否 | 项目 id 集合 |
| tag | String | 否 | 标签名（此接口不需要传，默认「造数失败」） |
| userName | String | 否 | 用户名 |

### 返回参数

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.result | Integer | 操作影响记录数（1 成功 0 失败） |

### 实现意图

国金证券定制功能。调用顺序：
1. `{"name":""}` 先清除所有"造数失败"标签
2. `{"name":"fund_account","values":["1000000"]}` 标记指定变量
3. `{"name":"stock_code","values":["11111"]}` 标记另一个变量
以上循环可重复，但每次打标签前必须调用步骤 1 清理。

### 调用链

```
DataSourceController.operateTag
└─ SourceConfigService.operateTagForNumberGenerationFailed
```

---

## 6. POST /v3/datasource/common/tag/operate — 通用标签操作

### 入口

`DataSourceController.CommonOperateTag(@RequestBody TagOperateDTO request)`

### 请求参数（TagOperateDTO，JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| name | String | 否 | 变量名（空 = 清除指定标签） |
| tag | String | 否 | 标签名 |
| values | List\<String\> | 否 | 变量值列表 |
| extNameList | Set\<String\> | 否 | 变量名扩展集合 |
| projectIdList | List\<Integer\> | 否 | 项目 id 集合 |
| userName | String | 否 | 用户名 |

### 返回参数

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.result | Integer | 操作影响记录数（1 成功 0 失败） |

### 实现意图

通用版本的标签操作，支持指定任意标签名称（而非固定的"造数失败"标签）。同样遵循先清除后标记的调用顺序。

### 调用链

```
DataSourceController.CommonOperateTag
└─ SourceConfigService.operateTag
```

---

## 7. POST /v3/datasource/getScriptParamDataNew — 提测业务获取数据源数据

### 入口

`DataSourceController.getScriptParamDataNew(@RequestBody SourceConfigDTO request)`

### 请求参数（SourceConfigDTO，JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| sourceConfigId | Long | 否 | 数据表 ID（无 @Valid/判空，全 DTO 非必填） |
| scriptNo | Integer | 否 | 脚本编号 |
| eid | Integer | 否 | 企业 id |
| projectid | Integer | 否 | 项目 id |
| sourceId | Long | 否 | 数据源 id |
| type | Integer | 否 | 类型 |
| tagList | List\<Integer\> | 否 | 包含标签 id 列表 |
| noHasTagList | List\<Integer\> | 否 | 排除标签 id 列表 |
| needAllTag / needGlobal | Integer | 否 | 标签/全局标记 |
| （其余 SourceConfigDTO 字段） | — | 否 | name/colName/paramValue/fuzzyByValue/caseSensitive/conditions 等 |

### 返回参数

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data | ScriptParamSourceData | 脚本参数源数据 |
| data.global | Map\<String,ScriptCellParamData\> | 全局参数（value 含 id/paramName/paramValue/paramDesc/scriptNo/colId/rowId 等） |
| data.local | Map\<Integer,List\<List\<ScriptCellParamData\>\>\> | 局部参数（按脚本分组） |
| data.localRowWithTagInfo | Map\<Integer,List\<List\<TagInfo\>\>\> | 局部行标签信息（TagInfo：id/eid/projectId/name） |

### 实现意图

为提测业务提供完整的数据源数据（变量定义 + 值 + 标签），对标 ApiServlet 入口的 `DataTableCtrl.getScriptParamData`，这是 MVC 版本。

### 调用链

```
DataSourceController.getScriptParamDataNew
└─ DatatableValuesService.getScriptParamDataNew
```

---

## 8. POST /v3/datasource/source/move — 批量移动数据源

### 入口

`DataSourceController.batchMove(@RequestBody @Valid DataSourceMoveRequestDTO request)`

### 请求参数（DataSourceMoveRequestDTO，JSON Body，@Valid）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| newParentId | Long | 是 | 目标父节点 ID（代码判空且 < 1 抛「要移入的父级ID错误」） |
| dataSourceType | Integer | 是 | 数据源类型（代码校验 DataSourceTypeEnum.checkDataSourceType，非法抛「数据源类型错误」） |
| sourceIds | List\<Long\> | 否 | 要移动的数据源 ID 列表 |
| projectId | Integer | 是 | 项目 ID（@NotNull） |
| eid | Integer | 否 | 企业 ID |
| uid | Integer | 否 | 用户 ID |
| userName | String | 否 | 用户名 |
| condition | SelectDTO | 否 | 筛选条件（name/colName/paramValue/tagName/caseSensitive 等） |

### 返回参数

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.result | Integer | 移动成功的记录数 |

### 实现意图

批量将选中的数据源节点移动到新的父目录下。校验：newParentId 必须有效、dataSourceType 必须合法。

### 调用链

```
DataSourceController.batchMove
├─ 参数校验：newParentId > 0、dataSourceType 合法
└─ SourceConfigService.batchMoveDataSource
```
