# service-QcTask — QC 任务（通知发送）接口

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/service/qc/QcTask.java`
> 类：`cn.testin.service.qc.QcTask extends GenericBaseService`（非 Spring MVC Controller，无注解路由）
> 入口方式：ApiServlet 入口，`action=qc`，`op=QcTask.add` 反射调用
> - **action**: `qc`（对应包 `cn.testin.service.qc`）
> - **入口格式**：`{"op": "QcTask.add", "action": "qc", "data": {...}}`
> 依赖：`IQcTaskService`（Spring Bean，继承自 `GenericBaseService`）
> 业务：新增 QC 通知任务记录（用于质量中心渠道回调/通知的消息落地）。
> 涉及表：`db_notice.qc_task`

## 方法列表总表

| # | 方法 | 说明 | 主要依赖 |
|---|---|---|---|
| 1 | add | 新增 QC 通知任务数据 | iQcTaskService.add |

统一返回：JSON 字符串，结构 `{ code, msg, data }`；`data` 内含 `result`。
参数校验失败返回 `CommonCode.paraInvalid` + 具体提示。

---

## 分发机制

- 入口：`/*`（ApiServlet）
- `action` 参数 = `qc`（定位到 `cn.testin.service.qc` 子包）
- `op` 参数 = `QcTask.add`
- 请求体中 `reqjson` 为业务 JSON

---

## 1. op=QcTask.add — 新增 QC 通知任务

### 请求格式
{"op": "QcTask.add", "action": "qc", "data": {"eid": ..., "projectid": ..., "tradeNo": "...", "content": "..."}}

### 请求参数（reqjson）

| 字段 | 必填 | 说明 |
|---|---|---|
| eid | 是 | 企业ID，正整数（无效返回 paraInvalid） |
| projectid | 是 | 项目组ID，非负整数 |
| tradeNo | 是 | 交易号，非空（无效返回 paraInvalid） |
| content | 是 | 通知内容，非空（无效返回 paraInvalid） |
| id | 否 | 主键ID |
| execNum | 否 | 通知执行次数，默认 0 |
| level | 否 | 通知级别，默认 0 |
| status | 否 | 通知状态，默认 1（有效） |
| callbackStatus | 否 | 回调状态，默认 0 |
| callbackNum | 否 | 回调通知次数，默认 0 |
| createtime | 否 | 创建时间，默认 `System.currentTimeMillis()` |
| updatetime | 否 | 更新时间，默认 `System.currentTimeMillis()` |
| publishtime | 否 | 推迟发布时间，默认当前时间 |
| expiretime | 否 | 过期时间，默认当前时间 + 6 小时 |

> 参数由 `DbQcTask.toBean(reqjson)` 转换后经 `validate(qcTask, apirequest)` 校验。vhost 字段自动设为 `Config.MODULE_NODE_ID`（模块编号，服务端自动填充）。

### 代码摘录

```java
public String add(ApiRequest apirequest) throws Exception {
    JSONObject reqjson = apirequest.getReqjson();
    DbQcTask qcTask = DbQcTask.toBean(reqjson);
    String vresult = this.validate(qcTask, apirequest);
    if (!"success".equals(vresult)) {
        return vresult;
    }
    Integer result = this.iQcTaskService.add(qcTask);
    JSONObject jObj = ApiUtil.getJSONobj(apirequest,
            CommonCode.success.getValue(), CommonCode.success.getDescr());
    Map<String, Object> datamap = new HashMap<String, Object>();
    datamap.put("result", result);
    jObj.put(ApiResponse.RES_DATA, datamap);
    return jObj.toString();
}
```

### 响应结构

`data.result` = 插入结果（Integer，>0 表示成功）。

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Integer | 插入结果（>0 成功，即新记录 id） |

### 涉及的数据库操作

`iQcTaskService.add(qcTask)` — 表 `db_notice.qc_task`，插入 qc_task 记录。

- 字段包括：id, vhost, trade_no, eid, projectid, content, exec_num, level, status, qc_return_result, callback_num, callback_status, createtime, updatetime, publishtime, expiretime

---

## 私有方法：validate

`validate(DbQcTask, ApiRequest)` — 校验 QC 任务参数：

- eid 必须为正整数
- projectid 必须为非负整数
- tradeNo 必须非空
- content 必须非空
- 校验失败返回 `ApiUtil.getResult` 拼装的错误 JSON；成功返回 "success"

---

## 备注

- 该类仅一个 public 方法 `add`，是 QC 通知任务的写入入口。QC 通知的后续处理（发送、回调等）由后台定时任务 `cn.testin.schedule.plan.QcTaskThread` 驱动，不属于本类范围。
- `vhost` 字段由 `DbQcTask.toBean()` 自动设置为 `Config.MODULE_NODE_ID`（标识当前模块节点），前端无需传入。
- `expiretime` 默认 = 创建时间 + 6 小时（21600000ms），即通知在 6 小时后过期。

相关文档：[00-分支索引](00-分支索引.md) · [service-QcCfg](service-QcCfg.md)
