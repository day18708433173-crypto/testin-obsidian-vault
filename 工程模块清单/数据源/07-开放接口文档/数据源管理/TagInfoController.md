# TagInfoController — 标签管理控制器（MVC）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/controller/TagInfoController.java`
> 类级路由：`/datasource/tag`（完整前缀 `/openapi/v3/datasource/tag`）
> Service 实现：`TagInfoService`
> 业务：标签的分页查询和更新。

## 接口列表总表

| # | 方法 | 路径 | 方法名 | 说明 |
|---|------|------|--------|------|
| 1 | GET | `/v3/datasource/tag/tags` | getTags | 分页获取标签列表 |
| 2 | PUT | `/v3/datasource/tag` | updateTag | 修改标签 |

涉及表：`tag_info`。

---

## 1. GET /v3/datasource/tag/tags — 分页获取标签列表

### 入口

`TagInfoController.getTags(projectId, eid, page=1, pageSize=20, name?)`

### 请求参数（Query）

| 字段 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| project_id | Integer | 是 | — | 项目 ID |
| eid | Integer | 是 | — | 企业 ID |
| page | Integer | 否 | 1 | 页码 |
| page_size | Integer | 否 | 20 | 每页条数 |
| name | String | 否 | — | 标签名模糊筛选 |

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.totalRow | Long | 总记录数 |
| data.totalPage | Integer | 总页数 |
| data.pageSize | Integer | 每页条数 |
| data.page | Integer | 页码 |
| data.list | List\<TagInfo\> | 标签列表 |
| data.list[].id | Integer | 标签 id |
| data.list[].eid | Integer | 企业 id |
| data.list[].projectId | Integer | 项目 id |
| data.list[].name | String | 标签名 |
| data.list[].status | Integer | 状态 1 正常 0 删除 |
| data.list[].createtime / updatetime | Long | 创建/更新时间 |

`ResponseResult<Object>`，data = `PageInfoList<TagInfo>` 分页结果。

### 调用链

```
TagInfoController.getTags
└─ TagInfoService.selectPage
```

---

## 2. PUT /v3/datasource/tag — 修改标签

### 入口

`TagInfoController.updateTag(@RequestBody UpdateTagRequestDTO request)`

### 请求参数（JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| tagId | Integer | 否 | 标签 id（无判空，但作为 updateById 主键） |
| tagName | String | 是 | 标签名（代码 `StringUtils.isBlank` 为空直接返回 false） |
| projectId | Integer | 否 | 项目 id（用于重名校验） |
| eid | Integer | 否 | 企业 id |
| userId | Integer | 否 | 用户 id |
| userName | String | 否 | 用户名 |

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.result | Boolean | 是否成功（true/false） |

`ResponseResult<ResultDTO>`，`data.result` = true/false。
