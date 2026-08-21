---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: 接口文档
entry: 长连接协议
---

# SourcecodeConfig

> mkey: `script` | 包路径: `cn.testin.trans.controller.req.script.SourcecodeConfig`

源码配置加载。

## loadCfg

### 协议命令

```
{ "mkey": "script", "op": "SourcecodeConfig.loadCfg", "reqid": "xxx", "data": {} }
```

### 实现意图

根据上位机 ucomid 加载源码配置列表（RealcfgSourcecodeConfig），用于指导上位机在测试过程中注入或替换应用源码。

### 请求消息字段

无需 data 字段，依靠 sessionKey 定位。

### 响应

```json
{
  "code": 0,
  "data": {
    "list": [{ /* RealcfgSourcecodeConfig JSON 列表 */ }]
  }
}
```

### 调用链

```
trans.controller.req.script.SourcecodeConfig.loadCfg(Session, RequestContext)
  → isourcecodeconfigservice.list(ucomid)    // [real-cfg](../../../平台配置（real-cfg）/07-开放接口文档/00-模块索引.md) realcfg_sourcecode_config
```

### 涉及表/SQL

- `realcfg_sourcecode_config` — 源码配置表
