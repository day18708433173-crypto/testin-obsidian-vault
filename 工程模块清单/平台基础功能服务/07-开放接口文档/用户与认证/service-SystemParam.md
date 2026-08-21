# service-SystemParam — 系统参数接口（ApiServlet）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/service/user/SystemParam.java`
> 基础类：`cn.testin.service.GenericBaseService`
> 机制：请求走 `ApiServlet /*` 入口，通过 `action=user` + `op=SystemParam.方法名` 路由到此类的对应 public 方法；每个方法的参数为 `ApiRequest`，返回 JSON 字符串。
> - **action**: `user`（对应包 `cn.testin.service.user`）
> - **入口格式**：`{"op": "SystemParam.方法名", "action": "user", "data": {...}}`
> 业务：系统参数（键值对配置）的增删改查，支持按分组检索、OEM 维护。

## op 列表总表

| #   | op                         | 方法名            | 说明                             |
| --- | -------------------------- | -------------- | ------------------------------ |
| 1   | SystemParam.add            | add            | 新增系统参数                         |
| 2   | SystemParam.list           | list           | 按分组查询参数列表（状态=ON）               |
| 3   | SystemParam.map            | map            | 按分组查询参数（Map 格式，key 为 paramKey） |
| 4   | SystemParam.maintain       | maintain       | 维护 OEM 信息（传入键值对批量更新）           |
| 5   | SystemParam.maintainSystem | maintainSystem | 维护系统参数（按 id 更新单条）              |

---

## 1. op=SystemParam.add — 新增系统参数

### 请求格式
{"op": "SystemParam.add", "action": "user", "data": {"paramTitle": "...", "paramValue": "...", "paramKey": "...", "paramGroup": "...", "paramStatus": ...}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| paramTitle | String | 是 | 参数标题（getString） |
| paramValue | String | 是 | 参数值（getString） |
| paramKey | String | 是 | 参数键（getString） |
| paramGroup | String | 是 | 参数分组（getString） |
| paramStatus | Integer | 是 | 参数状态（getInt） |

### 核心逻辑

构造 `DbSystemParam` 写入 `db_system_param`。

### 响应

`{ error_code, msg, data: { object: DbSystemParam(JSON) } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.object | Object | 新增的系统参数对象 |
| data.object.id | Long | 参数 ID |
| data.object.paramTitle | String | 参数标题 |
| data.object.paramKey | String | 参数键 |
| data.object.paramValue | String | 参数值 |
| data.object.paramGroup | String | 参数分组 |
| data.object.paramStatus | Integer | 参数状态 |

### 涉及表

- `db_system_param`（`DbSystemParam`）

---

## 2. op=SystemParam.list — 参数列表（按分组）

### 请求格式
{"op": "SystemParam.list", "action": "user", "data": {"paramGroup": "...", "paramKey": "..."}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| paramGroup | String | 是 | 参数分组（`isBlank` 判空，为空抛参数异常） |
| paramKey | String | 否 | 参数键（`optString` 筛选） |

### 核心逻辑

查询指定分组下 `status=ON` 的参数列表。

### 响应

`{ error_code, msg, data: { list: [DbSystemParam(JSON), ...] } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.list | Array | 系统参数列表 |
| data.list[].id | Long | 参数 ID |
| data.list[].paramTitle | String | 参数标题 |
| data.list[].paramKey | String | 参数键 |
| data.list[].paramValue | String | 参数值 |
| data.list[].paramGroup | String | 参数分组 |
| data.list[].paramStatus | Integer | 参数状态 |

---

## 3. op=SystemParam.map — 参数 Map

### 请求格式
{"op": "SystemParam.map", "action": "user", "data": {"paramGroup": "..."}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| paramGroup | String | 是 | 参数分组（`isBlank` 判空） |

### 核心逻辑

同 `list`，但返回 `Map<String, DbSystemParam>` 格式，键为 `paramKey`。无分页。

### 响应

`{ error_code, msg, data: { result: { "key1": DbSystemParam, "key2": ... } } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Object | Map，键为 paramKey，值为 DbSystemParam 对象 |
| data.result.<paramKey>.id | Long | 参数 ID |
| data.result.<paramKey>.paramTitle | String | 参数标题 |
| data.result.<paramKey>.paramKey | String | 参数键 |
| data.result.<paramKey>.paramValue | String | 参数值 |
| data.result.<paramKey>.paramGroup | String | 参数分组 |
| data.result.<paramKey>.paramStatus | Integer | 参数状态 |

---

## 4. op=SystemParam.maintain — 维护 OEM 信息

### 请求格式
{"op": "SystemParam.maintain", "action": "user", "data": {"paramKey1": "newValue1", "paramKey2": "newValue2", ...}}

### 请求参数

JSON 根级的任意 key-value 对，其中 key 对应 `paramKey`，value 为新的 `paramValue`。

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| <paramKey> | String | 否 | 动态键值对，键为 paramKey，值为新的 paramValue（遍历 `reqjson.keys()`） |

### 核心逻辑

遍历请求 JSON 的所有 key，按 `paramKey` 匹配更新 `db_system_param.paramValue`。

### 响应

`{ error_code, msg, data: { result: 实际更新的记录数 } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Integer | 实际更新的记录数 |

---

## 5. op=SystemParam.maintainSystem — 维护系统参数（按 ID）

### 请求格式
{"op": "SystemParam.maintainSystem", "action": "user", "data": {"id": ..., "paramTitle": "...", "paramKey": "...", "paramValue": "...", "paramGroup": "...", "paramStatus": ...}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| id | Long | 是 | 参数记录 ID（getLong） |
| paramTitle | String | 是 | 参数标题（getString） |
| paramKey | String | 是 | 参数键（getString） |
| paramValue | String | 是 | 参数值（getString） |
| paramGroup | String | 是 | 参数分组（getString） |
| paramStatus | Integer | 是 | 参数状态（getInt） |

### 核心逻辑

按主键 ID 更新单条系统参数记录（全字段覆盖）。

### 响应

`{ error_code, msg, data: { result: 影响行数 } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Integer | 影响行数 |

---

### 涉及表

- `db_system_param`（`DbSystemParam`）— 系统参数字典表（键值对存储）
