---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: 接口文档
entry: 长连接协议
---

# Script

> mkey: `script` | 包路径: `cn.testin.trans.controller.req.script.Script`

脚本文件获取。

## get

### 协议命令

```
{ "mkey": "script", "op": "Script.get", "reqid": "xxx", "data": { "scriptNo": 1, "scriptid": 100, "projectid": 10 } }
```

### 实现意图

根据 scriptNo 或 scriptid 获取脚本文件信息（含下载 URL 和 MD5）。

- 传入 `scriptid`：通过 `findScriptByScriptIdList` 查询（拿指定版本的脚本）
- 传入 `scriptNo`：通过 `findFinalScriptByScriptNoList` 查询（拿最新版本的脚本）
- scriptid 和 scriptNo 至少传一个

### 请求消息字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| data.scriptNo | int | 条件必填 | 脚本编号（拿最新版本） |
| data.scriptid | int | 条件必填 | 脚本 ID（拿指定版本） |
| data.projectid | int | 是 | 项目 ID |

### 响应

```json
{
  "code": 0,
  "data": {
    "scriptid": 100,
    "scriptNo": 1,
    "scriptName": "test.py",
    "scriptMd5": "abc123...",
    "scriptUrl": "http://file-server/scripts/1/test.py"
  }
}
```

### 调用链

```
trans.controller.req.script.Script.get(Session, RequestContext)
  → scriptapi.findScriptByScriptIdList(scriptidList, projectid)       // [real-test](../../../app处理服务（real-test）/07-开放接口文档/00-模块索引.md)
  → scriptapi.findFinalScriptByScriptNoList(scriptnoList, projectid)  // [real-test](../../../app处理服务（real-test）/07-开放接口文档/00-模块索引.md)
```

### 涉及表/SQL

- 通过 [app处理服务](../../../app处理服务（real-test）/07-开放接口文档/00-模块索引.md) 查询，不直接操作 DB。
