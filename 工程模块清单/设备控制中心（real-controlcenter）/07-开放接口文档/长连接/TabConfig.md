---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: 接口文档
entry: 长连接协议
---

# TabConfig

> mkey: `script` | 包路径: `cn.testin.trans.controller.req.script.TabConfig`

语音测试 Tab 配置查询。

## getTabConfigById

### 协议命令

```
{ "mkey": "script", "op": "TabConfig.getTabConfigById", "reqid": "xxx", "data": { "id": 1, "projectid": 10 } }
```

### 实现意图

根据 id 和 projectid 获取语音测试的 Tab 配置信息。

### 请求消息字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| data.id | int | 是 | 配置 ID |
| data.projectid | int | 是 | 项目 ID |

### 响应

```json
{
  "code": 0,
  "data": { "result": { /* TabConfigInfo JSON */ } }
}
```

### 调用链

```
trans.controller.req.script.TabConfig.getTabConfigById(Session, RequestContext)
  → UrlTabConfigApi.getTabConfigById(projectid, id)     // [real-test](../../../app处理服务（real-test）/07-开放接口文档/00-模块索引.md)
```

### 涉及表/SQL

- 通过 [app处理服务](../../../app处理服务（real-test）/07-开放接口文档/00-模块索引.md) 查询，不直接操作 DB。
