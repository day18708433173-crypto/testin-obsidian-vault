---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: 接口文档
entry: 长连接协议
---

# VocContentDetail

> mkey: `script` | 包路径: `cn.testin.trans.controller.req.script.VocContentDetail`

语音测试相关内容查询，包括语音内容详情和 DBC 文件获取。

## voiceInfo

### 协议命令

```
{ "mkey": "script", "op": "VocContentDetail.voiceInfo", "reqid": "xxx", "data": { "uniqKey": "xxx" } }
```

### 实现意图

通过 uniqKey 查询语音测试的内容详情。

### 请求消息字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| data.uniqKey | String | 是 | 唯一标识键 |

### 调用链

```
trans.controller.req.script.VocContentDetail.voiceInfo(Session, RequestContext)
  → scriptparameterapi.findByUniqKey(uniqKey)    // [real-test](../../../app处理服务（real-test）/07-开放接口文档/00-模块索引.md)
```

---

## getDbcFileById

### 协议命令

```
{ "mkey": "script", "op": "VocContentDetail.getDbcFileById", "reqid": "xxx", "data": { "id": 1 } }
```

### 实现意图

根据 id 获取 DBC 文件（CAN 数据库文件）信息。

### 请求消息字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| data.id | int | 是 | DBC 文件 ID |

### 调用链

```
trans.controller.req.script.VocContentDetail.getDbcFileById(Session, RequestContext)
  → AppPackageApi.getDbcFileById(id)     // [real-test](../../../app处理服务（real-test）/07-开放接口文档/00-模块索引.md)
```
