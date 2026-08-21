# OperateLogController -- 操作日志查询

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/controller/v3/log/OperateLogController.java`
> 类级路由：`/log`
> Service 接口：`cn.testin.business.interfaces.common.IOperateLogService`
> 实现类：`cn.testin.business.impl.common.OperateLogServiceImpl`
> 业务：操作日志的分页查询、详情查看，以及操作类型/操作对象的枚举字典查询（业务侧 + 系统侧两套）。

## 接口列表总表

| 方法 | 路径 | 方法名 | 说明 | 操作日志 |
|---|---|---|---|---|
| GET | `/v3/log/logs` | selectOperateLogByCondition | 分页条件查询操作日志 | 无 |
| GET | `/v3/log/logs/{operate_log_id}` | getDetailContentById | 按ID查询操作日志详情 | 无 |
| GET | `/v3/log/logs/operate_types` | getOperateTypeList | 获取业务操作类型枚举列表 | 无 |
| GET | `/v3/log/logs/operate_objects` | getOperateObjectList | 获取业务操作对象枚举列表 | 无 |
| GET | `/v3/log/logs/system/operate_types` | getSystemOperateTypeList | 获取系统操作类型枚举列表 | 无 |
| GET | `/v3/log/logs/system/operate_objects` | getSystemOperateObjectList | 获取系统操作对象枚举列表 | 无 |

统一响应包装：`ResponseResult<T>`；分页用 `BasePageListResponseDTO`；列表用 `BaseListResponseDTO`。

---

## 1. GET /v3/log/logs -- 分页条件查询操作日志

### 入口

`OperateLogController.selectOperateLogByCondition(@Valid OperateLogRequestDTO request)` -- OperateLogController.java（`@UnderlineToCamel`）

### 请求参数（OperateLogRequestDTO，Query 绑定，@Valid）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| module_type | Integer | 否 | 模块类型过滤 |
| operate_object | String | 否 | 操作对象过滤 |
| operate_type | String | 否 | 操作类型过滤 |
| project_id | Integer | 否 | 项目ID过滤 |
| create_start_time | Long | 否 | 创建起始时间（毫秒时间戳） |
| create_end_time | Long | 否 | 创建结束时间（毫秒时间戳） |
| page | Integer | 否 | 页码，缺省取 `PAGE_DEFAULT` |
| page_size | Integer | 否 | 页大小，缺省取 `PAGE_SIZE_DEFAULT` |

### 响应结构

`ResponseResult<BasePageListResponseDTO<OperateLogResponseDTO>>`，列表元素包含操作日志的全部字段：

| 字段 | 类型 | 说明 |
|---|---|---|
| id | Long | 日志主键 |
| operateType | String | 操作类型（如 TASK_STOP、TASK_DELETE、PLAN_INFO_UPDATE） |
| operateObject | String | 操作对象 |
| operateContent | String | 操作内容描述/变更前后的JSON对比 |
| moduleType | Integer | 模块类型 |
| projectId | Integer | 项目ID |
| userId / userName | Integer / String | 操作人 |
| createTime | Date | 创建时间 |
| requestId | String | 幂等请求ID |

### 实现意图

Controller 层补齐分页默认值后委托 Service 层：组装 `OperateLogConditionDTO` 条件 → `PageHelper` 分页查 `db_operate_log` → 实体转 DTO（`OperateLogResponseDTO.translateEntityToDTO`）→ 包装分页返回。

### 调用链

```
OperateLogController.selectOperateLogByCondition
└─ OperateLogServiceImpl.selectOperateLogByCondition
   ├─ OperateLogConditionDTO.builder 组装条件
   ├─ PageHelper.startPage + IOperateLogDao.selectOperateLogByCondition → db_operate_log
   └─ PageUtil.copyPageInfo + OperateLogResponseDTO.translateEntityToDTO
```

### 涉及表

| 表 | 操作 |
|---|---|
| db_operate_log | 读（分页条件查询） |

---

## 2. GET /v3/log/logs/{operate_log_id} -- 按ID查询详情

### 入口

`OperateLogController.getDetailContentById(@PathVariable Long operateLogId)` -- OperateLogController.java

### 请求参数

| 字段 | 来源 | 必填 | 说明 |
|---|---|---|---|
| operate_log_id | Path | 是 | 日志主键ID |

### 响应结构

`ResponseResult<OperateLogResponseDTO>`：单条日志详情（含完整 operateContent 变更对比）；不存在时返回 null data。

### 调用链

```
OperateLogController.getDetailContentById
└─ OperateLogServiceImpl.getDetailContentById
   └─ IOperateLogDao.selectById → db_operate_log
```

### 涉及表

| 表 | 操作 |
|---|---|
| db_operate_log | 读（主键查询） |

---

## 3. GET /v3/log/logs/operate_types -- 业务操作类型枚举

### 入口

`OperateLogController.getOperateTypeList()` -- OperateLogController.java

### 请求参数

无。

### 响应结构

`ResponseResult<BaseListResponseDTO<Map<String, Object>>>`：操作类型枚举列表，每项含 type/value/label 等键值。

### 实现意图

直接从枚举类 `OperateTypeEnum.getOperateTypeList()` 获取业务侧操作类型字典（如 TASK_STOP、TASK_DELETE、PLAN_INFO_UPDATE、PLAN_INFO_COPY 等），转为 List 返回。

### 调用链

```
OperateLogController.getOperateTypeList
└─ OperateTypeEnum.getOperateTypeList()（静态方法）
```

---

## 4. GET /v3/log/logs/operate_objects -- 业务操作对象枚举

### 入口

`OperateLogController.getOperateObjectList()` -- OperateLogController.java

### 请求参数

无。

### 响应结构

`ResponseResult<BaseListResponseDTO<Map<String, Object>>>`：操作对象枚举列表。

### 实现意图

直接从 `OperateObjectTypeEnum.getOperateObjectList()` 获取业务侧操作对象字典。

### 调用链

```
OperateLogController.getOperateObjectList
└─ OperateObjectTypeEnum.getOperateObjectList()（静态方法）
```

---

## 5. GET /v3/log/logs/system/operate_types -- 系统操作类型枚举

### 入口

`OperateLogController.getSystemOperateTypeList()` -- OperateLogController.java

### 请求参数

无。

### 响应结构

`ResponseResult<BaseListResponseDTO<Map<String, Object>>>`：系统侧操作类型枚举列表。

### 实现意图

同第 3 节，但取系统侧枚举 `OperateTypeEnum.getSystemOperateTypeList()`，用于系统管理界面的筛选下拉。

### 调用链

```
OperateLogController.getSystemOperateTypeList
└─ OperateTypeEnum.getSystemOperateTypeList()（静态方法）
```

---

## 6. GET /v3/log/logs/system/operate_objects -- 系统操作对象枚举

### 入口

`OperateLogController.getSystemOperateObjectList()` -- OperateLogController.java

### 请求参数

无。

### 响应结构

`ResponseResult<BaseListResponseDTO<Map<String, Object>>>`：系统侧操作对象枚举列表。

### 实现意图

同第 4 节，取系统侧枚举 `OperateObjectTypeEnum.getSystemOperateObjectList()`。

### 调用链

```
OperateLogController.getSystemOperateObjectList
└─ OperateObjectTypeEnum.getSystemOperateObjectList()（静态方法）
```

---

## 备注

- 操作日志的**写入**不在本 Controller：通过 `@OperateLog` 注解 AOP 切面 + `OperateLogServiceImpl.insertOperateLog` 异步写 Redis 队列再批量消费入库（`db_common.db_operate_log`）。本 Controller 仅负责**查询**。
- `insertOperateLog` 带幂等机制：`requestId = createTime + MD5(logValue)`，同 requestId 已存在则仅删 Redis 队列消息，不重复入库。
- `OperateLogResponseDTO` 通过 `translateEntityToDTO` 将实体转为前端友好格式（如 operateContent JSON 格式化）。
- 枚举接口为 4 个静态字典查询，无 DB 访问。
- 枚举类位置：`cn.testin.common.enums.OperateTypeEnum`、`OperateObjectTypeEnum`。

相关文档：[00-分支索引](00-分支索引.md)
