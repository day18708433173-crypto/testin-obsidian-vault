# service-SelectCtrl — 数据源全局检索（ApiServlet）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/service/source/SelectCtrl.java`
> 类级路由：`/source/SelectCtrl`（完整前缀 `/openapi/source/SelectCtrl`）
> 基类：`GenericBaseService`
> 业务：四种维度的全平台数据检索 — 按目录树、按变量名、按变量值、按标签名。

## 方法列表总表

| # | 路径 | 方法名 | 说明 |
|---|------|--------|------|
| 1 | `/selectSourceConfig` | selectSourceConfig | 按目录树搜索（名称/变量名/值/标签综合） |
| 2 | `/selectByVariableName` | selectByVariableName | 按变量名搜索 |
| 3 | `/selectByParamValue` | selectByParamValue | 按变量值搜索 |
| 4 | `/selectByTagName` | selectByTagName | 按标签名搜索 |

所有方法入参：`SelectDTO`（通过 reqjson 反序列化），返回 `IPage<T>` 分页结果。

涉及表：`source_config`、`datatable_col_config`、`datatable_values`、`datatable_tag_config`、`tag_info`。

---

## 方法详解

### 1. POST /selectSourceConfig — 按目录树搜索

综合搜索：可按 `sourceId`、`parentId`、`name`、`colName`、`paramValue`、`tagName` 等多维度过滤。支持 `fuzzyByValue` 控制值字段是否模糊匹配。

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| sourceId / parentId / name / colName / paramValue / tagName / caseSensitive | — | 否 | 搜索条件（SelectDTO） |
| fuzzyByValue | Integer | 否 | 0 精准 1 模糊（默认 0） |
| current / size | Integer | 否 | 分页 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.list | List\<SourceConfigVO\> | 目录视图列表 |
| data.list[].childrenList | List\<SourceConfigVO\> | 子节点列表 |
| data.list[].repeatScriptNoList | Set\<Integer\> | 重复编号列表 |
| data.list[].sourceId | Long | 数据源 id |
| data.list[].globalId | Long | 全局表 id |
| data.list[].excelList | List\<ExcelVO\> | excel 信息（url/fileName） |
| data.list[].urlList | List\<String\> | url 列表 |
| data.list[].url | String | 下载地址 |
| data.list[].source | SourceConfig | 所属数据源 |
| data.list[].repetitiveMap | Map\<String,Object\> | 旧脚本编号→多个新编号集合 |
| data.list[].tableData | DataTableVO | 全局表/实例表表格数据 |
| data.list[].（SourceConfig 字段） | — | id/eid/projectId/sourceId/parentId/name/type/sqlId/bind/scriptNo/scriptType/envId/dbAlias/dbConfigId/currentOrder/updateBy/status |
| data.page | Integer | 页码 |
| data.pageSize | Integer | 每页条数 |
| data.totalRow | Long | 总记录数 |
| data.totalPage | Long | 总页数 |

- Service：`SourceConfigService.selectPage(SelectDTO)` → `IPage<SourceConfigVO>`

### 2. POST /selectByVariableName — 按变量名搜索

在列配置中搜索变量名匹配的记录。

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| name | String | 否 | 变量名 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.list | List\<SelectVO\> | 全局搜索视图 |
| data.list[].sourceConfigName | String | 数据源名称 |
| data.list[].paramNameList | List\<String\> | 返回值列表 |
| data.list[].sourceConfig | SourceConfig | 当前目录信息 |
| data.list[].（ColConfig 字段） | — | id/eid/projectId/sourceConfigId/name/type/quoteType/descr/scope/colIndex/showInReport/sqlCol/updateBy |
| data.page | Integer | 页码 |
| data.pageSize | Integer | 每页条数 |
| data.totalRow | Long | 总记录数 |
| data.totalPage | Long | 总页数 |

- Service：`ColConfigService.selectByVariableName(SelectDTO)` → `IPage<SelectVO>`

### 3. POST /selectByParamValue — 按变量值搜索

在数据表值中搜索值匹配的记录。

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| name | String | 否 | 变量值 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.list | List\<SelectVO\> | 全局搜索视图（同 selectByVariableName：sourceConfigName/paramNameList/sourceConfig + ColConfig 字段） |
| data.page | Integer | 页码 |
| data.pageSize | Integer | 每页条数 |
| data.totalRow | Long | 总记录数 |
| data.totalPage | Long | 总页数 |

- Service：`DatatableValuesService.selectByParamValue(SelectDTO)` → `IPage<SelectVO>`

### 4. POST /selectByTagName — 按标签名搜索

查找被打上指定标签的数据行。

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| name | String | 否 | 标签名 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.list | List\<SelectVO\> | 全局搜索视图（同 selectByVariableName：sourceConfigName/paramNameList/sourceConfig + ColConfig 字段） |
| data.page | Integer | 页码 |
| data.pageSize | Integer | 每页条数 |
| data.totalRow | Long | 总记录数 |
| data.totalPage | Long | 总页数 |

- Service：`DatatableValuesService.selectByTagName(SelectDTO)` → `IPage<SelectVO>`

## 通用模式

```java
@PostMapping("/xxx")
public String xxx(ApiRequest apiRequest) throws Exception {
    JSONObject reqJson = apiRequest.getReqjson();
    IPage<XXX> result = null;
    try {
        SelectDTO dto = objectMapper.readValue(reqJson.toString(), SelectDTO.class);
        result = xxxService.xxx(dto);
    } catch (Exception e) {
        Logit.errorLog(e.getMessage(), new Throwable(e));
        return ApiUtil.getResult(apiRequest, CommonCode.execFailed.getValue(), e.getMessage());
    }
    return packageResult(apiRequest, result);
}
```
