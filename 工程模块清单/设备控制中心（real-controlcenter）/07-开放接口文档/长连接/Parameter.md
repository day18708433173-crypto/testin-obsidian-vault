---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: 接口文档
entry: 长连接协议
---

# Parameter

> mkey: `script` | 包路径: `cn.testin.trans.controller.req.script.Parameter`

脚本控件信息和脚本参数查询。

## controlList

### 协议命令

```
{ "mkey": "script", "op": "Parameter.controlList", "reqid": "xxx", "data": { "id": 1, "projectid": 10, "appid": 5, "fingerprint": "...", "packageName": "...", "name": "...", "pageNo": 1, "pageSize": 100 } }
```

### 实现意图

查询脚本控件信息（ScriptControl），支持按 id、projectid、appid、指纹、包名、名称过滤，返回分页结果。在内存中分页（先查全量再截取）。

### 请求消息字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| data.id | int | 否 | 控件 ID |
| data.projectid | int | 是 | 项目 ID |
| data.appid | int | 否 | 应用 ID |
| data.fingerprint | String | 否 | 控件指纹 |
| data.packageName | String | 否 | 包名 |
| data.name | String | 否 | 控件名称 |
| data.pageNo | int | 否 | 页码（默认 1） |
| data.pageSize | int | 否 | 每页条数（默认 100） |

### 响应

```json
{
  "code": 0,
  "data": {
    "totalRow": 50,
    "totalPage": 1,
    "pageNo": 1,
    "pageSize": 100,
    "list": [{ "id": 1, "projectid": 10, "name": "...", ... }]
  }
}
```

### 调用链

```
trans.controller.req.script.Parameter.controlList(Session, RequestContext)
  → scriptapi.listScriptControl(conditionmap)     // [real-test](../../../app处理服务（real-test）/07-开放接口文档/00-模块索引.md)
```

### 涉及表/SQL

- 通过 [app处理服务](../../../app处理服务（real-test）/07-开放接口文档/00-模块索引.md) 查询，不直接操作 DB。

---

## list

### 协议命令

```
{ "mkey": "script", "op": "Parameter.list", "reqid": "xxx", "data": { "projectid": 10, "appid": 5, "name": "paramName" } }
```

### 实现意图

查询脚本普通参数列表（兼容 V3.0 以前的脚本）。按 projectid、appid、name 精确查询。

### 请求消息字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| data.projectid | int | 是 | 项目 ID |
| data.appid | int | 是 | 应用 ID |
| data.name | String | 是 | 参数名称 |

### 调用链

```
trans.controller.req.script.Parameter.list(Session, RequestContext)
  → scriptparameterapi.list(projectid, appid, name)   // [real-test](../../../app处理服务（real-test）/07-开放接口文档/00-模块索引.md)
```
