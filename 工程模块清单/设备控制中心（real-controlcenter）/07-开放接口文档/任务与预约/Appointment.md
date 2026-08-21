---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: 接口文档
entry: ApiServlet
---

# Appointment

- **类全名**：`cn.testin.service.appointment.Appointment`
- **源码路径**：`real-controlcenter/src/main/java/cn/testin/service/appointment/Appointment.java`
- **职责**：设备预约管理：预约列表查询、按设备查询、新增预约、续约、取消、完成（提前结束）。业务逻辑封装在 `IDeviceAppointmentService` / `IDeviceAppointmentDAO`。

## op 一览表

| op | 方法 | 职责 |
| --- | --- | --- |
| list | `Appointment.list` | 预约记录分页查询 |
| listByDevice | `Appointment.listByDevice` | 按设备维度查询预约 |
| add | `Appointment.add` | 新增预约 |
| renewal | `Appointment.renewal` | 预约续约 |
| cancel | `Appointment.cancel` | 取消预约 |
| finish | `Appointment.finish` | 完成（结束）预约 |

---

### list (`Appointment.list`)

- **入口**：ApiServlet，action=appointment，op=Appointment.list
- **实现意图**：按企业/项目组/类型/时间范围/设备/申请人等条件分页查询设备预约记录。

**请求参数**

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| page | Integer | 是 | 页码（>0） |
| pageSize | Integer | 是 | 每页大小（<=Config.MaxSize） |
| eid | Integer | 是 | 企业 ID |
| projectid | Integer | 是 | 项目组 ID |
| type | Integer | 是 | 预约类型（>0） |
| startTime / endTime | Long | 否 | 时间范围 |
| deviceid | String | 否 | 设备 ID |
| applicantUserid | Integer | 否 | 申请人用户 ID |
| reqId | String | 否 | 预约单号 |

**返回参数**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 返回数据对象 |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页条数 |
| data.totalRow | Integer | 总记录数 |
| data.totalPage | Integer | 总页数 |
| data.list | JSONArray&lt;DeviceAppointment&gt; | 预约记录列表 |
**处理流程**：参数校验 → `iDeviceAppointmentDao.list(...)` 分页 → baseListToResData 返回。
**调用链**：`IDeviceAppointmentDAO.list`。
**涉及表与 SQL**：`device_appointment`（select 分页，DAO：`DeviceAppointmentDAOImpl`）。
**异常与校验**：page/pageSize/eid/projectid/type 非法 → paraInvalid；查询空 → unknown。

---

### listByDevice (`Appointment.listByDevice`)

- **入口**：ApiServlet，action=appointment，op=Appointment.listByDevice
- **实现意图**：按设备维度聚合查询预约信息（不分页），用于设备详情页展示预约占用情况。

**请求参数**

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| eid | Integer | 是 | 企业 ID |
| projectid | Integer | 是 | 项目组 ID |
| type | Integer | 是 | 预约类型 |
| startTime / endTime | Long | 否 | 时间范围 |
| deviceid | String | 否 | 设备 ID |
| applicantUserid | Integer | 否 | 申请人 |

**返回参数**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 返回数据对象 |
| data.result | JSONArray&lt;DeviceAppointmentByDevice&gt; | 按设备聚合的预约列表（无记录时缺省） |
**处理流程**：校验 → `deviceAppointmentService.listByDevice(...)` → 返回。
**调用链**：`IDeviceAppointmentService.listByDevice`。
**涉及表与 SQL**：`device_appointment`（select 聚合）。
**异常与校验**：eid/projectid/type 非法 → paraInvalid。

---

### add (`Appointment.add`)

- **入口**：ApiServlet，action=appointment，op=Appointment.add
- **实现意图**：新增设备预约（整包 reqjson 交给业务层处理，含预约设备、时段、申请人等）。

**请求参数**

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| onlineUserInfo | JSONObject | 是 | 在线用户信息 |
| projectid | Integer | 是 | 项目组 ID |
| eid | Integer | 是 | 企业 ID |
| 其他 | - | - | 预约明细（设备、时段等，由业务层解析） |

**返回参数**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 返回数据对象 |
| data.result | String | 预约单号 reqId |
**处理流程**：三项必填校验 → `deviceAppointmentService.add(reqJson)` → 返回 reqId。
**调用链**：`IDeviceAppointmentService.add`（可能联动通知 notice-manager）。
**涉及表与 SQL**：`device_appointment`（insert）。
**异常与校验**：必填缺失 → paraInvalid；业务层返回 null → APPOINTMENT_FAILED。

---

### renewal (`Appointment.renewal`)

- **入口**：ApiServlet，action=appointment，op=Appointment.renewal
- **实现意图**：对已有预约续约（延长预约时长）。

**请求参数**

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| eid | Integer | 是 | 企业 ID |
| projectid | Integer | 是 | 项目组 ID |
| reqId | String | 是 | 预约单号 |
| aRenewalPeriod | Long | 是 | 续约时长（>0） |

**返回参数**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 返回数据对象 |
| data.result | String | 业务层返回结果（非空时放入） |
**处理流程**：校验 → `deviceAppointmentService.renewal(reqId, renewalPeriod)` → 返回。
**调用链**：`IDeviceAppointmentService.renewal`。
**涉及表与 SQL**：`device_appointment`（update 时长/结束时间）。
**异常与校验**：eid/projectid/renewalPeriod/reqId 非法 → paraInvalid。

---

### cancel (`Appointment.cancel`)

- **入口**：ApiServlet，action=appointment，op=Appointment.cancel
- **实现意图**：取消预约。操作人可取 operatorUserid/operatorEmail，也可从 onlineUserInfo 兜底。

**请求参数**

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| eid | Integer | 是 | 企业 ID |
| projectid | Integer | 是 | 项目组 ID |
| reqId | String | 是 | 预约单号 |
| descr | String | 否 | 取消原因 |
| operatorUserid | Integer | 否 | 操作人 ID |
| operatorEmail | String | 否 | 操作人邮箱 |
| onlineUserInfo | JSONObject | 否 | 在线用户（兜底取 userid/email） |

**返回参数**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 返回数据对象 |
| data.result | Integer | 1 成功 / 0 失败 |
**处理流程**：校验 → `deviceAppointmentService.cancel(reqId, operatorUserid, operatorEmail, descr)` → 返回。
**调用链**：`IDeviceAppointmentService.cancel`（可能联动通知 notice-manager）。
**涉及表与 SQL**：`device_appointment`（update 状态为取消）。
**异常与校验**：eid/projectid/reqId 非法 → paraInvalid。

---

### finish (`Appointment.finish`)

- **入口**：ApiServlet，action=appointment，op=Appointment.finish
- **实现意图**：提前完成（结束）预约，释放设备。

**请求参数**

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| eid | Integer | 是 | 企业 ID |
| projectid | Integer | 是 | 项目组 ID |
| reqId | String | 是 | 预约单号 |
| onlineUserInfo | JSONObject | 是 | 在线用户信息，必含 email |

**返回参数**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 返回数据对象 |
| data.result | Integer | 1 成功 / 0 失败 |
**处理流程**：校验（含 onlineUserInfo.email）→ `deviceAppointmentService.finish(reqId, email)` → 返回。
**调用链**：`IDeviceAppointmentService.finish`。
**涉及表与 SQL**：`device_appointment`（update 状态为完成）。
**异常与校验**：eid/projectid/reqId/onlineUserInfo/email 缺失 → paraInvalid。

**关键代码摘录**

```java
// real-controlcenter/.../service/appointment/Appointment.java
if (reqJson.getJSONObject("onlineUserInfo").isNull("email")) {
    return ApiUtil.getResult(apirequest, CommonCode.paraInvalid.getValue(), msg);
}
String email = reqJson.getJSONObject("onlineUserInfo").getString("email");
boolean result = deviceAppointmentService.finish(reqId, email);
```
