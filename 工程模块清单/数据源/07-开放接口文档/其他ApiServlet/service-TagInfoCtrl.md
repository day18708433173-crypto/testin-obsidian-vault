# service-TagInfoCtrl — 标签管理（ApiServlet）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/service/source/TagInfoCtrl.java`
> 类级路由：`/source/TagInfoCtrl`（完整前缀 `/openapi/source/TagInfoCtrl`）
> 基类：`GenericBaseService`
> 业务：标签的批量新增/查询/删除，以及数据源下标签汇总查询。

## 方法列表总表

| # | 路径 | 方法名 | 说明 |
|---|------|--------|------|
| 1 | `/addList` | addList | 批量新增标签 |
| 2 | `/selectAll` | selectAll | 查询所有标签（支持条件过滤） |
| 3 | `/deleteList` | deleteList | 批量删除标签 |
| 4 | `/selectSourceTags` | selectSourceTags | 查询数据源下所有用到的标签 |

涉及表：`tag_info`、`source_config`、`datatable_tag_config`。

---

## 关键方法详解

### 1. POST /addList — 批量新增标签

**入参**：`TagInfoDTO`（含 `tagList` 标签信息列表）

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| tagList | List\<TagInfo\> | 否 | 标签列表（id/name/eid/projectId） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | Boolean | 是否新增成功 |

批量插入标签记录到 `tag_info`。

### 2. POST /selectAll — 查询所有标签

**入参**：`TagInfoDTO`（条件过滤，无分页）

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid / projectid / name | — | 否 | 过滤条件（TagInfoDTO） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.list | List\<TagInfoVO\> | 标签列表 |
| data.list[].id | Integer | 标签 id |
| data.list[].eid | Integer | 企业 id |
| data.list[].projectId | Integer | 项目 id |
| data.list[].name | String | 标签名字 |
| data.list[].status | Integer | 状态 1 正常 0 删除 |
| data.list[].createtime / updatetime | Long | 创建/更新时间 |

**返回**：`List<TagInfoVO>`

### 3. POST /deleteList — 批量删除标签

**入参**：`TagInfoDTO`（含 `deleteIdList` 要删除的标签 ID 列表）

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| deleteIdList | List\<Integer\> | 否 | 删除 id 列表 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | Boolean | 是否删除成功 |

### 4. POST /selectSourceTags — 查询数据源标签汇总

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectid | Integer | 否 | 项目 id |
| type | Integer | 否 | 类型 |
| sourceId | Integer | 否 | 数据源 id |
| scriptNos | JSONArray | 否 | 脚本编号列表 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.list | List\<TagInfoVO\> | 标签列表（每项字段同 selectAll：id/eid/projectId/name/status/createtime/updatetime） |

**实现流程**：
1. 从 reqjson 提取 `projectid`、`type`、`sourceId`、`scriptNos`
2. 调用 `sourceConfigService.selectSourceConfig()` 查询符合条件的数据源配置列表
3. 提取所有 `sourceConfigIds`
4. 调用 `tagInfoService.selectSourceTags(sourceConfigIds, projectid)` 获取这些数据源下使用的所有标签

用于前端展示某数据源/项目下可用的标签列表。
