# service-SourceConfigCtrl — 数据源配置管理（ApiServlet）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/service/source/SourceConfigCtrl.java`
> 类级路由：`/source/SourceConfigCtrl`（完整前缀 `/openapi/source/SourceConfigCtrl`）
> 基类：`GenericBaseService`
> 业务：数据源配置的完整 CRUD + 树操作 + 导入导出 + ES 同步 + 批量复制（~20 方法，最复杂的 ApiServlet 类）。

## 方法列表总表

| # | 路径 | 方法名 | 说明 |
|---|------|--------|------|
| 1 | `/add` | add | 新增数据源节点（目录/数据源/表/SQL） |
| 2 | `/updateById` | updateById | 按 ID 更新 |
| 3 | `/getTree` | getTree | 获取数据源树 |
| 4 | `/delete` | delete | 逻辑删除节点（批量） |
| 5 | — | batchDeleteInstanceTable | 批量删除实例表 |
| 6 | `/moveTo` | moveTo | 移动节点 |
| 7 | `/copyTo` | copyTo | 复制节点 |
| 8 | `/selectScriptNoList` | selectScriptNoList | 查询脚本编号列表 |
| 9 | `/selectUpperId` | selectUpperId | 获取父节点 ID |
| 10 | `/getById` | getById | 按 ID 获取 |
| 11 | `/selectSourceConfigByScriptNo` | selectSourceConfigByScriptNo | 按脚本编号查询数据源配置 |
| 12 | `/select` | select | 综合查询 |
| 13 | `/selectSourceByScriptNo` | selectSourceByScriptNo | 按脚本编号查询数据源列表 |
| 14 | `/selectSourceConfig` | selectSourceConfig | 查询数据源配置列表 |
| 15 | `/bindDefaultSource` | bindDefaultSource | 绑定默认数据源 |
| 16 | `/bindOtherSource` | bindOtherSource | 绑定其他数据源 |
| 17 | `/historyDataMigration` | historyDataMigration | 历史数据迁移 |
| 18 | `/selectInfoByScriptNo` | selectInfoByScriptNo | 按脚本编号查询详细信息 |
| 19 | `/syncEs` | syncEs | 同步 Elasticsearch |
| 20 | `/importVerify` | importVerify | 导入校验 |
| 21 | `/syncSourceName` | syncSourceName | 同步数据源名称 |
| 22 | — | getDataSourceBasicInfoList | 获取数据源基础信息列表 |
| 23 | — | getExportSourceTreeNode | 获取导出数据源树节点 |
| 24 | `/getSourceConfigByName` | getSourceConfigByName | 按名称查询数据源配置 |
| 25 | `/importDataSourceInfo` | importDataSourceInfo | 导入数据源信息 |
| 26 | `/batchCopyDataTableByScriptNos` | batchCopyDataTableByScriptNos | 按脚本编号批量复制数据表 |

所有方法签名：`String methodName(ApiRequest apiRequest)`，返回 JSON 字符串。

涉及表：`source_config`、`datatable_config`、`datatable_col_config`、`datatable_values`、`datatable_tag_config`、`source_sql`。

---

## 关键方法详解

### 1. POST /add — 新增节点

**请求参数（reqjson）**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| name | String | 是 | 名称 |
| type | Integer | 是 | 节点类型（0=目录 1=数据源 2=全局表 3=实例表 4=SQL管理 5=SQL） |
| parentId | Long | 是 | 父节点 ID（根节点传 0） |
| sourceId | Long | 是 | 所属数据源 ID（type=1 时先传 0） |
| projectid | Integer | 否 | 项目 ID |
| eid | Integer | 否 | 企业 ID |
| sqlId | Long | 否 | 关联 SQL id |
| scriptNo | Integer | 否 | 脚本编号 |
| scriptType | Integer | 否 | 脚本类型 |
| envId / dbAlias / dbConfigId | Integer/String/Integer | 否 | 数据库/环境相关 |
| updateBy | String | 否 | 更新人 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | SourceConfig | 新增后的节点实体（id/eid/projectId/sourceId/parentId/name/type/sqlId/bind/scriptNo/scriptType/envId/dbAlias/dbConfigId/currentOrder/updateBy/createtime/updatetime/status） |

**校验规则**：
- type=1（数据源）时 sourceId 暂设为 0L（新增时会修正）
- parentId=0 时 type 必须为 1（数据源），即根目录下不能创建非数据源的目录
- 必传字段缺失时抛 "参数不合法"

**异常处理**：捕获 `unique_scriptno` 唯一约束冲突 → 返回 "存在重复脚本编号，请刷新页面后重试"

### 2. POST /updateById — 按 ID 更新

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| id | Long | 是 | 节点 id |
| name | String | 是 | 名称 |
| type / parentId / sourceId 等 | — | 否 | 其余 SourceConfig 字段按需传 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | Boolean | 是否更新成功 |

### 3. POST /getTree — 获取数据源树

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| lazy | String | 否 | 0 懒加载，1 一次性加载全部 |
| eid | Integer | 否 | 企业 id |
| projectid | Integer | 否 | 项目 id |
| parentId / type / treeType | Long/Integer/String | 否 | 树过滤条件 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.list | List\<SourceConfigVO\> | 目录树节点（含 childrenList 子节点、sourceId、globalId、tableData 等） |

### 4. POST /delete — 删除节点

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| id | Long | 是 | 节点 id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | Boolean | 是否删除成功 |

**入参**：`ids`（JSONArray，要删除的 ID 列表）

逻辑删除（status→0），级联删除子节点（通过 `parent_id` 递归查找）。

### 5. batchDeleteInstanceTable — 批量删除实例表

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| condition | JSONObject | 否 | 查询条件（OperationVarValueDTO.condition，SelectDTO 结构：name/scriptNo/typeList/eid/projectid/colName/paramValue/fuzzyByValue/tagName/caseSensitive/sourceId/parentId/conditions/selectAllFlag/dataSourceIds） |
| replaceParam | JSONObject | 否 | 替换参数（ReplaceVarValueDTO：conditions/newVarValue/caseSensitive/updateTime） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.successCount | Integer | 成功数量 |
| data.failCount | Integer | 失败数量 |

### 6. POST /moveTo — 移动节点

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| id | Long | 是 | 节点 id |
| targetId | Long | 是 | 目标 id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | Boolean | 是否移动成功 |

### 7. POST /copyTo — 复制节点

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| id | Long | 是 | 节点 id |
| targetId | Long | 是 | 目标 id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | Boolean | 是否复制成功 |

### 8. POST /selectScriptNoList — 查询脚本编号列表

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| scriptNoList | List\<Integer\> | 否 | 脚本编号列表 |
| sourceId | Long | 否 | 数据源 id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | SourceConfigVO | 查询结果（含 repeatScriptNoList 重复编号） |

### 9. POST /selectUpperId — 获取父节点 ID

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| sourceConfigId | Long | 是 | 实例表 id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | SourceConfigVO | 含 sourceId / globalId |

### 10. POST /getById — 按 ID 获取

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| id | Long | 是 | 节点 id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.objInfo | SourceConfig | 节点实体 |

### 11. POST /selectSourceConfigByScriptNo — 按脚本编号查询（已废弃）

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| scriptNo | Integer | 否 | 脚本编号 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | SourceConfig | 数据源配置实体 |

### 12. POST /select — 综合查询

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| scriptNo | Integer | 是 | 脚本编号（validateParams 判空） |
| name | String | 是 | 名称（validateParams 判空） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.list | List\<SourceConfig\> | 节点列表 |

### 13. POST /selectSourceByScriptNo — 按脚本编号查询数据源列表

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| scriptNo | Integer | 是 | 脚本编号 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.list | List\<SourceConfigVO\> | 数据源列表 |

### 14. POST /selectSourceConfig — 查询数据源配置列表

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectid | Integer | 否 | 项目 id |
| type | Integer | 否 | 类型 |
| scriptNo | Integer | 否 | 脚本编号 |
| id | Integer | 否 | 节点 id |
| sourceId | Integer | 否 | 数据源 id |
| sourceIdList | JSONArray | 否 | 数据源 id 列表 |
| tagList | JSONArray | 否 | 包含标签 id 列表 |
| noHasTagList | JSONArray | 否 | 排除标签 id 列表 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.result | List\<SourceConfig\> | 数据源列表 |

### 15. POST /bindDefaultSource — 绑定默认数据源

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| scriptNo | Integer | 是 | 脚本编号 |
| sourceConfigId | Long | 是 | 实例表 id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | Boolean | 是否绑定成功 |

### 16. POST /bindOtherSource — 绑定其他数据源

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| scriptNo | Integer | 是 | 脚本编号 |
| targetId | Long | 是 | 目标文件夹 id |
| sourceConfigId | Long | 是 | 原实例表 id |
| updateBy | String | 是 | 更新人 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | Boolean | 是否绑定成功 |

### 17. POST /historyDataMigration — 历史数据迁移

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| — | — | — | 无业务参数 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | Boolean | 是否迁移成功 |

### 18. POST /selectInfoByScriptNo — 按脚本编号查询详细信息

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| scriptNo | Integer | 是 | 脚本编号 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | SourceConfigVO | 含 sourceConfigId/globalId |

### 19. POST /syncEs — 同步 ES

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| — | — | — | 无业务参数 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | Boolean | 是否同步成功 |

将所有 `source_config` 数据同步到 Elasticsearch 索引 `es_datasource`，用于全文检索。

### 20. POST /importVerify — 导入校验

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| url | String | 是 | 文件地址 |
| sourceConfigId | Long | 是 | 导入目标文件夹 id |
| updateBy | String | 是 | 更新人 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | SourceConfigVO | 校验结果 |
| data.data.urlList | List\<String\> | 下载 url 列表 |
| data.data.excelList | List\<ExcelVO\> | excel 信息列表（每项：url、fileName） |
| data.data.repetitiveMap | Map\<String,Object\> | 旧脚本编号→多个新编号集合 |
| data.data.（SourceConfig 字段） | — | id/eid/projectId/sourceId/parentId/name/type/scriptNo/scriptType 等 |

校验导入文件的格式和合法性（变量名冲突检测等），返回校验结果。

### 21. POST /syncSourceName — 同步数据源名称

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| scriptNo | Integer | 是 | 脚本编号 |
| scriptDesc | String | 是 | 脚本描述 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | Boolean | 是否同步成功 |

### 22. getDataSourceBasicInfoList — 获取数据源基础信息列表

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| （reqjson 条件） | JSONObject | 否 | 查询条件（代码未确认具体字段，透传 reqJson 到 service） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.list | List\<DataSourceResponseBaseDTO\> | 列表项含 dataSourceName(String)、id(Integer) |

### 23. getExportSourceTreeNode — 获取导出数据源树节点

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| （SourceConfigDTO 字段） | — | 否 | 如 projectid/scriptNoList 等 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.list | List\<SourceConfigVO\> | 树节点列表 |

### 24. POST /getSourceConfigByName — 按名称查询数据源配置

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业 id |
| projectId | Integer | 是 | 项目 id |
| name | String | 是 | 名称 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.list | List\<String\> | 数据源名称列表 |

### 25. POST /importDataSourceInfo — 执行导入

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 否 | 企业 id |
| projectId | Integer | 是 | 项目 id |
| dataSourceFileInfoList | List\<DataSourceFileInfoDTO\> | 否 | 导入文件信息（type/name/filePath/nextFile/scriptNo/newScriptNo/scriptName/fileUrl） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | Boolean | 是否导入成功（true） |

遍历解析后的节点树，使用策略模式按 type 分发：
- type=0 → DirectoryStrategy
- type=2 → GlobalSourceStrategy
- type=3 → InstanceStrategy

导入后调用 `syncEs` 同步 Elasticsearch。

### 26. POST /batchCopyDataTableByScriptNos — 批量复制

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| scriptNoMap | Map\<Integer,Integer\> | 否 | 新老脚本编号映射（老→新） |
| projectId | Integer | 否 | 项目 id |
| targetProjectId | Integer | 否 | 目标项目 id |
| eid | Integer | 否 | 企业 id |
| copyType | Integer | 否 | 1 复制数据源 2 复制实例表 |
| userName | String | 否 | 用户名 |
| scriptCreateDescMap | Map\<Integer,String\> | 否 | 新脚本 id 与描述映射 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | Boolean | 是否复制成功 |

按脚本编号列表批量复制数据表（含列定义 + 值 + 标签）到目标位置。

## 通用模式

所有方法遵循统一模式：

```java
@PostMapping("/xxx")
public String xxx(ApiRequest apiRequest) {
    JSONObject reqJson = apiRequest.getReqjson();
    try {
        // 1. 从 reqJson 反序列化 DTO
        // 2. 参数校验
        // 3. 调用 service 层
        // 4. return packageResult(apiRequest, result)
    } catch (Exception e) {
        Logit.errorLog(e.getMessage(), new Throwable(e));
        return ApiUtil.getResult(apiRequest, CommonCode.execFailed.getValue(), e.getMessage());
    }
}
```
